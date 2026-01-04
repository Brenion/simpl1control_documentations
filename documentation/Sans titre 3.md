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
    
    ```bash
    sudo openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:4096 -out ca.key
    ```
    
- **Étape 2 : Création du certificat auto-signé CA**
    
    ```bash
    sudo openssl req -new -x509 -days 3650 -key ca.key -out ca.crt -subj "/CN=MyMosquittoCA"
    ```
    
- **Étape 3 : Déploiement du certificat CA**
    
    ```bash
    sudo mkdir -p /etc/mosquitto/certs
    sudo cp ca.crt /etc/mosquitto/certs/
    sudo chown mosquitto:mosquitto /etc/mosquitto/certs/ca.crt
    sudo chmod 640 /etc/mosquitto/certs/ca.crt
    ```
    

#### 2.2.2 Génération du certificat serveur (avec SAN) Génération du certificat serveur (avec SAN)

  
Pour que les clients valident correctement le certificat, l’adresse IP fixe ou le nom DNS du broker doit figurer dans le SAN.

- **Étape 1 : Création du fichier SAN**


```
    [ req ]
    distinguished_name = req_distinguished_name
    req_extensions = v3_req
    prompt = no
    
    [ req_distinguished_name ]
    CN = 192.168.0.102
    
    [ v3_req ]
    subjectAltName = @alt_names
    
    [ alt_names ]
    IP.1 = 192.168.0.102
```

- **Étape 2 : Clé privée serveur & CSR**
    
    ```bash
    openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out server.key
    openssl req -new -key server.key -out server.csr -config san.cnf
    ```
    
- **Étape 3 : Signature du certificat**
    
    ```bash
    openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key 
      -CAcreateserial -out server.crt -days 365 
      -extensions v3_req -extfile san.cnf
    ```
    
- **Étape 4 : Déploiement et sécurisation** 
```
sudo cp ca.crt server.crt server.key /etc/mosquitto/certs/ sudo chown mosquitto:mosquitto /etc/mosquitto/certs/ca.crt  
/etc/mosquitto/certs/server.crt /etc/mosquitto/certs/server.key sudo chmod 640 /etc/mosquitto/certs/ca.crt  
/etc/mosquitto/certs/server.crt /etc/mosquitto/certs/server.key

```

#### 2.2.3 Génération du certificat client (port 8883) (port 8883)
```

```bash
cd /etc/mosquitto/ca_certificates/
# Par exemple pour l'ESP32 encodeur
sudo openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out client-esp-encodeur.key
sudo openssl req -new -key client-esp-encodeur.key 
  -out client-esp-encodeur.csr -subj "/CN=esp32-encodeur"
sudo openssl x509 -req -in client-esp-encodeur.csr 
  -CA ca.crt -CAkey ca.key -CAcreateserial 
  -out client-esp-encodeur.crt -days 365
```

_Répétez pour chaque client en changeant ******``******._

#### 2.2.4 Gestion des utilisateurs et mots de passe

```bash
sudo mosquitto_passwd -c /etc/mosquitto/passwd <utilisateur>
sudo mosquitto_passwd /etc/mosquitto/passwd <autre_utilisateur>
```

Intégrer dans `mosquitto.conf` :

```conf
password_file /etc/mosquitto/passwd
```

#### 2.2.5 Fichier `/etc/mosquitto/opmosquitto.conf`

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

### 2.3 Intégration MQTT dans le code ESP32

#### 2.3.1 Port 1883 (non sécurisé)

```cpp
#include <PubSubClient.h>
WiFiClient espClient;
PubSubClient client(espClient);
void setup_mqtt_nonsecure() {
  client.setServer("192.168.0.102", 1883);
}
```

#### 2.3.2 Port 8883 (TLS)

```cpp
#include <WiFiClientSecure.h>
WiFiClientSecure secureClient;
secureClient.setCACert(ca_crt);
secureClient.setCertificate(client_crt);
secureClient.setPrivateKey(client_key);
PubSubClient secureMqtt(secureClient);
void setup_mqtt_secure() {
  secureMqtt.setServer("192.168.0.102", 8883);
}
```

#### 2.3.3 Bascule automatique

```cpp
void setup_mqtt() {
  setup_mqtt_secure();
  if (secureMqtt.connect("ESP32Client", mqtt_user, mqtt_password)) client = secureMqtt;
  else setup_mqtt_nonsecure();
}
```

## 3. Dump complet carte vierge

- Flasher un firmware temporaire
    
- Lire et transmettre les 64 blocs MIFARE
    

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
sudo openssl x509 -req -in client-esp-encodeur.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out client-esp-encodeur.crt -days 365
```

**ESP32 Lecteur**

```bash
sudo openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out client-esp-lecteur.key
sudo openssl req -new -key client-esp-lecteur.key -out client-esp-lecteur.csr -subj "/CN=esp32-lecteur"
sudo openssl x509 -req -in client-esp-lecteur.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out client-esp-lecteur.crt -days 365
```

**Backend NodeJS**

```bash
sudo openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out client-backend.key
sudo openssl req -new -key client-backend.key -out client-backend.csr -subj "/CN=backend-nodejs"
sudo openssl x509 -req -in client-backend.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out client-backend.crt -days 3650
```

**MQTT Explorer**

```bash
sudo openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out client-mqexplorer.key
sudo openssl req -new -key client-mqexplorer.key -out client-mqexplorer.csr -subj "/CN=mqtt-explorer"
sudo openssl x509 -req -in client-mqexplorer.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out client-mqexplorer.crt -days 365
```

|Client|Clé privée|Certificat client|
|---|---|---|
|ESP32 Encodeur|client-esp-encodeur.key|client-esp-encodeur.crt|
|ESP32 Lecteur|client-esp-lecteur.key|client-esp-lecteur.crt|
|Backend NodeJS|client-backend.key|client-backend.crt|
|MQTT Explorer|client-mqexplorer.key|client-mqexplorer.crt|

Chaque client doit obtenir : sa clé privée, son certificat, et le fichier commun `ca.crt`.

**1.**           **Préparation – Début de Mission**

**1 – Points à respecter**

- Ne pas mentionner Vincent.
- Être clair et transparent dans mes échanges.
- M’appuyer sur mon expérience en théâtre pour mieux communiquer et gérer les interactions.
- Être conscient que certaines personnes sont présentes depuis longtemps, avec des avis tranchés, politisés et provenant de groupes différents. avec de l'influence, et provenant de directions différentes (direction santé, direction accueil et enfance, etc)
- Être aidé par Gilles pour faciliter ma transition.
- Assurer le suivi des tâches, la documentation et appliquer la rigueur en gestion de projet.
- Gérer les risques et tenir un registre des risques à jour.

**2 – Questions pertinentes à poser**

- Quels sont les critères pour considérer la mission comme un succès ?
- Quelles seront les tâches quotidiennes ?
- Quelles forces puis-je apporter à l’équipe ?
- Quelle est la date officielle de début de mission ?

**3 – Communication et vocabulaire**

- Utiliser un vocabulaire précis : **stakeholder (partie prenante)**, livrables, jalons, risques, etc.
- Rester aligné avec les valeurs d’Isabelle : authenticité, sincérité et intelligence relationnelle.