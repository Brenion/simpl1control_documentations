
## **MoSCoW du démonstrateur domotique de bureau**



### **Must Have (Indispensable)**

- **API**
    
    - Recevoir et enregistrer les données du compteur d'énergie.
    - Recevoir des instructions du backend afin d'actionner ses sorties (allumage du chauffage / ouverture de la porte sécurisée).
    - gestion des pannes IOT perte contact avec le capteur de température / la porte / le compteur

- **Backend**
    
    - Communication avec l’API pour recevoir les données d'énergie et envoyer des commandes aux pré-actionneurs.
    - Enregistrement des données en temps réel dans la base de données.
    - Interface entre la base de données et l’application frontend.
    - Réception MQTT d’un capteur de température déporté.

- **Frontend**
    
    - Tableau de bord affichant les mesures de température et les statistiques énergétiques.
    - Interface pour envoyer des commandes aux actionneurs (chauffage).

- **Hardware**
    
    - Lecteur de carte/digicode permettant d'ouvrir une porte sécurisée.
    - Capteur WiFi envoyant en temps réel la température d'une pièce.

---

### **Should Have (Important mais pas essentiel au démonstrateur initial)**

- **Backend**
    
    - **Communication IoT** avec le protocole Zigbee.
    - **Enregistrement des actionneurs et pré-actionneurs** tiers.
    - Enregistrement  de scénarios domotiques** (ex : activation automatique des lumières après 18h).

- **Frontend**
    
    - **Gestion avancée des actions différées**.
    - **Programmation de scénarios domotiques** (ex : activation automatique des lumières après 18h).
    - **Supervision énergétique plus détaillée**, avec alertes (ex : consommation anormale détectée).
    - **Authentification et gestion des utilisateurs**, avec une interface permettant de gérer les droits d’accès aux commandes domotiques.

---

### **Could Have **

- **API**
    
    - Récupération automatique des tarifs énergétiques pour calculer le coût total.
	- **Historique avancé des mesures et actions**
    - Graphiques interactifs sur plusieurs semaines/mois.

---

### **Won’t Have (Hors du périmètre actuel)**

- **Interface mobile optimisée** avec PWA (Progressive Web App).
- **Sécurité avancée avec chiffrement bout en bout** pour les communications API.
- rapport
