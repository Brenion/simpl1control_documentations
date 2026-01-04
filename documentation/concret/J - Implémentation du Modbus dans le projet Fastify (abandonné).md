
### Table des matières

1. Introduction
    - Objectif de cette documentation
    - Pourquoi intégrer Modbus ?
        
2. Prérequis
    - Matériel utilisé : Compteur Schneider IEM2055
    - Communication via Modbus TCP
        
3. Installation des dépendances
    - Présentation du package utilisé
    - Commande d’installation
        
4. Mise en place de la communication Modbus
    - Création d’un service dédié
    - Lecture/écriture des registres (exemples concrets)
    - Gestion des erreurs
        
5. Intégration avec les routes Fastify
    - Endpoint pour lire ou écrire des données Modbus
        
6. Tests & Débogage
    - Conseils pour tester localement
    - Logs utiles
        
7. Perspectives   
    - (Section supprimée : Intégration avec MQTT / Home Assistant)

---

### 1. Introduction

#### Objectif de cette documentation

Cette documentation a pour but de détailler l'intégration du protocole Modbus dans le backend Fastify du projet. L'objectif principal est de permettre la communication fiable avec un compteur Schneider IEM2055, en assurant une conversion correcte des données vers un format exploitable (Float32).

#### Pourquoi intégrer Modbus ?

Il est particulièrement difficile de communiquer en Modbus avec un Siemens LOGO! 8.4 lorsqu’il s’agit de gérer des entiers sur 64 bits. Afin de contourner cette limitation et de permettre une lecture cohérente des données, nous avons choisi de déléguer la communication Modbus à notre backend. Ce backend jouera le rôle d’interface entre le compteur Schneider IEM2055 et le reste du système, en convertissant les données reçues en format **Float32** pour une utilisation finale correcte.

### 2. Prérequis

#### Matériel utilisé : Compteur Schneider IEM2055

Le Schneider IEM2055 est un compteur d’énergie modulaire destiné à la surveillance de la consommation électrique dans des installations industrielles ou tertiaires. Il permet la mesure de divers paramètres électriques (tension, courant, puissance, énergie, etc.) et communique via le protocole Modbus RTU. Il s’installe facilement sur rail DIN et s’intègre dans un réseau de supervision énergétique.

#### Communication via Modbus TCP

Le compteur IEM2055 utilise nativement le protocole Modbus RTU (série). Pour l’intégration avec notre backend (qui utilise Modbus TCP), un convertisseur passerelle Modbus RTU vers Modbus TCP est nécessaire. Cela permet de faire transiter les trames série (RS-485) du compteur vers un réseau IP accessible par le backend.

```
┌──────────────────────┐   RS-485   ┌────────────────────────────┐   TCP/IP  ┌────────────────────┐ 
│ Compteur Schneider   │◄──────────►│ Passerelle Modbus RTU/TCP  │◄─────────►│ Backend Fastify    │
│ IEM2055 (Modbus RTU) │            │ (ex: Schneider EGX100)     │           │ (Modbus TCP Client)│
└──────────────────────┘            └────────────────────────────┘           └────────────────────┘
```

Cette architecture permet une communication fiable entre le backend Fastify et le compteur, tout en respectant les contraintes matérielles existantes.
    

### 3. Installation des dépendances

- Présentation du package utilisé
    
- Commande d’installation
    

### 4. Mise en place de la communication Modbus

- Création d’un service dédié
    
- Lecture/écriture des registres (exemples concrets)
    
- Gestion des erreurs
    

### 5. Intégration avec les routes Fastify

- Endpoint pour lire ou écrire des données Modbus
    

### 6. Tests & Débogage

- Conseils pour tester localement
    
- Logs utiles
    

### 7. Perspectives

- _(Section supprimée : Intégration avec MQTT / Home Assistant)_