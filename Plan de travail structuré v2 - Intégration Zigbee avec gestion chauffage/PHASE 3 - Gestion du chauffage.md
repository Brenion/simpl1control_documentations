## PHASE 3 : Gestion du chauffage (Semaines 9-12 | 40-48h)

---

> **Principes architecturaux non négociables pour cette phase :**
>
> - Les devices custom MQTT et LOGO suivent le **même flux de traitement** que les devices Zigbee (`capabilities.controls`, `device_states`, WebSocket). Aucun traitement spécifique dans le core backend.
> - Le Siemens Logo est un `device` avec `protocol=logo` et `capabilities.blocks` (Q/I/M/V). La communication passe par le même flux que tout device.
> - Une zone de chauffage peut référencer des devices multi-protocole simultanément (TRV Zigbee ET/OU blocs LOGO).
> - La source de vérité des températures courantes est `device_states.state[key]` — jamais un champ dédié sur une autre table.
> - Topics MQTT avec préfixe `s1c/` : `s1c/zigbee2mqtt/`, `s1c/devices/`, `s1c/internal/telemetry/`.

---

### A - Architecture chauffage et driver unifié (PRÉREQUIS — à faire avant toute implémentation)

**User Stories :**

- **SC-US-PH3-NEW-01** : En tant que développeur, je veux que les devices custom MQTT et LOGO suivent le même flux de traitement que les devices Zigbee (`capabilities.controls`, `device_states`, WebSocket) sans traitement spécifique dans le core
- **SC-US-PH3-NEW-02** : En tant que développeur, je veux un parser LOGO qui traduit les blocs Q/I/M/V en payload standardisé exploitable par le même flux MQTT que les autres devices
- **SC-US-PH3-NEW-03** : En tant que développeur, je veux définir le schéma complet de `heating_zones` et `heating_schedules` avant d'implémenter

**Tâches :**

- **SC-PH3-T00** : Produire document `SC-PH3-T00_schema-heating.md` : schéma `heating_zones` / `heating_schedules`, articulation avec `devices` (protocol=logo, protocol=zigbee pour TRV, protocol=mqtt pour capteurs) (SC-US-PH3-NEW-03)
- **SC-PH3-LOGO-T01** : Valider que le flux générique MQTT (UPSERT `device_states` + publish `s1c/internal/telemetry/{deviceId}/{key}` + EMIT WebSocket) couvre nativement les devices custom MQTT sans code spécifique (SC-US-PH3-NEW-01)
- **SC-PH3-LOGO-T02** : Implémenter parser payload MQTT LOGO → normaliser en payload clé/valeur → injecter dans le même flux générique que Zigbee et custom MQTT (SC-US-PH3-NEW-02)
- **SC-PH3-LOGO-T03** : Implémenter traduction commande `capabilities.controls` → payload LOGO (`{ "Q1": true }`) → publish sur `devices.publish` (SC-US-PH3-NEW-02)
- **SC-PH3-LOGO-T04** : Implémenter watchdog LOGO : passer `technicalStatus=unreachable` si `lastSeenAt` > 2min (même job périodique que les autres devices) (SC-US-PH3-NEW-02)
- **SC-PH3-LOGO-T05** : Créer migration `heating_zones` (schéma à définir dans SC-PH3-T00) (SC-US-PH3-01)
- **SC-PH3-LOGO-T06** : Créer migration `heating_schedules` (schéma à définir dans SC-PH3-T00) (SC-US-PH3-02)
- **SC-PH3-LOGO-T07** : Implémenter modèles ORM `HeatingZone`, `HeatingSchedule` (SC-US-PH3-01)
- **SC-PH3-LOGO-T08** : Créer seeders `heating_zones` et `heating_schedules` réalistes (SC-US-PH3-01)

### B - API REST Chauffage

**User Stories :**

- **SC-US-PH3-01** : En tant qu'utilisateur, je veux définir des zones de chauffage avec des températures cibles différentes
- **SC-US-PH3-02** : En tant qu'utilisateur, je veux créer des plannings horaires de chauffage par zone
- **SC-US-PH3-03** : En tant qu'utilisateur, je veux pouvoir forcer manuellement une température dans une zone (override)
- **SC-US-PH3-04** : En tant qu'utilisateur, je veux associer des devices multi-protocole à chaque zone (TRV Zigbee ET/OU blocs LOGO, selon le schéma défini dans SC-PH3-T00)
- **SC-US-PH3-05** : En tant qu'utilisateur, je veux voir l'état actuel de chaque zone (température, consigne, demande)

**Tâches :**

- **SC-PH3-T01** : Créer endpoint `GET /api/heating/zones` (liste zones) (SC-US-PH3-01)
- **SC-PH3-T02** : Créer endpoint `POST /api/heating/zones` (créer zone) (SC-US-PH3-01)
- **SC-PH3-T03** : Créer endpoint `PUT /api/heating/zones/:id` (modifier zone) (SC-US-PH3-01)
- **SC-PH3-T04** : Créer endpoint `DELETE /api/heating/zones/:id` (supprimer zone) (SC-US-PH3-01)
- **SC-PH3-T05** : Créer endpoint `GET /api/heating/zones/:id/status` (état temps réel) (SC-US-PH3-05)
- **SC-PH3-T06** : Créer endpoint `POST /api/heating/zones/:id/override` (forcer température) (SC-US-PH3-03)
- **SC-PH3-T07** : Créer endpoint `GET /api/heating/schedules` (liste plannings) (SC-US-PH3-02)
- **SC-PH3-T08** : Créer endpoint `POST /api/heating/schedules` (créer planning) (SC-US-PH3-02)
- **SC-PH3-T09** : Créer endpoint `PUT /api/heating/schedules/:id` (modifier planning) (SC-US-PH3-02)
- **SC-PH3-T10** : Créer endpoint `POST /api/heating/schedules/:id/activate` (activer sur zone) (SC-US-PH3-02)
- **SC-PH3-T11** : Implémenter validation des payloads chauffage avec `fastify-type-provider-zod` (SC-US-PH3-01 à SC-US-PH3-05)
- **SC-PH3-T12** : Créer tests unitaires pour controllers chauffage (SC-US-PH3-01 à SC-US-PH3-05)

### C - Logique de régulation

**User Stories :**

- **SC-US-PH3-06** : En tant que système, je veux réguler le chauffage automatiquement selon les demandes des zones
- **SC-US-PH3-07** : En tant qu'administrateur, je veux que le système soit évolutif (changement source chauffage)
- **SC-US-PH3-08** : En tant que système, je veux commander les vannes thermostatiques Zigbee par zone
- **SC-US-PH3-09** : En tant que système, je veux communiquer avec le Siemens Logo via MQTT — le LOGO est un `device` avec `protocol=logo`, `capabilities.blocks` (Q/I/M/V) ; la communication passe par le même flux que tout device, avec publish sur `devices.publish` topic

**Tâches :**

- **SC-PH3-T13** : Créer service de régulation (boucle 30s) — lit `device_states.state[key]` pour la température courante, pas un champ dédié (SC-US-PH3-06)
- **SC-PH3-T14** : Implémenter logique calcul demande par zone (hystérésis ±0.5°C) (SC-US-PH3-06)
- **SC-PH3-T15** : Implémenter agrégation demandes multi-zones (SC-US-PH3-06)
- **SC-PH3-T16** : Implémenter commande vannes TRV Zigbee via MQTT sur `s1c/zigbee2mqtt/[friendlyName]/set` (SC-US-PH3-08)
- **SC-PH3-T17** : Implémenter communication MQTT avec Siemens Logo : le LOGO est un `device` avec `protocol=logo`, `capabilities.blocks` (Q/I/M/V) — la commande publie sur `devices.publish` topic avec le payload `{ "Q1": true }` (SC-US-PH3-09)
- **SC-PH3-T18** : Créer interface abstraite `HeatingSource` (SC-US-PH3-07)
- **SC-PH3-T19** : Implémenter `SiemensLogoHeatingSource` : encapsule la traduction `heating_zone → blocs LOGO → commande MQTT` via l'abstraction `HeatingSource` (SC-US-PH3-07, SC-US-PH3-09)
- **SC-PH3-T20** : Préparer structure pour `HeatPumpHeatingSource` (documentation) (SC-US-PH3-07)
- **SC-PH3-T21** : Implémenter watchdog Logo : passer `technicalStatus=unreachable` sur `device_states` si `lastSeenAt` > 2min (SC-US-PH3-09)
- **SC-PH3-T22** : Implémenter logging décisions régulation (SC-US-PH3-06)
- **SC-PH3-T23** : Gérer priorités zones (mode antigel, etc.) (SC-US-PH3-06)
- **SC-PH3-T24** : Créer tests unitaires pour service régulation (SC-US-PH3-06 à SC-US-PH3-09)
- **SC-PH3-T25** : Créer tests intégration communication Logo (SC-US-PH3-09)

### D - Préparation installation physique

**User Stories :**

- **SC-US-PH3-10** : En tant qu'administrateur, je veux documenter le câblage entre le Siemens Logo et le chauffage
- **SC-US-PH3-11** : En tant qu'administrateur, je veux que l'installation soit modulaire pour changement futur de chaudière

**Tâches :**

- **SC-PH3-T26** : Analyser câblage actuel du chauffage (circulateur, thermostat) (SC-US-PH3-10)
- **SC-PH3-T27** : Identifier entrées/sorties Logo nécessaires (SC-US-PH3-10)
- **SC-PH3-T28** : Créer schéma de câblage détaillé (Logo ↔ Relais ↔ Chaudière) (SC-US-PH3-10)
- **SC-PH3-T29** : Prévoir système découplage pour remplacement chaudière (SC-US-PH3-11)
- **SC-PH3-T30** : Documenter sécurités électriques nécessaires (disjoncteur, protection) (SC-US-PH3-10)
- **SC-PH3-T31** : Créer procédure installation avec photos/schémas (SC-US-PH3-10)
- **SC-PH3-T32** : Prévoir emplacement physique Logo + alimentation (SC-US-PH3-10)
- **SC-PH3-T33** : Créer checklist validation post-installation (SC-US-PH3-10)
- **SC-PH3-T34** : Tester connexion MQTT Logo ↔ Raspberry Pi (WiFi/Ethernet) sur topic `s1c/devices/` (SC-US-PH3-09)

### E - Tâches frontend (accompagnent les features chauffage)

**User Stories :**

- **SC-US-PH3-01** : En tant qu'utilisateur, je veux définir des zones de chauffage avec des températures cibles différentes
- **SC-US-PH3-05** : En tant qu'utilisateur, je veux voir l'état actuel de chaque zone (température, consigne, demande)

**Tâches :**

- **SC-PH3-FE-T01** : Définir types TypeScript `HeatingZone`, `HeatingSchedule` (schéma défini dans SC-PH3-T00) (SC-US-PH3-01)
- **SC-PH3-FE-T02** : Créer service API `heating/zones` et `heating/schedules` (CRUD + override + activate) (SC-US-PH3-01)
- **SC-PH3-FE-T03** : Créer store `heatingZones` (zones actives, température courante depuis `device_states`, demande) (SC-US-PH3-05)
- **SC-PH3-FE-T04** : Implémenter mise à jour temps réel des zones via WebSocket (température courante = `device_states.state[key]` des TRV/LOGO associés) (SC-US-PH3-05)
