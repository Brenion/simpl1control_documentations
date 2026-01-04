# 📘 Étape 2 – Configuration de la communication Modbus TCP dans *LOGO! Soft Comfort*

## 🎯 Objectif

Configurer un Siemens LOGO! (v8.4.1) en **Modbus TCP Client** pour lire des données depuis un compteur d’énergie **Schneider iEM2055**, afin de :

- Stocker les données dans des variables internes du LOGO!
- Les afficher sur l’écran intégré du LOGO!
- Permettre la navigation via le bouton "flèche vers le bas"
- Rafraîchir les données toutes les 3 secondes

---

## 🔧 Partie 1 : Configuration de la connexion Modbus TCP

### 1.1 Ouvrir la configuration

1. Lancer *LOGO! Soft Comfort*
2. Ouvrir ton projet
3. Aller dans le menu ***"Tools"*** > ***"Modbus Connection"***

---

### 1.2 Créer une connexion Modbus TCP

- Dans la fenêtre *"Modbus Connection"* :
  - Ne pas cocher "Client" ni "Server"
  - Saisir les paramètres :
    - **IP Address** : adresse IP du compteur Schneider (ex. `192.168.0.100`)
    - **PORT** : `502` (par défaut pour Modbus TCP)
  - Cocher **"Accept all connection request in server side"** si visible (optionnel ici)

---

### 1.3 Compléter la table *Data transfer*

> Cette section permet de lier chaque **variable interne du LOGO!** à une **adresse Modbus du compteur Schneider**.

| ID  | Start Address (LOGO!) | Length | Direction | Start Address (Compteur) | Length | Unit ID | Description associée                             |
|-----|------------------------|--------|-----------|---------------------------|--------|---------|--------------------------------------------------|
| 1   | VW0                   | 20     | ←         | IR // 51                  | 20     | 1       | Modèle de compteur (UTF8)                        |
| 2   | VW40                  | 2      | ←         | IR // 3029                | 2      | 1       | Tension (float 32 bits)                          |
| 3   | VW44                  | 2      | ←         | IR // 3001                | 2      | 1       | Courant (float 32 bits)                          |
| 4   | VW48                  | 2      | ←         | IR // 3055                | 2      | 1       | Puissance active (float 32 bits)                 |
| 5   | VW52                  | 2      | ←         | IR // 3069                | 2      | 1       | Puissance réactive (float 32 bits)               |
| 6   | VW56                  | 2      | ←         | IR // 3077                | 2      | 1       | Puissance apparente (float 32 bits)              |
| 7   | VW60                  | 4      | ←         | IR // 3205                | 4      | 1       | Énergie active directe (Int64)                   |
| 8   | VW68                  | 4      | ←         | IR // 3209                | 4      | 1       | Énergie active inverse (Int64)                   |
| 9   | VW76                  | 4      | ←         | IR // 3213                | 4      | 1       | Énergie active totale (Int64)                    |
| 10  | VW84                  | 4      | ←         | IR // 3221                | 4      | 1       | Énergie réactive directe (Int64)                 |
| 11  | VW92                  | 4      | ←         | IR // 3225                | 4      | 1       | Énergie réactive inverse (Int64)                 |
| 12  | VW100                 | 4      | ←         | IR // 3229                | 4      | 1       | Énergie réactive totale (Int64)                  |

---

### 📌 Explication des colonnes

- **Start Address (LOGO!)** : variable interne dans laquelle les données reçues sont stockées (ex : `VW40`)
- **Length** : nombre de registres utilisés dans le LOGO!
- **Direction** :
  - `←` = le LOGO! **lit** les données depuis le compteur (fonction 04 - Input Register)
  - `→` = le LOGO! **écrit** vers le compteur (non utilisé ici)
- **Start Address (Compteur)** : adresse dans le compteur, précédée de son **type de registre**
  - `IR //` = Input Register (lecture seule)
  - `HR //` = Holding Register (lecture/écriture)
- **Unit ID** : identifiant de l'esclave (souvent `1` pour le Schneider iEM2055)
- **Description** : signification ou usage de la donnée

---

## 📦 Partie 2 : Association aux variables internes du LOGO!

Chaque registre Modbus correspond à **1 mot (16 bits)** dans le LOGO!, c’est-à-dire un `VW`.  
Les données sont donc lues séquentiellement et placées dans `VW0`, `VW2`, `VW4`, etc.

---

## 📺 Partie 3 : Configuration de l'affichage

### 3.1 Activer l’écran

- Aller dans ***"Program"***
- Ouvrir le ***"Display"***
- Activer l’écran HMI (afficheur intégré du LOGO!)

---

### 3.2 Ajouter des pages d’affichage

#### 🖥️ Page 1 – Informations principales :
- Date du jour (via bloc d’horloge système)
- Modèle de l’appareil (affichage UTF8 à partir de `VW0`)
- Tension (`VW40`)
- Courant (`VW44`)
- Puissance active (`VW48`), réactive (`VW52`), apparente (`VW56`)

#### 🖱️ Page 2 – Énergie active :
- Directe (`VW60`)
- Inverse (`VW68`)
- Totale (`VW76`)

#### 🖱️ Page 3 – Énergie réactive :
- Directe (`VW84`)
- Inverse (`VW92`)
- Totale (`VW100`)

---

### 3.3 Navigation entre les pages

- Utiliser un bouton *"Down"* (flèche vers le bas)
- Créer un compteur interne (ex. `M0`, `M1`, `V0`, etc.)
- Utiliser ce compteur comme index pour afficher la page sélectionnée via un bloc ***"MUX"***

---

## ⏱️ Partie 4 : Rafraîchissement toutes les 3 secondes

### 4.1 Générer une impulsion cyclique

- Aller dans ***"Program"***
- Ajouter un bloc ***"Pulse Generator"*** :
  - **Période** : 3 secondes
- Utiliser cette impulsion pour :
  - Déclencher une actualisation logique
  - Forcer la mise à jour de l’affichage si nécessaire

> 💡 Il est aussi possible de conditionner l’affichage ou la lecture via un bit déclenché à chaque impulsion.

---

## 📎 Annexes

### 📄 Documentation utilisée

- [Manuel utilisateur Schneider iEM2055 (FR)](https://www.productinfo.schneider-electric.com/iem2050/5b3e7b76d674a900015d9be8/iEM2050%20User%20Manual/French/BM_iEM2000seriesUserManual_French_fr_0000212068.ditamap)

---
