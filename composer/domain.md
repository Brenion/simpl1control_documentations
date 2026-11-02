# Synchronisation profil client (front-office → back-office)

## Purpose

Propager vers le back-office les modifications de profil client effectuées depuis l'espace client (`PATCH /client`), via le pattern outbox/inbox existant. Le cabinet reçoit une fiche miroir à jour et une trace d'audit identifiant le client-user auteur.

## User stories

- En tant que client-user, quand je modifie le profil de mon client, le cabinet voit les mêmes informations à jour dans sa base miroir.
- En tant qu'admin/gestionnaire back-office, je peux consulter dans `audit_log` qu'un client-user a modifié le profil client et quand.

## Acceptance criteria

- Un `PATCH /client` réussi côté front-office émet atomiquement (même transaction SQL) un event `client.profile_updated.v1` dans la table `outbox` extranet.
- Le payload contient uniquement les **champs effectivement modifiés** parmi : `name`, `vatNumber`, `isVatSubject`, `addressStreet`, `addressZip`, `addressCity`, `addressCountry`, `sector`. Les champs absents du payload ne sont pas touchés côté back-office.
- Le payload inclut toujours `clientId` (UUID) et `updatedByClientUserId` (UUID du client-user authentifié).
- Les champs `planType`, `planLevel`, `creditBalance` et `status` ne sont **jamais** inclus dans le payload ni modifiés par cette synchro.
- Le worker back-office (`events-fetcher-tick`) récupère l'event via `GET /relay/events`, insère dans `inbox`, exécute la projection, puis acquitte via `POST /relay/events/ack`.
- La projection back-office met à jour la ligne `client` miroir (même UUID) avec les champs du payload (sémantique patch partiel).
- La projection crée une ligne `audit_log` : `actor` = `updatedByClientUserId`, `action` = `client.profile_updated.v1`, `resource` = `clientId`, `details` = payload des champs modifiés.
- Rejeu idempotent : un event déjà traité (même `event_id`) ne double pas la mise à jour ni l'audit.

## Business rules & invariants

- **Unidirectionnelle** : front-office → back-office uniquement pour cette feature. Le back-office ne pousse pas de modifications client vers le front-office ici.
- **Pull initié par le back-office** : le front-office reste passif ; le worker intranet interroge `/relay/events`.
- **Corrélation par UUID** : `client.id` est identique en front-office et back-office. La création du client (et des client-users) est toujours initiée côté back-office, qui fournit les UUID à enregistrer côté front-office.
- **Last-write-wins** : en cas de modification concurrente BO et FO, la dernière écriture gagne (pas de détection de conflit).
- **Pas de synchro client_user** : les modifications, créations et désactivations de `client_user` depuis le front-office ne déclenchent aucun event (hors périmètre ; gestion client_user réservée au back-office).
- **Champs BO-only protégés** : `planType`, `planLevel`, `creditBalance`, `status` ne transitent jamais dans le payload et ne sont jamais écrasés par la projection.

## Edge cases & error states

- **Worker down / extranet injoignable** : les events s'accumulent dans `outbox` front-office ; rattrapage automatique au redémarrage du worker (comportement standard outbox).
- **Rejeu idempotent** : `ON CONFLICT (event_id) DO NOTHING` sur `inbox` ; projection non rejouée si déjà ingéré.
- **Client inexistant côté BO** : situation anormale (création client toujours BO). Comportement attendu : log warning, aucune création à la volée, event acquitté pour ne pas bloquer la file.
- **Modification concurrente BO/FO** : last-write-wins, pas d'erreur remontée au client.
- **Projection sur champs identiques** : seuls les champs présents dans le payload sont appliqués ; les autres restent inchangés côté BO.

## Out of scope

- Synchronisation des `client_user` (création, modification, désactivation) depuis le front-office.
- Modification ou suppression de `client_user` depuis le front-office (abandonné produit).
- Création de client depuis le front-office (aucune route existante aujourd'hui).
- Indicateur visuel back-office « modifié par le client le … » (noté comme évolution future possible).
- Monitoring / alerting sur `outbox_dead` ou `inbox.processed_at IS NULL` (noté comme évolution future possible).
- Migration `client_email` → `client_id` sur les tables métier existantes.
- Propagation BO → FO des modifications client.

## Assumptions

- [Anomalie BO] Si le client BO est introuvable malgré l'invariant « création BO d'abord », la projection log un warning, n'insère pas d'`audit_log`, et l'event est quand même acquitté côté FO pour ne pas bloquer la file.
- [Audit] `audit_log.actor` porte le `clientUserId` (UUID), pas l'email. Consultable par tous les admins/gestionnaires back-office (pas de filtre par assignation avocat — notion inexistante en Phase 1).
- [UI BO] Mise à jour silencieuse de la fiche miroir ; pas de notification UI dans cette US.
- [Doc métier] Le tableau et le payload de `client.profile_updated.v1` seront précisés dans `docs/metier/events-outbox-inbox.md` dans la même PR.

## Payload `client.profile_updated.v1`

```json
{
  "clientId": "uuid",
  "updatedByClientUserId": "uuid",
  "changes": {
    "name": "Acme SA",
    "vatNumber": "BE0123456789",
    "isVatSubject": true,
    "addressStreet": "Rue Example 1",
    "addressZip": "1000",
    "addressCity": "Bruxelles",
    "addressCountry": "BE",
    "sector": "CORPORATE"
  }
}
```

- `changes` : sous-objet contenant **uniquement** les clés réellement modifiées par le `PATCH /client`. Clés autorisées : `name`, `vatNumber`, `isVatSubject`, `addressStreet`, `addressZip`, `addressCity`, `addressCountry`, `sector`.
- Aucune autre clé autorisée dans `changes`.

## Technical approach

- **Front-office** : ajouter `publishEvent` dans `@client-libs/events-backend` (même pattern que BO) ; appeler depuis `UpdateClientRoute` dans la même transaction que le flush client ; calculer le delta `changes` à partir des attributs reçus dans le body PATCH.
- **Back-office worker** : projection `client.profile_updated.v1` dans `@admin-libs/events-backend/src/projections/` — PATCH partiel sur `client` + INSERT `audit_log`.
- **Infra existante** : routes `GET /relay/events` et `POST /relay/events/ack` (FO) ; `events-fetcher-tick` (BO worker) — aucune modification du mécanisme relay/worker.
- **Tests** : unit/integration sur émission atomique FO, projection BO (happy path + client BO absent + rejeu idempotent).
