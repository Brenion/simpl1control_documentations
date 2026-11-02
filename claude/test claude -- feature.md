
## commande

```
/nox (racine)/feature-resolve 1780573340658828474 on arrive a la Synchronisation front -> back

bien regarder aux documents @docs/metier/events-outbox-inbox.md et

@docs/techniques/worker-outbox.md
```

## reponse

```
Il y a déjà un events-fetcher-tick.ts côté back-office. Je le lis.

  Searched for 2 patterns, read 10 files (ctrl+o to expand)

⏺ Maintenant j'ai tout le contexte nécessaire. Je fais le bilan avant de vous poser
  mes questions.

  ---
  Complexité : Medium
  
  L'infrastructure bout-en-bout est déjà en place :
  - Front-office : table outbox + GET /relay/events + POST /relay/events/ack
  - Back-office : makeEventsFetcherTick déjà intégré dans startWorker, table inbox,
  projections: Record<string, Projection> prête (mais vide)
  - Event type CLIENT_PROFILE_UPDATED_V1 déjà déclaré dans event-types.ts

  Ce qui manque :
  1. UpdateClientRoute ne publie pas encore dans l'outbox front-office lors d'un
  PATCH client
  2. UpdateRoute (client_user) idem
  3. Aucune projection côté back-office pour traiter client.profile_updated.v1
  4. Pas d'event type ni de projection pour les modifications de client_user

  ---
  Questions (phase 2)

  1. Payload de client.profile_updated.v1 — snapshot ou delta ?
  La route UpdateClientRoute accepte des champs optionnels. Le payload émis doit-il
  contenir l'intégralité des champs modifiables du client (snapshot : name, 
  vatNumber, isVatSubject, addressStreet, addressZip, addressCity, addressCountry, 
  sector) même si seul un champ a changé ? Ou uniquement les deltas (les champs
  effectivement fournis dans le PATCH) ?

  2. "Un signalement est tracé" — que faire concrètement ?
  Le catalogue events-outbox-inbox.md dit "un signalement est tracé" côté back-office
   quand client.profile_updated.v1 arrive. Que signifie-t-il concrètement : écriture
  dans la table audit_log back-office ? Un flag pendingReview sur l'entité client du
  back-office ? Simplement le processed_at de l'inbox suffit ?

  3. Modifications de client_user — nouvel event type ou dans profile_updated ?
  client.user_updated.v1 n'existe pas dans le catalogue. Pour notifier le back-office
   d'un client_user modifié, faut-il créer un nouveau type client.user_updated.v1 (à
  documenter dans le catalogue) ? Ou intègre-t-on cette info dans un payload enrichi
  de client.profile_updated.v1 ?

  4. Quels champs client_user synchroniser côté back-office ?
  Le back-office ClientUser possède : firstName, lastName, email, phone, language, 
  isActive, role. Quand la projection s'exécute, quels champs doit-elle écraser ?
  Notamment : email (risqué si utilisé comme identifiant) et isActive (soft-delete)
  sont-ils inclus ?

  5. Soft-delete d'un client_user front-office
  Si un client_user est supprimé (soft-delete) depuis le front-office, faut-il
  déclencher un event vers le back-office ? Si oui, c'est le même
  client.user_updated.v1 avec isActive: false, ou un event distinct ?

  6. Conflit de données
  La projection back-office écrase-t-elle toujours les champs reçus, sans condition ?
   Ou faut-il ignorer/signaler si le back-office a des données plus récentes (ex. un
  admin a modifié le client côté intranet entre-temps) ?
```

## commande
  
```
1 - seul les changement doivent etre emis + le client et le client_user afin de
  savoir pour qui et pourquoi
  2 - lorsque le message arrive il doit être enregistré dans l'inbox. ensuite il 
  faut faire la save du client en back-office quand cela est fait ca doit 
  disparaitre de l'inbox. l'audit_log permet de savoir qui a emit le message. si 
  l'enregistrement rate il faut que ca reste en inbox. 
  3 - le client_user ne peut être modifier par le front-office les route vont être
  supprimée sur le back-office peut le faire.
  4 - aucun vu la response a la 3
  5 - pareil que 4
  6 - pour l'instant on par sur le ecrasement quoi qu'il arrive mais le noté pour
  le futur 
```  

## Réponse

```
  Points clés à valider avant de continuer :

  1. Delta = champs du PATCH, pas de diff avant/après — si le client envoie la même
  valeur, un event est quand même émis. C'est le choix le plus simple mais ça génère
  des events "vides de sens". Confirmé ?
  2. Un PATCH avec zéro attribut → pas d'event émis. Confirmé ?
  3. Champ null dans le PATCH (ex. effacer un vatNumber) → inclus dans le payload.
  Confirmé ?
  4. Assumption sur la projection : si le client_id du payload n'existe pas dans le
  back-office, la projection lance une erreur et l'event est retenté. OK ou faut-il
  logger + skip silencieusement ?

  Si tout est bon, je passe au plan d'implémentation.
```
  
## commande

```
   non pas d'implémentation tu fini fini la doc avec les reponses a tes nouvelle 
  question
  
  1 - non si aucun champ dnas le FO a ce moment la on ne fait pas de patch
  2 - pas d'event
  3 - oui (mais a annoté comme point d'attention pour le futur)
  4 - le worker normalement recommence 10 fois puis il va mettre dnas le
  outbox_dead? vérifie 
```
  
## Reponse

```
Spec à jour. Voici le résumé des corrections apportées :

  Q1 — Delta sans diff : le frontend ne soumet pas de PATCH si rien n'a changé → à l'API, chaque PATCH entrant a au moins un vrai changement.
   Documenté comme assumption.

  Q2 — PATCH vide : pas d'event. Géré côté UI, mais le backend l'absorbe aussi si attributes = {}.

  Q3 — null valide : oui, inclus dans le payload. Ajouté comme point d'attention futur dans les assumptions : un null issu d'un bug UI est
  indiscernable d'un effacement intentionnel.

  Q4 — Vérification du worker : non, le mécanisme 10 tentatives + outbox_dead n'existe pas pour le fetcher. C'est exclusif au publisher. Le
  eventsFetcherRunner retente indéfiniment toutes les 2 s sans dead-letter. Un event dont la projection échoue de façon permanente (ex.
  client_id inconnu) bloque la file. Documenté comme limitation connue dans les assumptions + edge cases.

  La doc est à docs/features/us002-sync-front-to-back/domain.md. Quand tu veux passer à l'implémentation, dis-le moi.
```

## commande

```
aucune modification pour l'sintant on reste en analyse. la reponse est simple (soit celui-ci ou soit celui-ci)
  
  je veux que tu lise les deux domain @docs/features/client-profile-sync/domain.md  et @docs/features/us002-sync-front-to-back/domain.md
  etr que tu me dise quel est le meilleur des deux (celui qui permettra de mieux faire l'impémentation demander) 
  
  tu dois dire le nom du fichier et en 4 point simple pourquoi 
```

## Réponse 

```
docs/features/client-profile-sync/domain.md

  1. Payload complet et nommé — il documente la structure JSON avec les noms de champs exacts (clientId, updatedByClientUserId, changes) et
  liste explicitement les clés autorisées dans changes. L'autre doc ne le fait pas.
  2. Comportement client-BO introuvable correct — il dit : log warning + acquitter l'event pour ne pas bloquer la file. Mon doc dit : lancer
  une erreur → retry infini, ce qui est un choix pire opérationnellement et contredit l'architecture existante.
  3. User stories des deux côtés — il exprime le besoin côté client-user ET côté admin, ce qui ancre mieux les critères d'acceptance. Le mien
   n'a que le côté back-office.
  4. Approche technique plus précise — il identifie la création d'un publishEvent dans @client-libs/events-backend (mirror du pattern
  back-office) plutôt qu'un inline em.create() ad hoc, et note que events-outbox-inbox.md doit être mis à jour dans la même PR.
```