## PHASE 0 : Audit et préparation (Semaines 1-2 | 15-20h)

### A - Audit du projet existant

**User Stories :**

- **SC-US-PH0-01** : En tant que développeur, je veux auditer la structure du code backend pour identifier les dettes techniques
- **SC-US-PH0-02** : En tant que développeur, je veux auditer le frontend pour vérifier la qualité du code
- **SC-US-PH0-03** : En tant que développeur, je veux documenter l'architecture MQTT actuelle (topics, payload, clients)
- **SC-US-PH0-04** : En tant que développeur, je veux lister les tests existants et identifier les manques
- **SC-US-PH0-05** : En tant que développeur, je veux analyser l'intégration actuelle du Siemens Logo et sa communication MQTT

**Tâches :**

- **SC-PH0-T01** : Cartographier les topics MQTT existants (chauffage Siemens Logo, autres appareils) (SC-US-PH0-03)
- **SC-PH0-T02** : Analyser le protocole de communication avec le Siemens Logo (topics, payloads, fréquence) (SC-US-PH0-05)
- **SC-PH0-T03** : Vérifier la convention de nommage (variables, fichiers, topics MQTT) (SC-US-PH0-01)
- **SC-PH0-T04** : Identifier les bibliothèques obsolètes backend et frontend (SC-US-PH0-01, SC-US-PH0-02)
- **SC-PH0-T05** : Lister les tests existants et analyser la couverture actuelle (SC-US-PH0-04)
- **SC-PH0-T06** : Documenter l'architecture actuelle avec diagrammes (C4 ou équivalent) (SC-US-PH0-03)
- **SC-PH0-T07** : Analyser la structure du code frontend existant (SC-US-PH0-02)
- **SC-PH0-T08** : Analyser la structure du code backend existant (SC-US-PH0-01)

### B - Remise à niveau technique

**User Stories :**

- **SC-US-PH0-06** : En tant que développeur, je veux mettre à jour les dépendances vulnérables
- **SC-US-PH0-07** : En tant que développeur, je veux refactoriser le code problématique identifié
- **SC-US-PH0-08** : En tant que développeur, je veux mettre en place les outils d'analyse de couverture de tests

**Tâches :**

- **SC-PH0-T09** : Mettre à jour Node.js, npm, et dépendances backend (SC-US-PH0-06)
- **SC-PH0-T10** : Mettre à jour les dépendances frontend (SC-US-PH0-06)
- **SC-PH0-T11** : Refactoriser les parties critiques identifiées lors de l'audit (SC-US-PH0-07)
- **SC-PH0-T12** : Configurer Vitest pour l'analyse de couverture de tests backend (SC-US-PH0-08)
- **SC-PH0-T13** : Configurer Vitest pour l'analyse de couverture de tests frontend (SC-US-PH0-08)
- **SC-PH0-T14** : Optimiser les requêtes PostgreSQL si nécessaire (SC-US-PH0-07)
- **SC-PH0-T15** : Nettoyer le code mort et harmoniser le nommage selon les conventions (SC-US-PH0-07)
- **SC-PH0-T16** : Générer un rapport initial de couverture de tests (baseline) (SC-US-PH0-08)
