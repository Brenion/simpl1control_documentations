## 1. Présentation personnelle

- 1.1 Parcours scolaire et professionnel
- 1.2 Compétences acquises
- 1.3 Rôle et missions au sein de l'entreprise

## 2. Présentation de l'entreprise

- 2.1 Historique et activités principales
- 2.2 Organisation et équipe
- 2.3 Lien entre l’entreprise et le projet de domotique

## 3. Introduction du projet

- 3.1 Contexte du projet
- 3.2 Problématique
- 3.3 Objectifs du projet
- 3.4 Méthodologie de travail

## 4. État de l'art

- 4.1 Protocoles de communication IoT : MQTT et Modbus
- 4.2 Vers un système domotique ouvert et déployable pour les espaces de travail

## 5. Cahier des charges

- 5.1 Besoins fonctionnels
- 5.2 Besoins techniques
- 5.3 Contraintes du projet
- 5.4 Architecture cible

## 6. Conception et Développement

- 6.1 Choix des technologies et approche technique
    - 6.1.1 Microcontrôleur ESP32 (prototype de test)
    - 6.1.2 Transition prévue vers Zigbee MQTT
        
- 6.2 Développement du serveur domotique
    - 6.2.1 Backend : serveur Fastify (Node.js)
    - 6.2.2 Frontend : interface React basique
        
- 6.3 Intégration des capteurs, actionneurs
    
- 6.4 Gestion de l'accès sécurisé par badge
    - 6.4.1 Lecture du badge RFID/NFC avec ESP32
    - 6.4.2 Vérification d'accès via backend
    - 6.4.3 Contrôle de l'ouverture de porte magnétique via Siemens LOGO!
        
- 6.5 Choix et abandon de certaines pistes de développement

## 7. Mise en œuvre

- 7.1 Déploiement du système prototype
- 7.2 Configuration des équipements et réseau
- 7.3 Tests et validation des flux de communication
- 7.4 Mise en place et tests du contrôle d'accès par badge

## 8. Sécurité du système

- 8.1 Sécurisation des communications (ex : MQTT avec TLS, authentification)
- 8.2 Gestion sécurisée des accès utilisateurs (badges, droits)
- 8.3 Sécurité du serveur backend (Fastify)
- 8.4 Risques potentiels et mesures d'atténuation

## 9. Analyse et Résultats

- 9.1 Analyse technique
- 9.2 Estimation des coûts de développement pour un projet à grande échelle
- 9.3 Retours d'expérience

## 10. Conclusion et perspectives

- 10.1 Bilan du projet de démonstration
- 10.2 Pistes d'amélioration
- 10.3 Perspectives pour un projet final à grande échelle

## Annexes

- A. Schémas techniques
- B. Extraits de code (Fastify, React, Arduino, Siemens LOGO!)
- C. Liste des équipements utilisés
- D. Glossaire des termes utilisés