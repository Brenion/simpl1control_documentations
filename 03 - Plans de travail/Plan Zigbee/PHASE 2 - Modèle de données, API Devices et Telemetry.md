## PHASE 2 : Modèle de données, API Devices et Telemetry (Semaines 6-8 | 40-50h)

---

> **Prérequis de démarrage (ventilés depuis Phase 1) :**
>
> - **SC-PH2-PRE-T01** : Vérifier que `MQTT_START=true` dans `development.env` avant tout développement MQTT en Phase 2
> - **SC-PH2-PRE-T02** : Vérifier que `test.env` pointe sur `localhost` pour le container Mosquitto (cohérence avec Phase 1 T23)
> - **SC-PH2-PRE-T03** : Documenter et valider la convention topics `s1c/` dans l'architecture MQTT du projet (source de vérité pour toutes les phases suivantes)

---

### A - Package telemetry (PRÉREQUIS ABSOLU — à bootstrapper EN PREMIER)

> `packages/telemetry` doit être bootstrappé et fonctionnel **avant** que le backend commence à publier sur `s1c/internal/telemetry/`. Sans service actif pour consommer ces messages, ils sont perdus. Le service doit tourner en dev dès le début de Phase 2, en parallèle du développement backend.

**User Stories :**

- **SC-US-PH2-NEW-01** : En tant que développeur, je veux bootstrapper le package `packages/telemetry` (Fastify v5 + TypeORM) à partir du socle backend, avec connexion TimescaleDB et client MQTT
- **SC-US-PH2-NEW-02** : En tant que système, je veux historiser les données IoT dans TimescaleDB selon les policies configurées, avec agrégation automatique
- **SC-US-PH2-NEW-03** : En tant que super admin, je veux configurer les policies d'historisation par device et par clé de donnée

**Tâches :**

- **SC-PH2-TEL-T01** : Bootstrapper `packages/telemetry` : copier socle Fastify v5 + TypeORM du backend, retirer auth/users/devices/seeders (SC-US-PH2-NEW-01)
- **SC-PH2-TEL-T02** : Configurer `data-source.ts` vers TimescaleDB (variables `TIMESCALE_HOST`, `TIMESCALE_PORT`, `TIMESCALE_DB`) (SC-US-PH2-NEW-01)
- **SC-PH2-TEL-T03** : Ajouter service `timescaledb` + service `telemetry` dans `docker-compose.yml` de dev (avec volumes persistants) (SC-US-PH2-NEW-01)
- **SC-PH2-TEL-T04** : Créer migration hypertable `device_history` (time, deviceId, key, value, raw jsonb) (SC-US-PH2-NEW-02)
- **SC-PH2-TEL-T05** : Créer Continuous Aggregate `agg_15min` (rétention 1 an) (SC-US-PH2-NEW-02)
- **SC-PH2-TEL-T06** : Créer Continuous Aggregate `agg_1d` (rétention indéfinie) (SC-US-PH2-NEW-02)
- **SC-PH2-TEL-T07** : Configurer Retention Policy : `device_history` 30 jours brut (SC-US-PH2-NEW-02)
- **SC-PH2-TEL-T08** : Implémenter client MQTT abonné à `s1c/internal/telemetry/#` (SC-US-PH2-NEW-02)
- **SC-PH2-TEL-T09** : Implémenter INSERT `device_history` à chaque message reçu (SC-US-PH2-NEW-02)
- **SC-PH2-TEL-T10** : Valider en dev : publier un message test sur `s1c/internal/telemetry/test/temperature` et vérifier l'INSERT dans TimescaleDB (SC-US-PH2-NEW-02)
- **SC-PH2-TEL-T11** : Implémenter routing automatique `timeWindow` → brut / `agg_15min` / `agg_1d` (SC-US-PH2-NEW-02)
- **SC-PH2-TEL-T12** : Créer endpoint `GET /telemetry/:deviceId/:key?timeWindow=...` (SC-US-PH2-NEW-02)

### B - Table telemetry_policies (backend principal)

**User Stories :**

- **SC-US-PH2-NEW-03** : En tant que super admin, je veux configurer les policies d'historisation par device et par clé de donnée

**Tâches :**

- **SC-PH2-POL-T01** : Créer migration `telemetry_policies` (id, deviceId FK, key, enabled, minInterval, onChangeOnly) (SC-US-PH2-NEW-03)
- **SC-PH2-POL-T02** : Implémenter modèle ORM `TelemetryPolicy` (SC-US-PH2-NEW-03)
- **SC-PH2-POL-T03** : Créer endpoints CRUD `GET/POST/PUT/DELETE /api/telemetry-policies` (super_admin uniquement) (SC-US-PH2-NEW-03)
- **SC-PH2-POL-T04** : Implémenter logique d'évaluation des policies dans le flux MQTT du backend principal (SC-US-PH2-NEW-03)
- **SC-PH2-POL-T05** : Créer seeders policies par défaut (température 120s, state onChangeOnly, batterie 3600s) (SC-US-PH2-NEW-03)

### C - Analyse et adaptation schéma existant

**User Stories :**

- **SC-US-PH2-01** : En tant que développeur, je veux analyser le schéma de base de données existant et identifier les tables à modifier
- **SC-US-PH2-02** : En tant que développeur, je veux créer le schéma de base de données complet pour le système en tenant compte de l'existant

**Tâches :**

- **SC-PH2-T01** : Analyser schéma de base de données existant (tables, relations, contraintes) (SC-US-PH2-01)
- **SC-PH2-T02** : Identifier tables existantes à modifier/étendre (SC-US-PH2-01)
- **SC-PH2-T03** : Documenter structure actuelle (ER diagram de l'existant) (SC-US-PH2-01)
- **SC-PH2-T04** : Créer plan de migration (nouvelles tables + modifications tables existantes) (SC-US-PH2-01)

### D - Migrations et modèles de données

**User Stories :**

- **SC-US-PH2-03** : En tant que développeur, je veux migrer uniquement les tables prérequis cross-feature en Phase 2 — les autres migrations suivent leur phase naturelle
- **SC-US-PH2-04** : En tant que développeur, je veux implémenter les modèles ORM correspondants
- **SC-US-PH2-05** : En tant que développeur, je veux mettre à jour les seeders existants et créer de nouveaux seeders réalistes
- **SC-US-PH2-NEW-04** : En tant qu'utilisateur, je veux créer et gérer des profils d'interface adaptés à mes appareils (mobile, tablette, desktop)
- **SC-US-PH2-NEW-05** : En tant qu'utilisateur, je veux organiser mes tuiles dans des pages au sein de mon profil
- **SC-US-PH2-NEW-06** : En tant qu'utilisateur, je veux créer des tuiles configurables pointant vers un device, un groupe ou une scène Zigbee
- **SC-US-PH2-NEW-07** : En tant que système, je veux gérer les groupes Zigbee et les scènes firmware Zigbee via MQTT bridge
- **SC-US-PH2-NEW-08** : En tant qu'administrateur, je veux choisir les keys à exposer dans une tuile parmi les capacités d'un device

**Tâches :**

- **SC-PH2-T06** : Créer migration pour table `devices` : supprimer `ieee_address`, `brand`, `model`, `is_online`, `current_state` — ajouter `protocol` enum (zigbee/mqtt/logo/zigbee_group), `friendlyName`, `adminStatus`, `publish` (topic), `subscribe` (topic), `capabilities` jsonb (SC-US-PH2-03)
- **SC-PH2-T14** : Implémenter modèle ORM `Device` : refonte complète selon nouveau schéma SC-PH2-T01 (capabilities.controls, protocol enum, sans room_id, sans current_state) (SC-US-PH2-04)
- **SC-PH2-DS-T01** : Créer migration `device_states` (deviceId PK FK, technicalStatus, lastSeenAt, updatedAt, state jsonb) (SC-US-PH2-14)
- **SC-PH2-DS-T02** : Implémenter modèle ORM `DeviceState` (SC-US-PH2-14)
- **SC-PH2-DS-T03** : Implémenter job périodique : passer `technicalStatus=offline` si `lastSeenAt` > seuil configurable (SC-US-PH2-14)
- **SC-PH2-PR-T01** : Créer migration `profiles` (id, userId FK, name, deviceType enum, isDefault) (SC-US-PH2-NEW-04)
- **SC-PH2-PR-T02** : Créer migration `pages` (id, profileId FK, name, position) (SC-US-PH2-NEW-05)
- **SC-PH2-PR-T03** : Créer migration `cards` (id, pageId FK, name, isActive, position, targetType enum, targetId loose, selectedKeys varchar[]) (SC-US-PH2-NEW-06)
- **SC-PH2-PR-T04** : Implémenter modèles ORM `Profile`, `Page`, `Card` (SC-US-PH2-NEW-04/05/06)
- **SC-PH2-PR-T05** : Créer profil `isDefault` automatiquement à la création d'un user (règle applicative) (SC-US-PH2-NEW-04)
- **SC-PH2-PR-T06** : Créer endpoints CRUD profiles/pages/cards (SC-US-PH2-NEW-04/05/06)
- **SC-PH2-PR-T07** : Implémenter logique détection profil : User-Agent → deviceType → profil matching → fallback isDefault (SC-US-PH2-NEW-03)
- **SC-PH2-PR-T08** : Créer tests unitaires pour controllers `profiles`, `pages`, `cards` (SC-US-PH2-NEW-04/05/06)
- **SC-PH2-ZG-T01** : Créer migration `zigbee_groups` (id, deviceId FK, groupId int, friendlyName) (SC-US-PH2-NEW-07)
- **SC-PH2-ZG-T02** : Créer migration `zigbee_scenes` (id, groupId FK, sceneId int, name) (SC-US-PH2-NEW-07)
- **SC-PH2-ZG-T03** : Implémenter modèles ORM `ZigbeeGroup`, `ZigbeeScene` (SC-US-PH2-NEW-07)
- **SC-PH2-ZG-T04** : Créer endpoints CRUD `GET/POST/PUT/DELETE /api/zigbee-groups` (SC-US-PH2-NEW-07)
- **SC-PH2-ZG-T05** : Créer endpoints CRUD `GET/POST/PUT/DELETE /api/zigbee-groups/:id/scenes` (SC-US-PH2-NEW-07)
- **SC-PH2-ZG-T06** : Implémenter appel MQTT bridge `s1c/zigbee2mqtt/bridge/request/scene/...` pour rappel scène firmware (SC-US-PH2-NEW-07)
- **SC-PH2-ZG-T07** : Créer tests unitaires pour controllers `zigbee-groups` et `zigbee-groups/:id/scenes` (SC-US-PH2-NEW-07)
- **SC-PH2-T21** : Mettre à jour seeders existants pour compatibilité nouveau schéma `devices` (capabilities jsonb, protocol enum, sans room_id) (SC-US-PH2-05)
- **SC-PH2-SEED-T01** : Créer seeders `devices` réalistes avec `capabilities.controls` complets par protocole (zigbee, mqtt, logo) (SC-US-PH2-05)
- **SC-PH2-SEED-T02** : Créer seeders `profiles/pages/cards` (1 profil default, 2 pages, 5 cartes variées) (SC-US-PH2-05)
- **SC-PH2-SEED-T03** : Créer seeders `telemetry_policies` par défaut (SC-US-PH2-NEW-03)

### E - Impact frontend et adaptations nécessaires

**User Stories :**

- **SC-US-PH2-06** : En tant que développeur, je veux identifier les impacts frontend des modifications de schéma
- **SC-US-PH2-07** : En tant que développeur, je veux adapter le code frontend existant aux nouveaux modèles

**Tâches :**

- **SC-PH2-T28** : Analyser code frontend existant (composants, services, stores) (SC-US-PH2-06)
- **SC-PH2-T29** : Identifier composants impactés par modifications schéma (SC-US-PH2-06)
- **SC-PH2-T30** : Documenter liste des adaptations nécessaires frontend (SC-US-PH2-06)
- **SC-PH2-T31** : Adapter interfaces TypeScript/types frontend pour nouveaux modèles (SC-US-PH2-07)
- **SC-PH2-T32** : Adapter services API frontend pour nouvelles routes (SC-US-PH2-07)
- **SC-PH2-T33** : Adapter stores/state management pour nouveaux modèles (SC-US-PH2-07)
- **SC-PH2-T34** : Adapter composants existants utilisant ancien schéma (SC-US-PH2-07)
- **SC-PH2-T36** : Tester frontend avec nouveau schéma et seeders (SC-US-PH2-07)

### F - API REST Gestion appareils

**User Stories :**

- **SC-US-PH2-08** : En tant qu'utilisateur, je veux voir tous mes appareils disponibles avec leur état actuel en temps réel — l'état temps réel vient de `device_states.state` (table dédiée), pas d'un champ sur `devices`
- **SC-US-PH2-09** : En tant qu'utilisateur, je veux ajouter un nouvel appareil Zigbee en activant le mode appairage
- **SC-US-PH2-10** : En tant qu'utilisateur, je veux renommer mes appareils avec des noms compréhensibles
- **SC-US-PH2-11** : En tant qu'utilisateur, je veux organiser mes tuiles dans des pages au sein de mon profil
- **SC-US-PH2-12** : En tant qu'utilisateur, je veux contrôler manuellement chaque appareil (allumer/éteindre, régler)
- **SC-US-PH2-13** : En tant qu'utilisateur, je veux supprimer un appareil et le retirer du réseau Zigbee

**Tâches :**

- **SC-PH2-T37** : Créer/adapter endpoint `GET /api/devices` avec filtres (`protocol`, `adminStatus`, `technicalStatus`) — filtre basé sur `device_states.technicalStatus` pour le statut online/offline (SC-US-PH2-08)
- **SC-PH2-T38** : Créer/adapter endpoint `GET /api/devices/:id` pour détails appareil (SC-US-PH2-08)
- **SC-PH2-T39** : Créer endpoint `POST /api/devices` pour enregistrement manuel (SC-US-PH2-09)
- **SC-PH2-T40** : Créer/adapter endpoint `PUT /api/devices/:id` pour modification (`friendlyName`, `adminStatus`, `capabilities`) — champ `room` supprimé (SC-US-PH2-10)
- **SC-PH2-T41** : Créer endpoint `DELETE /api/devices/:id` avec unpair Zigbee (SC-US-PH2-13)
- **SC-PH2-T42** : Créer endpoint `POST /api/devices/:id/command` pour commandes MQTT (SC-US-PH2-12)
- **SC-PH2-T43** : Créer endpoint `POST /api/devices/pairing/start` (activer permit_join via `s1c/zigbee2mqtt/bridge/request/permit_join`) (SC-US-PH2-09)
- **SC-PH2-T44** : Créer endpoint `POST /api/devices/pairing/stop` (désactiver permit_join) (SC-US-PH2-09)
- **SC-PH2-T45** : Créer endpoint `GET /api/devices/:id/state` — retourne `device_states` (table dédiée), pas un champ sur `devices` (SC-US-PH2-08)
- **SC-PH2-T48** : Implémenter validation des payloads avec `fastify-type-provider-zod` (Zod uniquement — pas Joi) pour tous les endpoints (SC-US-PH2-08 à SC-US-PH2-13)
- **SC-PH2-T49** : Créer tests unitaires pour controllers devices (SC-US-PH2-08 à SC-US-PH2-13)

### G - Services MQTT et synchronisation

**User Stories :**

- **SC-US-PH2-14** : En tant que système, je veux écouter les messages MQTT et mettre à jour les états des appareils en temps réel — flux complet : UPSERT `device_states` → évaluation `telemetry_policies` → PUBLISH `s1c/internal/telemetry/{deviceId}/{key}` → EMIT WebSocket
- **SC-US-PH2-15** : En tant que système, je veux détecter automatiquement les nouveaux appareils Zigbee
- **SC-US-PH2-16** : En tant que système, je veux synchroniser la base de données avec Zigbee2MQTT au démarrage

**Tâches :**

- **SC-PH2-T50** : Adapter/étendre service MQTT Listener existant pour `s1c/zigbee2mqtt/#` (SC-US-PH2-14)
- **SC-PH2-T51** : Mettre à jour le service MQTT Listener pour les topics `s1c/devices/` et `s1c/zigbee2mqtt/` selon le protocole du device — topics avec préfixe `s1c/` (SC-US-PH2-14)
- **SC-PH2-T52** : Implémenter le flux complet de mise à jour états : UPSERT `device_states` → évaluation `telemetry_policies` → PUBLISH `s1c/internal/telemetry/{deviceId}/{key}` → EMIT WebSocket (SC-US-PH2-14)
- **SC-PH2-T53** : Implémenter détection nouveaux appareils Zigbee : mapping `exposes` → `capabilities.controls` selon les règles documentées (numeric/binary/enum → widget déduit) (SC-US-PH2-15)
- **SC-PH2-T54** : Implémenter gestion `technicalStatus` sur `device_states` : `online` / `offline` / `unreachable` — job périodique sur `lastSeenAt` (timeout configurable) (SC-US-PH2-14)
- **SC-PH2-T55** : Créer service de synchronisation au démarrage (fetch `s1c/zigbee2mqtt/bridge/devices`) (SC-US-PH2-16)
- **SC-PH2-T56** : Implémenter détection appareils orphelins (en DB mais pas Z2M) (SC-US-PH2-16)
- **SC-PH2-T57** : Adapter/créer service WebSocket pour broadcast états temps réel au frontend (SC-US-PH2-14)
- **SC-PH2-T58** : Créer tests unitaires pour services MQTT (avec mock broker) (SC-US-PH2-14 à SC-US-PH2-16)

### H - Mise à jour firmware IoT

**User Stories :**

- **SC-US-PH2-17** : En tant qu'utilisateur, je veux être notifié quand une mise à jour firmware est disponible pour un appareil
- **SC-US-PH2-18** : En tant qu'utilisateur, je veux mettre à jour le firmware de mes appareils Zigbee facilement
- **SC-US-PH2-19** : En tant que système, je veux vérifier automatiquement les mises à jour disponibles chaque semaine

**Tâches :**

- **SC-PH2-T59** : Créer endpoint `GET /api/devices/:id/updates` (check OTA via Z2M) (SC-US-PH2-17)
- **SC-PH2-T60** : Créer endpoint `POST /api/devices/:id/update` (lancer OTA update) (SC-US-PH2-18)
- **SC-PH2-T61** : Créer endpoint `GET /api/devices/:id/update/status` (progression update) (SC-US-PH2-18)
- **SC-PH2-T62** : Implémenter service de vérification périodique (cron hebdomadaire) (SC-US-PH2-19)
- **SC-PH2-T63** : Implémenter gestion progression via MQTT Z2M (SC-US-PH2-18)
- **SC-PH2-T64** : Implémenter notification utilisateur (WebSocket) pour progression (SC-US-PH2-18)
- **SC-PH2-T65** : Logging historique updates — décision en suspens : jsonb dans `device_states` ou table dédiée à trancher avant implémentation (SC-US-PH2-19)
- **SC-PH2-T66** : Créer tests unitaires pour service updates (SC-US-PH2-17 à SC-US-PH2-19)

### I - Tâches frontend (accompagnent les features backend)

**User Stories :**

- **SC-US-PH2-08** : En tant qu'utilisateur, je veux voir tous mes appareils disponibles avec leur état actuel en temps réel
- **SC-US-PH2-NEW-04** : En tant qu'utilisateur, je veux créer et gérer des profils d'interface adaptés à mes appareils
- **SC-US-PH2-NEW-05** : En tant qu'utilisateur, je veux organiser mes tuiles dans des pages au sein de mon profil
- **SC-US-PH2-NEW-06** : En tant qu'utilisateur, je veux créer des tuiles configurables pointant vers un device, un groupe ou une scène Zigbee

**Tâches :**

- **SC-PH2-FE-T01** : Définir types TypeScript `Device`, `DeviceState`, `Capability`, `Control` conformes au schéma SC-PH2-T01 (SC-US-PH2-08)
- **SC-PH2-FE-T02** : Définir types TypeScript `Profile`, `Page`, `Card` (SC-US-PH2-NEW-04/05/06)
- **SC-PH2-FE-T03** : Définir types TypeScript `ZigbeeGroup`, `ZigbeeScene`, `TelemetryPolicy` (SC-US-PH2-NEW-07)
- **SC-PH2-FE-T04** : Créer service API `devices` (GET liste, GET détail, POST, PUT, DELETE, POST command) (SC-US-PH2-08)
- **SC-PH2-FE-T05** : Créer service API `profiles`, `pages`, `cards` (CRUD complet) (SC-US-PH2-NEW-04/05/06)
- **SC-PH2-FE-T06** : Créer service API `zigbee-groups` et `zigbee-groups/:id/scenes` (SC-US-PH2-NEW-07)
- **SC-PH2-FE-T07** : Créer service API `telemetry` (GET historique avec timeWindow) vers `packages/telemetry` (SC-US-PH2-NEW-02)
- **SC-PH2-FE-T08** : Créer store `devices` (liste, état courant par deviceId depuis `device_states`, filtre par protocol/adminStatus) (SC-US-PH2-08)
- **SC-PH2-FE-T09** : Créer store `profiles/pages/cards` (profil actif, pages du profil actif, cards par page) (SC-US-PH2-NEW-04/05/06)
- **SC-PH2-FE-T10** : Implémenter client WebSocket : abonnement aux events `device_states` — mise à jour du store `devices` à chaque message (SC-US-PH2-14)
- **SC-PH2-FE-T11** : Implémenter logique de détection de profil : User-Agent → deviceType → appel API → store profil actif (SC-US-PH2-NEW-04