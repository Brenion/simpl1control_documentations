## PHASE 7 : Tests, optimisation et documentation (Semaines 22-24 | 25-30h)

### A - Tests backend

**User Stories :**

- **SC-US-PH7-01** : En tant que développeur, je veux une couverture de tests backend ≥70%

**Tâches :**

- **SC-PH7-T01** : Créer tests unitaires services métier (automatisations, chauffage) (SC-US-PH7-01)
- **SC-PH7-T02** : Créer tests unitaires utilitaires (helpers, validators) (SC-US-PH7-01)
- **SC-PH7-T03** : Créer tests intégration tous endpoints API REST (SC-US-PH7-01)
- **SC-PH7-T04** : Créer tests authentification/autorisation (SC-US-PH7-01)
- **SC-PH7-T05** : Créer tests gestion erreurs (SC-US-PH7-01)
- **SC-PH7-T06** : Créer tests MQTT avec mock broker (SC-US-PH7-01)
- **SC-PH7-T07** : Créer tests latence communication MQTT (SC-US-PH7-01)
- **SC-PH7-T08** : Créer tests scénarios automatisations complexes (SC-US-PH7-01)
- **SC-PH7-T09** : Créer tests gestion conflits automatisations (SC-US-PH7-01)
- **SC-PH7-T10** : Créer tests performance (100+ rules) (SC-US-PH7-01)
- **SC-PH7-T11** : Créer tests simulation variations température (SC-US-PH7-01)
- **SC-PH7-T12** : Créer tests validation logique hystérésis (SC-US-PH7-01)
- **SC-PH7-T13** : Créer tests agrégation demandes multi-zones (SC-US-PH7-01)
- **SC-PH7-T14** : Générer rapport couverture ≥70% (SC-US-PH7-01)

### B - Tests frontend

**User Stories :**

- **SC-US-PH7-02** : En tant que développeur, je veux des tests frontend complets

**Tâches :**

- **SC-PH7-T15** : Créer tests composants critiques (DeviceCard, AutomationBuilder) (SC-US-PH7-02)
- **SC-PH7-T16** : Créer tests interactions utilisateur (clics, inputs) (SC-US-PH7-02)
- **SC-PH7-T17** : Créer tests E2E Playwright : ajouter appareil Zigbee (SC-US-PH7-02)
- **SC-PH7-T18** : Créer tests E2E : créer automatisation (SC-US-PH7-02)
- **SC-PH7-T19** : Créer tests E2E : activer scène (SC-US-PH7-02)
- **SC-PH7-T20** : Créer tests E2E : configurer zone chauffage (SC-US-PH7-02)
- **SC-PH7-T21** : Tester cross-browser (Chrome, Firefox, Safari) (SC-US-PH7-02)
- **SC-PH7-T22** : Tester mobile viewport (SC-US-PH7-02)

### C - Tests système

**User Stories :**

- **SC-US-PH7-03** : En tant qu'administrateur, je veux valider la stabilité et performance du système

**Tâches :**

- **SC-PH7-T23** : Tester charge 30+ devices simultanés (SC-US-PH7-03)
- **SC-PH7-T24** : Tester 50+ automatisations actives (SC-US-PH7-03)
- **SC-PH7-T25** : Tester 10 zones chauffage (SC-US-PH7-03)
- **SC-PH7-T26** : Mesurer latence réponse, CPU, RAM sous charge (SC-US-PH7-03)
- **SC-PH7-T27** : Tester perte connexion MQTT (reconnexion auto) (SC-US-PH7-03)
- **SC-PH7-T28** : Tester crash Zigbee2MQTT (redémarrage, sync) (SC-US-PH7-03)
- **SC-PH7-T29** : Tester coupure électrique (persistance états) (SC-US-PH7-03)
- **SC-PH7-T30** : Tester devices offline (timeouts, fallbacks) (SC-US-PH7-03)
- **SC-PH7-T31** : Tester 5+ marques devices Zigbee (compatibilité) (SC-US-PH7-03)
- **SC-PH7-T32** : Tester OTA updates (SC-US-PH7-03)
- **SC-PH7-T33** : Tester stabilité réseau maillé Zigbee (mesh) (SC-US-PH7-03)
- **SC-PH7-T34** : Scanner vulnérabilités (npm audit, Snyk) (SC-US-PH7-03)
- **SC-PH7-T35** : Tester injection SQL (parameterized queries) (SC-US-PH7-03)
- **SC-PH7-T36** : Tester MQTT ACL (accès non autorisé) (SC-US-PH7-03)

### D - Optimisation Raspberry Pi 5

**User Stories :**

- **SC-US-PH7-04** : En tant qu'administrateur, je veux que le système reste performant sur Raspberry Pi 5

**Tâches :**

- **SC-PH7-T37** : Profiler avec htop, vmstat (identifier bottlenecks) (SC-US-PH7-04)
- **SC-PH7-T38** : Profiler Node.js avec --inspect (SC-US-PH7-04)
- **SC-PH7-T39** : Optimiser requêtes DB (ajouter indexes) (SC-US-PH7-04)
- **SC-PH7-T40** : Analyser requêtes lentes (EXPLAIN ANALYZE) (SC-US-PH7-04)
- **SC-PH7-T41** : Optimiser connection pooling PostgreSQL (SC-US-PH7-04)
- **SC-PH7-T42** : Configurer logs production (level: error) (SC-US-PH7-04)
- **SC-PH7-T43** : Activer compression responses API (gzip) (SC-US-PH7-04)
- **SC-PH7-T44** : Optimiser QoS MQTT (0 pour états, 1 pour commandes) (SC-US-PH7-04)
- **SC-PH7-T45** : Configurer retained messages MQTT judicieusement (SC-US-PH7-04)
- **SC-PH7-T46** : Implémenter debouncing publish MQTT (SC-US-PH7-04)
- **SC-PH7-T47** : Réduire polling interval Z2M si nécessaire (SC-US-PH7-04)
- **SC-PH7-T48** : Désactiver frontend Z2M (utiliser uniquement le vôtre) (SC-US-PH7-04)
- **SC-PH7-T49** : Configurer log level Z2M: warning (SC-US-PH7-04)
- **SC-PH7-T50** : Configurer logrotate (rotation logs) (SC-US-PH7-04)
- **SC-PH7-T51** : Optimiser swappiness (vm.swappiness=10) (SC-US-PH7-04)
- **SC-PH7-T52** : Configurer PM2 pour gestion process (SC-US-PH7-04)
- **SC-PH7-T53** : Configurer watchdog systemd (auto-restart) (SC-US-PH7-04)
- **SC-PH7-T54** : Créer script monitoring ressources (alerte >80%) (SC-US-PH7-04)
- **SC-PH7-T55** : Créer health check endpoint `/api/health` (SC-US-PH7-04)
- **SC-PH7-T56** : Valider CPU <60% en utilisation normale (SC-US-PH7-04)

### E - Documentation

**User Stories :**

- **SC-US-PH7-05** : En tant qu'utilisateur/développeur, je veux une documentation complète et claire

**Tâches :**

- **SC-PH7-T57** : Rédiger guide installation (prérequis, étapes, troubleshooting) (SC-US-PH7-05)
- **SC-PH7-T58** : Documenter flash firmware Sonoff ZBDongle-E (SC-US-PH7-05)
- **SC-PH7-T59** : Documenter installation Zigbee2MQTT (SC-US-PH7-05)
- **SC-PH7-T60** : Rédiger guide utilisateur (premiers pas, screenshots) (SC-US-PH7-05)
- **SC-PH7-T61** : Documenter ajout appareil (SC-US-PH7-05)
- **SC-PH7-T62** : Documenter création automatisation (exemples concrets) (SC-US-PH7-05)
- **SC-PH7-T63** : Documenter création scène (SC-US-PH7-05)
- **SC-PH7-T64** : Documenter configuration chauffage (zones, plannings) (SC-US-PH7-05)
- **SC-PH7-T65** : Documenter mise à jour appareil (SC-US-PH7-05)
- **SC-PH7-T66** : Créer FAQ (SC-US-PH7-05)
- **SC-PH7-T67** : Générer documentation API (Swagger/OpenAPI) (SC-US-PH7-05)
- **SC-PH7-T68** : Documenter tous endpoints avec exemples cURL (SC-US-PH7-05)
- **SC-PH7-T69** : Documenter schémas requêtes/réponses (SC-US-PH7-05)
- **SC-PH7-T70** : Documenter codes erreur (SC-US-PH7-05)
- **SC-PH7-T71** : Créer diagrammes architecture (C4 niveau 2-3) (SC-US-PH7-05)
- **SC-PH7-T72** : Créer schéma base de données (ER diagram) (SC-US-PH7-05)
- **SC-PH7-T73** : Documenter flow MQTT (topics, payloads) (SC-US-PH7-05)
- **SC-PH7-T74** : Créer guide contribution (conventions, PR process) (SC-US-PH7-05)
- **SC-PH7-T75** : Créer flowchart automatisation (SC-US-PH7-05)
- **SC-PH7-T76** : Créer flowchart régulation chauffage (SC-US-PH7-05)
- **SC-PH7-T77** : Créer schéma câblage Siemens Logo (SC-US-PH7-05)