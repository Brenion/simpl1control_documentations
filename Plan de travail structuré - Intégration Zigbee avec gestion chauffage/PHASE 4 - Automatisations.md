## PHASE 4 : Automatisations (Semaines 13-15 | 25-30h)

### A - API REST Automatisations

**User Stories :**

- **SC-US-PH4-01** : En tant qu'utilisateur, je veux créer une automatisation "quand interrupteur pressé, allumer lampe"
- **SC-US-PH4-02** : En tant qu'utilisateur, je veux définir plusieurs actions pour un même déclencheur
- **SC-US-PH4-03** : En tant qu'utilisateur, je veux ajouter des conditions (heure, état, température)
- **SC-US-PH4-04** : En tant qu'utilisateur, je veux activer/désactiver temporairement mes automatisations
- **SC-US-PH4-05** : En tant qu'utilisateur, je veux voir l'historique d'exécution avec détails

**Tâches :**

- **SC-PH4-T01** : Créer endpoint `GET /api/automations` avec filtres (SC-US-PH4-04)
- **SC-PH4-T02** : Créer endpoint `POST /api/automations` (créer automatisation) (SC-US-PH4-01)
- **SC-PH4-T03** : Créer endpoint `PUT /api/automations/:id` (modifier) (SC-US-PH4-02, SC-US-PH4-03)
- **SC-PH4-T04** : Créer endpoint `DELETE /api/automations/:id` (supprimer) (SC-US-PH4-04)
- **SC-PH4-T05** : Créer endpoint `PATCH /api/automations/:id/toggle` (enable/disable) (SC-US-PH4-04)
- **SC-PH4-T06** : Créer endpoint `GET /api/automations/:id/logs` avec pagination (SC-US-PH4-05)
- **SC-PH4-T07** : Créer endpoint `POST /api/automations/:id/test` (simulation) (SC-US-PH4-01)
- **SC-PH4-T08** : Implémenter validation payloads automatisations (SC-US-PH4-01 à SC-US-PH4-03)
- **SC-PH4-T09** : Créer tests unitaires pour controllers automatisations (SC-US-PH4-01 à SC-US-PH4-05)

### B - Moteur d'automatisation

**User Stories :**

- **SC-US-PH4-06** : En tant que système, je veux exécuter automatiquement les automatisations selon leurs déclencheurs
- **SC-US-PH4-07** : En tant que système, je veux supporter différents types de déclencheurs et conditions
- **SC-US-PH4-08** : En tant que système, je veux logger toutes les exécutions d'automatisations

**Tâches :**

- **SC-PH4-T10** : Créer service moteur automatisation (SC-US-PH4-06)
- **SC-PH4-T11** : Implémenter chargement automatisations actives en mémoire (SC-US-PH4-06)
- **SC-PH4-T12** : Implémenter écoute événements MQTT (tous devices) (SC-US-PH4-06)
- **SC-PH4-T13** : Implémenter identification automatisations correspondantes (SC-US-PH4-06)
- **SC-PH4-T14** : Implémenter évaluation conditions (heure, jour, état, température) (SC-US-PH4-07)
- **SC-PH4-T15** : Implémenter exécution actions (publish MQTT, activer scène, chauffage) (SC-US-PH4-06)
- **SC-PH4-T16** : Implémenter logging exécution avec métriques (SC-US-PH4-08)
- **SC-PH4-T17** : Supporter déclencheurs événement appareil (changement état, bouton) (SC-US-PH4-07)
- **SC-PH4-T18** : Supporter déclencheurs changement température/luminosité (SC-US-PH4-07)
- **SC-PH4-T19** : Supporter déclencheurs intervalle temps (cron-like) (SC-US-PH4-07)
- **SC-PH4-T20** : Supporter déclencheurs événements système (lever/coucher soleil) (SC-US-PH4-07)
- **SC-PH4-T21** : Supporter conditions comparaison (==, <, >, !=) (SC-US-PH4-07)
- **SC-PH4-T22** : Supporter combinaisons logiques (AND, OR, NOT) (SC-US-PH4-07)
- **SC-PH4-T23** : Supporter actions avec délais (wait X seconds) (SC-US-PH4-06)
- **SC-PH4-T24** : Implémenter gestion priorités (conflits entre automatisations) (SC-US-PH4-06)
- **SC-PH4-T25** : Optimiser performances (debouncing, éviter boucles infinies) (SC-US-PH4-06)
- **SC-PH4-T26** : Créer tests unitaires moteur automatisation (SC-US-PH4-06 à SC-US-PH4-08)
- **SC-PH4-T27** : Tester charge CPU avec 20+ automatisations actives (SC-US-PH4-06)