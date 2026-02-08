---
tags: [index, navigation, table-des-matieres]
created: 2026-02-08
---

# Table des Matières - Documentation Domotique

## Installation & Configuration de base

| Fichier | Description |
|---------|-------------|
| [[A - Installation et Test de Mosquitto (MQTT) sur un Raspberry Pi]] | Installation Mosquitto, tests, MQTT Explorer, ESP32 basique |
| [[B - Documentation du Backend]] | Backend Fastify, TypeORM, PostgreSQL, JWT, WebSocket |

## Communication IoT & MQTT

| Fichier | Description |
|---------|-------------|
| [[C - MQTT, ESP32 et Backend]] | Communication ESP32/Backend via MQTT, entités |
| [[D - MQTT entre siemens logo et backend]] | Configuration LOGO, communication Modbus TCP |

## Automates & Modbus

| Fichier | Description |
|---------|-------------|
| [[E - le modbus]] | Modbus TCP, compteur énergie Schneider |
| [[F - Gestion du chauffage sur le logo]] | Algorithme chauffage LOGO |
| [[G - Implémentation du Modbus dans le projet Fastify (abandonné)]] | Version abandonnée |

## Système d'Accès RFID & Badges

| Fichier | Description |
|---------|-------------|
| [[H - Système d'accès RFID]] | Concepts RFID, MFRC522, architecture |
| [[I - 00 - Plan d'action complet – Système de badge AES128 avec backend Fastify et tests réels]] | Plan global badges AES128 |
| [[I - 00bis - Système d'Accès par Badge AES‑128 (TFE)]] | Spécifications TFE |
| [[I - 01 - Gestion des badges et accessLog (Entités et Seeder)]] | Entités TypeORM, seeders |
| [[I - 02 - Dérivation de clé AES128 avec HKDF (HKDF-SHA256)]] | Cryptographie HKDF |
| [[I - 02 -PasswordService – Fiche Technique (revenir dessus)]] | Service mot de passe |
| [[I - 03 - API backend - POST badges]] | API création badges |
| [[I - 04 - Intégration MQTT Backend - Lecteur de badge (access-reader)]] | Topic access/reader |
| [[I - 05 - Mise en œuvre technique – Encodeur ESP32]] | ESP32 complet : WiFi, TLS, MQTT, RFID |
| [[I - 05 bis - Intégration du Port MQTT Sécurisé (TLS) dans le Backend]] | Backend double port 1883/8883 |

## Production & Déploiement

| Fichier | Description |
|---------|-------------|
| [[J - mise en production]] | Notes brutes de production |
| [[K - Récapitulatif production]] | Résumé compact : compose, scripts |
| [[L - Guide de Mise en Production]] | **Guide complet** : SSH, Docker, TLS, Mosquitto, scripts |

## Résumé

| Fichier | Description |
|---------|-------------|
| [[W - RÉSUMÉ]] | Vue d'ensemble du projet |

## Annexes

| Fichier | Description |
|---------|-------------|
| [[Z - Annexe 1 - MoSCoW]] | Priorisation MoSCoW |
| [[Z - Annexe 2 - Retro planning]] | Planning rétrospectif |
| [[Z - Annexe 3 - objectif lecteur de carte]] | Objectifs lecteur |
| [[Z - annexe 4 - tableau_final_heures_minutes pour algo LOGO]] | Tableau heures LOGO |
| [[Z - annexe 5 - Contrôle d'accès – Spécification fonctionnelle]] | Specs contrôle accès |
| [[Z - annexe 6 - Structure backend]] | Structure du backend |

---

## Navigation par Thème

### SSH
- [[L - Guide de Mise en Production#1.1 Configuration SSH]] - Configuration complète
- [[J - mise en production]] - Notes originales

### Certificats TLS
- [[L - Guide de Mise en Production#2. Génération des Certificats TLS]] - Guide complet CA + serveur + clients
- [[I - 05 - Mise en œuvre technique – Encodeur ESP32#2.2.1 Génération du certificat de l'autorité (CA)]] - Détails certificats
- [[I - 05 bis - Intégration du Port MQTT Sécurisé (TLS) dans le Backend]] - Backend TLS

### MQTT / Mosquitto
- [[A - Installation et Test de Mosquitto (MQTT) sur un Raspberry Pi#1. Installation de Mosquitto sur le Raspberry Pi]] - Installation de base
- [[L - Guide de Mise en Production#3. Configuration Mosquitto]] - Configuration production (1883 + 8883)

### Docker
- [[L - Guide de Mise en Production#1.3 Installation de Docker]] - Installation
- [[L - Guide de Mise en Production#1.4 Création du registre privé Docker]] - Registre privé
- [[L - Guide de Mise en Production#6. Docker Compose Production]] - Compose production
- [[K - Récapitulatif production]] - Résumé scripts

### ESP32
- [[I - 05 - Mise en œuvre technique – Encodeur ESP32#1. Préparation matérielle]] - Brochage complet
- [[I - 05 - Mise en œuvre technique – Encodeur ESP32#2.3 Intégration MQTT TLS uniquement]] - Code TLS complet
- [[L - Guide de Mise en Production#8. Intégration ESP32]] - Résumé production
- [[A - Installation et Test de Mosquitto (MQTT) sur un Raspberry Pi#7. Préparation de l'ESP32 pour l'envoi de messages MQTT]] - Code basique (port 1883)

### PostgreSQL / Backend
- [[B - Documentation du Backend]] - Documentation complète
- [[L - Guide de Mise en Production#4. Configuration Backend]] - Variables production
- [[L - Guide de Mise en Production#6.2 Fichier production/db.env]] - Credentials DB

---

## Annexes Techniques (dossier séparé)

Voir le dossier `/annexe a la documentation/` :
- Commande SSL-TLS
- Architecture MQTT Topics
- ws et wss
- fortigate
- LOGO_Modbus_TCP_FR
- LOGO_Functional_Blocks
- Tutoriel Logo
- Intégration Zigbee
