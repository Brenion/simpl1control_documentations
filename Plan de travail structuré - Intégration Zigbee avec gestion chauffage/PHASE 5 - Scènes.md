## PHASE 5 : Scènes (Semaines 16-17 | 15-20h)

### A - API REST Scènes

**User Stories :**

- **SC-US-PH5-01** : En tant qu'utilisateur, je veux créer une scène "Cinéma" (lumières tamisées, stores, chauffage)
- **SC-US-PH5-02** : En tant qu'utilisateur, je veux activer une scène d'un seul clic
- **SC-US-PH5-03** : En tant qu'utilisateur, je veux capturer l'état actuel pour créer une scène rapidement
- **SC-US-PH5-04** : En tant qu'utilisateur, je veux déclencher une scène via une automatisation

**Tâches :**

- **SC-PH5-T01** : Créer endpoint `GET /api/scenes` (liste scènes) (SC-US-PH5-01)
- **SC-PH5-T02** : Créer endpoint `POST /api/scenes` (créer manuelle ou capture) (SC-US-PH5-01, SC-US-PH5-03)
- **SC-PH5-T03** : Créer endpoint `PUT /api/scenes/:id` (modifier) (SC-US-PH5-01)
- **SC-PH5-T04** : Créer endpoint `DELETE /api/scenes/:id` (supprimer) (SC-US-PH5-01)
- **SC-PH5-T05** : Créer endpoint `POST /api/scenes/:id/activate` (activer scène) (SC-US-PH5-02)
- **SC-PH5-T06** : Créer endpoint `POST /api/scenes/capture` (snapshot états devices) (SC-US-PH5-03)
- **SC-PH5-T07** : Implémenter validation payloads scènes (SC-US-PH5-01, SC-US-PH5-03)
- **SC-PH5-T08** : Créer tests unitaires pour controllers scènes (SC-US-PH5-01 à SC-US-PH5-04)

### B - Service d'activation scènes

**User Stories :**

- **SC-US-PH5-05** : En tant que système, je veux activer une scène en appliquant tous les états définis
- **SC-US-PH5-06** : En tant que système, je veux gérer l'activation de scènes complexes (50+ devices)

**Tâches :**

- **SC-PH5-T09** : Créer service activation scène (SC-US-PH5-05)
- **SC-PH5-T10** : Implémenter récupération device_states depuis DB (SC-US-PH5-05)
- **SC-PH5-T11** : Implémenter publication commandes MQTT pour tous devices (SC-US-PH5-05)
- **SC-PH5-T12** : Inclure états zones chauffage dans scènes (SC-US-PH5-05)
- **SC-PH5-T13** : Gérer ordre exécution (séquentiel pour éviter surcharge) (SC-US-PH5-06)
- **SC-PH5-T14** : Implémenter timeout par device (3s max) (SC-US-PH5-05)
- **SC-PH5-T15** : Implémenter logging activation (succès/échec par device) (SC-US-PH5-05)
- **SC-PH5-T16** : Intégrer scènes comme actions dans automatisations (SC-US-PH5-04)
- **SC-PH5-T17** : Créer tests unitaires service activation scènes (SC-US-PH5-05, SC-US-PH5-06)
- **SC-PH5-T18** : Tester scènes complexes (50+ devices) (SC-US-PH5-06)