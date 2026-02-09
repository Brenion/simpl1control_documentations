## PHASE 2 : Modèle de données et API Devices (Semaines 6-8 | 30-35h)

### A - Analyse et adaptation schéma existant

**User Stories :**

- **SC-US-PH2-01** : En tant que développeur, je veux analyser le schéma de base de données existant et identifier les tables à modifier
- **SC-US-PH2-02** : En tant que développeur, je veux créer le schéma de base de données complet pour le système en tenant compte de l'existant

**Tâches :**

- **SC-PH2-T01** : Analyser schéma de base de données existant (tables, relations, contraintes) (SC-US-PH2-01)
- **SC-PH2-T02** : Identifier tables existantes à modifier/étendre (SC-US-PH2-01)
- **SC-PH2-T03** : Documenter structure actuelle (ER diagram de l'existant) (SC-US-PH2-01)
- **SC-PH2-T04** : Créer plan de migration (nouvelles tables + modifications tables existantes) (SC-US-PH2-01)

### B - Migrations et modèles de données

**Diagramme entités complet :**

```
┌─────────────────────┐
│       Device        │ [NOUVELLE TABLE ou MODIFICATION EXISTANTE]
├─────────────────────┤
│ id (PK)             │
│ friendly_name       │
│ ieee_address        │
│ type                │ (zigbee/mqtt/siemens/virtual)
│ category            │ (light/switch/sensor/thermostat/valve)
│ manufacturer        │
│ model               │
│ capabilities        │ (JSONB: {on_off, dimming, temp, etc.})
│ current_state       │ (JSONB)
│ last_seen           │
│ is_online           │
│ firmware_version    │
│ update_available    │
│ room_id (FK)        │
│ created_at          │
│ updated_at          │
└─────────────────────┘
        │ 1
        ├─────────────────┐
        │ N               │ N
┌─────────────────────┐ ┌──────────────────────┐
│     Automation      │ │     HeatingZone      │ [NOUVELLES TABLES]
├─────────────────────┤ ├──────────────────────┤
│ id (PK)             │ │ id (PK)              │
│ name                │ │ name                 │
│ description         │ │ room_id (FK)         │
│ enabled             │ │ target_temp          │
│ trigger_type        │ │ current_temp         │
│ trigger_device_id   │ │ valve_device_id (FK) │
│ trigger_conditions  │ │ temp_sensor_id (FK)  │
│ actions             │ │ schedule_id (FK)     │
│ conditions          │ │ mode                 │ (auto/manual/off)
│ priority            │ │ is_active            │
│ created_at          │ │ min_temp             │
│ last_triggered      │ │ max_temp             │
└─────────────────────┘ └──────────────────────┘

┌─────────────────────┐
│       Scene         │ [NOUVELLE TABLE]
├─────────────────────┤
│ id (PK)             │
│ name                │
│ icon                │
│ device_states       │ (JSONB: [{device_id, state}])
│ created_at          │
│ updated_at          │
└─────────────────────┘

┌─────────────────────┐
│        Room         │ [NOUVELLE TABLE ou MODIFICATION EXISTANTE]
├─────────────────────┤
│ id (PK)             │
│ name                │
│ icon                │
│ floor               │
│ order               │
└─────────────────────┘

┌──────────────────────┐
│   HeatingSchedule    │ [NOUVELLE TABLE]
├──────────────────────┤
│ id (PK)              │
│ name                 │
│ zone_id (FK)         │
│ periods              │ (JSONB: [{day, start, end, temp}])
│ is_active            │
│ created_at           │
└──────────────────────┘

┌──────────────────────┐
│   AutomationLog      │ [NOUVELLE TABLE]
├──────────────────────┤
│ id (PK)              │
│ automation_id (FK)   │
│ triggered_at         │
│ execution_time_ms    │
│ success              │
│ error_message        │
└──────────────────────┘

┌──────────────────────┐
│   DeviceUpdate       │ [NOUVELLE TABLE]
├──────────────────────┤
│ id (PK)              │
│ device_id (FK)       │
│ from_version         │
│ to_version           │
│ started_at           │
│ completed_at         │
│ status               │ (pending/in_progress/success/failed)
│ error_message        │
└──────────────────────┘
```

**User Stories :**

- **SC-US-PH2-03** : En tant que développeur, je veux créer/modifier les tables nécessaires avec migrations
- **SC-US-PH2-04** : En tant que développeur, je veux implémenter les modèles ORM correspondants
- **SC-US-PH2-05** : En tant que développeur, je veux mettre à jour les seeders existants et créer de nouveaux seeders réalistes

**Tâches :**

- **SC-PH2-T05** : Créer migration pour table `rooms` (nouvelle ou modification existante) (SC-US-PH2-03)
- **SC-PH2-T06** : Créer migration pour table `devices` (nouvelle ou modification existante) (SC-US-PH2-03)
- **SC-PH2-T07** : Créer migration pour table `heating_zones` (SC-US-PH2-03)
- **SC-PH2-T08** : Créer migration pour table `heating_schedules` (SC-US-PH2-03)
- **SC-PH2-T09** : Créer migration pour table `automations` (SC-US-PH2-03)
- **SC-PH2-T10** : Créer migration pour table `automation_logs` (SC-US-PH2-03)
- **SC-PH2-T11** : Créer migration pour table `scenes` (SC-US-PH2-03)
- **SC-PH2-T12** : Créer migration pour table `device_updates` (SC-US-PH2-03)
- **SC-PH2-T13** : Implémenter/modifier modèle ORM `Room` avec relations (SC-US-PH2-04)
- **SC-PH2-T14** : Implémenter/modifier modèle ORM `Device` avec relations (SC-US-PH2-04)
- **SC-PH2-T15** : Implémenter modèle ORM `HeatingZone` avec relations (SC-US-PH2-04)
- **SC-PH2-T16** : Implémenter modèle ORM `HeatingSchedule` (SC-US-PH2-04)
- **SC-PH2-T17** : Implémenter modèle ORM `Automation` (SC-US-PH2-04)
- **SC-PH2-T18** : Implémenter modèle ORM `AutomationLog` (SC-US-PH2-04)
- **SC-PH2-T19** : Implémenter modèle ORM `Scene` (SC-US-PH2-04)
- **SC-PH2-T20** : Implémenter modèle ORM `DeviceUpdate` (SC-US-PH2-04)
- **SC-PH2-T21** : Mettre à jour seeders existants pour compatibilité nouveau schéma (SC-US-PH2-05)
- **SC-PH2-T22** : Créer seeders réalistes pour `rooms` (5-8 pièces type maison) (SC-US-PH2-05)
- **SC-PH2-T23** : Créer seeders réalistes pour `devices` (15-20 devices variés) (SC-US-PH2-05)
- **SC-PH2-T24** : Créer seeders réalistes pour `heating_zones` (3-5 zones) (SC-US-PH2-05)
- **SC-PH2-T25** : Créer seeders réalistes pour `heating_schedules` (plannings jour/nuit) (SC-US-PH2-05)
- **SC-PH2-T26** : Créer seeders réalistes pour `automations` (5-7 automatisations type) (SC-US-PH2-05)
- **SC-PH2-T27** : Créer seeders réalistes pour `scenes` (3-5 scènes type) (SC-US-PH2-05)

### C - Impact frontend et adaptations nécessaires

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
- **SC-PH2-T35** : Créer composants frontend temporaires/basiques pour nouvelles entités (devices, rooms) (SC-US-PH2-07)
- **SC-PH2-T36** : Tester frontend avec nouveau schéma et seeders (SC-US-PH2-07)

### D - API REST Gestion appareils

**User Stories :**

- **SC-US-PH2-08** : En tant qu'utilisateur, je veux voir tous mes appareils disponibles avec leur état actuel en temps réel
- **SC-US-PH2-09** : En tant qu'utilisateur, je veux ajouter un nouvel appareil Zigbee en activant le mode appairage
- **SC-US-PH2-10** : En tant qu'utilisateur, je veux renommer mes appareils avec des noms compréhensibles
- **SC-US-PH2-11** : En tant qu'utilisateur, je veux organiser mes appareils par pièce
- **SC-US-PH2-12** : En tant qu'utilisateur, je veux contrôler manuellement chaque appareil (allumer/éteindre, régler)
- **SC-US-PH2-13** : En tant qu'utilisateur, je veux supprimer un appareil et le retirer du réseau Zigbee

**Tâches :**

- **SC-PH2-T37** : Créer/adapter endpoint `GET /api/devices` avec filtres (room, type, category, online) (SC-US-PH2-08)
- **SC-PH2-T38** : Créer/adapter endpoint `GET /api/devices/:id` pour détails appareil (SC-US-PH2-08)
- **SC-PH2-T39** : Créer endpoint `POST /api/devices` pour enregistrement manuel (SC-US-PH2-09)
- **SC-PH2-T40** : Créer/adapter endpoint `PUT /api/devices/:id` pour modification (rename, room) (SC-US-PH2-10, SC-US-PH2-11)
- **SC-PH2-T41** : Créer endpoint `DELETE /api/devices/:id` avec unpair Zigbee (SC-US-PH2-13)
- **SC-PH2-T42** : Créer endpoint `POST /api/devices/:id/command` pour commandes MQTT (SC-US-PH2-12)
- **SC-PH2-T43** : Créer endpoint `POST /api/devices/pairing/start` (activer permit_join Z2M) (SC-US-PH2-09)
- **SC-PH2-T44** : Créer endpoint `POST /api/devices/pairing/stop` (désactiver permit_join Z2M) (SC-US-PH2-09)
- **SC-PH2-T45** : Créer endpoint `GET /api/devices/:id/state` pour état actuel (SC-US-PH2-08)
- **SC-PH2-T46** : Créer/adapter endpoint `GET /api/rooms` (liste pièces) (SC-US-PH2-11)
- **SC-PH2-T47** : Créer/adapter endpoint `POST /api/rooms` (créer pièce) (SC-US-PH2-11)
- **SC-PH2-T48** : Implémenter validation des payloads (Zod ou Joi) pour tous endpoints (SC-US-PH2-08 à SC-US-PH2-13)
- **SC-PH2-T49** : Créer tests unitaires pour controllers devices (SC-US-PH2-08 à SC-US-PH2-13)

### E - Services MQTT et synchronisation

**User Stories :**

- **SC-US-PH2-14** : En tant que système, je veux écouter les messages MQTT et mettre à jour les états des appareils en temps réel
- **SC-US-PH2-15** : En tant que système, je veux détecter automatiquement les nouveaux appareils Zigbee
- **SC-US-PH2-16** : En tant que système, je veux synchroniser la base de données avec Zigbee2MQTT au démarrage

**Tâches :**

- **SC-PH2-T50** : Adapter/étendre service MQTT Listener existant pour `zigbee2mqtt/#` (SC-US-PH2-14)
- **SC-PH2-T51** : Vérifier service MQTT Listener existant pour `heating/#` (SC-US-PH2-14)
- **SC-PH2-T52** : Implémenter mise à jour états devices en DB depuis messages MQTT (SC-US-PH2-14)
- **SC-PH2-T53** : Implémenter détection nouveaux appareils Zigbee (auto-découverte) (SC-US-PH2-15)
- **SC-PH2-T54** : Implémenter gestion online/offline avec timeout (5 min) (SC-US-PH2-14)
- **SC-PH2-T55** : Créer service de synchronisation au démarrage (fetch `zigbee2mqtt/bridge/devices`) (SC-US-PH2-16)
- **SC-PH2-T56** : Implémenter détection appareils orphelins (en DB mais pas Z2M) (SC-US-PH2-16)
- **SC-PH2-T57** : Adapter/créer service WebSocket pour broadcast états temps réel au frontend (SC-US-PH2-14)
- **SC-PH2-T58** : Créer tests unitaires pour services MQTT (avec mock broker) (SC-US-PH2-14 à SC-US-PH2-16)

### F - Mise à jour firmware IoT

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
- **SC-PH2-T65** : Implémenter logging historique updates (table `device_updates`) (SC-US-PH2-19)
- **SC-PH2-T66** : Créer tests unitaires pour service updates (SC-US-PH2-17 à SC-US-PH2-19)