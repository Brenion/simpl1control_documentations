## 🎯 Objectif

Gérer un système de contrôle d'accès basé sur une carte **MIFARE Classic** dans un premier temps, avec une architecture prévue pour évoluer vers **MIFARE DESFire**. La carte contient un **token opaque** (identifiant chiffré de type AES-128, mais **traité uniquement au niveau du backend**). Le système repose sur un **lecteur NFC lecture seule**, qui transmet ce token vers un backend Node.js chargé de la validation.

---

## ✅ OPTION A - Lecture d'un token via carte MIFARE Classic (solution de départ)

### ✨ Objectif

Lire un **token opaque** inscrit sur une carte MIFARE Classic, le transmettre au backend via un lecteur PN532, et effectuer la vérification d'accès de manière sécurisée côté serveur.

### ✅ Avantages

- Simplicité matérielle (lecteur PN532 + ESP32)
    
- Aucun traitement cryptographique côté client (lecture brute du token)
    
- Backend seul détient la clé de vérification AES-128 → centralisation de la sécurité
    

### ❌ Inconvénients

- Pas de protection matérielle du token sur la carte (carte clonable)
    
- Le canal ESP32 → backend doit être sécurisé (HTTPS ou MQTT avec TLS)
    

### ✍️ Étapes de mise en place

1. ⚡ **Lecture du token** via ESP32 + PN532 (lecture seule d'un secteur de la carte)
    
2. 📡 **Transmission du token brut** au backend via HTTP/MQTT
    
3. 🔐 **Déchiffrement et vérification AES-128** dans le backend (Node.js)
    
4. ✅ **Décision d'autorisation ou refus** côté backend, commande d'ouverture éventuelle
    

### 📄 Exemple de contenu dans la carte MIFARE Classic

```
Bloc 8 : 8 octets aléatoires (token AES généré par le backend)
```

### 🖌️ Schéma Option A

```
[Carte MIFARE Classic] --(token opaque)--> [Lecteur PN532 + ESP32] --(MQTT ou HTTP sécurisé)--> [Backend Node.js] --(décryptage AES)--> [Décision accès]
```

---

## 🔄 OPTION B - Lecture d'un identifiant utilisateur sécurisé sur carte DESFire

### ✨ Objectif

Lire un fichier sécurisé contenant un ID utilisateur sur la carte DESFire (ex: "user42"), avec authentification AES directement sur la carte.

### ✅ Avantages

- Authentification matérielle forte (AES natif sur la carte)
    
- Impossible de cloner la carte sans la clé
    
- Meilleure conformité avec des exigences domotiques renforcées
    

### ❌ Inconvénients

- Requiert un lecteur NFC compatible AES (ex: ACR1252U)
    
- Demande une configuration initiale des cartes plus complexe
    

### ✍️ Étapes de mise en place

1. 🚀 **Remplacer le lecteur** par un ACR1252U ou équivalent
    
2. 📔 **Encoder la carte** avec application + fichier contenant un ID sécurisé
    
3. 💡 **Lire et authentifier via le backend** à l’aide d’une librairie PC/SC (Node.js)
    
4. 🧲 **Vérification d’accès et commande d’action**
    

### 🖌️ Schéma Option B

```
[Carte DESFire] --(auth AES + ID)--> [Lecteur ACR1252U] --> [Backend Node.js] --> [BDD] --> [Autorisation / Relais]
```

---

## 📚 À propos du lecteur ACR1252U

### Description

L’**ACR1252U** est un lecteur de cartes à puce USB compatible avec les cartes **MIFARE DESFire EV1/EV2**. Il supporte les standards ISO/IEC 14443 A/B et intègre nativement le support des **commandes AES**, indispensables pour accéder aux données sécurisées sur les cartes DESFire.

### Caractéristiques clés :

- ✅ Support complet de MIFARE DESFire EV1/EV2, y compris les commandes ISO7816-4
    
- ✅ Compatible avec les bibliothèques **PC/SC** (Windows/Linux/macOS)
    
- ✅ Communication via **USB** (fonctionne en plug-and-play sur Raspberry Pi)
    
- 🔐 Gestion de la cryptographie **AES-128** en natif
    
- 🔌 Alimentation par USB, pas de montage électronique requis
    

### Utilisation dans le projet :

- **Remplace le PN532** dans la chaîne
    
- **Permet d’accéder aux fichiers protégés AES** sur les cartes DESFire
    
- **Librairies Node.js compatibles** : `nfc-pcsc`, `pcsclite`, ou interfaçage bas niveau en C
    

### Prix indicatif : ~45–60 €

➡️ Recommandé pour une solution domotique sécurisée, évolutive vers une implémentation à grande échelle.

---

## ❗ Justification de l'utilisation de MIFARE Classic (ou lecture UID)

### Contexte du démonstrateur

Ce démonstrateur a pour objectif de **valider la faisabilité** d'un système de contrôle d'accès NFC intégré à une solution domotique, tout en **évaluant les ressources humaines et matérielles nécessaires** à la transition vers une version sécurisée basée sur DESFire.

### Raisons techniques

- 🔧 **Simplicité matérielle** : le lecteur PN532 est compatible avec l'ESP32, facile à intégrer
    
- 💰 **Réduction des coûts initiaux** : pas besoin de lecteur AES pour la phase de test
    
- 🧪 **Évaluation de la chaîne logicielle** : permet de valider la communication entre lecteur, backend, et module d'action (relais)
    

### Simulation réaliste

Le système en lecture UID permet de simuler le comportement final d'une solution DESFire sécurisée, tout en maintenant la souplesse pour le développement.

### Évolution planifiée

L'architecture logicielle est **prévue pour évoluer sans changement majeur** lorsque le lecteur DESFire AES sera introduit, garantissant une continuité de l'effort de développement et de maintenance.

---

## 💸 Coût d’un système sécurisé avec DESFire et lecteur industriel

|Élément|Référence/Exemple|Prix approximatif|
|---|---|---|
|Lecteur NFC compatible AES|HID OMNIKEY 5427CK / ACR1252U|45–120 €|
|Carte MIFARE DESFire EV1|NXP 4K, EV1|2–4 € / carte|
|Câblage ou boîtier technique|Boîtier IP65, connectique|15–30 €|
|Raspberry Pi (backend)|Pi 4 ou Pi Zero|30–60 €|

➡️ **Total pour une borne sécurisée : environ 100–200 €**

Ce budget peut être progressivement absorbé dans une stratégie domotique de moyen terme, en particulier dans les contextes résidentiels ou semi-professionnels où la sécurité devient un critère différenciant.