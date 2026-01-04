
## Proposition de stages : mise en place d’un démonstrateur d’un système domotisé et automatisé de bureau  
  

## Lieu du stage

Triptyk SPRL, chaussée de Binche 177A, 7000 Mons

Maitre de stage :  
  
Monsieur Gilles Bertand.  
  

## Cahier des charges

  
Ce stage a pour objectif de développer un démonstrateur d’un système domotisé et automatisé pour un bureau. Le but principal sera de mettre en place la communication entre une API et un backend qui servira de lien entre une base de données et l’API. Voici les différents composants et leurs rôles :

**Backend de Gestion des Capteurs et Préactionneurs**

- **Communication avec l’API (Automate Programmable)** : Le backend recevra des données provenant des capteurs via l’API et enverra des instructions aux préactionneurs, toujours via l’API. Par exemple, il pourra recevoir des informations sur la température et l'humidité et envoyer des commandes pour ajuster le chauffage ou la climatisation.
- **Gestion des Données** : Il servira de gestionnaire externe, enregistrant les données dans une base de données et récupérant les instructions enregistrées dans celle-ci pour les exécuter au moment opportun. Par exemple, une commande de fermeture des volets à une certaine heure peut être programmée et stockée dans la base de données.

**Backend de Communication avec l’Application Frontend**

- **Lien avec la Base de Données** : Un second backend sera responsable de la communication entre la base de données et une application web, accessible sur mobile, tablette et PC.
- **Affichage des Données** : L’application pourra recevoir les données et les afficher dans un tableau de bord de statistiques. Par exemple, elle pourra montrer des graphiques en temps réel de la consommation énergétique du bureau.
- **Gestion des Instructions** : Des panneaux supplémentaires dans l’application permettront de gérer le bureau avec des instructions directes ou enregistrées qui seront lancées en différé. Par exemple, un utilisateur pourra programmer l'activation de l'éclairage à une heure précise ou ajuster la luminosité directement depuis son appareil.

Ce projet intégrera donc divers composants technologiques pour offrir une solution de gestion domotique et automatisée, rendant le bureau intelligent et adaptable aux besoins des utilisateurs.

## Schéma de principe

![[Pasted image 20250210222407.png]]

## Diagramme

![[Pasted image 20250210222431.png]]


## Description des interactions :

1. **L’Application** envoie des instructions au **Backend**.
2. **Le Backend** envoie des données traitées à **l’Application** pour affichage.
3. **Le Backend** enregistre des instructions différées à exécuter dans la **Base de données**.
4. **Le Backend** reçoit et traite des données de la **Base de données**.
5. **Le Backend** envoie des instructions directes au **backend API**.
6. **Le Backend** **API** enregistre les données reçues par l'**API**.
7. **Le Backend API** requête la base de donnée pour vérifier si des instructions différées doivent être activée.
8. **Le Backend API** traite et envois les instructions a l’**API**.
9. **Le Backend API** traite les données reçues par l’**API** afin de les enregistré dans la **Base de données** .
10. **L’API** gère les préactionneurs
11. **Préactionneurs** commandes les **Actionneurs**.
12. **L’API** reçoit les données des divers **Capteurs**

Matériel nécessaire :  
  
PC : pour le développement,  
Raspberry PI : communiquant avec l’API

Serveur : un serveur pour le déploiement

API : plusieurs arduino peuvent servir en tant que démonstrateur

Capteurs : capteurs environnementaux (hydrométrique, température, co2…), capteur de carte RFID, carpteur de mouvement…

Autre : microcontrôleur (esp32...)

Cette liste est non exhaustive surtout niveau des capteurs et matériels autour de ceux-ci.

Budget :  
  
la plupart du matériel est déjà à portée de main et a l’heure actuel un budget fixe n’est possible.

            Prévisionnelle :

-       Raspberry Pi 5 8Go: 91,95€ (MChobby)

-       HAT M.2 PCIe officiel pour Raspberry-Pi 5 : 13,5€ (MCHobby)

-       LDR dollatek 5Pc : 6 ,98€ (Amazon.fr)

-       Schneider iEM 2055 : 177,63€

-       Provision: 200€

Total = 490,06€

  

## Exemple de principes et Matériels Nécessaires pour un Système Domotisé de Bureau

**_Capteur de Température_**

**Principe :** Le capteur de température mesure la température ambiante et transmet ces données au système domotique. Les données sont ensuite affichées sur un tableau de bord visuel et utilisées pour contrôler des équipements comme le chauffage ou la climatisation.

**Matériel Nécessaire :**

1. **Capteur de Température** : DHT22.
2. **Microcontrôleur** : Arduino et Raspberry Pi.
3. **Backends** : Serveur Node.js ou Rust avec MQTT.
4. **Base de Données** : PostgreSQL.
5. **Frontend Application** : Framework Ember.
6. **Communication** : WiFi Module (ESP32).

**Exemple de Fonctionnement :**

1. Le capteur de température mesure la température ambiante.
2. Les données sont envoyées au microcontrôleur.
3. Le microcontrôleur envoie au Raspberry.
4. Le backend récupère les données et les enregistre dans la base de données.
5. L'application frontend récupère les données de la base de données et les affiche sur un tableau de bord visuel.

**_Capteur de Luminosité_**

**Principe :** Le capteur de luminosité mesure l’intensité lumineuse et transmet ces données au système domotique. Les données sont affichées sur un tableau de bord visuel et utilisées pour contrôler des équipements comme les volets ou l'éclairage.

**Matériel Nécessaire :**

6. **Capteur de Luminosité** : BH1750 ou LDR (Light Dependent Resistor).
7. **Microcontrôleur** : Arduino et Raspberry Pi.
8. **Backend** : Serveur Node.js ou Rust avec MQTT.
9. **Base de Données** : PostgreSQL.
10. **Frontend Application** : Framework Ember.
11. **Communication** : WiFi Module (ESP32).

**Exemple de Fonctionnement :**

12. Le capteur de luminosité mesure l’intensité lumineuse.
13. Les données sont envoyées au microcontrôleur.
14. Le microcontrôleur envoie au raspberry.
15. Le backend récupère les données et les enregistre dans la base de données.
16. L'application frontend récupère les données de la base de données et les affiche sur un tableau de bord visuel.

**_  
Porte à Ouverture par RFID_**

**Principe :** Le système RFID permet de contrôler l’accès à la porte en scannant une carte RFID. Les informations de la carte sont vérifiées et, si valides, la porte est déverrouillée. Il pourrait être nécessaire de sécurisé une zone particulière du bâtiment qui lorsque la carte est valide une alarme par capteur de présence

**Matériel Nécessaire :**

17. **Lecteur RFID** : RC522.
18. **Microcontrôleur** : Arduino et Raspberry Pi.
19. **Relais** : Pour contrôler la serrure électrique.
20. **Serrure Électrique** : Serrure magnétique ou motorisée.
21. **Backend** : Serveur Node.js ou Rust avec MTTQ.
22. **Base de Données** : PostgreSQL.
23. **Frontend Application** : Framework Ember.
24. **Communication** : WiFi Module (ESP32).

**Exemple de Fonctionnement :**

25. La carte RFID est scannée par le lecteur.
26. Le lecteur envoie les données au microcontrôleur.
27. Le microcontrôleur vérifie les données via l'API.
28. Le backend valide les informations et renvoie la réponse.
29. Si la carte est valide, le microcontrôleur active le relais pour déverrouiller la porte.
30. Les informations d’accès sont enregistrées dans la base de données pour des raisons de sécurité et de suivi.

**_Supervision de l'Énergie Électrique avec un Compteur Modulaire_**

#### **Principe**

Le système de supervision de l'énergie électrique permet de surveiller la consommation d'électricité en temps réel à l'aide d'un compteur modulaire. Les données collectées sont envoyées à un backend qui les enregistre dans une base de données et les affiche sur un tableau de bord visuel. Cela permet de suivre la consommation énergétique, d'identifier les pics de consommation et de prendre des mesures pour optimiser l'utilisation de l'énergie.

#### **Matériel Nécessaire**

1.     **Compteur Modulaire d'Énergie Électrique** :

- Exemples : Schneider Electric iEM2000,
- Fonctionnalités : Mesure de la consommation électrique (kWh), courant, tension, puissance, etc.

2.     **Microcontrôleur** :

- Exemples : Arduino, ESP8266, ESP32, Raspberry Pi.
- Fonctionnalités : Collecte des données du compteur et transmission au backend.
- **Module de Communication** :
- Exemples : ESP32
- Fonctionnalités : Envoi des données collectées au backend via Internet.

3.     **Backend** :

- Exemples : Serveur Node.js ou Rust.
- Fonctionnalités : Traitement des données, stockage dans la base de données, fourniture des données à l'application frontend.

4.     **Base de Données** :

- Exemples : PostgreSQL
- Fonctionnalités : Stockage des données de consommation énergétique.

5.     **Frontend Application** :

- Framework : EmberJS
- Fonctionnalités : Affichage des données de consommation sur un tableau de bord visuel, graphiques, alertes.

#### **Exemple de Fonctionnement**

1.     **Collecte des Données** :

- Le compteur modulaire mesure la consommation électrique en temps réel et transmet les données au microcontrôleur.

2.     **Transmission des Données** :

- Le microcontrôleur, équipé d'un module de communication WiFi envoie les données au backend via une API.

3.     **Traitement et Stockage** :

- Le backend reçoit les données, les traite et les enregistre dans la base de données.

4.     **Affichage des Données** :

- L'application frontend récupère les données de la base de données et les affiche sur un tableau de bord visuel. Le tableau de bord permet de visualiser les données en temps réel, de consulter l'historique de la consommation, et de configurer des alertes en cas de dépassement de seuils définis.

Pour Finir …  
  
Ce stage vise à évaluer la faisabilité de l'implantation d'un système domotique au sein de l'entreprise en termes de temps de travail et de coût. Bien que ce projet ne soit pas la réalité, il en est un reflet. Pour mettre en place un système domotique professionnel, il pourrait être nécessaire d'utiliser du matériel plus spécialisé et de respecter les normes belges et européennes, afin que ce démonstrateur puisse devenir une solution professionnelle viable.

**Exemple de bénéfices potentiels de ce projet :**

**Réduction des coûts énergétiques** : Un système domotique optimisé peut réduire considérablement la consommation d'énergie de l'entreprise, entraînant ainsi des économies substantielles sur les factures d'électricité.

**Amélioration du confort et du bien-être des employés** : En intégrant des capteurs de température et d'autres dispositifs domotiques, l'entreprise peut créer un environnement de travail optimal, adapté aux besoins individuels des développeurs. Cela peut contribuer à une meilleure productivité et à une diminution des interruptions liées à des conditions inconfortables.