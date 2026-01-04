# Plan de présentation (15 minutes)

## 1. Titre & accroche (20-30s)

- Titre du projet
    
- Ton nom
    
- Mention de la démonstration live
    

## 2. Présentation de l’entreprise (30-45s)

- Triptyk : transformation numérique, automatisation, optimisation énergétique
    
- Lien direct avec le projet
    

## 3. Contexte du projet (1min)

- Brève introduction au besoin général : solutions propriétaires limitées et coûteuses
    
- Pourquoi une solution ouverte et déployable pour PME
    

## 4. Présentation succincte & objectifs (1min30)

- Démonstrateur de supervision énergétique
    
- Contrôle d’accès RFID
    
- Interface web déployable
    
- **Objectifs du projet :**
    
    - Montrer faisabilité d’une solution domotique ouverte
        
    - Démontrer communication temps réel et sécurité
        
    - Offrir une solution déployable et évolutive pour PME
        

## 5. Architecture matérielle (1min)

- Raspberry Pi, Siemens LOGO!, relais, lampe
    
- ESP32 (simulateur), compteur Schneider
    
- 📝 _Pense-bête : mentionner programmation LOGO! avec SoftComfort et firmware ESP32 en C++ (fait partie du matériel)._
    

## 6. Schéma de communication matérielle (30s)

- Schéma visuel montrant les connexions entre les différents matériels (LOGO!, relais, ESP32, compteur, Raspberry Pi)
    
- Mettre en évidence les flux physiques (câblage, signaux)
    
- Protocoles : MQTT, Modbus
    
- Flux de données : capteurs → broker → backend → dashboard
    
- Sécurité : TLS, authentification
    

## 7. Architecture logicielle (1min30)

- Backend : Node.js
    
- Base de données : PostgreSQL
    
- Conteneurisation : Docker (environnement englobant)
    
- Frontend : Ember/React
    

## 8. Transition vers démonstration (30s)

- Expliquer ce que le public va voir en direct
    

## 9. Démonstration live (4min)

- Connexion
    
- Dashboard LOGO et température
    
- Interface devices
    
- Enregistrement d’un badge
    
- Ouverture de la porte
    
- Visualisation dans l’historique
    

## 10. Résultats obtenus & problèmes rencontrés (1min30)

- Fonctionnement en temps réel validé
    
- Communication sécurisée et interopérable
    
- Déploiement via Docker
    
- Problèmes : bugs mineurs, latence possible
    
- Solutions envisagées : correctifs futurs
    

## 11. Améliorations (1min30)

- Sécurité filaire porte
    
- Ajout Zigbee/Matter
    
- Interface PWA mobile
    

## 12. Conclusion & appel aux questions (30s)

- Récapitulatif et ouverture sur le déploiement futur
    
- Invitation aux questions
    

---

# Scénario de démonstration (≈ 4 minutes)

1. **Connexion** (30s)
    
    - Page de login et authentification
        
2. **Dashboard LOGO** (30s)
    
    - Montrer entrées/sorties et relais (lampe allumée/éteinte)
        
3. **Dashboard température (simulateur)** (40s)
    
    - Variation de 15° à 23° en temps réel
        
4. **Enregistrement d’une carte** (50s)
    
    - Ajouter une carte dans l’interface utilisateur et valider
        
5. **Ouverture de la porte** (50s)
    
    - Tester avec carte valide (option : carte non valide)
        
6. **Visualisation dans l’historique** (30s)
    
    - Montrer que l’événement est loggé
        
7. **Interface Devices** (20s)
    
    - Présenter rapidement les équipements connectés
        
8. **Clôture** (10s)
    
    - Résumer : supervision, contrôle, traçabilité



### **Axes d’amélioration techniques**

- **Architecture plus professionnelle :**
    
    - Passer la sécurité de la porte en filaire plutôt qu’en Wi-Fi.
        
    - Utiliser du matériel plus spécialisé respectant les normes belges et européennes.
        
- **Backend :**
    
    - Ajout du support du protocole **Zigbee** pour les IoT.
        
    - Enregistrement d’actionneurs tiers et gestion plus avancée des scénarios domotiques.
        
- **Frontend :**
    
    - Ajout d’une interface d’authentification et de gestion des utilisateurs.
        
    - Programmation d’actions différées et scénarios automatiques.
        
    - Supervision énergétique avec alertes en cas de consommation anormale.
        
- **API et Données :**
    
    - Récupération automatique des **tarifs énergétiques** pour calculer les coûts.
        
    - Historique avancé des mesures et actions.
        
    - Graphiques interactifs sur plusieurs semaines/mois.
        
- **Sécurité :**
    
    - Audit de sécurité et **pentest** du système.
        
    - Intégration d’une solution **ZTNA** ou VPN pour sécuriser les accès réseau.
        
    - Analyse du code avec **SonarQube/Snyk** pour éviter injections SQL et failles RGPD.
        

---

### 🚀 **Évolutions fonctionnelles**

- Déploiement d’un **lecteur de badge plus sécurisé** avec chiffrement AES-128 (voire AES-256 si compatible).
    
- Création d’un **enregistreur de badges autonome**, sans dépendance PC.
    
- Mise en place d’un **reset administrateur** pour lever les alertes plutôt qu’un simple redémarrage.
    
- Possibilité de connecter plusieurs capteurs et d’avoir un setpoint global optimisé (gestion multi-capteurs).
    
- Développement d’une **interface mobile (PWA)** pour une meilleure accessibilité.
    

---

### 📊 **Perspectives générales**

- Rédiger un **cahier des charges plus détaillé** pour la suite.
    
- Effectuer une **analyse plus fine des besoins** de l’entreprise et des futurs clients.
    
- Rendre la solution **clé en main et industrialisable** pour PME et espaces de travail.
    
- Garantir **interopérabilité** et modularité pour intégrer des équipements de différentes marques.
    
- Objectif final : un système professionnel **sécurisé, scalable et simple à déployer**.