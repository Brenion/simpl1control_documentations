### Projet : Domotique et automatisation d'un bâtiment

#### Objectifs principaux

- Gérer les éléments du bâtiment (lumières, porte avec badge, chauffage).
- Collecter les données de chaque élément pour une supervision en temps réel.
- Proposer un dashboard pour la supervision et le contrôle.
- Gérer des utilisateurs avec des rôles différenciés permettant des niveaux d'interaction variés.

#### Matériel disponible

- **Automate programmable** : API Logo V8 (Siemens).
- **Microcontrôleurs** :
    - Raspberry Pi 4 (4 Go).
    - Arduino Uno.
    - Arduino Uno Wifi.
    - ESP avec capteur de température DHT22.
- **Connectivité Zigbee** :
    - Ampoules connectées.
    - Interrupteurs connectés.
    - Vannes connectées.
- **Accès contrôlé** :
    - Badges MIFARE (non écrivables pour le moment, évolution vers des badges sécurisés prévue).
- **Divers** :
    - LEDs.
    - Relais.

#### Technologies utilisées

- **Frontend** : React, Typescript, Node.js.
- **Backend** : Fastify, Node.js.
- **Communication** : MQTT.
- **API de supervision** : SoftControl.
- **Déploiement** : Docker (Frontend, Backend, Base de données).
- **Sécurisation** : VPN ZeroTier.

#### Exigences du cahier des charges

1. **Application de supervision** :
    
    - Intégration à l'API Logo V8 si possible.
    - Sinon, déploiement en environnement externe via Docker.
    
2. **Communication et collecte des données** :
    
    - Utilisation du protocole MQTT pour connecter les éléments et remonter les données.
    
3. **Gestion des utilisateurs** :
    
    - Mise en place de rôles différenciés (exemple : Administrateur, Utilisateur standard).
    - Possibilité de création, modification et suppression des comptes.
    
4. **Sécurité** :
    
    - Implémentation d'un VPN (ZeroTier) pour sécuriser les communications.
    - Préparation pour l'utilisation de badges MIFARE sécurisés.

5. **Supervision et contrôle** :
    
    - Création d'un dashboard centralisé permettant :
        - Visualisation des données en temps réel.
        - Contrôle des différents équipements (lumières, chauffage, etc.).
        - Configuration des règles d'automatisation.
    
6. **Scalabilité et maintenance** :
    
    - Modularité pour ajouter de nouveaux équipements ou capteurs.
    - Documentation claire pour l'installation et la maintenance.

### Contraintes techniques et mode de fonctionnement choisi

#### 1. **Commandes via les sorties de l'API Logo V8 :**

**a. Commande du chauffage via un output (contact relais) :**

- **Faisabilité :** Oui, il est possible d'utiliser une sortie relais du Logo V8 pour commander le chauffage. Le Logo V8 peut activer ou désactiver un relais en fonction des conditions programmées (température, horaires, etc.).
    
- **Scénario d'utilisation :**
    
    - Le Logo V8 reçoit une mesure de température (via une entrée analogique ou numérique connectée à un capteur).
    - Une règle logique ou un seuil configuré dans le programme interne active ou désactive le relais, qui commande une chaudière ou un chauffage électrique.
    - Nous passerons par un relais secondaire plus puissant afin de protéger le Logo en cas de problème.
    - Toutes les données seront enregistrées dans un historique, incluant la demande du badge et l'état du Logo.

**b. Commande de l'ouverture des portes via un output (relais) :**

- **Faisabilité :** Oui, mais cela dépend du mécanisme d'ouverture des portes :
    
    - Les badges sécurisés étant nécessaires, le Logo V8 seul n’ayant pas de fonctionnalité native pour lire ou authentifier des badges.
    - Nous le couplerons avec un lecteur de badges externe connecté à un microcontrôleur ESP32 qui enverra les données via Zigbee en MQTT à notre serveur (backend) pour gérer la logique d’authentification.
    - Si le badge est autorisé, une commande sera envoyée au Logo (toujours en MQTT) qui ouvrira un relais dédié.
    - En parallèle, le Logo enverra un message notificatif en MQTT contenant un code statut (porte ouverte, erreur d'ouverture, etc.).
    - Toutes les données seront enregistrées dans un historique, incluant la demande du badge et l'état du Logo.

#### 2. **Application de gestion/supervision :**

**Externaliser la supervision :** Nous allons opter pour l'externalisation de la supervision et aussi de la communication entre les éléments et le Logo.

**Schéma de communication :**

```
Element IoT ---------> Backend de communication ----------> Logo
		       MQTT                                 MQTT
```

- **Avantages :**
    
    - Flexibilité : Un serveur externe (Docker) permet de développer une interface utilisateur riche et personnalisée avec React et Fastify.
    - Gestion avancée : Possibilité de gérer des rôles utilisateurs, des graphiques dynamiques, des notifications, etc.
    - Scalabilité : Facilité d’ajouter des équipements ou des fonctionnalités à l’avenir.
        
- **Limites :**
    
    - Dépendance à un serveur externe pour fonctionner.
    - Nécessité de configurer des passerelles MQTT entre le Logo V8 et l’application de gestion.

### Définition de l'architecture logicielle

#### 1. **Frontend**

- **Technologies principales** : React, TypeScript.
- **Outils additionnels** :
    - **Vitest** pour les tests.
    - **MUI (Material-UI)** pour une interface utilisateur moderne et cohérente.
        
- **Structure** :
    
    - **Dashboard principal** :
        - Visualisation des données des capteurs (température, états des relais, etc.).
        - Contrôle des équipements connectés (lumières, chauffage, ouverture des portes).
            
    - **Gestion des utilisateurs** :
        - Création, modification, suppression des comptes utilisateurs.
        - Attribution des rôles (administrateur, utilisateur standard).
            
    - **Notifications et logs** :
        - Affichage des erreurs ou statuts (portes ouvertes, erreurs, etc.).
            
    - **Communication** :
        - HTTP pour la communication avec le backend.

#### 2. **Backend**

- **Technologies principales** : Fastify (Node.js).
    
- **Modules clés** :
    
    - **Gestion des données MQTT** :
        - Réception des messages des équipements via MQTT.
        - Transmission des commandes au Logo ou aux équipements Zigbee.
            
    - **API REST** :
        - Fournir des endpoints pour le frontend (authentification, données des capteurs, contrôle des équipements).
            
    - **Base de données** :
        - Historisation des données (demandes de badges, états des équipements).
        - Gestion des utilisateurs et des rôles.
            
    - **Notifications** :        
        - Envoi d’alertes au frontend en temps réel (WebSocket ou autre).

    - **Tests** :
        - Utilisation de **Vitest** pour les tests unitaires et d'intégration.

#### 3. **MQTT**

- **Broker MQTT** : Mosquitto.
- **Rôles** :
    - Centralisation de la communication entre tous les éléments (Logo V8, ESP32, backend, équipements Zigbee).
    - Gestion des topics :
        - `devices/<id>/status` : État des équipements (relais, capteurs).
        - `devices/<id>/command` : Commandes vers les équipements.
        - `notifications/<id>` : Notifications système.
            
- **Sécurisation** :
    - Authentification des clients MQTT.
    - Utilisation de TLS pour chiffrer les communications.
        

#### 4. **Collaboration et gestion du code**

- **Versionnement** :
    
    - Utilisation de **Git** pour la gestion du code source.
    - Structuration en branches :
        - Une branche principale (`main`) pour le code stable.
        - Branches de fonctionnalités (`feature/nom_fonctionnalité`) pour les développements spécifiques.
        - Branches de correction (`hotfix/nom_correction`) pour les correctifs urgents.
            
- **Automatisation** :
    
    - Intégration de **GitHub Actions** pour automatiser :
        - L'exécution des tests (frontend, backend) à chaque pull request.
        - Le déploiement des images Docker sur un registre privé après validation.
        - La vérification du linting et des bonnes pratiques de code.

### Planification des phases de développement

#### Mise en place de la communication entre équipements via MQTT

**Étape 1 : Configuration initiale du broker MQTT**

- Installer Mosquitto sur le serveur dédié.
- Configurer les topics de base pour les équipements (ex. `devices/<id>/status`, `devices/<id>/command`).
- Activer l’authentification des clients (fichier d’utilisateur/mot de passe).
- Configurer le chiffrement TLS pour sécuriser les communications.


**Étape 2 : Connexion des équipements**

- **Sous-étape 1** : Configurer le Logo V8 pour publier des messages sur les topics MQTT.
- **Sous-étape 2** : Connecter un ESP32 pour envoyer les données de capteurs (ex. température) vers le broker.
- **Sous-étape 3** : Tester la communication des équipements Zigbee via un coordinateur (ex. Zigbee2MQTT).
- **Sous-étape 4** : Valider les connexions en surveillant les publications via un client MQTT (ex. MQTT Explorer).


**Étape 3 : Implémentation des mécanismes de publication et souscription**

- Développer un script ou un service pour écouter les topics de statut et de commande.
- Créer un simulateur pour tester les équipements en l’absence de matériel réel.
- Vérifier la cohérence des données transmises sur les topics.

#### Mise en place du contrôle de version avec Git et GitHub

**Étape 1 : Initialisation des dépôts**

- Créer un dépôt GitHub pour chaque composant (Frontend, Backend, Documentation).
- Initialiser les dépôts locaux et les lier aux dépôts distants sur GitHub.
- Configurer les branches principales (`main`) pour chaque dépôt.


**Étape 2 : Mise en place des workflows Git**

- Définir une stratégie de branchement :
    
    - `main` : Branche stable pour le déploiement.
    - `develop` : Branche pour l’intégration des fonctionnalités.
    - Branches fonctionnelles : Une branche par fonctionnalité ou correction.
        
- Configurer des templates de pull request et des conventions de commit (ex. Conventional Commits).


**Étape 3 : Intégration des outils CI/CD**

- Ajouter des workflows GitHub Actions pour :
    - Tester automatiquement les modifications (Frontend et Backend).
    - Analyser la qualité du code (linting, tests).
    - Déployer les images Docker sur un registre (si applicable).

#### Développement du backend

**Étape 1 : Gestion des données MQTT**

- Implémenter la réception des messages des équipements via MQTT.
- Ajouter un module pour transmettre les commandes aux équipements Zigbee et au Logo.
- Tester la publication et la souscription aux topics définis.


**Étape 2 : Gestion des utilisateurs**

- Créer une base de données pour stocker les utilisateurs et leurs rôles.
- Implémenter une API REST pour l’inscription, la connexion et la gestion des rôles.
- Ajouter des tests unitaires avec Vitest pour les fonctionnalités utilisateur.


**Étape 3 : Gestion des équipements**

- Mettre en place un système d’identification unique pour chaque équipement.
- Stocker les états des équipements dans une base de données.
- Exposer des endpoints pour récupérer et mettre à jour les informations des équipements.


**Étape 4 : Historisation des données**

- Enregistrer les messages reçus (statuts et commandes) dans une base de données.
- Implémenter un système de requêtes pour récupérer les historiques par équipement.
- Tester la performance et la fiabilité du stockage.


**Étape 5 : Statistiques des capteurs et actionneurs**

- Ajouter un module pour calculer et stocker des statistiques d’utilisation et de fonctionnement (journalières, mensuelles, trimestrielles, annuelles).
- Générer des rapports de performance pour les capteurs (ex. température moyenne, pics d’utilisation).
- Inclure des données sur l’utilisation des actionneurs (ex. cycles d’ouverture de porte, temps de fonctionnement du chauffage).
- Exposer des endpoints pour récupérer ces statistiques et les afficher dans le frontend.


**Étape 6 : Notifications en temps réel**

- Ajouter un module pour envoyer des notifications via WebSocket.
- Intégrer des alertes en cas d’équipement hors ligne ou de seuil dépassé.
- Tester l’envoi des notifications sur le frontend.


**Étape 7 : Validation et tests**

- Valider toutes les fonctionnalités implémentées dans un environnement d’intégration.
- Effectuer des tests complets avec Vitest pour garantir la stabilité du backend.

#### Création du frontend

**Étape 1 : Configuration de l’environnement**

- Initialiser le projet Next.js avec les dépendances nécessaires (MUI, WebSocket, Axios, etc.).
- Configurer le thème et les composants globaux avec MUI.


**Étape 2 : Développement des fonctionnalités principales**

- **Sous-étape 1** : Créer le tableau de bord principal avec des graphiques en temps réel pour les données des capteurs et statistiques
- **Sous-étape 2** : Ajouter une section pour lister les équipements connectés avec leurs états.
- **Sous-étape 3** : Implémenter les contrôles interactifs pour gérer les équipements (ex. allumer/éteindre les lumières).


**Étape 3 : Gestion des utilisateurs**

- Développer les pages pour la création, la modification, et la suppression des utilisateurs.
- Intégrer la gestion des rôles (administrateur, utilisateur standard) avec des restrictions d’accès sur certaines sections.


**Étape 4 : Notifications et logs**

- Intégrer un système de notifications en temps réel pour afficher les erreurs et les événements importants.
- Ajouter une page dédiée aux journaux système pour consulter les actions passées.


**Étape 5 : Tests et validation**

- Effectuer des tests unitaires et d’intégration avec Vitest pour chaque composant.
- Valider le fonctionnement complet dans un environnement simulé.


Pour mener à bien le projet d'automatisation du bâtiment, voici les liens vers les documentations officielles des technologies que nous allons utiliser :

- **Mosquitto (Broker MQTT)** : [Documentation officielle de Mosquitto](https://mosquitto.org/documentation/)
- **Fastify (Backend)** : [Documentation officielle de Fastify](https://fastify.dev/docs/latest/)
- **React (Frontend)** : [Documentation officielle de React](https://fr.react.dev)
- **Material-UI (MUI) pour le design** : [Documentation officielle de Material-UI](https://mui.com/material-ui/getting-started/overview/)
- **Docker (Conteneurisation)** : [Documentation officielle de Docker](https://docs.docker.com/)
- **TypeScript (Superset de JavaScript)** : [Documentation officielle de TypeScript](https://www.typescriptlang.org/docs/)
- **Node.js (Environnement d'exécution JavaScript)** : [Documentation officielle de Node.js](https://nodejs.org/en/docs/)
- **Vitest (Framework de test)** : [Documentation officielle de Vitest](https://vitest.dev/guide/)

Pour intégrer le protocole MQTT avec le Siemens LOGO!, voici des ressources techniques officielles et communautaires qui vous guideront dans cette démarche :

7. **Manuel du Siemens LOGO!** : Ce document fournit des informations détaillées sur les fonctionnalités et la configuration du LOGO!.
    [Siemens Industry Support](https://support.industry.siemens.com/cs/attachments/16527461/Logo_f.pdf?utm_source=chatgpt.com)
8. **Discussion sur l'intégration du Siemens LOGO! 8.4 avec MQTT** : Cette discussion de la communauté Home Assistant offre des perspectives pratiques sur l'utilisation du LOGO! avec MQTT.
    [Home Assistant Community](https://community.home-assistant.io/t/siemens-logo-8-4-mqtt/747049?utm_source=chatgpt.com)
9. **Démonstration de la communication bidirectionnelle MQTT avec le Siemens LOGO! V8.4** : Cet article présente une démonstration en direct de l'utilisation du LOGO! avec MQTT.
	[BlocIot](https://blociot.com/2024/02/18/siemens-logo-v8-4-unboxing-and-live-demo-of-bi-directional-mqtt-communication-with-raspberry-pi-5/?utm_source=chatgpt.com)

# Tâches à accomplir

#### **Configurer le broker MQTT**

1.1 : Installer Mosquitto sur le serveur dédié.  
1.2 : Configurer l’authentification des clients MQTT avec un fichier d’utilisateurs et mots de passe.  
1.3 : Activer le chiffrement TLS pour sécuriser les communications.  
1.4 : Définir les topics de base pour les équipements (ex. `devices/<id>/status`, `devices/<id>/command`).  
1.5 : Tester le fonctionnement du broker avec un client MQTT (ex. MQTT Explorer).

#### **Initialiser le dépôt GitHub**

2.1 : Créer le dépôt GitHub pour le frontend, le backend, et la documentation.  
2.2 : Initialiser le dépôt local et configurer les connexions avec GitHub.  
2.3 : Mettre en place la structure des branches (`main`, `develop`, branches fonctionnelles).  
2.4 : Ajouter des templates de pull request et des conventions de commit (ex. Conventional Commits).  
2.5 : Configurer les workflows CI/CD avec GitHub Actions (tests, linting).

#### **Configurer l’environnement backend**

3.1 : Initialiser le projet Fastify avec les dépendances nécessaires (MQTT, REST API, WebSocket).  
3.2 : Définir la structure de base du projet (routes, contrôleurs, middlewares).  
3.3 : Ajouter la configuration Docker pour le déploiement.  
3.4 : Tester le démarrage du backend dans l’environnement local.

#### **Initialiser le projet Next.js**

4.1 : Créer le projet avec Next.js et TypeScript.  
4.2 : Ajouter les dépendances nécessaires (MUI, Axios, WebSocket).  
4.3 : Configurer le thème et les composants globaux avec MUI.  
4.4 : Tester l’affichage d’une page basique dans l’environnement local.

#### **Implémenter les endpoints utilisateur**

5.1 : Créer l’API REST pour l’inscription des utilisateurs (POST `/users/signup`).  
5.2 : Ajouter l’API REST de connexion (POST `/users/login`) avec un token JWT.  
5.3 : Implémenter l’API REST pour récupérer les utilisateurs (GET `/users`).  
5.4 : Ajouter la gestion des rôles utilisateur (admin, utilisateur standard).

#### **Implémenter la gestion des équipements**

6.1 : Créer une table dans la base de données pour les équipements connectés.  
6.2 : Ajouter une API REST pour enregistrer de nouveaux équipements (POST `/devices`).  
6.3 : Développer une API REST pour mettre à jour les états des équipements (PATCH `/devices/:id`).  
6.4 : Implémenter une API REST pour récupérer la liste des équipements (GET `/devices`).

#### **Ajouter le module d'historisation**

7.1 : Créer une table pour stocker les données historiques des équipements (statuts et commandes).  
7.2 : Implémenter un service pour enregistrer automatiquement les données reçues via MQTT.  
7.3 : Ajouter une API REST pour consulter les historiques d’un équipement (GET `/devices/:id/history`).

#### **Ajouter le calcul des statistiques**

8.1 : Créer une table ou des vues pour stocker les statistiques des capteurs/actionneurs.  
8.2 : Implémenter un service pour calculer les statistiques (journalières, mensuelles, etc.).  
8.3 : Ajouter une API REST pour récupérer les statistiques par période (GET `/devices/:id/statistics`).

#### **Développer le tableau de bord principal**

9.1 : Afficher les données des capteurs en temps réel (graphiques, widgets).  
9.2 : Ajouter une liste des équipements connectés avec leurs états.  
9.3 : Intégrer des contrôles interactifs pour les équipements (ex. allumer/éteindre).

#### **Intégrer les notifications en temps réel**

10.1 : Configurer un service WebSocket dans le backend pour gérer les notifications.  
10.2 : Implémenter la réception des notifications dans le frontend.  
10.3 : Afficher les notifications dans le tableau de bord.
