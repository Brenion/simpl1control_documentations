# Mise en œuvre technique – Encodeur ESP32

## 1. Préparation matérielle

- **ESP32-WROOM-32 (carte principale)**
    
    - **Modèle** : ESP32-WROOM-32
        
    - **Type** : Microcontrôleur Wi-Fi + Bluetooth double cœur (SoC ESP32-D0WDQ6)
        
    - **Mémoire** :
        
        - RAM : 520 KB SRAM
            
        - Flash externe : typiquement 4 Mo
            
    - **Connectivité** :
        
        - Wi-Fi 802.11 b/g/n (2,4 GHz)
            
        - Bluetooth 4.2 BR/EDR + BLE
            
    - **GPIO disponibles** : environ 25 selon brochage
        
    - **Alimentation** : 5 V via USB, régulateur intégré pour 3,3 V
        
- **MFRC522 câblé en SPI**
    
    - **Interface** : SPI
        
    - **Brochage ESP32 ↔ MFRC522** :
        
        |MFRC522 Pin|ESP32 Pin|Fonction|
        |---|---|---|
        |SDA (SS)|GPIO 5|Chip Select|
        |SCK|GPIO 18|Horloge SPI|
        |MOSI|GPIO 23|Données vers le lecteur|
        |MISO|GPIO 19|Données depuis le lecteur|
        |IRQ|Non utilisé|(optionnel)|
        |GND|GND|Masse|
        |RST|GPIO 22|Reset module|
        |3.3 V|3.3 V|Alimentation|
        
    - **Bibliothèque** : [MFRC522](https://github.com/miguelbalboa/rfid)
        
- **LED WS2812 sur GPIO 4**
    
    - **Brochage** :
        
        |WS2812 Pin|ESP32 Pin|Fonction|
        |---|---|---|
        |DIN|GPIO 4|Donnée|
        |GND|GND|Masse|
        |5 V|5 V|Alimentation|
        
    - **Bibliothèque** : [Adafruit_NeoPixel](https://github.com/adafruit/Adafruit_NeoPixel)
        
    - **Initialisation typique** :
        
        ```cpp
        Adafruit_NeoPixel strip(1, 4, NEO_GRB + NEO_KHZ800);
        ```
        

## 2. Configuration réseau

Ce chapitre couvre :

1. **Connexion WiFi** de l’ESP32
    
2. **Configuration du broker Mosquitto**
    
3. **Intégration MQTT** dans le code ESP32
    

---

### 2.1 Connexion WiFi

**Objectif** : connecter l’ESP32 au réseau local.

```cpp
#include <WiFi.h>
const char* ssid     = "<TON_SSID>";
const char* password = "<TON_MOT_DE_PASSE>";
void setup_wifi() {
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) { delay(500); }
}
```

### 2.2 Configuration du broker Mosquitto

#### 2.2.1 Génération du certificat de l’autorité (CA)

- **Étape 1 : Génération de la clé privée CA**
    
    ```
    sudo openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:4096 -out ca.key
    ```
    
- **Étape 2 : Création du certificat auto-signé CA**
    
    ```
    sudo openssl req -new -x509 -days 3650 -key ca.key -out ca.crt -subj "/CN=MyMosquittoCA"
    ```
    
- **Étape 3 : Déploiement du certificat CA**
    
    ```
    sudo mkdir -p /etc/mosquitto/certs
    sudo cp ca.crt /etc/mosquitto/certs/
    sudo chown mosquitto:mosquitto /etc/mosquitto/certs/ca.crt
    sudo chmod 640 /etc/mosquitto/certs/ca.crt
    ```
    

#### 2.2.2 Génération du certificat serveur (avec SAN) 

  
Pour que les clients valident correctement le certificat, l’adresse IP fixe ou le nom DNS du broker doit figurer dans le SAN.

- **Étape 1 : Création du fichier SAN**
    
    ```
  
    ```
    
- **Étape 2 : Clé privée serveur & CSR**
    
    ```
    sudo openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out server.key
    sudo openssl req -new -key server.key -out server.csr -config san.cnf
    ```
    
- **Étape 3 : Signature du certificat**
    
    ```
    sudo openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key \
      -CAcreateserial -out server.crt -days 365 \
      -extensions req_ext -extfile san.cnf
    ```
    
- **Étape 4 : Déploiement et sécurisation**

    ```
    sudo cp ca.crt server.crt server.key /etc/mosquitto/certs/ sudo chown mosquitto:mosquitto /etc/mosquitto/certs/ca.crt  
	/etc/mosquitto/certs/server.crt /etc/mosquitto/certs/server.key sudo chmod 640 /etc/mosquitto/certs/ca.crt  
	/etc/mosquitto/certs/server.crt /etc/mosquitto/certs/server.key
	```


#### 2.2.3 Génération du certificat client (port 8883)

```bash
cd /etc/mosquitto/certs/sudo 
# Par exemple pour l'ESP32 encodeur
sudo openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out client-esp-encodeur.key
sudo openssl req -new -key client-esp-encodeur.key \
  -out client-esp-encodeur.csr -subj "/CN=esp32-encodeur"
sudo openssl x509 -req -in client-esp-encodeur.csr \
  -CA ca.crt -CAkey ca.key -CAcreateserial \
  -out client-esp-encodeur.crt -days 365
```

_Répétez pour chaque client en changeant ******`/CN=...`******._

#### 2.2.4 Gestion des utilisateurs et mots de passe

```bash
sudo mosquitto_passwd -c /etc/mosquitto/passwd <utilisateur>
sudo mosquitto_passwd /etc/mosquitto/passwd <autre_utilisateur>
```

Intégrer dans `mosquitto.conf` :

```conf
password_file /etc/mosquitto/passwd
```

#### 2.2.5 Fichier `/etc/mosquitto/mosquitto.conf`

```conf
listener 1883
allow_anonymous true
listener 8883
protocol mqtt
cafile /etc/mosquitto/ca_certificates/ca.crt
certfile /etc/mosquitto/ca_certificates/server.crt
keyfile /etc/mosquitto/ca_certificates/server.key
require_certificate true
use_identity_as_username true
password_file /etc/mosquitto/passwd
```

#### 2.3 Intégration MQTT TLS uniquement

Cette section explique, étape par étape et en langage clair, comment connecter votre ESP32 à un broker MQTT via TLS (port 8883) en utilisant **PROGMEM** pour stocker vos certificats.

---

##### 2.3.1 Étape 1 : Préparer et intégrer les certificats avec PROGMEM

Vous avez besoin de trois fichiers PEM :

- `ca.crt` : certificat de l’autorité de certification
    
- `client.crt` : certificat de l’ESP32
    
- `client.key` : clé privée de l’ESP32
    

**Objectif :** convertir ces fichiers PEM en tableaux C et les inclure directement dans le firmware de l’ESP32.

1. Ouvrez un terminal dans le dossier de votre projet Arduino.
    
2. Exécutez les commandes suivantes :
    
    ```bash
    xxd -i ca.crt    > ca_crt.h
    xxd -i client.crt > client_crt.h
    xxd -i client.key > client_key.h
    ```
    
3. Déplacez `ca_crt.h`, `client_crt.h` et `client_key.h` dans le même dossier que votre sketch `.ino`.
    
4. Dans votre code, incluez ces headers :
    

> **Pourquoi PROGMEM ?**
> 
> - Stockage en **mémoire flash**, sans partition supplémentaire.
>     
> - Pas d’overhead de système de fichiers ni d’utilisation de RAM lors de la lecture.
>     

---

##### 2.3.2 Étape 2 : Installer le support ESP32 et les bibliothèques nécessaires

1. **Ajouter le gestionnaire de cartes ESP32** :
    
    - Dans l’IDE Arduino, ouvrez **Fichier > Préférences**.
        
    - Dans **URL de gestionnaire de cartes supplémentaires**, ajoutez :
        
        ```
        https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
        ```
        
2. **Installer le package ESP32** :
    
    - Allez dans **Outils > Type de carte > Gestionnaire de cartes**, recherchez **esp32** et installez **ESP32 by Espressif Systems**.
        
    - Sélectionnez votre modèle d’ESP32 dans **Outils > Type de carte** (par ex. « ESP32 Dev Module »).
        
3. **Vérifier les bibliothèques intégrées** :
    
    - Le support ESP32 inclut automatiquement **WiFi** et **WiFiClientSecure**.
        
    - Vous n’avez pas besoin d’installer `WiFiClientSecure` manuellement.
        
4. **Installer PubSubClient** :
    
    - Ouvrez **Croquis > Inclure une bibliothèque > Gérer les bibliothèques…**, recherchez **PubSubClient** et installez-la.
        

---

##### 2.3.3 Étape 3 : Code complet avec PROGMEM

```cpp
#include <WiFi.h>
#include <WiFiClientSecure.h>
#include <PubSubClient.h>

// Include des certificats générés
#include "ca_crt.h"
#include "client_esp_encodeur_crt.h"
#include "client_esp_encodeur_key.h"

// Paramètres Wi-Fi
const char* ssid       = "*******************";
const char* password   = "***********";

// Paramètres MQTT
const char* mqtt_server   = "192.168.0.102";
const int   mqtt_port     = 8883;
const char* mqtt_user     = "admin";
const char* mqtt_password = "*****************";
const char* topic         = "test/esp32";

// Client sécurisé et MQTT
WiFiClientSecure secureClient;
PubSubClient        client(secureClient);

// Fonction de connexion au Wi-Fi
void setup_wifi() {
    delay(10);
    Serial.println("Connexion au WiFi...");
    WiFi.begin(ssid, password);
    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
    }
    Serial.println("WiFi connecté");
    Serial.print("Adresse IP: ");
    Serial.println(WiFi.localIP());
}

// Fonction de reconnexion au broker MQTT
void reconnect() {
    int attempts = 0;
    while (!client.connected() && attempts < 10) {
        Serial.println("Connexion au broker MQTT...");
        if (client.connect("ESP32Client", mqtt_user, mqtt_password)) {
            Serial.println("Connecté au broker MQTT");
        } else {
            Serial.print("Échec, rc=");
            Serial.print(client.state());
            Serial.println(" Nouvelle tentative dans 5 secondes...");
            delay(5000);
            attempts++;
        }
    }
    if (attempts >= 10) {
        Serial.println("Impossible de se connecter au broker MQTT après 10 tentatives.");
    }
}

void setup() {
    Serial.begin(115200);
    setup_wifi();

    // Charger les certificats
    secureClient.setCACert((const char*)ca_crt);
    secureClient.setCertificate((const char*)client_esp_encodeur_crt);
    secureClient.setPrivateKey((const char*)client_esp_encodeur_key);

    // Configuration du serveur MQTT
    client.setServer(mqtt_server, mqtt_port);
}

void loop() {
    if (!client.connected()) {
        reconnect();
    }
    client.loop();

    // Publier un message toutes les 5 secondes
    static unsigned long lastMsg = 0;
    if (millis() - lastMsg > 5000) {
        lastMsg = millis();
        const char* message = "Coucou je suis ESP32, es-tu là?";
        Serial.print("Envoi du message: ");
        Serial.println(message);
        if (client.connected()) {
            client.publish(topic, message);
        } else {
            Serial.println("Impossible d'envoyer le message, non connecté au broker MQTT.");
        }
    }
}
```

---

##### 2.3.4 Explications simples

- **PROGMEM** stocke directement vos certificats en **mémoire flash**, sans partition ni système de fichiers.
    
- **WiFiClientSecure** gère le chiffrement TLS.
    
- **client.loop()** maintient la connexion active et traite les messages.
    

## 3. Dump complet carte vierge

### Installation de la bibliothèque RFID

Avant d’utiliser le code, tu dois installer la bibliothèque nécessaire pour gérer le lecteur RFID PN532 :

1. Ouvre l’IDE Arduino.
    
2. Va dans le menu _Sketch → Include Library → Manage Libraries…_
    
3. Recherche **Adafruit PN532**.
    
4. Installe la bibliothèque officielle **Adafruit PN532** (publiée par Adafruit).
    

Cette section explique clairement comment configurer ton ESP32 pour effectuer un dump complet d'une carte RFID MIFARE Classic, en affichant toutes les données lues directement sur le moniteur série. Cette partie est destinée uniquement aux phases de test et ne sera plus utilisée en fonctionnement normal du système.

### Tester la lecture complète de la carte

- Ouvre le moniteur série (_Tools → Serial Monitor_) à **115200 baud**.
    
- Approche ta carte RFID près du lecteurPN532.
    
- Le moniteur série affichera :
    
    - Le type exact de carte détectée.
        
    - Toutes les données, clés A et B comprises, secteur par secteur et bloc par bloc, clairement structurées.
        

Voici un exemple concret de l'affichage attendu :

```
Secteur 0
Bloc 0: 04 D3 2A 6B 9C 34 80 00 14 A2 34 12 01 02 03 04
Bloc 1: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
...
```

Chaque ligne indique précisément le contenu d'un bloc spécifique de la carte RFID.### Fonctionnement du code Arduino

Voici le code Arduino complet intégrant la lecture RFID à ton code MQTT existant, sans en modifier le fonctionnement actuel :

```cpp
#include <SPI.h>
#include <WiFi.h>
#include <WiFiClientSecure.h>
#include <PubSubClient.h>
#include <Adafruit_PN532.h>

// Include des certificats générés
#include "ca_crt.h"
#include "client_esp_encodeur_crt.h"
#include "client_esp_encodeur_key.h"
#include "Arduino.h"

// Paramètres Wi-Fi
const char* ssid       = "**************";
const char* password   = "***********";

// Paramètres MQTT
const char* mqtt_server   = "192.168.0.102";
const int   mqtt_port     = 8883;
const char* mqtt_user     = "admin";
const char* mqtt_password = "*****************";
const char* topic         = "test/esp32";

// Pins SPI pour PN532
#define SCK_PIN 18
#define MISO_PIN 19
#define MOSI_PIN 23
#define SS_PIN 5

// Initialisation du PN532 en mode SPI
Adafruit_PN532 nfc(SS_PIN);

// Client sécurisé et MQTT
WiFiClientSecure secureClient;
PubSubClient client(secureClient);

// Fonction de connexion au Wi-Fi
void setup_wifi() {
    delay(10);
    Serial.println("Connexion au WiFi...");
    WiFi.begin(ssid, password);
    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
    }
    Serial.println("WiFi connecté");
    Serial.print("Adresse IP: ");
    Serial.println(WiFi.localIP());
}

// Fonction de reconnexion au broker MQTT
bool reconnect() {
    static unsigned long lastAttemptTime = 0;
    static int attempts = 0;

    if (client.connected()) {
        return true; // Déjà connecté
    }

    unsigned long now = millis();
    if (now - lastAttemptTime > 5000 || lastAttemptTime == 0) {
        lastAttemptTime = now;
        Serial.println("Connexion au broker MQTT...");

        // Générer un ID client unique
        String clientId = "ESP32Client_" + String(random(0xffff));

        // Convertir en const char* pour la méthode connect()
        if (client.connect(clientId.c_str(), mqtt_user, mqtt_password)) {
            Serial.println("Connecté au broker MQTT");
            attempts = 0; // Réinitialiser les tentatives
            return true;
        } else {
            Serial.print("Échec, rc=");
            Serial.print(client.state());
            Serial.println(" Nouvelle tentative dans 5 secondes...");
            attempts++;
        }
    }

    if (attempts >= 10) {
        Serial.println("Trop de tentatives de reconnexion. Abandon.");
        return false;
    }

    return false;
}

void CardReaderInit()
{
    // Initialisation du PN532
    Serial.println("Initialisation du PN532...");
    nfc.begin();

    uint32_t versiondata = nfc.getFirmwareVersion();
    if (!versiondata)
    {
        Serial.println("Impossible de trouver le PN532. Vérifiez les connexions !");
        while (1)
            ; // Boucle infinie
    }

    // Afficher les informations du firmware
    Serial.print("Version du firmware PN532 : ");
    Serial.println((versiondata >> 16) & 0xFF, HEX);

    // Configurer le PN532 pour lire les cartes
    nfc.SAMConfig();
    Serial.println("PN532 prêt. Placez une carte sur le lecteur.");
}

void readCardDump()
{
    uint8_t success;
    uint8_t uid[] = { 0, 0, 0, 0, 0, 0, 0 }; // UID de la carte
    uint8_t uidLength;

    // Essayer de lire une carte
    success = nfc.readPassiveTargetID(PN532_MIFARE_ISO14443A, uid, &uidLength);

    if (success) {
        String message = "UID de la carte : ";
        Serial.print(message);
        for (uint8_t i = 0; i < uidLength; i++) {
            Serial.print(" 0x");
            Serial.print(uid[i], HEX);
        }
        Serial.println();

        if (client.connected()) {
            for (uint8_t i = 0; i < uidLength; i++) {
                message += String(uid[i], HEX) + " ";
            }
            client.publish(topic, message.c_str());
            Serial.println("UID publié sur MQTT.");
        }
    } else {
        Serial.println("Aucune carte détectée.");
    }
}

void setup() {
    Serial.begin(115200);
    SPI.begin(SCK_PIN, MISO_PIN, MOSI_PIN, SS_PIN);

    CardReaderInit();
    setup_wifi();

    secureClient.setCACert((const char*)ca_crt);
    secureClient.setCertificate((const char*)client_esp_encodeur_crt);
    secureClient.setPrivateKey((const char*)client_esp_encodeur_key);
    
    client.setServer(mqtt_server, mqtt_port);
}

void loop() {
    // Vérifiez si le Wi-Fi est connecté
    if (WiFi.status() != WL_CONNECTED) {
        Serial.println("WiFi déconnecté. Reconnexion...");
        setup_wifi(); // Reconnectez au Wi-Fi si nécessaire
    }

    // Vérifiez si le client MQTT est connecté
    if (!client.connected()) {
        static unsigned long lastReconnectAttempt = 0;
        unsigned long now = millis();

        // Essayez de vous reconnecter toutes les 10 secondes
        if (now - lastReconnectAttempt > 10000) {
            lastReconnectAttempt = now;
            if (reconnect()) {
                lastReconnectAttempt = 0; // Réinitialisez le compteur si la reconnexion réussit
            }
        }
    } else {
        // Si connecté, maintenez la connexion MQTT
        client.loop();
    }

    // Lire les données de la carte RFID
    readCardDump();
}
```

### Tester la lecture complète de la carte

- Ouvre le moniteur série (_Tools → Serial Monitor_) configuré à **115200 baud**.
    
- Approche une carte RFID du lecteur.
    
- Toutes les données s'afficheront clairement dans le moniteur série.
    

### Prochaine étape

La sauvegarde de ces informations dans un fichier texte (`.txt`) pour consultation ultérieure.
    

## 4. Génération backend des données de badge

- Enregistrer UID
    
- Générer blocs 8–11 (AES, type, HMAC, trailer)
    

## 5. Publication backend MQTT

- Topic `access/encoder`
    

## 6. Déploiement du code ESP32

- WiFi
    
- MQTT (1883+8883)
    
- Détection MFRC522
    
- Lecture UID
    
- Écriture blocs
    
- WS2812
    
- ACK
    

## 7. Validation avec lecteur ESP32

- Scanner et vérifier
    

## 8. Tests de scénario

- Maître / utilisateur / erreurs / restauration
    

---

## Mémento : Génération des certificats clients

**Pour chaque client, répétez avec CN unique**

**ESP32 Encodeur**

```bash
sudo openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out client-esp-encodeur.key
sudo openssl req -new -key client-esp-encodeur.key -out client-esp-encodeur.csr -subj "/CN=esp32-encodeur"
sudo openssl x509 -req -in client-esp-encodeur.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out client-esp-encodeur.crt -days 365 -extfile san.cnf -extensions v3_req
```

**ESP32 Lecteur**

```bash
sudo openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out client-esp-lecteur.key
sudo openssl req -new -key client-esp-lecteur.key -out client-esp-lecteur.csr -subj "/CN=esp32-lecteur"
sudo openssl x509 -req -in client-esp-lecteur.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out client-esp-lecteur.crt -days 365 -extfile san.cnf -extensions v3_req
```

**Backend NodeJS**

```bash
sudo openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out client-backend.key
sudo openssl req -new -key client-backend.key -out client-backend.csr -subj "/CN=backend-nodejs"
sudo openssl x509 -req -in client-backend.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out client-backend.crt -days 365 -extfile san.cnf -extensions v3_req

```

**MQTT Explorer**

```bash
sudo openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out client-mqexplorer.key
sudo openssl req -new -key client-mqexplorer.key -out client-mqexplorer.csr -subj "/CN=mqtt-explorer"
sudo openssl x509 -req -in client-mqexplorer.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out client-mqexplorer.crt -days 365 -extfile san.cnf -extensions v3_req
```

|Client|Clé privée|Certificat client|
|---|---|---|
|ESP32 Encodeur|client-esp-encodeur.key|client-esp-encodeur.crt|
|ESP32 Lecteur|client-esp-lecteur.key|client-esp-lecteur.crt|
|Backend NodeJS|client-backend.key|client-backend.crt|
|MQTT Explorer|client-mqexplorer.key|client-mqexplorer.crt|

Chaque client doit obtenir : sa clé privée, son certificat, et le fichier commun `ca.crt`.



doc a modifier avec ceci 

# Documentation TLS Mosquitto – Architecture Domotique

## Objectif

Sécuriser les communications MQTT entre les éléments suivants via TLS :

- **Serveur Mosquitto** sur Raspberry Pi (192.168.0.102)
    
- **ESP32 Lecteur de carte** (client 1)
    
- **ESP32 Enregistreur** (client 2)
    
- **Backend Node.js** (client 3)
    
- **MQ Explorer** (client 4)
    

---

## 1. Prérequis

- Raspberry Pi avec Mosquitto installé
    
- OpenSSL installé (par défaut sur Raspberry Pi)
    
- Accès SSH au Raspberry Pi
    

### 1.1 Activer l'accès SSH au Raspberry Pi

1. Insérer la carte microSD du Raspberry Pi dans votre ordinateur.
    
2. Ouvrir la partition `boot`.
    
3. Créer un fichier vide nommé `ssh` (sans extension) à la racine de la partition `boot` :
    
    ```bash
    touch /Volumes/boot/ssh
    ```
    
4. Insérer la carte SD dans le Raspberry Pi et démarrer.
    
5. Se connecter en SSH depuis un terminal :
    
    ```bash
    ssh pi@192.168.0.102
    # mot de passe par défaut : raspberry
    ```
    

> L'accès SSH permet de configurer le serveur Mosquitto, générer les certificats avec OpenSSL, transférer les fichiers vers les clients et surveiller les logs. C'est la méthode centrale pour administrer le Raspberry Pi à distance sans interface graphique.

---

## 2. Génération des certificats

### 2.1. Créer une autorité de certification (CA)

```bash
cd /etc/mosquitto/certs
sudo openssl genrsa -out ca.key 2048
sudo openssl req -x509 -new -nodes -key ca.key -sha256 -days 3650 -out ca.crt
```

### 2.2. Certificat du serveur (Mosquitto)

```bash
sudo openssl genrsa -out server.key 2048
sudo openssl req -new -key server.key -out server.csr
sudo openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key \
  -CAcreateserial -out server.crt -days 365 -sha256
```

### 2.3. Certificats pour les clients (répéter pour chaque client)

> Chaque certificat client **doit avoir un sujet unique**, en particulier le champ "Common Name (CN)", pour que Mosquitto les distingue correctement.

#### Exemple pour le lecteur ESP32 :

```bash
sudo openssl genrsa -out client_reader.key 2048
sudo openssl req -new -key client_reader.key -out client_reader.csr
sudo openssl x509 -req -in client_reader.csr -CA ca.crt -CAkey ca.key \
  -CAcreateserial -out client_reader.crt -days 365 -sha256
```

Changer les noms pour `client_writer`, `client_backend`, `client_mqexplorer` pour les autres clients.

---

## 3. Configuration de Mosquitto

### 3.1. Fichier de configuration

Modifier `/etc/mosquitto/mosquitto.conf` ou ajouter un fichier dans `/etc/mosquitto/conf.d/tls.conf`

```conf
listener 8883
cafile /etc/mosquitto/certs/ca.crt
certfile /etc/mosquitto/certs/server.crt
keyfile /etc/mosquitto/certs/server.key
require_certificate true
use_identity_as_username true
```

### 3.2. Redémarrer le service Mosquitto

```bash
sudo systemctl restart mosquitto
```

---

## 4. Déploiement des certificats clients

Copier sur chaque client :

- `ca.crt`
    
- `client_<role>.crt`
    
- `client_<role>.key`
    

**⚠️ Ne jamais copier `ca.key` ou `server.key` sur un autre appareil !**

---

## 5. Configuration des clients

### 5.1. ESP32 (lecteur et enregistreur)

Utiliser la bibliothèque MQTT TLS de l’ESP32 :

```cpp
client.setCACert(ca_cert);
client.setCertificate(client_cert);
client.setPrivateKey(client_key);
client.connect("client_reader", "192.168.0.102", 8883);
```

Remplacer par `client_writer` pour l’enregistreur.

### 5.2. Backend Node.js (avec `mqtt`)

```js
const mqtt = require('mqtt');
const fs = require('fs');
const client = mqtt.connect('mqtts://192.168.0.102:8883', {
  key: fs.readFileSync('client_backend.key'),
  cert: fs.readFileSync('client_backend.crt'),
  ca: fs.readFileSync('ca.crt'),
  clientId: 'client_backend',
  rejectUnauthorized: true
});
```

### 5.3. MQ Explorer

Configurer les certificats dans l'interface :

- Serveur : `192.168.0.102:8883`
    
- CA file : `ca.crt`
    
- Cert file : `client_mqexplorer.crt`
    
- Key file : `client_mqexplorer.key`
    

---

## 6. Test de connexion

- Démarrer tous les clients un à un.
    
- Vérifier dans les logs de Mosquitto :
    

```bash
sudo journalctl -u mosquitto -f
```

- S’assurer qu’aucune erreur de certificat ou d’authentification ne s’affiche.
    

---

## 7. Sécurisation supplémentaire (facultatif)

- Générer de nouveaux certificats tous les 12 mois
    
- Passer à MQTT sur port 8883 avec firewall ouvert uniquement sur ce port
    
- Activer le chiffrement du disque sur le Raspberry Pi
    

---

## Références

- Page officielle : [https://mosquitto.org/man/mosquitto-tls-7.html](https://mosquitto.org/man/mosquitto-tls-7.html)
    
- Tutoriel vidéo : [https://asciinema.org/a/201826](https://asciinema.org/a/201826)