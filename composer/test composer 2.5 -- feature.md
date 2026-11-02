## commande

```
/nox (racine)/feature-resolve 1780573340658828474 on arrive a la Synchronisation front -> back

bien regarder aux documents @docs/metier/events-outbox-inbox.md et

@docs/techniques/worker-outbox.md
```

## Reponse 

```
## Étape 1 — Carte et classification

Carte : [US002 - Compléter profil client](vscode-file://vscode-app/Applications/Cursor.app/Contents/Resources/app/out/vs/code/electron-sandbox/workbench/1780573340658828474) — Sprint 01, colonne en cours.

Périmètre restant (checklist « Synchronisation front → back », 3 tâches ouvertes) :

1. Quand un client est modifié en front-office → notification vers le back-office
2. Quand un client_user est modifié en front-office → notification vers le back-office
3. Le back-office traite ces notifications (worker) et met à jour ses données miroir

État du code (pour cadrer les questions) :

- Infra relay déjà en place : `GET /relay/events`, `POST /relay/events/ack` côté extranet ; `events-fetcher-tick` côté intranet worker
- `client.profile_updated.v1` existe dans le catalogue d'events, mais aucune émission depuis les routes FO (`PATCH /client`, `/users/*`) et aucune projection BO (`projections/index.ts` est vide)
- Le front-office n'a pas encore de helper `publishEvent` (contrairement au back-office)
- La spec profil client (`docs/features/client-profile/domain.md`) exclut explicitement la synchro — c'est bien la tranche qu'on attaque maintenant

Classification : Medium — intégration sur une infra outbox/inbox existante, avec des projections métier et des règles de payload à figer. Pas Light (deux entités, champs BO-only à protéger, signalement métier). Pas Deep non plus (pas de nouveau concept domaine, pas de migration de schéma attendue).

---

## Étape 2 — Interrogatoire (questions uniquement)

### Axis 1 — Acceptance criteria

1. Le ticket parle de « notification » pour client et client_user séparément, mais le catalogue métier ne liste qu'un seul event `client.profile_updated.v1` (« modification du profil »). Faut-il un seul event pour les deux cas, ou deux events distincts (ex. `client.profile_updated.v1` + un event dédié aux client_users) ?
    
2. Quelles opérations front-office doivent déclencher une synchro ?
    
    - Client : uniquement `PATCH /client` ?
    - Client_user : `PATCH /users/:id` seulement, ou aussi `POST /users` (création) et `DELETE /users/:id` (désactivation soft `isActive=false`) ?
3. Pour chaque event, quels champs exactement doivent transiter dans le payload ?
    
    - Client : tous les champs modifiables FO (`name`, `vatNumber`, `isVatSubject`, adresse, `sector`) ? Les champs BO-only (`planType`, `planLevel`, `creditBalance`, `status`) sont-ils exclus du payload et jamais écrasés côté BO ?
    - Client_user : `email`, `firstName`, `lastName`, `phone`, `language`, `isActive` ? Le password est-il explicitement exclu (jamais dans le payload, jamais synchronisé) ?
4. Le doc métier dit « la fiche client est mise à jour côté cabinet ; un signalement est tracé ». Qu'est-ce que « signalement » signifie concrètement ici ?
    
    - Une ligne dans `audit_log` ?
    - Un flag/UI visible pour l'avocat ?
    - Autre chose ?

### Axis 2 — Edge cases & error states

5. Client/client_user inexistant côté BO au moment où l'event arrive (miroir pas encore créé, ou IDs désynchronisés) : 404 silencieux + log warn ? création du miroir à la volée ? bascule en dead letter ?
    
6. Mise à jour concurrente : un gestionnaire BO modifie la fiche client pendant qu'un client-user FO envoie un `PATCH /client` — last-write-wins côté BO (comme confirmé pour le profil FO), ou le BO a priorité et ignore l'event ?
    
7. Changement d'email sur un client_user FO : si l'email existe déjà côté BO (contrainte unique globale), la projection échoue — retry infini jusqu'à dead ? log + skip ? autre comportement ?
    
8. Désactivation (`isActive=false`) d'un client_user FO : le miroir BO doit refléter `isActive=false` ? Doit-il aussi déclencher le signalement du point 4 ?
    
9. Worker down / extranet injoignable : on accepte la consistance éventuelle (events s'accumulent dans `outbox` FO, rattrapés au redémarrage) — ou y a-t-il une SLA/latence max attendue ?
    
10. Rejeu idempotent : si le même event est rejoué (crash entre commit BO et ack HTTP), la projection doit être un upsert idempotent sur `(id)` — confirmé ?
    

### Axis 3 — Data model & persistence

11. Les clients FO et BO partagent-ils le même UUID (`client.id`, `client_user.id`) comme clé de corrélation pour la projection ? Ou faut-il un mapping externe ?
    
12. La synchro est-elle unidirectionnelle FO→BO uniquement pour cette US (le BO ne pousse jamais de modifications client vers le FO dans ce périmètre) ?
    
13. Faut-il persister côté BO autre chose que la mise à jour des tables miroir `client` / `client_user` (ex. table d'historique des modifications client) — ou dériver à la volée depuis `inbox` + `audit_log` suffit ?
    

### Axis 4 — API contract & breaking changes

14. Le payload de `client.profile_updated.v1` (ou l'event retenu) doit-il inclure un `updated_by` (clientUserId de l'acteur FO) pour alimenter le signalement / audit_log ? Si oui, quels champs d'identité minimum (`clientUserId`, `email`, les deux) ?
    
15. Si on retient un event par entité plutôt qu'un event générique, faut-il documenter le payload dans `docs/metier/events-outbox-inbox.md` avant implémentation (nouvelle ligne ou précision sur `client.profile_updated.v1`) ?
    

### Axis 5 — Auth & permissions

16. Côté projection BO, y a-t-il une vérification métier au-delà de l'idempotence (ex. refuser la synchro si le client BO est en statut `INACTIVE`/`révoqué`) — ou toute modification FO valide côté API FO est acceptée sans condition côté BO ?
    
17. Le signalement (point 4) est-il visible par tous les admins/gestionnaires BO, ou seulement l'avocat assigné au dossier (s'il existe une notion d'assignation — pas visible dans le code actuel) ?
    

### Axis 6 — UX / display

18. Côté BO, après synchro, faut-il un indicateur visuel sur la fiche client (« modifié par le client le … ») — ou la simple mise à jour silencieuse des champs miroir suffit pour cette US ?
    
19. En cas d'échec permanent (event en `outbox_dead` côté FO, ou `processed_at IS NULL` côté BO sans projection), y a-t-il une action ops attendue (alerte, dashboard) — ou hors scope pour l'instant ?
    

---

Réponds à ces 19 points (numérotés) et je synthétise la domain spec dans `docs/features/client-profile-sync/domain.md` (ou un slug que tu préfères) avant toute branche ou code.
```


## Réponse

```
Domain spec écrite dans `docs/features/client-profile-sync/domain.md`.

### Synthèse de tes réponses

|Décision|Retenu|
|---|---|
|Périmètre sync|`PATCH /client` uniquement — pas de synchro `client_user`|
|Payload|Delta (`changes`) + `clientId` + `updatedByClientUserId`|
|Champs protégés|`planType`, `planLevel`, `creditBalance`, `status` jamais touchés|
|Signalement|Ligne `audit_log` avec le `clientUserId` auteur|
|Corrélation|Même UUID ; création toujours côté BO|
|Concurrence|Last-write-wins|
|Rattrapage|Events en file jusqu'au redémarrage worker|
|UI BO|Silencieuse (indicateur futur noté)|
|Ops dead letter|Hors scope (noté pour plus tard)|

### Point 5 — création client FO

Aucune route de création client côté front-office trouvée. Seules routes client FO : `GET /client` et `PATCH /client` dans `packages/front-office/@libs/users-backend/src/init.ts`. Les clients FO sont seedés en tests via `createClient()` dans le setup de test, pas via API publique.

### Assumptions à valider

La section Assumptions couvre surtout le cas anormal « client BO introuvable » : log warning, pas de création à la volée, event acquitté quand même pour ne pas bloquer la file.

Domain spec écrite dans `docs/features/client-profile-sync/domain.md`. Merci de relire la section Assumptions en particulier. Approuve pour continuer, ou signale les changements souhaités.
```

## Commande 
```
aucune modification pour l'sintant on reste en analyse. la reponse est simple (soit celui-ci ou soit celui-ci)

je veux que tu lise les deux domain @docs/features/client-profile-sync/domain.md et @docs/features/us002-sync-front-to-back/domain.md

etr que tu me dise quel est le meilleur des deux (celui qui permettra de mieux faire l'impémentation demander)

tu dois dire le nom du fichier et en 4 point simple pourquoi
```

## Réponse 

```
docs/features/us002-sync-front-to-back/domain.md`

1. Il pointe le code existant — `UpdateClientRoute`, `OutboxEventEntity`, `eventsFetcherRunner`, `projections/index.ts` : tu sais exactement où coder, sans inventer un helper `publishEvent`.
    
2. Il couvre les cas PATCH concrets — body vide → pas d'event, `null` = changement valide : règles directement applicables dans la route FO.
    
3. Il décrit le vrai comportement de l'infra — retry sans dead-letter, idempotence via `ON CONFLICT`, rollback si projection en échec : tu évites de mal brancher la projection sur le worker existant.
    
4. Il cadrre le scope immédiat — pas de migration, pas de synchro `client_user`, dépendance `@client-libs/events-backend` : moins de décisions à prendre au moment de coder.
```