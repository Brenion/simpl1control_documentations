## Tier 0 — Mise en conformité Migration (AVANT tout le reste)

Ces tâches sont un prérequis absolu. Sans elles, tout ce qui suit en Phase 2 reposera sur du sable.

| #   | Tâche                                                                                                               | Action concrète                                      |
| --- | ------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------- |
| 0.1 | Passer `DB_SYNCHRONIZE=false` dans `development.env`                                                                | 1 ligne à changer                                    |
| 0.2 | Corriger le script `seed:run` — extraire `seed:run:dev` sans le double drop                                         | `package.json`                                       |
| 0.3 | Corriger le script `regen:dev` pour qu'il n'appelle plus `seed:run` (qui re-drop)                                   | `package.json`                                       |
| 0.4 | Régénérer la migration initiale pour qu'elle corresponde à l'entité `DeviceEntity` actuelle (7 colonnes manquantes) | Nouvelle migration ou mise à jour de `1747259713078` |
| 0.5 | Valider `migration:run` sur base vierge → vérifier que le schéma résultant = entités                                | Test de non-régression                               |

---

## Tier 1 — Fondations bloquantes (inchangé)

|#|Tâche|Pourquoi en premier|
|---|---|---|
|1|SC-PH2-T01|Analyser le schéma existant|
|2|SC-PH2-T02|Identifier ce qu'on modifie vs ce qu'on crée|
|3|SC-PH2-T03|Documenter l'ER existant|
|4|SC-PH2-T04|Plan de migration|
|5|SC-PH2-T05|Migration `rooms`|
|6|SC-PH2-T06|Migration `devices` (modification)|
|7|SC-PH2-T13|ORM `Room`|
|8|SC-PH2-T14|ORM `Device` (modification)|

---

## Tier 2 — Modèle de données

|#|Tâche|Dépend de|
|---|---|---|
|9|SC-PH2-T07|Migration `heating_zones`|
|10|SC-PH2-T08|Migration `heating_schedules`|
|11|SC-PH2-T09|Migration `automations`|
|12|SC-PH2-T10|Migration `automation_logs`|
|13|SC-PH2-T11|Migration `scenes`|
|14|SC-PH2-T12|Migration `device_updates`|
|15|SC-PH2-T15|ORM `HeatingZone`|
|16|SC-PH2-T16|ORM `HeatingSchedule`|
|17|SC-PH2-T17|ORM `Automation`|
|18|SC-PH2-T18|ORM `AutomationLog`|
|19|SC-PH2-T19|ORM `Scene`|
|20|SC-PH2-T20|ORM `DeviceUpdate`|

---

## Tier 3 — Seeders

|#|Tâche|Dépend de|
|---|---|---|
|21|SC-PH2-T21|MAJ seeders existants|
|22|SC-PH2-T22|Seeders `rooms`|
|23|SC-PH2-T23|Seeders `devices`|
|24|SC-PH2-T24|Seeders `heating_zones`|
|25|SC-PH2-T25|Seeders `heating_schedules`|
|26|SC-PH2-T26|Seeders `automations`|
|27|SC-PH2-T27|Seeders `scenes`|

---

## Tier 4 — API REST

|#|Tâche|Dépend de|
|---|---|---|
|28|SC-PH2-T37|`GET /api/devices` avec filtres|
|29|SC-PH2-T38|`GET /api/devices/:id`|
|30|SC-PH2-T40|`PUT /api/devices/:id`|
|31|SC-PH2-T46|`GET /api/rooms`|
|32|SC-PH2-T47|`POST /api/rooms`|
|33|SC-PH2-T45|`GET /api/devices/:id/state`|
|34|SC-PH2-T42|`POST /api/devices/:id/command`|
|35|SC-PH2-T39|`POST /api/devices`|
|36|SC-PH2-T41|`DELETE /api/devices/:id` + unpair|
|37|SC-PH2-T43|`POST /api/devices/pairing/start`|
|38|SC-PH2-T44|`POST /api/devices/pairing/stop`|
|39|SC-PH2-T48|Validation Zod tous endpoints|
|40|SC-PH2-T49|Tests controllers|

---

## Tier 5 — Services MQTT / Sync

|#|Tâche|Dépend de|
|---|---|---|
|41|SC-PH2-T55|Sync démarrage `bridge/devices`|
|42|SC-PH2-T50|MQTT Listener `zigbee2mqtt/#`|
|43|SC-PH2-T51|Vérifier MQTT Listener `heating/#`|
|44|SC-PH2-T52|MAJ état devices en DB depuis MQTT|
|45|SC-PH2-T53|Auto-découverte nouveaux appareils|
|46|SC-PH2-T54|Gestion online/offline timeout 5 min|
|47|SC-PH2-T56|Détection devices orphelins|
|48|SC-PH2-T57|WebSocket broadcast états temps réel|
|49|SC-PH2-T58|Tests services MQTT|

---

## Tier 6 — Impact Frontend

|#|Tâche|Dépend de|
|---|---|---|
|50|SC-PH2-T28|Analyser frontend existant|
|51|SC-PH2-T29|Identifier composants impactés|
|52|SC-PH2-T30|Documenter adaptations nécessaires|
|53|SC-PH2-T31|Adapter interfaces TypeScript|
|54|SC-PH2-T32|Adapter services API frontend|
|55|SC-PH2-T33|Adapter stores/state management|
|56|SC-PH2-T34|Adapter composants existants|
|57|SC-PH2-T35|Composants basiques devices/rooms|
|58|SC-PH2-T36|Tests frontend|

---

## Tier 7 — OTA / Firmware

|#|Tâche|Dépend de|
|---|---|---|
|59|SC-PH2-T65|Logging historique `device_updates`|
|60|SC-PH2-T59|`GET /api/devices/:id/updates`|
|61|SC-PH2-T63|Progression OTA via MQTT|
|62|SC-PH2-T60|`POST /api/devices/:id/update`|
|63|SC-PH2-T61|`GET /api/devices/:id/update/status`|
|64|SC-PH2-T64|Notification WebSocket progression|
|65|SC-PH2-T62|Cron hebdomadaire vérification OTA|
|66|SC-PH2-T66|Tests service updates|

---

## Vue d'ensemble

[TIER 0] Fix migration infra (synchronize=false, scripts, migration complète)

↓

[TIER 1] Analyse + migrations Room & Device (pivot central)

↓

[TIER 2] Autres migrations + ORMs (en parallèle entre elles)

↓

[TIER 3] Seeders (T21 en urgence dès T06 terminé)

↓

[TIER 4] API REST [TIER 6 T28-T30] Analyse frontend (parallèle)

↓ ↓

[TIER 5] Services MQTT [TIER 6 T31-T36] Adaptation frontend

↓

[TIER 7] OTA / Firmware (dernier)