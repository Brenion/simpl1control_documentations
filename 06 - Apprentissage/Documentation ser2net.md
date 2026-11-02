Zigbee2MQTT

## Architecture

Z2M tourne dans Docker et se connecte au broker Mosquitto interne (`simpl1controlMQTT`).

Le topic MQTT utilisé est `s1c/zigbee2mqtt`.

Le frontend est accessible sur `http://localhost:8099`.

La connexion au dongle Zigbee diffère selon l'environnement :

- **Dev (Mac)** : via `ser2net` (TCP) — ser2net tourne sur le Mac, hors Docker

- **Prod (Raspberry Pi)** : via `--device` natif Linux

---

## Développement (Mac)

### Prérequis — ser2net

brew install ser2net

### 1. Trouver le path du dongle

Branche le ZBDongle-E et exécute :

ls /dev/cu.usb*

Sur ce projet le dongle apparaît sous `/dev/cu.usbserial-2120`.  
Utiliser toujours le préfixe `cu` et non `tty` sur macOS.

### 2. Configurer ser2net

Édite `$(brew --prefix)/etc/ser2net/ser2net.yaml` et ajoute :

connection: &con00

accepter: tcp,20108

connector: serialdev,/dev/cu.usbserial-2120,115200n81,local

options:

kickolduser: true

### 3. Lancer ser2net

ser2net -c $(brew --prefix)/etc/ser2net/ser2net.yaml

Vérifier qu'il écoute bien sur le port 20108 :

lsof -i :20108

# doit afficher ser2net en LISTEN

### 4. Lancer Z2M

# D'abord le stack principal (Mosquitto doit tourner avant Z2M)

docker compose -f docker-compose.yml up -d

# Ensuite Z2M

docker compose -f docker-compose.z2m.dev.yml --env-file common.env up -d

### 5. Premier démarrage — onboarding

Au premier démarrage (pas de `configuration.yaml`), Z2M lance une page d'onboarding.  
Accessible sur `http://localhost:8099`.

> Note : pendant l'onboarding Z2M écoute en interne sur le port 8080.  
> Le mapping `8099:8080` dans le compose s'occupe de la traduction — ne pas changer le port dans le formulaire.

Après soumission, faire un hard refresh (`Cmd+Shift+R`) ou ouvrir en navigation privée.

### 6. Accès au frontend

http://localhost:8099

---

## Variables d'environnement

Les variables communes dev/prod sont dans `common.env` (ignoré par Git — à créer manuellement).  
Voir le contenu attendu ci-dessous :

Z2M_IMAGE=ghcr.io/koenkk/zigbee2mqtt

Z2M_TZ=Europe/Brussels

Z2M_FRONTEND_PORT=8099

ZIGBEE2MQTT_CONFIG_MQTT_SERVER=mqtt://simpl1controlMQTT:1883

ZIGBEE2MQTT_CONFIG_MQTT_USER=admin

ZIGBEE2MQTT_CONFIG_MQTT_PASSWORD=<mot_de_passe>

ZIGBEE2MQTT_CONFIG_MQTT_BASE_TOPIC=s1c/zigbee2mqtt

ZIGBEE2MQTT_CONFIG_FRONTEND_ENABLED=true

ZIGBEE2MQTT_CONFIG_ADVANCED_CHANNEL=25

---

## Production (Raspberry Pi)

### 1. Trouver le path du dongle

ls -la /dev/serial/by-id/

Mettre à jour `ZIGBEE2MQTT_CONFIG_SERIAL_PORT` et `devices:` dans  
`production/docker-compose.z2m.prod.yml` avec le path exact.

### 2. Lancer Z2M

docker compose -f docker-compose.production.yaml up -d

docker compose -f docker-compose.z2m.prod.yml --env-file common.env up -d

---

## Réseau Zigbee

|Paramètre|Valeur|
|---|---|
|Canal|25 (isolé du canal 11 du réseau HA existant)|
|Adaptateur|EmberZNet (ZBDongle-E)|
|`rtscts`|false|
|`pan_id`|52488|