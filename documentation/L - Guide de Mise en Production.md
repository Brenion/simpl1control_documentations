---
tags: [production, docker, certificats, tls, mqtt, deployment, raspberry-pi]
created: 2026-02-07
---

# Guide de Mise en Production - Système Domotique

Ce guide compile toutes les procédures nécessaires pour déployer le système domotique sur un Raspberry Pi en réseau fermé.

**Architecture** : Frontend React (Vite/Nginx) → Backend Node (Fastify) → PostgreSQL + MQTT (Mosquitto)

---

## 1. Préparation du Serveur (Raspberry Pi)

### 1.1 Configuration SSH 

Générer une clé SSH sur la machine de développement :
```bash
ssh-keygen -t ed25519 -f ~/.ssh/raspberry-tfe
```

Copier la clé publique sur le Raspberry Pi :
```bash
# Afficher la clé publique
cat ~/.ssh/raspberry-tfe.pub

# Copier sur le Raspberry (méthode 1 - ssh-copy-id)
ssh-copy-id -i ~/.ssh/raspberry-tfe.pub <VOTRE_USER>@<IP_RASPBERRY>

# OU méthode manuelle : sur le Raspberry, ajouter la clé dans ~/.ssh/authorized_keys
ssh <VOTRE_USER>@<IP_RASPBERRY>
mkdir -p ~/.ssh && chmod 700 ~/.ssh
echo "<CONTENU_CLE_PUBLIQUE>" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

Créer un fichier config pour simplifier la connexion :

```
# ~/.ssh/config
Host tfe
  Hostname <IP_RASPBERRY>
  User <VOTRE_USER>
  IdentityFile ~/.ssh/raspberry-tfe
```

Connexion simplifiée :

```bash
ssh tfe
```

### 1.2 Mise à jour du système

```bash
sudo apt update && sudo apt upgrade -y
```

### 1.3 Installation de Docker

```bash
# Ajouter la clé GPG Docker
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Ajouter le repository Docker
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

# Installer Docker
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Vérifier l'installation
sudo docker run hello-world
```

### 1.4 Création du registre privé Docker

```bash
docker run -d -p 5000:5000 --restart always --name registry registry:3
```

Le registre sera accessible sur `<IP_SERVEUR>:5000`.

### 1.5 Configuration du Docker Context

Sur la machine de développement, créer un contexte pour le Pi :

```bash
docker context create tfe --docker "host=ssh://tfe"
docker context use tfe
```

---

## 2. Génération des Certificats TLS

### 2.1 Certificat de l'Autorité de Certification (CA)

```bash
cd /etc/mosquitto/certs

# Clé privée CA (4096 bits, valide 10 ans)
sudo openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:4096 -out ca.key

# Certificat auto-signé CA
sudo openssl req -new -x509 -days 3650 -key ca.key -out ca.crt -subj "/CN=MyMosquittoCA"
```

### 2.2 Certificat Serveur (avec SAN)

Créer le fichier `san.cnf` pour inclure l'adresse IP :

```ini
[ req ]
distinguished_name = req_distinguished_name
req_extensions = v3_req
prompt = no

[ req_distinguished_name ]
CN = <IP_SERVEUR>

[ v3_req ]
subjectAltName = @alt_names

[ alt_names ]
IP.1 = <IP_SERVEUR>
```

Générer le certificat serveur :

```bash
# Clé privée serveur
sudo openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out server.key

# Demande de signature (CSR)
sudo openssl req -new -key server.key -out server.csr -config san.cnf

# Signature du certificat
sudo openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key \
  -CAcreateserial -out server.crt -days 365 \
  -extensions v3_req -extfile san.cnf
```

### 2.3 Certificats Clients

**Backend Node.js :**

```bash
sudo openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out client-backend.key
sudo openssl req -new -key client-backend.key -out client-backend.csr -subj "/CN=backend"
sudo openssl x509 -req -in client-backend.csr -CA ca.crt -CAkey ca.key \
  -CAcreateserial -out client-backend.crt -days 3650
```

**ESP32 Encodeur :**

```bash
sudo openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out client-esp-encodeur.key
sudo openssl req -new -key client-esp-encodeur.key -out client-esp-encodeur.csr -subj "/CN=esp32-encodeur"
sudo openssl x509 -req -in client-esp-encodeur.csr -CA ca.crt -CAkey ca.key \
  -CAcreateserial -out client-esp-encodeur.crt -days 365
```

**ESP32 Lecteur :**

```bash
sudo openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out client-esp-reader.key
sudo openssl req -new -key client-esp-reader.key -out client-esp-reader.csr -subj "/CN=esp32-reader"
sudo openssl x509 -req -in client-esp-reader.csr -CA ca.crt -CAkey ca.key \
  -CAcreateserial -out client-esp-reader.crt -days 3650
```

**MQTT Explorer :**

```bash
sudo openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out client-mqtt-explorer.key
sudo openssl req -new -key client-mqtt-explorer.key -out client-mqtt-explorer.csr -subj "/CN=mqtt-explorer"
sudo openssl x509 -req -in client-mqtt-explorer.csr -CA ca.crt -CAkey ca.key \
  -CAcreateserial -out client-mqtt-explorer.crt -days 3650
```

### 2.4 Déploiement des certificats

```bash
sudo mkdir -p /etc/mosquitto/certs
sudo cp ca.crt server.crt server.key /etc/mosquitto/certs/
sudo chown mosquitto:mosquitto /etc/mosquitto/certs/*
sudo chmod 640 /etc/mosquitto/certs/*
```

### 2.5 Tableau récapitulatif

| Client | Clé privée | Certificat |
|--------|-----------|------------|
| ESP32 Encodeur | client-esp-encodeur.key | client-esp-encodeur.crt |
| ESP32 Lecteur | client-esp-reader.key | client-esp-reader.crt |
| Backend NodeJS | client-backend.key | client-backend.crt |
| MQTT Explorer | client-mqtt-explorer.key | client-mqtt-explorer.crt |

Chaque client reçoit : sa clé privée, son certificat, et `ca.crt`.

---

## 3. Configuration Mosquitto

### 3.1 Installation

```bash
sudo apt install -y mosquitto mosquitto-clients
sudo systemctl enable mosquitto
sudo systemctl start mosquitto
```

### 3.2 Gestion des utilisateurs

```bash
# Créer le premier utilisateur
sudo mosquitto_passwd -c /etc/mosquitto/passwd admin

# Ajouter d'autres utilisateurs
sudo mosquitto_passwd /etc/mosquitto/passwd autre_user
```

### 3.3 Configuration `/etc/mosquitto/mosquitto.conf`

```conf
# Port non sécurisé (réseau local)
listener 1883
allow_anonymous false
password_file /etc/mosquitto/passwd

# Port TLS sécurisé
listener 8883
protocol mqtt
cafile /etc/mosquitto/certs/ca.crt
certfile /etc/mosquitto/certs/server.crt
keyfile /etc/mosquitto/certs/server.key
require_certificate true
use_identity_as_username true
```

### 3.4 Redémarrage et test

```bash
sudo systemctl restart mosquitto

# Test publication locale
mosquitto_pub -h localhost -t "test/topic" -m "Hello" -u admin -P "mot_de_passe"

# Test TLS
mosquitto_pub -h localhost -p 8883 --cafile ca.crt \
  --cert client-backend.crt --key client-backend.key \
  -t "test/topic" -m "Hello TLS"
```

---

## 4. Configuration Backend

### 4.1 Variables d'environnement (`production.env`)

```env
NODE_ENV=production
PORT=3001
DATABASE_URL=postgresql://<USER_DB>:<MOT_DE_PASSE_DB>@db:5432/<USER_DB>_dev
MQTT_BASE_URL=<IP_SERVEUR>
MQTT_CLIENT_ID=backend
MQTT_USERNAME=admin
MQTT_PASSWORD=mot_de_passe
MQTT_PORT_PLAIN=1883
MQTT_PORT_TLS=8883
MQTT_KEEPALIVE=60
MQTT_CLEAN_SESSION=true
FRONTEND_URL=http://<IP_SERVEUR>/
```

### 4.2 Connexion MQTT TLS dans le backend

```typescript
if(process.env.MQTT_BASE_URL) {
  const createMqttOptions = (port: string, secure: boolean): MqttOptions => ({
    baseUrl: `${port === "8883" ? "mqtts" : "mqtt"}://${process.env.MQTT_BASE_URL}:${port}`,
    clientId: process.env.MQTT_CLIENT_ID ?? (() => { throw new Error("MQTT_CLIENT_ID is required"); })(),
    username: process.env.MQTT_USERNAME,
    password: process.env.MQTT_PASSWORD,
    keepalive: Number(process.env.MQTT_KEEPALIVE) || 60,
    clean: process.env.MQTT_CLEAN_SESSION === "true",
    ...(secure && {
      key: fs.readFileSync("./certs/client_backend.key"),
      cert: fs.readFileSync("./certs/client_backend.crt"),
      ca: fs.readFileSync("./certs/ca.crt"),
      rejectUnauthorized: false,
    }),
  });

  if (process.env.MQTT_PORT_PLAIN) {
    const optionsPlain = createMqttOptions(process.env.MQTT_PORT_PLAIN, false);
    setTimeout(() => mqttServer(optionsPlain, "plain"), 1500);
  }

  if (process.env.MQTT_PORT_TLS) {
    const optionsTls = createMqttOptions(process.env.MQTT_PORT_TLS, true);
    setTimeout(() => mqttServer(optionsTls, "tls"), 1500);
  }
}
```

### 4.3 Préparation des certificats backend

```bash
mkdir certs
cp client_backend.key certs/
cp client_backend.crt certs/
cp ca.crt certs/
```

---

## 5. Dockerfiles

### 5.1 Backend (`packages/backend/Dockerfile`)

```dockerfile
FROM node:20-alpine AS build
WORKDIR /app
RUN npm i -g pnpm
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml ./
RUN pnpm install --frozen-lockfile
COPY . .
RUN pnpm build

FROM node:20-alpine AS production
WORKDIR /app
ENV NODE_ENV=production
RUN npm i -g pnpm
COPY --from=build /app/package.json /app/pnpm-lock.yaml ./
RUN pnpm install --prod --frozen-lockfile
COPY --from=build /app/dist ./dist
COPY --from=build /app/certs ./certs
EXPOSE 3001
HEALTHCHECK --interval=10s --timeout=3s --start-period=15s --retries=3 \
  CMD wget -qO- http://127.0.0.1:3001/health || exit 1
CMD ["node","dist/server.js"]
```

### 5.2 Frontend (`packages/frontend/Dockerfile`)

```dockerfile
FROM node:20-alpine AS build
WORKDIR /app
RUN npm i -g pnpm
COPY package.json pnpm-lock.yaml ./
RUN pnpm install --frozen-lockfile
COPY . .
ENV VITE_API_BASE_URL=/api
RUN pnpm build

FROM nginx:alpine AS production
WORKDIR /usr/share/nginx/html
RUN rm -rf ./*
COPY --from=build /app/dist .
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
HEALTHCHECK --interval=10s --timeout=3s --start-period=5s --retries=3 \
  CMD wget -qO- http://127.0.0.1/ || exit 1
```

### 5.3 Nginx (`packages/frontend/nginx.conf`)

```nginx
map $http_upgrade $connection_upgrade {
  default upgrade;
  '' close;
}

server {
  listen 80;
  server_name _;

  root /usr/share/nginx/html;
  index index.html;

  # SPA
  location / {
    try_files $uri /index.html;
  }

  # Proxy API -> backend
  location /api/ {
    proxy_pass http://backend:3001/;
    proxy_http_version 1.1;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_read_timeout 3600s;
  }

  # WebSocket
  location /ws/ {
    proxy_pass http://backend:3001/ws/;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $connection_upgrade;
    proxy_set_header Host $host;
    proxy_read_timeout 3600s;
  }
}
```

---

## 6. Docker Compose Production

### 6.1 Fichier `production/docker-compose.production.yml`

```yaml
name: myapp

services:
  db:
    image: <IP_SERVEUR>:5000/postgres:15-alpine
    env_file:
      - ./db.env
    volumes:
      - dbdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U $$POSTGRES_USER -d $$POSTGRES_DB -h 127.0.0.1"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 15s
    restart: always
    networks: [appnet]

  backend:
    image: <IP_SERVEUR>:5000/backend:vX.Y.Z
    env_file:
      - ./production.env
    environment:
      NODE_ENV: production
      PORT: 3001
    depends_on:
      db:
        condition: service_healthy
    restart: always
    networks: [appnet]

  frontend:
    image: <IP_SERVEUR>:5000/frontend:vX.Y.Z
    depends_on:
      - backend
    ports:
      - "80:80"
    restart: always
    networks: [appnet]

  seeder:
    image: <IP_SERVEUR>:5000/backend:vX.Y.Z
    depends_on:
      db:
        condition: service_healthy
    env_file:
      - ./production.env
    command: ["node", "dist/seed.js"]
    restart: "no"
    profiles: ["seed"]
    networks: [appnet]

volumes:
  dbdata:

networks:
  appnet:
    driver: bridge
```

### 6.2 Fichier `production/db.env`

```env
POSTGRES_DB=<USER_DB>_dev
POSTGRES_USER=<USER_DB>
POSTGRES_PASSWORD=<MOT_DE_PASSE_DB>
```

---

## 7. Scripts de Déploiement

### 7.1 Arborescence

```
<repo>/
├─ packages/
│  ├─ backend/
│  │  └─ Dockerfile
│  └─ frontend/
│     ├─ Dockerfile
│     └─ nginx.conf
└─ production/
   ├─ docker-compose.production.yml
   ├─ production.env
   ├─ db.env
   ├─ VERSION
   ├─ tag.sh
   ├─ build.sh
   ├─ push_deploy.sh
   └─ artifacts/
```

### 7.2 Script `production/tag.sh`

```sh
#!/usr/bin/env sh
# usage: ./production/tag.sh vX.Y.Z
set -e

VERSION_FILE="./production/VERSION"
COMPOSE_FILE="./production/docker-compose.production.yml"
REG="<IP_SERVEUR>:5000"

if [ -z "$1" ]; then
  echo "Usage: ./production/tag.sh vX.Y.Z"
  exit 1
fi
NEW_VERSION="$1"

echo "$NEW_VERSION" | grep -Eq '^v[0-9]+\.[0-9]+\.[0-9]+$' || {
  echo "Format invalide. Attendu: vX.Y.Z"; exit 1; }

if [ -f "$VERSION_FILE" ]; then
  CURRENT_VERSION=$(cat "$VERSION_FILE")
  [ "$NEW_VERSION" = "$CURRENT_VERSION" ] && {
    echo "Version $NEW_VERSION déjà enregistrée"; exit 1; }
fi

echo "$NEW_VERSION" > "$VERSION_FILE"
echo "VERSION enregistrée: $NEW_VERSION"

# Mise à jour docker-compose.production.yml
sed -i.bak -E "s|(image:[[:space:]]*${REG}/backend:)[^\"'[:space:]]+|\\1${NEW_VERSION}|g"  "$COMPOSE_FILE"
sed -i.bak -E "s|(image:[[:space:]]*${REG}/frontend:)[^\"'[:space:]]+|\\1${NEW_VERSION}|g" "$COMPOSE_FILE"
rm -f "$COMPOSE_FILE.bak"
echo "docker-compose.production.yml mis à jour"
```

### 7.3 Script `production/build.sh`

```sh
#!/usr/bin/env sh
# Build images ARM64 + export .tar
set -e

SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
ROOT_DIR="$(cd "${SCRIPT_DIR}/.." && pwd)"
VERSION_FILE="${ROOT_DIR}/production/VERSION"
ARTIFACTS_DIR="${ROOT_DIR}/production/artifacts"

BACK_CTX="${ROOT_DIR}/packages/backend"
FRONT_CTX="${ROOT_DIR}/packages/frontend"
BACK_DOCKERFILE="${BACK_CTX}/Dockerfile"
FRONT_DOCKERFILE="${FRONT_CTX}/Dockerfile"

REG="${REG:-<IP_SERVEUR>:5000}"

[ -f "${VERSION_FILE}" ] || { echo "VERSION introuvable. Lancez ./production/tag.sh vX.Y.Z"; exit 1; }
VERSION="$(tr -d '\r\n' < "${VERSION_FILE}")"

mkdir -p "${ARTIFACTS_DIR}"

echo "VERSION = ${VERSION}"
echo "REG     = ${REG}"

docker buildx create --name prod_builder --use >/dev/null 2>&1 || true
docker buildx inspect --bootstrap >/dev/null 2>&1 || true

echo "Build backend (linux/arm64)..."
docker buildx build --platform linux/arm64 \
  -t "${REG}/backend:${VERSION}" \
  -f "${BACK_DOCKERFILE}" \
  --load \
  "${BACK_CTX}"

echo "Build frontend (linux/arm64)..."
docker buildx build --platform linux/arm64 \
  -t "${REG}/frontend:${VERSION}" \
  -f "${FRONT_DOCKERFILE}" \
  --load \
  "${FRONT_CTX}"

echo "Prépare postgres:15-alpine (linux/arm64)..."
docker pull --platform linux/arm64 postgres:15-alpine
docker tag postgres:15-alpine "${REG}/postgres:15-alpine"

echo "Export des images..."
docker save "${REG}/backend:${VERSION}"  -o "${ARTIFACTS_DIR}/backend_${VERSION}.tar"
docker save "${REG}/frontend:${VERSION}" -o "${ARTIFACTS_DIR}/frontend_${VERSION}.tar"
docker save "${REG}/postgres:15-alpine"  -o "${ARTIFACTS_DIR}/postgres_15-alpine.tar"

echo "Build terminé. Artifacts dans ${ARTIFACTS_DIR}"
```

### 7.4 Script `production/push_deploy.sh`

```sh
#!/usr/bin/env sh
# Charge, push et déploie sur le Pi
set -e

SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
ROOT_DIR="$(cd "${SCRIPT_DIR}/.." && pwd)"
VERSION_FILE="${ROOT_DIR}/production/VERSION"
ARTIFACTS_DIR="${ROOT_DIR}/production/artifacts"
COMPOSE_FILE="${ROOT_DIR}/production/docker-compose.production.yml"
REG="${REG:-<IP_SERVEUR>:5000}"

[ -f "${VERSION_FILE}" ] || { echo "VERSION introuvable"; exit 1; }
VERSION="$(tr -d '\r\n' < "${VERSION_FILE}")"

for f in "backend_${VERSION}.tar" "frontend_${VERSION}.tar" "postgres_15-alpine.tar"; do
  [ -f "${ARTIFACTS_DIR}/$f" ] || { echo "Manque ${ARTIFACTS_DIR}/$f"; exit 1; }
done

echo "Load images dans le daemon du Pi (context tfe)..."
docker --context tfe load -i "${ARTIFACTS_DIR}/backend_${VERSION}.tar"
docker --context tfe load -i "${ARTIFACTS_DIR}/frontend_${VERSION}.tar"
docker --context tfe load -i "${ARTIFACTS_DIR}/postgres_15-alpine.tar"

echo "Push vers registre ${REG}..."
docker --context tfe push "${REG}/backend:${VERSION}"
docker --context tfe push "${REG}/frontend:${VERSION}"
docker --context tfe push "${REG}/postgres:15-alpine"

echo "Déploiement compose..."
docker --context tfe compose -f "${COMPOSE_FILE}" up -d

echo "Déploiement terminé."
docker --context tfe ps
```

---

## 8. Intégration ESP32

### 8.1 Brochage matériel

**MFRC522 (SPI) :**

| MFRC522 Pin | ESP32 Pin | Fonction |
|-------------|-----------|----------|
| SDA (SS) | GPIO 5 | Chip Select |
| SCK | GPIO 18 | Horloge SPI |
| MOSI | GPIO 23 | Données vers lecteur |
| MISO | GPIO 19 | Données depuis lecteur |
| RST | GPIO 22 | Reset module |
| GND | GND | Masse |
| 3.3V | 3.3V | Alimentation |

**LED WS2812 :**

| WS2812 Pin | ESP32 Pin | Fonction |
|------------|-----------|----------|
| DIN | GPIO 4 | Donnée |
| GND | GND | Masse |
| 5V | 5V | Alimentation |

### 8.2 Certificats en PROGMEM

Convertir les certificats en headers C :

```bash
xxd -i ca.crt > ca_crt.h
xxd -i client-esp-encodeur.crt > client_crt.h
xxd -i client-esp-encodeur.key > client_key.h
```

### 8.3 Code ESP32 connexion MQTT TLS

```cpp
#include <WiFi.h>
#include <WiFiClientSecure.h>
#include <PubSubClient.h>

// Certificats
#include "ca_crt.h"
#include "client_crt.h"
#include "client_key.h"

// Paramètres Wi-Fi
const char* ssid = "VOTRE_SSID";
const char* password = "VOTRE_MOT_DE_PASSE";

// Paramètres MQTT
const char* mqtt_server = "<IP_SERVEUR>";
const int mqtt_port = 8883;
const char* mqtt_user = "admin";
const char* mqtt_password = "mot_de_passe";
const char* topic = "test/esp32";

WiFiClientSecure secureClient;
PubSubClient client(secureClient);

void setup_wifi() {
    Serial.println("Connexion au WiFi...");
    WiFi.begin(ssid, password);
    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
    }
    Serial.println("WiFi connecté");
    Serial.println(WiFi.localIP());
}

void reconnect() {
    while (!client.connected()) {
        Serial.println("Connexion MQTT...");
        if (client.connect("ESP32Client", mqtt_user, mqtt_password)) {
            Serial.println("Connecté au broker MQTT");
        } else {
            Serial.print("Échec, rc=");
            Serial.println(client.state());
            delay(5000);
        }
    }
}

void setup() {
    Serial.begin(115200);
    setup_wifi();

    // Charger les certificats
    secureClient.setCACert((const char*)ca_crt);
    secureClient.setCertificate((const char*)client_crt);
    secureClient.setPrivateKey((const char*)client_key);

    client.setServer(mqtt_server, mqtt_port);
}

void loop() {
    if (!client.connected()) {
        reconnect();
    }
    client.loop();

    static unsigned long lastMsg = 0;
    if (millis() - lastMsg > 5000) {
        lastMsg = millis();
        client.publish(topic, "Hello depuis ESP32 TLS");
    }
}
```

---

## 9. Checklist de Déploiement

### Première installation

- [ ] SSH configuré sur le Raspberry Pi
- [ ] Docker installé
- [ ] Registre privé Docker actif (port 5000)
- [ ] Certificats CA générés
- [ ] Certificats serveur générés (avec SAN)
- [ ] Certificats clients générés
- [ ] Mosquitto configuré (ports 1883 + 8883)
- [ ] Docker context `tfe` créé

### À chaque déploiement

1. [ ] Définir la version : `./production/tag.sh v1.0.0`
2. [ ] Builder les images : `./production/build.sh`
3. [ ] Basculer sur réseau fermé
4. [ ] `docker context use tfe`
5. [ ] Déployer : `./production/push_deploy.sh`
6. [ ] Vérifier : `http://<IP_SERVEUR>/`

---

## 10. Commandes Utiles

### Logs et diagnostic

```bash
docker --context tfe ps
docker --context tfe logs backend --tail=100
docker --context tfe logs frontend --tail=50
```

### Seeder (première fois)

```bash
docker --context tfe compose -f production/docker-compose.production.yml up -d db
docker --context tfe compose -f production/docker-compose.production.yml --profile seed up --abort-on-container-exit seeder
docker --context tfe compose -f production/docker-compose.production.yml up -d
```

### Accès PostgreSQL

```bash
docker --context tfe exec -it <NOM_CONTENEUR_DB> sh -lc \
  'PGPASSWORD="$POSTGRES_PASSWORD" psql -U "$POSTGRES_USER" -d "$POSTGRES_DB" -c "SELECT * FROM public.users LIMIT 50;"'
```

### Rollback

```bash
# Éditer docker-compose.production.yml avec version précédente
docker --context tfe compose -f production/docker-compose.production.yml up -d
```

### Nettoyage

```bash
docker --context tfe image prune -a
```

---

## Références

- [[J - mise en production]]
- [[K - Récapitulatif production]]
- [[A - Installation et Test de Mosquitto (MQTT) sur un Raspberry Pi]]
- [[I - 05 - Mise en œuvre technique – Encodeur ESP32]]
- [[I - 05 bis - Intégration du Port MQTT Sécurisé (TLS) dans le Backend]]
