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
        
    - **Brochage recommandé ESP32 ↔ MFRC522** :
        
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

Ce chapitre décrit :

1. **Connexion WiFi** de l’ESP32
    
2. **Configuration du broker Mosquitto** (répertoire, certificats, mot de passe, configuration)
    
3. **Intégration MQTT** dans le code ESP32 (ports et fallback)
    

---

### 2.1 Connexion WiFi

**Objectif** : connecter l’ESP32 au réseau local avant toute communication MQTT.

```cpp
#include <WiFi.h>

const char* ssid     = "<TON_SSID>";
const char* password = "<TON_MOT_DE_PASSE>";

void setup_wifi() {
  Serial.print("Connexion au WiFi '" + String(ssid) + "' ... ");
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print('.');
  }
  Serial.println();
  Serial.println("WiFi connecté. IP = " + WiFi.localIP().toString());
}
```

Appelé dans `setup()` juste après `Serial.begin()`.

---

### 2.2 Configuration du broker Mosquitto

#### 2.2.1 Création du répertoire et permissions

```bash
sudo mkdir -p /etc/mosquitto/ca_certificates
sudo chown root:mosquitto /etc/mosquitto/ca_certificates
sudo chmod 750 /etc/mosquitto/ca_certificates
```

#### 2.2.2 Génération des certificats

Placez-vous dans `/etc/mosquitto/ca_certificates` ou ajoutez `-out /etc/mosquitto/ca_certificates/<fichier>` à chaque commande.

1. **Clé privée de l’AC**
    
```bash
openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:4096 -out ca.key
```
    
2. **Certificat auto-signé de l’AC**
    
```bash
openssl req -new -x509 -days 3650 -key ca.key -out ca.crt -subj "/CN=MyMosquittoCA"
```
    
3. **Clé privée serveur + CSR**
    
```bash
openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:4096 -out server.key
openssl req -new -key server.key -out server.csr -subj "/CN=mosquitto.local"
    ```
    
4. **Signature du CSR**
    
```bash
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 365
```
    
5. **Déploiement**
    
    ```bash
    sudo cp ca.crt server.crt server.key /etc/mosquitto/ca_certificates/
    sudo chown root:mosquitto /etc/mosquitto/ca_certificates/{ca.crt,server.crt,server.key}
    sudo chmod 640 /etc/mosquitto/ca_certificates/{ca.crt,server.crt,server.key}
    ```
    

#### 2.2.3 Création d’un utilisateur et d’un mot de passe MQTT

Pour limiter l’accès à votre broker Mosquitto (notamment sur le port sécurisé), définissez un ou plusieurs comptes utilisateurs à l’aide d’un fichier de mots de passe Mosquitto.

**Étapes :**

1. **Créer (ou écraser) le fichier de mots de passe et ajouter un utilisateur**  
    Remplacez `<utilisateur>` par le nom souhaité (ex : `admin`).
    
    ```bash
    sudo mosquitto_passwd -c /etc/mosquitto/passwd <utilisateur>
    ```
    
    ➡️ On vous demandera de saisir le mot de passe.
    
    - Pour ajouter d’autres utilisateurs sans écraser les précédents :
        
        ```bash
        sudo mosquitto_passwd /etc/mosquitto/passwd <autre_utilisateur>
        ```
        
2. **Vérifier le contenu du fichier**
    
    ```bash
    sudo cat /etc/mosquitto/passwd
    ```
    
    (les utilisateurs apparaissent sous forme hashée)
    
3. **Intégrer ce fichier dans la configuration**
    
    ```conf
    password_file /etc/mosquitto/passwd
    ```
    
    (déjà présent dans l’exemple de configuration)
    
4. **Redémarrer Mosquitto**
    
    ```bash
    sudo systemctl restart mosquitto
    ```
    

#### 2.2.4 Fichier `/etc/mosquitto/mosquitto.conf`

```conf
pid_file /var/run/mosquitto.pid

# Listener non sécurisé (1883)
listener 1883
allow_anonymous true

# Listener sécurisé (8883)
listener 8883
protocol mqtt
cafile /etc/mosquitto/ca_certificates/ca.crt
certfile /etc/mosquitto/ca_certificates/server.crt
keyfile /etc/mosquitto/ca_certificates/server.key
require_certificate true
use_identity_as_username true
password_file /etc/mosquitto/passwd

# Logs
log_dest syslog
log_dest stdout
log_type warning
connection_messages true
```

Redémarrer et vérifier :

```bash
sudo systemctl restart mosquitto
sudo systemctl status mosquitto
sudo journalctl -u mosquitto -f
```

---

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
secureClient.setCACert(ca_cert);
secureClient.setCertificate(client_cert);
secureClient.setPrivateKey(client_key);
PubSubClient secureMqtt(secureClient);

void setup_mqtt_secure() {
  secureMqtt.setServer("192.168.0.102", 8883);
}
```

#### 2.3.3 Logique de bascule (fallback)

```cpp
void setup_mqtt() {
  setup_mqtt_secure();
  if (secureMqtt.connect("ESP32Client", mqtt_user, mqtt_password)) {
    client = secureMqtt;
  } else {
    setup_mqtt_nonsecure();
  }
}
```

## 3. Dump complet carte vierge

Dump complet carte vierge

- Flasher un firmware temporaire pour lire les 64 blocs MIFARE
    
- Transmettre ou enregistrer ce dump (port série ou MQTT local)
    

## 4. Génération backend des données de badge

- Enregistrer l’UID dans le seeder
    
- Générer les blocs :
    
    - Bloc 8 : clé dérivée AES(uid + secret)
        
    - Bloc 9 : type de badge (“maitre” ou “utilisateur”)
        
    - Bloc 10 : signature HMAC(uid + secret)
        
    - Bloc 11 (trailer) : version + clé A + bits d’accès + clé B
        

## 5. Publication backend MQTT

- Émettre vers `access/encoder` :
    

```json
{
  "uid": "DEADBEEF",
  "type": "maitre",
  "derivedKey": "...",
  "signature": "...",
  "version": 1,
  "keyA": "A1A2A3A4A5A6",
  "keyB": "B1B2B3B4B5B6",
  "accessBits": "787788"
}
```

## 6. Déploiement du code ESP32

- Initialiser le WiFi
    
- Initialiser la double connexion MQTT (1883 + 8883)
    
- Boucle de détection MFRC522
    
- Lecture de l’UID
    
- Vérification de l’UID avec message MQTT
    
- Écriture successive des blocs :
    
    1. Bloc 8 : clé dérivée
        
    2. Bloc 9 : type
        
    3. Bloc 10 : signature
        
    4. Bloc 11 : trailer (clé A + accessBits + version + clé B)
        
- Gestion du retour visuel par WS2812
    
- Publication ACK sur `access/encoder/ack`
    

## 7. Validation avec lecteur ESP32

- Scanner sur un second ESP32 lecteur
    
- Lire : UID, type, signature, version
    
- Vérifier rôle “maitre” ou “utilisateur”
    
- Enregistrer les événements côté backend (optionnel)
    

## 8. Tests de scénario (validation terrain)

- Encodage carte “maitre” → succès
    
- Encodage carte “utilisateur” → succès
    
- UID erroné → refus d’écriture
    
- Erreur trailer → échec confirmé
    
- Restauration carte → dump réinjecté