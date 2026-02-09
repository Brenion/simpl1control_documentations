## PHASE 1 : Infrastructure Zigbee et environnement développement (Semaines 3-5 | 15-20h)

### A - Configuration environnement développement

**User Stories :**

- **SC-US-PH1-01** : En tant que développeur, je veux un environnement de développement qui simule la production sans compromettre la sécurité
- **SC-US-PH1-02** : En tant que développeur, je veux pouvoir travailler en HTTP/WS en développement et basculer facilement en HTTPS/WSS

**Tâches :**

- **SC-PH1-T01** : Créer configuration environnement développement (variables .env.development) (SC-US-PH1-01)
- **SC-PH1-T02** : Vérifier configuration HTTP/WS existante pour développement local (SC-US-PH1-02)
- **SC-PH1-T03** : Préparer configuration HTTPS/WSS (désactivée en dev, documentation pour activation Phase 8) (SC-US-PH1-02)
- **SC-PH1-T04** : Vérifier configuration Mosquitto existante (ports 1883/8883, authentification) (SC-US-PH1-01)
- **SC-PH1-T05** : Documenter configuration MQTT actuelle (users, ACL, topics) (SC-US-PH1-01)
- **SC-PH1-T06** : Documenter procédure de basculement dev → production (SC-US-PH1-01)
- **SC-PH1-T07** : Créer script de vérification configuration environnement (SC-US-PH1-01)

### B - Infrastructure Zigbee

**User Stories :**

- **SC-US-PH1-03** : En tant qu'administrateur, je veux flasher le firmware Ember sur ma clé Sonoff ZBDongle-E pour une meilleure stabilité
- **SC-US-PH1-04** : En tant qu'administrateur, je veux installer Zigbee2MQTT et le connecter à mon broker MQTT existant
- **SC-US-PH1-05** : En tant que système, je veux que Zigbee2MQTT cohabite avec les topics MQTT du Siemens Logo

**Tâches :**

- **SC-PH1-T08** : Télécharger firmware Ember pour ZBDongle-E (SC-US-PH1-03)
- **SC-PH1-T09** : Flasher le firmware Ember sur la clé Sonoff (SC-US-PH1-03)
- **SC-PH1-T10** : Documenter la procédure de flash pour futures mises à jour (SC-US-PH1-03)
- **SC-PH1-T11** : Installer Zigbee2MQTT sur Raspberry Pi 5 (installation native) (SC-US-PH1-04)
- **SC-PH1-T12** : Configurer `configuration.yaml` Z2M (port série, connexion broker MQTT existant 1883/8883) (SC-US-PH1-04)
- **SC-PH1-T13** : Définir base topic `zigbee2mqtt/` et valider architecture topics globale avec MQTT existant (SC-US-PH1-05)
- **SC-PH1-T14** : Activer frontend Zigbee2MQTT (port 8080 local) (SC-US-PH1-04)
- **SC-PH1-T15** : Configurer démarrage automatique Zigbee2MQTT (systemd) (SC-US-PH1-04)
- **SC-PH1-T16** : Optimiser configuration Z2M pour Raspberry Pi 5 (logs, verbosité) (SC-US-PH1-04)
- **SC-PH1-T17** : Tester connexion Z2M au broker MQTT existant (publier/recevoir messages) (SC-US-PH1-04)
- **SC-PH1-T18** : Tester appairage d'un appareil Zigbee de test (SC-US-PH1-04)
- **SC-PH1-T19** : Vérifier cohabitation topics MQTT (Zigbee + Siemens + existants) sans conflits (SC-US-PH1-05)

**Architecture topics MQTT (validation avec existant) :**

```
mqtt://localhost:1883 (développement) | mqtts://localhost:8883 (TLS)
├── zigbee2mqtt/              [NOUVEAU]
│   ├── bridge/
│   │   ├── state
│   │   ├── devices
│   │   ├── config
│   │   └── request/permit_join
│   ├── [device_friendly_name]/
│   │   ├── get
│   │   ├── set
│   │   ├── availability
│   │   └── [endpoint]
├── heating/                  [EXISTANT/ÉTENDU]
│   ├── logo/
│   │   ├── status
│   │   ├── temperature/[zone_id]
│   │   ├── setpoint/[zone_id]
│   │   └── valve/[zone_id]
│   ├── zones/               [NOUVEAU]
│   │   └── [zone_id]/
│   │       ├── current_temp
│   │       ├── target_temp
│   │       ├── valve_position
│   │       └── demand
│   └── schedule/            [NOUVEAU]
│       ├── active
│       └── [zone_id]/override
├── [topics_existants]/       [EXISTANT - À DOCUMENTER]
└── system/                   [NOUVEAU]
    ├── updates
    └── health
```
