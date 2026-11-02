# US002 — Synchronisation profil client front → back

## Purpose

Quand un client met à jour son profil depuis le front-office, propager les changements vers le back-office via l'event `client.profile_updated.v1` et le mécanisme outbox/inbox existant.

## User stories

- En tant que back-office, je veux que les modifications de profil client saisies dans l'espace client soient répercutées automatiquement dans mes propres données, sans intervention manuelle.

## Acceptance criteria

- Quand `PATCH /api/v1/client` réussit avec au moins un attribut fourni, un event `client.profile_updated.v1` est inséré dans la table `outbox` front-office dans la même transaction que la mise à jour du client.
- Le payload de l'event contient uniquement les champs effectivement envoyés dans le PATCH (delta), plus `client_id` et `modified_by_user_id`.
- Un champ `null` dans le PATCH est considéré comme un changement valide et est inclus dans le payload.
- Le worker back-office (`eventsFetcherRunner`) récupère l'event via `GET /relay/events`, l'insère dans `inbox`, applique la projection, et ack.
- La projection met à jour uniquement les champs présents dans `changes` sur l'entité `client` back-office (écrasement sans condition).
- La projection crée une entrée dans `audit_log` back-office : `actor = modified_by_user_id`, `action = 'client.profile_updated.v1'`, `resource = client_id`.
- Si la projection échoue (erreur DB, client introuvable), la transaction rollback, `processed_at` reste null, l'event n'est pas acké, et sera rejoué au prochain tick du `eventsFetcherRunner` (toutes les 2 s par défaut).

## Business rules & invariants

- Seuls les champs "self-service" peuvent être mis à jour par ce mécanisme : `name`, `vatNumber`, `isVatSubject`, `addressStreet`, `addressZip`, `addressCity`, `addressCountry`, `sector`.
- Les champs `planType`, `planLevel`, `creditBalance`, `status` du back-office ne sont jamais touchés par cette projection.
- Un event acké côté front-office mais dont la projection a déjà été traitée (conflict `event_id`) est ignoré silencieusement (ON CONFLICT DO NOTHING — idempotence déjà garantie par `events-fetcher-tick`).

## Edge cases & error states

- **Aucun champ fourni dans le PATCH** : le front-office (UI Ember) ne soumet pas de PATCH si rien n'a changé — cette situation ne devrait pas atteindre l'API. Si elle arrive quand même (appel direct), `attributes = {}` → pas d'event émis.
- **Champ `null` dans le PATCH** : valide, inclus dans le payload et appliqué par la projection. Point d'attention futur : un `null` non intentionnel (bug UI) efface un champ côté back-office sans moyen de le distinguer d'une vraie suppression.
- **Client introuvable côté back-office** : la projection lève une erreur → transaction rollback → event reste dans l'`outbox` front-office → retenté indéfiniment toutes les 2 s. **Pas de mécanisme de dead-letter côté fetcher** : contrairement au publisher (admin→client) qui a `MAX_ATTEMPTS` + `outbox_dead`, le `eventsFetcherRunner` ne limite pas les tentatives. Un event bloquant monopolise la file jusqu'à ce que le problème soit résolu manuellement.
- **Back-office down** : les events s'accumulent dans l'`outbox` front-office (non ackés) et seront traités à la remontée du worker.
- **Projection déjà traitée** (doublon suite à crash entre commit et ack) : ON CONFLICT DO NOTHING sur la table `inbox` absorbe l'insertion, `processed_at` reste tel quel, event acké — idempotence garantie par l'infrastructure existante.
- **Conflit de données** (un admin a modifié le client back-office entre-temps) : la projection écrase quoi qu'il arrive. Limitation connue, à adresser dans une future version avec `updated_at` dans le payload ou un mécanisme de version optimiste.

## Out of scope

- Synchronisation des `client_user` (modifications, soft-delete) — les routes de modification client_user front-office seront supprimées ; seul le back-office peut modifier les `client_user`.
- Notification temps réel côté front-office (WebSocket, SSE).
- Route `PATCH /clients/{id}` côté back-office (non nécessaire pour cette sync — c'est la projection qui écrit).

## Assumptions

- [Data] Le payload contient les champs présents dans `request.body.data.attributes` sans comparaison avant/après (delta = corps du PATCH). L'UI ne soumet pas de PATCH si rien n'a changé, donc tout PATCH entrant représente au moins un vrai changement.
- [Data — point d'attention futur] Un attribut `null` dans le PATCH est considéré valide et sera appliqué par la projection. Un null non intentionnel issu d'un bug UI ne peut pas être distingué d'un effacement volontaire. À surveiller si des incidents de données nulles inattendues remontent.
- [Infrastructure] L'`outbox` front-office et l'`inbox` back-office existent déjà en base (migrations `20260527090005` et `20260527090255`) — aucune migration supplémentaire n'est nécessaire.
- [Idempotence] La garantie de non-doublon est fournie par l'`ON CONFLICT DO NOTHING` dans `events-fetcher-tick` existant — la projection n'a pas besoin de la re-implémenter.
- [Résilience — limitation connue] Le `eventsFetcherRunner` (direction client→admin) n'a pas de mécanisme `outbox_dead` ni de limite de tentatives. Un event dont la projection échoue de façon permanente bloque la file et sera retenté indéfiniment. À traiter dans une future US si le besoin de dead-letter dans cette direction se confirme.

## Technical approach

- **Front-office** : `UpdateClientRoute` ajoute `em.create(OutboxEventEntity, ...)` avant `em.flush()` — MikroORM englobe les deux dans la même transaction implicite.
- **Back-office** : nouvelle projection `'client.profile_updated.v1'` dans `packages/back-office/@libs/events-backend/src/projections/index.ts`, enregistrée au démarrage de l'app.
- **Dépendance** : `@client-libs/events-backend` ajoutée comme workspace dep de `@client-libs/users-backend` pour importer `OutboxEventEntity`.
- **Pas de migration** : tables `outbox`, `inbox`, `audit_log` déjà créées.
- **Test** : intégration sur la projection (payload → client mis à jour + audit_log créé).
