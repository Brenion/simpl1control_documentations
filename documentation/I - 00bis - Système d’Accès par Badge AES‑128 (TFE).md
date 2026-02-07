# 🎯 Cahier de Suivi – Système d’Accès par Badge AES‑128 (TFE)

| Élément         | UID | AES128 (clé dérivée) | Key A                 | Key B                 | Access Bits |
| --------------- | --- | -------------------- | --------------------- | --------------------- | ----------- |
| 📇 Carte NFC    | ✅   | ✅ (secteur bloc 1)   | ✅ (bloc 3)            | ✅ (bloc 3)            | ✅ (bloc 3)  |
| 📟 ESP lecteur  | ✅   | ❌ (jamais stockée)   | ✅                     | ❌                     | ❌           |
| 📟 ESP encodeur | ✅   | ❌ (temporaire)       | ✅                     | ✅                     | ✅           |
| 🔐 Backend      | ✅   | ❌ (recalculable)     | ✅ (chiffré ou dérivé) | ✅ (chiffré ou dérivé) | ✅ (stocké)  |

## ✅ Contexte

Prototype de système d'accès par badge RFID (**Mifare Classic 1K**) avec gestion centralisée des accès via MQTT sécurisé.  
Le système est déployé comme **démonstrateur** dans un environnement de test (boîtiers imprimés en 3D). Dans un futur déploiement professionnel certains choix (alimentation, support RFID) pourront évoluer.

## 🏗️ Architecture Générale

- **1 × ESP32 Encodeur** : encode les cartes avec les clés **AES‑128** générées par le backend.
    
- **1 × ESP32 Lecteur Porte** : lit les cartes et demande la validation d'accès via MQTT.
    
- **Backend Fastify** : gestion des utilisateurs, génération des clés, traitement des accès, agrégation des logs.
    
- **Broker MQTT sécurisé** : déjà existant, assure la communication entre les ESP32 et le backend (TLS + authentification).
    

## 📦 Étapes pour mettre en place le système

### 1. Architecture Technique

- **Base de données** : PostgreSQL
    
- **MQTT sécurisé** : TLS + authentification (voir §5 pour la procédure d’activation)
    
- **Stockage chiffré** des clés AES‑128 (pgcrypto ou HashiCorp Vault)
    
- **Alimentation** : modules alimentés via **USB 5 V** (suffisant pour le démonstrateur). Une version industrielle pourra exploiter PoE ou 12 V régulé.
    

### 2. Développement à réaliser

#### 2.1 🛠️ Backend Fastify (REST + MQTT)

- Exposer une **API REST** pour :
    
    1. Créer un employé + mot de passe (hashé Bcrypt)
        
    2. Générer une clé AES‑128 (dérivée via KDF, cf. §5.1) pour une carte
        
    3. Envoyer cette clé à l’encodeur via MQTT
        
- Écouter les **topics MQTT** pour :
    
    1. Recevoir les données lues sur les cartes (`access/reader`)
        
    2. Valider/refuser l’accès et publier la réponse (`access/reader/ack`)
        
- **Logs & traçabilité** : chaque tentative d’accès est enregistrée dans la table `access_logs` (PostgreSQL). Des dashboards seront ajoutés plus tard dans la partie front/monitoring.
    

#### 2.2 Modèle d’autorisations et rôles

| Rôle                     | Capacités principales                                           |
| ------------------------ | --------------------------------------------------------------- |
| **Administrateur**       | CRUD employés, configuration système, lecture complète des logs |
| **Responsable Sécurité** | Génération/révocation de badges, audit des accès                |
| **Employé**              | Consultation de son historique personnel                        |

Les rôles sont portés dans le JWT et contrôlés via des middlewares d’autorisation.

#### 2.3 Tolérance aux pannes & retours utilisateur (ESP32)

- Reconnexion automatique au broker (exponentiel back‑off).
    
- File d’attente locale (SPIFFS) lorsque le broker est indisponible.
    
- **Interface utilisateur :**
    
    - Écran **Nokia 5110 LCD** pour messages courts (_connecté_, _hors‑ligne_, _NFC OK/KO_).
        
    - **LED RGB adressable WS2812** : code couleur (vert = OK, rouge = erreur, bleu = reconnexion, jaune = écriture NFC).
        
- Retries sur l’écriture NFC (3 tentatives) avant alerte.
    

#### 2.4 🛠️ ESP32 Encodeur (R/W)

1. Se connecter au broker MQTT sécurisé
    
2. Attendre une commande d’encodage sur `access/encoder`
    
3. **Personnaliser les clés A & B** des secteurs utilisateur du badge :
    
    - La clé **A_encodeur** est connue seulement de l’encodeur et du backend (droits : `write|read`).
        
    - La clé **B_lecteur** est lue par le lecteur, droits : `read only`.
        
    - Les bits d’accès sont configurés en conséquence (`C1C2C3 = 100`).
        
4. Écrire la clé AES‑128 dérivée via KDF dans un bloc de données (crypté).
    
5. Confirmer à `access/encoder/ack` avec l’UID et le statut.
    

#### 2.5 🛠️ ESP32 Lecteur Porte (Read‑only)

1. Lire la clé via le _NFC Module V3_ en utilisant **B_lecteur** (lecture seule).
    
2. Publier la clé lue + UID sur `access/reader`
    
3. Attendre la réponse du backend (accès autorisé ou refusé). Le lecteur ne possède jamais **A_encodeur**, empêchant toute écriture.
    

### 3. Prototype Physique (Version Finale pour démonstrateur)

- **Modélisation CAO** des boîtiers (Fusion 360, SolidWorks ou FreeCAD) avant impression.
    
- Deux boîtiers séparés : un pour l’encodeur, un pour le lecteur.
    
- Impression 3D PLA (couleur distincte pour différencier encodeur/lecteur).
    
- Intégration ESP32 + MFRC522 + LED WS2812 + LCD 5110 dans chaque boîtier.
    
- **Alimentation USB 5 V** via adaptateur secteur ou batterie externe.
    

### 4. Tests Fonctionnels (scénarios de badges). Tests Fonctionnels (scénarios de badges)

|Cas|Description|Résultat attendu|
|---|---|---|
|1|**Carte inconnue** (UID non enregistré)|Refus, code RED, log _unknown_card_|
|2|**Carte employé**|Refus (porte secure), code YELLOW, log _employee_denied_|
|3|**Carte admin**|Autorisé, code GREEN, log _admin_granted_|
|4|**Carte développeur**|Autorisé, code GREEN, log _dev_granted_|

### 5. Sécurité avancée

- **Pas d'identifiant personnel** stocké sur la carte : seulement une clé AES‑128 dérivée.
    
- **Clés A & B personnalisées** : lecteur incapable d’écrire.
    
- **TLS sur MQTT** : Activé via certificat auto‑signé ou AC interne (mosquitto > 2.0, cf. procédure ci‑dessous).
    
- Séparation stricte des droits encodeur (_write_) / lecteur (_read_).
    

#### 5.1 KDF : dérivation de la clé badge

```text
master_key      -- secret 256 bits (stocké en Vault)
card_uid        -- 4 ou 7 octets
purpose         -- "BADGE_KEY"

Per-card AES‑128 = Truncate_128bits( HKDF( SHA‑256, master_key, card_uid || purpose ) )
```

- Pourquoi ? Une fuite d’une clé badge n’expose pas le master_key.
    
- Implémentation : bibliothèque `crypto` native Node.js (backend) et `mbedtls` côté encodeur pour vérification optionnelle.
    

#### 5.2 Mise en place TLS sur Mosquitto (résumé)

1. Générer CA, serveur.crt et serveur.key (OpenSSL).
    
2. Config `mosquitto.conf` :
    
    ```
    listener 8883
    cafile /etc/mosquitto/ca.crt
    certfile /etc/mosquitto/server.crt
    keyfile /etc/mosquitto/server.key
    require_certificate true
    ```
    
3. Redémarrer le service et tester avec `mosquitto_pub` / `mosquitto_sub` + option `--tls-cert`.
    

### 6. Recette & Critères d’acceptation

|#|Critère|Condition de succès|
|---|---|---|
|R‑01|Création employé|L’API retourne _201 Created_ et l’employé apparaît en DB|
|R‑02|Encodage badge|Clés A & B modifiées, clé dérivée écrite, événement loggé|
|R‑03|Scénarios badges|Les 4 cas de §4 respectent le résultat attendu, temps < 300 ms|
|R‑04|Perte broker|L’ESP32 se reconnecte en < 60 s, file d’attente vidée|

### 7. Bill of Materials (BOM)

|Composant|Référence|Qté|Remarques|
|---|---|---|---|
|MCU|**ESP32‑DEVKIT‑C**|2|Wi‑Fi + MQTT|
|Module RFID|**HW‑147 (MFRC522)**|2|Interface SPI|
|Écran LCD|**Nokia 5110 LCD**|2|84×48 px|
|LED RGB|**WS2812 (NeoPixel) × 1**|2|Code couleur état|
|Boîtier|Impression 3D PLA|2|Modèle personnalisé|
|Alimentation|Adaptateur **USB 5 V 1 A**|2|Démonstrateur|
|Divers|Câblage Dupont, vis M3|—|—|

---

> _Monitoring_ : dashboards Grafana/Loki seront intégrés ultérieurement dans la partie front et ne font pas partie du périmètre immédiat du TFE.

### 8. Planning TFE – échéance 25 mai 2025

#### 8.1 Charges estimées (hors modélisation CAO)

|Phase / Sous‑tâche|Action humaine restant à faire|Charge réaliste|
|---|---|---|
|Backend – endpoints badge + HKDF|Ajouter tables `badges` & `access_logs`, KDF, 2 routes, insert logs|**8 h**|
|Backend – topics MQTT + validations|Souscrire `access/reader`, publier ACK, mapping rôles/cas|**4 h**|
|Certificats clients & ACL Mosquitto|Générer CA + 2 certs (encodeur/lecteur), mettre ACL minimaliste|**3 h**|
|Firmware ESP32 Encodeur|MFRC522 init (HW‑147), écriture clé dérivée, modif clés A/B, LED/LCD|**10 h**|
|Firmware ESP32 Lecteur|Lecture clé+UID, publish, attente ACK, LED/LCD|**7 h**|
|Tolérance pannes (reconnect + cache)|Implémenter file SPIFFS + back‑off sur firmwares|**4 h**|
|Assemblage & câblage boîtiers|Lancer impression, montage, tests mécaniques|**1 h**|
|Tests scénarios (4 cas) + logs|Exécuter, corriger, remplir tableau|**3 h**|
|Documentation finale|MAJ cahier, captures, BOM, export PDF|**3 h**|
|**Total**||**43 h**|

#### 8.2 Répartition selon disponibilités (44 h planifiées)

|Date|Jour|Objectif du créneau|Durée (h)|Cumul|
|---|---|---|---|---|
|12/05|**Lun**|Backend : tables SQL, logique HKDF (part 1)|3|3|
|14/05|**Mer**|Backend : routes `/badges`, tests Postman (part 2)|3|6|
|16/05|**Ven**|MQTT backend : subscribe/publish, mapping cas|3|9|
|17/05|**Sam**|Firmware encodeur : MFRC522 init (HW‑147), écriture clé dérivée, modif clés A/B, LED/LCD|5|14|
|18/05|**Dim**|Finaliser encodeur + début firmware lecteur (lecture + publish)|8|22|
|19/05|**Lun**|Lecteur : attente ACK, feedback LED/LCD|3|25|
|21/05|**Mer**|Génération CA, certs encodeur/lecteur, ACL mosquitto 8883|3|28|
|23/05|**Ven**|Reconnect/cache SPIFFS (2 firmwares) + scénarios badges|3|31|
|24/05|**Sam**|Assemblage boîtiers, collecte photos, corrections mineures|5|36|
|25/05|**Dim**|Documentation finale, export PDF, freeze repo, tampon imprévus|8|44|

> _Monitoring : dashboards Grafana/Loki restent hors périmètre immédiat du TFE._ : dashboards Grafana/Loki restent hors périmètre immédiat du TFE.*

#### 8.3 Étapes opérationnelles (sans contrainte de date)

1. **Préparer la base et la dérivation de clés**
    
    - Créer les tables `badges` et `access_logs`.
        
    - Stocker `MASTER_KEY` dans `.env` ou Vault.
        
    - Implémenter la fonction HKDF `deriveBadgeKey(uid)` avec test unitaire.
        
2. **Exposer les endpoints "badges"**
    
    - Route `POST /badges` : reçoit UID et rôle, calcule la clé dérivée, insère en DB, renvoie la clé.
        
    - Prévoir deux badges de test (admin, employé).
        
3. **Brancher le backend sur MQTT**
    
    - Souscrire `access/reader`, vérifier UID + clé.
        
    - Publier `access/reader/ack` (_granted_ / _denied_).
        
    - Insérer chaque tentative dans `access_logs`.
        
4. **Développer le firmware de l’encodeur (ESP32 + MFRC522)**
    
    - Câbler le module **HW‑147 (MFRC522)** en SPI.
        
    - Lire l’UID d’une carte vierge, le publier au backend.
        
    - Recevoir la clé dérivée ; écrire la clé dans un bloc, modifier les clés A/B (`MIFARE_SetAccessBits`).
        
    - Afficher le statut sur l’écran Nokia 5110 et la LED WS2812.
        
5. **Développer le firmware du lecteur (ESP32 + MFRC522)**
    
    - Lire UID + clé dérivée (lecture seule, clé B).
        
    - Publier au backend, attendre l’ACK.
        
    - LED verte/rouge + LCD « ACCÈS / REFUS ».
        
6. **Mettre en place la sécurité Mosquitto**
    
    - Générer CA + certificats encodeur & lecteur.
        
    - Activer le listener 8883 avec `require_certificate true` pour ces appareils.
        
7. **Résilience**
    
    - Ajouter file SPIFFS et reconnexion exponentielle dans les deux firmwares.
        
    - LED bleue / LCD « Reconnect » si broker indisponible.
        
8. **Assembler le prototype physique**
    
    - Imprimer les boîtiers, intégrer ESP32, MFRC522, LED WS2812, LCD 5110.
        
    - Alimenter le tout en USB 5 V.
        
9. **Tester les scénarios de badge**
    
    1. Carte inconnue → refus
        
    2. Carte employé → refus
        
    3. Carte admin → accès
        
    4. Carte développeur → accès
        
    
    - Vérifier LED, LCD et entrées `access_logs`.
        
10. **Documenter et livrer**
    
    - Captures Postman, photos du montage, tableau de résultats.
        
    - Mettre à jour le cahier, exporter PDF, tag `v1.0‑TFE`.