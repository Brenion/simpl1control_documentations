## PHASE 8 : Sécurisation et déploiement production (Semaines 25-27 | 25-30h)

### A - Sécurisation HTTPS/WSS

**User Stories :**

- **SC-US-PH8-01** : En tant qu'administrateur, je veux sécuriser l'accès via HTTPS/WSS
- **SC-US-PH8-02** : En tant qu'administrateur, je veux sécuriser le broker MQTT avec TLS

**Tâches :**

- **SC-PH8-T01** : Générer certificats SSL (Let's Encrypt ou auto-signés intranet) (SC-US-PH8-01)
- **SC-PH8-T02** : Configurer Nginx reverse proxy pour HTTPS (SC-US-PH8-01)
- **SC-PH8-T03** : Configurer Nginx pour WSS (SC-US-PH8-01)
- **SC-PH8-T04** : Migrer toutes connexions WebSocket vers WSS (SC-US-PH8-01)
- **SC-PH8-T05** : Configurer renouvellement automatique certificats (SC-US-PH8-01)
- **SC-PH8-T06** : Configurer Mosquitto avec TLS (port 8883) (SC-US-PH8-02)
- **SC-PH8-T07** : Définir ACL Mosquitto production strictes (SC-US-PH8-02)
- **SC-PH8-T08** : Créer users MQTT avec mots de passe forts (SC-US-PH8-02)
- **SC-PH8-T09** : Tester performance SSL/TLS sur Raspberry Pi 5 (SC-US-PH8-01)

### B - Sécurisation application

**User Stories :**

- **SC-US-PH8-03** : En tant qu'administrateur, je veux sécuriser l'application backend
- **SC-US-PH8-04** : En tant qu'administrateur, je veux que le système vérifie les mises à jour

**Tâches :**

- **SC-PH8-T10** : Implémenter authentification JWT ou session (SC-US-PH8-03)
- **SC-PH8-T11** : Implémenter rate limiting (express-rate-limit) (SC-US-PH8-03)
- **SC-PH8-T12** : Implémenter validation inputs stricte (Zod/Joi) (SC-US-PH8-03)
- **SC-PH8-T13** : Configurer headers sécurité (helmet.js) (SC-US-PH8-03)
- **SC-PH8-T14** : Créer service vérification mises à jour système (GitHub releases) (SC-US-PH8-04)
- **SC-PH8-T15** : Implémenter signature paquets (vérification hash) (SC-US-PH8-04)
- **SC-PH8-T16** : Implémenter rollback automatique si échec update (SC-US-PH8-04)
- **SC-PH8-T17** : Configurer firewall ufw (fermer tous ports sauf 443) (SC-US-PH8-03)
- **SC-PH8-T18** : Configurer fail2ban (monitoring tentatives intrusion) (SC-US-PH8-03)
- **SC-PH8-T19** : Préparer configuration VPN WireGuard (documentation future) (SC-US-PH8-03)

### C - Déploiement production

**User Stories :**

- **SC-US-PH8-05** : En tant qu'administrateur, je veux déployer le système en production sur Raspberry Pi 5

**Tâches :**

- **SC-PH8-T20** : Backup complet configuration développement (SC-US-PH8-05)
- **SC-PH8-T21** : Clean install Raspberry Pi OS stable (SC-US-PH8-05)
- **SC-PH8-T22** : Hardening sécurité OS (SSH keys, fail2ban, ufw) (SC-US-PH8-05)
- **SC-PH8-T23** : Installer stack complète selon guide installation (SC-US-PH8-05)
- **SC-PH8-T24** : Configurer variables environnement production (SC-US-PH8-05)
- **SC-PH8-T25** : Build production frontend optimisé (SC-US-PH8-05)
- **SC-PH8-T26** : Configurer systemd services (backend, Z2M, Mosquitto) (SC-US-PH8-05)
- **SC-PH8-T27** : Configurer PM2 pour backend Node.js (SC-US-PH8-05)
- **SC-PH8-T28** : Tests smoke post-déploiement (endpoints, WebSocket, MQTT) (SC-US-PH8-05)
- **SC-PH8-T29** : Configurer backups automatiques DB (cron dump PostgreSQL) (SC-US-PH8-05)
- **SC-PH8-T30** : Configurer monitoring (script alerte ressources) (SC-US-PH8-05)

### D - Installation physique chauffage (Septembre)

**User Stories :**

- **SC-US-PH8-06** : En tant qu'administrateur, je veux installer physiquement le Siemens Logo
- **SC-US-PH8-07** : En tant qu'administrateur, je veux valider le bon fonctionnement complet du système

**Tâches :**

- **SC-PH8-T31** : Préparer matériel (Logo, relais, câbles, outils) (SC-US-PH8-06)
- **SC-PH8-T32** : Couper disjoncteur général (sécurité) (SC-US-PH8-06)
- **SC-PH8-T33** : Installer Logo dans tableau/boîtier dédié (SC-US-PH8-06)
- **SC-PH8-T34** : Câbler alimentation Logo (SC-US-PH8-06)
- **SC-PH8-T35** : Câbler relais commande chaudière selon schéma (SC-US-PH8-06)
- **SC-PH8-T36** : Connecter feedback état chaudière (optionnel) (SC-US-PH8-06)
- **SC-PH8-T37** : Vérifier polarités, serrage, isolation (SC-US-PH8-06)
- **SC-PH8-T38** : Tester continuité au multimètre (SC-US-PH8-06)
- **SC-PH8-T39** : Programmer logique Logo (Logo Soft Comfort) (SC-US-PH8-06)
- **SC-PH8-T40** : Tester E/S Logo (inputs, outputs) (SC-US-PH8-06)
- **SC-PH8-T41** : Configurer communication Ethernet/WiFi Logo (SC-US-PH8-06)
- **SC-PH8-T42** : Configurer client MQTT dans Logo (SC-US-PH8-06)
- **SC-PH8-T43** : Vérifier connexion Logo ↔ Mosquitto (SC-US-PH8-06)
- **SC-PH8-T44** : Tester commandes MQTT → Logo → chaudière (SC-US-PH8-06)
- **SC-PH8-T45** : Valider retours états (feedback) (SC-US-PH8-06)
- **SC-PH8-T46** : Calibrer vannes thermostatiques Zigbee (SC-US-PH8-06)
- **SC-PH8-T47** : Test fonctionnel demande zone → Logo → chaudière ON (SC-US-PH8-07)
- **SC-PH8-T48** : Test arrêt demande → chaudière OFF (SC-US-PH8-07)
- **SC-PH8-T49** : Tester sécurités (coupure MQTT, Logo offline) (SC-US-PH8-07)
- **SC-PH8-T50** : Tester avec plannings horaires réels (SC-US-PH8-07)
- **SC-PH8-T51** : Mesurer latence réaction bout-en-bout (SC-US-PH8-07)

### E - Configuration finale et validation

**User Stories :**

- **SC-US-PH8-08** : En tant qu'utilisateur, je veux que le système soit prêt pour usage quotidien

**Tâches :**

- **SC-PH8-T52** : Apairer tous devices Zigbee finaux (SC-US-PH8-08)
- **SC-PH8-T53** : Renommer devices selon convention [pièce]_[type]_[n] (SC-US-PH8-08)
- **SC-PH8-T54** : Assigner devices aux pièces (SC-US-PH8-08)
- **SC-PH8-T55** : Vérifier qualité signal Zigbee (LQI, RSSI) (SC-US-PH8-08)
- **SC-PH8-T56** : Ajuster emplacements si signal faible (SC-US-PH8-08)
- **SC-PH8-T57** : Tester contrôle depuis interface pour chaque device (SC-US-PH8-08)
- **SC-PH8-T58** : Créer toutes pièces/zones chauffage réelles (SC-US-PH8-08)
- **SC-PH8-T59** : Configurer plannings chauffage quotidiens (SC-US-PH8-08)
- **SC-PH8-T60** : Créer automatisations maison (besoins réels) (SC-US-PH8-08)
- **SC-PH8-T61** : Créer scènes utiles (départ, retour, nuit, film) (SC-US-PH8-08)
- **SC-PH8-T62** : Tests utilisateur final (famille/cohabitants) (SC-US-PH8-08)
- **SC-PH8-T63** : Période observation 7 jours (noter bugs, améliorations) (SC-US-PH8-08)
- **SC-PH8-T64** : Ajustements paramètres chauffage (températures, plannings) (SC-US-PH8-08)
- **SC-PH8-T65** : Valider stabilité (uptime, crashs) (SC-US-PH8-08)
- **SC-PH8-T66** : Documenter anomalies et résolutions (SC-US-PH8-08)
- **SC-PH8-T67** : Formation utilisateurs (démonstration interface) (SC-US-PH8-08)
- **SC-PH8-T68** : Créer guide urgence (que faire si...) (SC-US-PH8-08)
- **SC-PH8-T69** : Backup final configuration production (SC-US-PH8-08)