## PHASE 3 : Gestion du chauffage (Semaines 9-12 | 35-40h)

### A - API REST Chauffage

**User Stories :**

- **SC-US-PH3-01** : En tant qu'utilisateur, je veux définir des zones de chauffage (pièces) avec des températures cibles différentes
- **SC-US-PH3-02** : En tant qu'utilisateur, je veux créer des plannings horaires de chauffage par zone
- **SC-US-PH3-03** : En tant qu'utilisateur, je veux pouvoir forcer manuellement une température dans une zone (override)
- **SC-US-PH3-04** : En tant qu'utilisateur, je veux associer des vannes thermostatiques Zigbee à chaque zone
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
- **SC-PH3-T11** : Implémenter validation des payloads chauffage (SC-US-PH3-01 à SC-US-PH3-05)
- **SC-PH3-T12** : Créer tests unitaires pour controllers chauffage (SC-US-PH3-01 à SC-US-PH3-05)

### B - Logique de régulation

**User Stories :**

- **SC-US-PH3-06** : En tant que système, je veux réguler le chauffage automatiquement selon les demandes des zones
- **SC-US-PH3-07** : En tant qu'administrateur, je veux que le système soit évolutif (changement source chauffage)
- **SC-US-PH3-08** : En tant que système, je veux commander les vannes thermostatiques Zigbee par zone
- **SC-US-PH3-09** : En tant que système, je veux communiquer avec le Siemens Logo via MQTT

**Tâches :**

- **SC-PH3-T13** : Créer service de régulation (boucle 30s) (SC-US-PH3-06)
- **SC-PH3-T14** : Implémenter logique calcul demande par zone (hystérésis ±0.5°C) (SC-US-PH3-06)
- **SC-PH3-T15** : Implémenter agrégation demandes multi-zones (SC-US-PH3-06)
- **SC-PH3-T16** : Implémenter commande vannes TRV Zigbee via MQTT (SC-US-PH3-08)
- **SC-PH3-T17** : Implémenter communication MQTT avec Siemens Logo (SC-US-PH3-09)
- **SC-PH3-T18** : Créer interface abstraite `HeatingSource` (SC-US-PH3-07)
- **SC-PH3-T19** : Implémenter `SiemensLogoHeatingSource` (SC-US-PH3-07, SC-US-PH3-09)
- **SC-PH3-T20** : Préparer structure pour `HeatPumpHeatingSource` (documentation) (SC-US-PH3-07)
- **SC-PH3-T21** : Implémenter watchdog Logo (alerte si offline > 2min) (SC-US-PH3-09)
- **SC-PH3-T22** : Implémenter logging décisions régulation (SC-US-PH3-06)
- **SC-PH3-T23** : Gérer priorités zones (mode antigel, etc.) (SC-US-PH3-06)
- **SC-PH3-T24** : Créer tests unitaires pour service régulation (SC-US-PH3-06 à SC-US-PH3-09)
- **SC-PH3-T25** : Créer tests intégration communication Logo (SC-US-PH3-09)

### C - Préparation installation physique

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
- **SC-PH3-T34** : Tester connexion MQTT Logo ↔ Raspberry Pi (WiFi/Ethernet) (SC-US-PH3-09)
