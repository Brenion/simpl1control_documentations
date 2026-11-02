# LOGO! – Communication Modbus/TCP avec l’exemple du SENTRON PAC3200

## Informations légales

Ce document présente un exemple d'application entièrement configuré pour **LOGO! 8** dans le cadre d'une communication avec un appareil compatible Modbus/TCP, ici le **SENTRON 7KM PAC3200**.

---

## 1. Introduction et description de la tâche

Cet exemple démontre une mise en œuvre fonctionnelle avec LOGO!. Les exigences de sécurité fonctionnelle (ex. arrêt d'urgence) ne sont pas couvertes. L’utilisateur doit se conformer aux directives en vigueur.

LOGO! propose :
- des blocs fonctionnels prédéfinis : minuterie hebdomadaire, générateur d’impulsions, minuterie astro, etc.
- des possibilités d'extension du programme selon les besoins.
- une disposition de câblage simple (en étoile).
- une communication via Modbus/TCP, S7, KNX.

### Public cible

Spécialistes de l’installation électrique ou de l’automatisation.

### Objectif

Transférer et afficher des grandeurs électriques (tension, courant, puissance, fréquence) du PAC3200 vers LOGO! via Modbus/TCP.

---

## 2. Composants utilisés

### 2.1 Matériel – LOGO!

- LOGO!Soft Comfort V8.2
- LOGO! 12/24 RCE
- LOGO! TDE (optionnel)
- LOGO! Power 24V / 1.3A

### 2.2 Matériel – SENTRON 7KM PAC3200

- Appareil de mesure PAC3200
- Alimentation multi-tension
- Communication intégrée via Modbus/TCP
- Affichage des paramètres réseau
- Mesure monophasée, biphasée ou triphasée
- Entrées/sorties numériques
- Interface utilisateur avec 4 touches

---

## 3. Mise en service

### 3.1 Mise en service – PAC3200

#### Paramètres de base – Entrée de tension
- Type de connexion : **1P2W**
- Menu : **_MAIN MENU_ → SETTINGS → BASIC PARAMETERS → VOLTAGE INPUTS**

#### Paramètres de base – Entrée de courant
- I primaire : 100A, I secondaire : 1A
- Rapport de transformation = 100:1

#### Communication
- Protocole par défaut : SEAbus/TCP → à changer pour **Modbus/TCP**
- Menu : **_MAIN MENU_ → SETTINGS → COMMUNICATION**
- Adresse IP : 192.168.111.32
- Masque : 255.255.255.0

### 3.2 Mise en service de l'exemple
- Dézipper le fichier fourni
- Charger le programme `.lsc` dans LOGO!Soft Comfort
- Transférer vers le module LOGO!
- LOGO! : IP = 192.168.111.3
- Tous les équipements doivent être dans le même sous-réseau

---

## 4. Configuration de la communication

### 4.1 Modbus/TCP

- Protocole client/serveur basé sur TCP/IP (port 502)
- Le PAC3200 agit en **serveur**, LOGO! en **client**
- LOGO! récupère les registres Modbus 2 (tension) et 14 (courant)
- Les données sont des **valeurs flottantes** sur 4 octets

### 4.2 Paramétrage dans LOGO!Soft Comfort

- Menu : **_Tools → Ethernet connections_**
- Ajouter une connexion Modbus
- Définir les propriétés du serveur (PAC3200)
- Adresses Modbus à configurer dans le tableau de transfert

---

## 5. Exemple de programme

- Lecture des données : bloc Modbus → mémoire variable LOGO!
- Conversion via **F/I Converter** (flottant → entier)
- Affichage sur LOGO! TDE
- Déclenchement d'une alarme si la puissance active dépasse un seuil

---

## 6. Annexes

### 6.1 Service et support
- Support en ligne Siemens : [support.industry.siemens.com](https://support.industry.siemens.com)

### 6.2 Liens et documentation

| Réf. | Sujet | Lien |
|------|-------|------|
| \ | Siemens Industry Online Support | https://support.industry.siemens.com |
| \ | Cette documentation | https://support.industry.siemens.com/cs/ww/en/view/109779762 |
| \ | Manuel utilisateur LOGO! 8 | https://support.industry.siemens.com/cs/ww/en/view/109741041 |
| \ | Modules LOGO! | http://www.siemens.com/logo |
| \ | Manuel PAC3200 | https://support.industry.siemens.com/cs/ww/en/view/26504150 |
| \ | Formation en ligne Modbus & LOGO! | https://wbt.siemens.com/sitrain/LOGO-MODBUS_EN/story_html5.html?lms=1 |

### 6.3 Historique des modifications

| Version | Date | Modification |
|---------|------|--------------|
| V1.0 | 07/2020 | Première version |

