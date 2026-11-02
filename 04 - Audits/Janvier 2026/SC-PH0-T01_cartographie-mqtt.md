# SC-PH0-T01 — Cartographie des topics MQTT existants

> **User Story** : SC-US-PH0-03 — En tant que développeur, je veux documenter l'architecture MQTT actuelle (topics, payload, clients)
> **Tâche** : Cartographier les topics MQTT existants et vérifier que l'architecture est complète et fonctionnelle lorsque la passerelle zigbee2mqtt → MQTT sera mise en place
> **Projet** : simpl1Control v1.1.2
> **Date** : 10 février 2026
> **Méthode** : Analyse statique du code source (read-only, aucune modification)

---

## Table des matières

1. [Vue d'ensemble de l'architecture MQTT](#1-vue-densemble-de-larchitecture-mqtt)
2. [Configuration et connexion](#2-configuration-et-connexion)
3. [Cartographie complète des topics](#3-cartographie-complète-des-topics)
4. [Payloads et formats de messages](#4-payloads-et-formats-de-messages)
5. [Clients MQTT — Qui publie, qui souscrit](#5-clients-mqtt--qui-publie-qui-souscrit)
6. [Chaîne de traitement des messages (pipeline)](#6-chaîne-de-traitement-des-messages-pipeline)
7. [Pont MQTT → WebSocket (temps réel)](#7-pont-mqtt--websocket-temps-réel)
8. [Tâches CRON liées à MQTT](#8-tâches-cron-liées-à-mqtt)
9. [Analyse de compatibilité zigbee2mqtt](#9-analyse-de-compatibilité-zigbee2mqtt)
10. [Constats et recommandations](#10-constats-et-recommandations)
11. [Synthèse](#11-synthèse)

---

## 1. Vue d'ensemble de l'architecture MQTT

### 1.1 Schéma global

```
                          ┌──────────────────────┐
                          │   Broker MQTT         │
                          │   192.168.3.100       │
                          │  :1883 (plain)        │
                          │  :8883 (TLS/mTLS)     │
                          └───────┬──────┬────────┘
                                  │      │
                  ┌───────────────┘      └──────────────┐
                  │                                     │
         ┌────────▼─────────┐                ┌──────────▼──────────┐
         │  Client Plain    │                │  Client TLS (mTLS)  │
         │  clientId:       │                │  clientId:           │
         │  "client"        │                │  "client-backend"    │
         │  port: 1883      │                │  port: 8883          │
         └────────┬─────────┘                └──────────┬──────────┘
                  │                                     │
                  └──────────┬──────────────────────────┘
                             │
                    ┌────────▼────────┐
                    │  Backend Fastify │
                    │  MqttService     │
                    │  (Singleton/port)│
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────────┐
              │              │                  │
    ┌─────────▼──┐   ┌──────▼──────┐   ┌───────▼───────┐
    │handleReader │   │processTemp  │   │ DataHistories │
    │Access (NFC) │   │→ LOGO PLC   │   │ Repository    │
    └─────────┬──┘   └──────┬──────┘   └───────┬───────┘
              │              │                  │
              │              │           ┌──────▼──────┐
              │              │           │  KeyMapper   │
              │              │           │  → Realtime  │
              │              │           │    Hub → WS  │
              │              │           └─────────────┘
              │              │
    ┌─────────▼──┐   ┌──────▼──────┐
    │ LOGO PLC   │   │ LOGO PLC    │
    │ (alertes   │   │ (température│
    │  portes)   │   │  chauffage) │
    └────────────┘   └─────────────┘
```

### 1.2 Principes architecturaux

L'architecture MQTT repose sur les principes suivants :

- **Dual-connexion** : deux clients MQTT parallèles — un non-chiffré (port 1883) et un TLS avec mutual TLS (port 8883)
- **Topics dynamiques** : les topics sont stockés en base de données (colonnes `subscribe` et `publish` de l'entité `DeviceEntity`)
- **Singleton par URL** : `MqttService` utilise un pattern Singleton avec clé = `baseUrl` (ex: `mqtt://192.168.3.100:1883`)
- **Routage par pattern** : le handler `on("message")` route les messages selon que le topic contient `/controller/reader-nfc-`, `/sensor/` ou `/API/`
- **Persistance systématique** : tout message reçu est sauvegardé dans `DataHistoriesRepository` (historique complet)

---

## 2. Configuration et connexion

### 2.1 Variables d'environnement MQTT

| Variable | Valeur (dev) | Description |
|---|---|---|
| `MQTT_BASE_URL` | `192.168.3.100` | Adresse IP du broker |
| `MQTT_PORT_PLAIN` | `1883` | Port non-chiffré |
| `MQTT_PORT_TLS` | `8883` | Port TLS/mTLS |
| `MQTT_USERNAME` | `admin` | Identifiant broker |
| `MQTT_PASSWORD` | `***` | Mot de passe broker |
| `MQTT_CLIENT_ID_PLAIN` | `client` | ID client plain |
| `MQTT_CLIENT_ID` | `client-backend` | ID client TLS |
| `MQTT_KEEPALIVE` | `60` | Keepalive en secondes |
| `MQTT_CLEAN_SESSION` | `true` | Session propre à chaque connexion |
| `MQTT_START` | `false` (dev) | Active/désactive le démarrage MQTT |

**Fichier** : `development.env`

### 2.2 Processus d'initialisation (server.ts)

```
1. Démarrage Fastify (port 3000)
2. Si MQTT_START = true :
   a. Construction MqttOptions pour client plain (mqtt://) et TLS (mqtts://)
   b. Initialisation parallèle via Promise.all([initMqttClient(plain), initMqttClient(tls)])
   c. Pour chaque client : MqttService.initAsync(options) → connexion → reloadAllSubscribers(port)
   d. Après init complète : pingPongMqttDevices() (vérification statut devices)
```

### 2.3 Connexion TLS (mTLS)

Pour la connexion sécurisée, le service charge trois fichiers de certificats :

- `./certs/client-backend.key` — Clé privée du client
- `./certs/client-backend.crt` — Certificat du client
- `./certs/ca.crt` — Certificat de l'autorité de certification

**Constat** : `rejectUnauthorized` est défini à `false`, ce qui désactive la vérification du certificat serveur (sécurité : **sévérité 3**)

### 2.4 Rechargement des abonnements (reload-mqtt.service.ts)

Au démarrage, `reloadAllSubscribers(port)` :
1. Récupère tous les devices ayant un champ `subscribe` non-vide via `deviceRepository.findSubscribers()`
2. Pour chacun, appelle `mqttService.ensureSubscription(device.subscribe)` — abonnement idempotent
3. Log le nombre total de topics rechargés

**Note** : ce rechargement se fait indépendamment pour chaque client MQTT (plain et TLS). Tous les devices sont abonnés sur les deux connexions sans distinction basée sur `isSecure`.

---

## 3. Cartographie complète des topics

### 3.1 Convention de nommage des topics

Le pattern général observé est : `unifyIots/{type}/{deviceId}/{direction}`

| Segment | Valeurs observées | Description |
|---|---|---|
| Préfixe | `unifyIots` | Namespace projet |
| Type | `controller`, `sensor`, `API` | Catégorie de device |
| DeviceId | `reader-nfc-01`, `custom-sensor-dth-01`, `logo` | Identifiant unique |
| Direction | `get` (subscribe), `set` (publish) | Sens du flux |

### 3.2 Topics Subscribe (Backend écoute)

| Topic | Device type | Sécurité | Source |
|---|---|---|---|
| `unifyIots/controller/reader-nfc-01/get` | NFC Reader | TLS (8883) | device.seeder.ts |
| `unifyIots/controller/writer-nfc-01/get` | NFC Writer | TLS (8883) | device.seeder.ts |
| `unifyIots/sensor/custom-sensor-dth-01/get` | Capteur temp/hum | Par défaut | device.seeder.ts |
| `unifyIots/API/logo/get` | Siemens LOGO 8.4 | Plain (1883) | device.seeder.ts (isSecure:false) |
| `{device.subscribe}` (dynamique) | Tout device actif | Selon `isSecure` | reload-mqtt.service.ts |

**Mécanisme de routage** (dans `mqtt.service.ts`, handler `on("message")`) :

```
Si topic contient "/controller/reader-nfc-"  →  handleReaderAccess()
Si topic contient "/sensor/" OU "/API/"      →  processTemperature()
Toujours                                     →  dataHistoriesRepository.saveDataHistory()
```

### 3.3 Topics Publish (Backend envoie)

| Topic | Payload | Contexte | Fichier source |
|---|---|---|---|
| `{reader.publish}` | `{ received: true, uid }` | ACK lecture badge NFC (step 0) | handleReaderAccess.ts |
| `{reader.publish}` | `{ keyA: "..." }` | Envoi keyA au reader NFC (step 0) | handleReaderAccess.ts |
| `{reader.publish}` | `{ access: "granted"\|"denied" }` | Résultat validation badge (step 1) | handleReaderAccess.ts |
| `{reader.publish}` | `{ uid, access: "denied", source: "badge-not-found" }` | Badge non trouvé | handleReaderAccess.ts |
| `unifyIots/API/logo/set` | `{ state: { open: { value: [1] } } }` | Ouverture porte après badge OK | handleReaderAccess.ts |
| `unifyIots/API/logo/set` | `{ state: { doorAlert: { value: [1] } } }` | Alerte module NFC absent | handleReaderAccess.ts |
| `unifyIots/API/logo/set` | `{ state: { clearAlert: { value: [1] } } }` | Levée d'alerte après 60s | handleReaderAccess.ts |
| `{logo.publish}` | `{ state: { temperature: { value: [T] }, heatSetpoint: { value: [1850] } } }` | Publication température min vers LOGO | logo-publisher.service.ts |
| `{device.publish}` | `"isOnline"` (JSON string) | Ping pour vérifier statut device | ping-pong.service.ts |
| `{publishTopic}` | `{setCommand}` (configurable) | Commande contrôleur | monitorControllerPresence.ts |

### 3.4 Topics dynamiques (base de données)

Chaque device en DB possède deux champs optionnels :

- `subscribe: string | null` — topic sur lequel le backend écoute les messages du device
- `publish: string | null` — topic sur lequel le backend envoie des commandes au device

**Seeders (données initiales)** :

| Device | deviceId | subscribe | publish | isSecure |
|---|---|---|---|---|
| Capteur DHT | `custom-sensor-dth-01` | `unifyIots/sensor/custom-sensor-dth-01/get` | *(null)* | true (défaut) |
| Writer NFC | `writer-nfc-01` | `unifyIots/controller/writer-nfc-01/get` | `unifyIots/controller/writer-nfc-01/set` | true (défaut) |
| Reader NFC | `reader-nfc-01` | `unifyIots/controller/reader-nfc-01/get` | `unifyIots/controller/reader-nfc-01/set` | true (défaut) |
| LOGO 8.4 | `logo-01` | `unifyIots/API/logo/get` | `unifyIots/API/logo/set` | **false** |

---

## 4. Payloads et formats de messages

### 4.1 Messages NFC Reader → Backend (subscribe)

**Step 0 — Présentation badge** :
```json
{
  "uid": "A1B2C3D4",
  "seed": "1dea7ca1d7c819bb12cce6dc7c51d56c",
  "step": "0"
}
```

**Step 1 — Vérification clé dérivée** :
```json
{
  "derivedKey": "hexadecimal...",
  "uid": "A1B2C3D4",
  "step": "1"
}
```

**Message système** :
```json
{ "message": "PN532 ready !" }
```

**Erreur module** :
```json
{ "error": "nfc_module_absent" }
```

### 4.2 Messages Backend → NFC Reader (publish)

```json
{ "received": true, "uid": "A1B2C3D4" }
{ "keyA": "hexadecimal..." }
{ "access": "granted" }
{ "access": "denied" }
{ "uid": "A1B2C3D4", "access": "denied", "source": "badge-not-found" }
```

### 4.3 Messages Capteurs → Backend (subscribe)

**Format capteur température/humidité** :
```json
{
  "payload": {
    "temperature": 22.5,
    "humidity": 45.3
  }
}
```

Le service `extractTemperatureFromMessage()` cherche les clés `temperature`, `temp`, ou `t` dans `payload`, puis multiplie par 100 (centièmes de degré).

### 4.4 Messages Backend → LOGO PLC (publish)

**Température + consigne chauffage** :
```json
{
  "state": {
    "temperature": { "value": [2250] },
    "heatSetpoint": { "value": [1850] }
  }
}
```

**Commandes de contrôle** :
```json
{ "state": { "open": { "value": [1] } } }
{ "state": { "doorAlert": { "value": [1] } } }
{ "state": { "clearAlert": { "value": [1] } } }
```

### 4.5 Messages LOGO PLC → Backend (subscribe)

**Format Siemens LOGO 8.4 (state/reported)** :
```json
{
  "state": {
    "reported": {
      "courant": { "value": [12.5] },
      "tension": { "value": [230] },
      "EActivDirect": { "value": [1500] },
      "EActivTotal": { "value": [50000] },
      "HeaterStatus": { "value": [1] }
    }
  }
}
```

### 4.6 Ping-Pong (vérification statut)

**Backend → Device (publish)** :
```json
"isOnline"
```
(Note : c'est un JSON string, pas un objet — `JSON.stringify("isOnline")`)

**Device → Backend (subscribe, réponse attendue)** :
```json
{ "isOnline": "true" }
```
ou
```json
{ "isOnline": "1" }
```

---

## 5. Clients MQTT — Qui publie, qui souscrit

### 5.1 Vue matricielle

| Composant | Rôle | Subscribe topics | Publish topics | Port |
|---|---|---|---|---|
| **MqttService (plain)** | Client broker | Tous les devices (via reloadAllSubscribers) | *(dépend du contexte)* | 1883 |
| **MqttService (TLS)** | Client broker | Tous les devices (via reloadAllSubscribers) | *(dépend du contexte)* | 8883 |
| **handleReaderAccess** | Consommateur | `/controller/reader-nfc-*/get` | `{reader.publish}` + `unifyIots/API/logo/set` | 8883 (hardcodé) |
| **processTemperature** | Consommateur | `/sensor/*/get` + `/API/*/get` | *(indirect via logo-publisher)* | — |
| **logo-publisher.service** | Producteur | *(aucun)* | `{logo.publish}` (ex: `unifyIots/API/logo/set`) | 1883 (hardcodé) |
| **ping-pong.service** | Producteur + Consommateur | `{device.subscribe}` (réponses) | `{device.publish}` (ping "isOnline") | Selon `device.isSecure` |
| **monitorControllerPresence** | Producteur + Consommateur | `{subscribeTopic}` (réponse) | `{publishTopic}` (commande) | 8883 (hardcodé) |
| **addDevice.route** | Gestionnaire | `{device.subscribe}` (ensureSubscription) | *(aucun)* | Selon `device.isSecure` |
| **updateDevice.route** | Gestionnaire | `{device.subscribe}` (ensure + unsubscribe ancien) | *(aucun)* | Selon `device.isSecure` |
| **DataHistoriesRepository** | Persistance | *(indirectement, via MqttService)* | *(aucun MQTT, publie vers WebSocket via RealtimeHub)* | — |

### 5.2 Devices IoT physiques (côté terrain)

| Device | Publie vers broker | Souscrit depuis broker | Protocole |
|---|---|---|---|
| Reader NFC (ESP32) | `unifyIots/controller/reader-nfc-XX/get` | `unifyIots/controller/reader-nfc-XX/set` | TLS |
| Writer NFC | `unifyIots/controller/writer-nfc-01/get` | `unifyIots/controller/writer-nfc-01/set` | TLS |
| Capteur DHT | `unifyIots/sensor/custom-sensor-dth-01/get` | *(aucun subscribe connu)* | TLS (défaut) |
| Siemens LOGO 8.4 | `unifyIots/API/logo/get` | `unifyIots/API/logo/set` | Plain |

---

## 6. Chaîne de traitement des messages (pipeline)

### 6.1 Pipeline global à la réception

```
Message MQTT reçu sur topic T
    │
    ├─── Si topic contient "/controller/reader-nfc-"
    │         └─── handleReaderAccess(topic, message)
    │               ├─── Étape 0 : validation UID + envoi keyA
    │               └─── Étape 1 : validation derivedKey + accès granted/denied
    │                         └─── Publish → LOGO (open porte)
    │                         └─── Log dans AccessLogEntity
    │
    ├─── Si topic contient "/sensor/" OU "/API/"
    │         └─── processTemperature(topic, message)
    │               └─── Extraction temp → cache en mémoire (Map)
    │
    └─── Toujours (après handler spécifique)
              └─── dataHistoriesRepository.saveDataHistory(topic, message)
                    ├─── Recherche device par topic (findOneBy subscribe = topic)
                    ├─── Parse JSON payload
                    ├─── KeyMapper.map(device, payload) → événements clé/valeur
                    │       ├─── DSL Adapter (metadata JSONPath rules)
                    │       ├─── Siemens LOGO 8.4 Adapter (state.reported)
                    │       └─── Generic Reported Value Adapter (fallback)
                    ├─── RealtimeHub.publish(deviceId, key, value) → WebSocket clients
                    └─── Persist DataHistoryEntity (device, timestamp, payload)
```

### 6.2 Pipeline publication température LOGO (CRON)

```
CRON toutes les 5 secondes
    │
    └─── logoPublishJob()
          └─── scheduleLogoPublishingTick()
                ├─── Cherche device model "logo 8.4" en DB
                ├─── Filtre temperatureCache (entrées < 5s)
                ├─── Prend la température la plus basse
                └─── publishTemperatureToLogo(temp, logo)
                      └─── MqttService.getInstance(1883).publish(logo.publish, ...)
```

### 6.3 Pipeline ping-pong (CRON)

```
CRON toutes les 5 secondes (+ 1 appel au démarrage)
    │
    └─── pingPongMqttDevices()
          └─── Pour chaque device actif avec publish:
                ├─── Publish "isOnline" sur device.publish
                ├─── setTimeout(2s) → marque device offline
                └─── registerSubscription(device.subscribe)
                      └─── Si réponse isOnline === "true"|"1"
                            └─── clearTimeout + marque device online
```

---

## 7. Pont MQTT → WebSocket (temps réel)

### 7.1 Architecture du pont

```
MQTT Message reçu
    └─── DataHistoriesRepository.saveDataHistory()
          └─── KeyMapper.map(device, payload)
                └─── [KeyValueEvent{deviceId, key, value}]
                      └─── RealtimeHub.publish(deviceId, key, value)
                            └─── Pour chaque WebSocket abonné au canal "deviceId::key"
                                  └─── ws.send({ deviceId, key, value })
```

### 7.2 KeyMapper — Adapters

Le `KeyMapper` transforme les payloads MQTT bruts en événements `{deviceId, key, value}` normalisés. Trois adapters sont enregistrés par ordre de priorité :

1. **DSL Adapter** (priorité haute) : utilise les règles JSONPath définies dans `device.metadata`. Exemple :
   ```json
   {
     "format": "jsonpath",
     "rules": [
       { "out": "temperature", "path": "$.payload.temperature", "cast": "float" },
       { "out": "humidity", "path": "$.payload.humidity", "cast": "float" }
     ]
   }
   ```

2. **Siemens LOGO 8.4 Adapter** : match si `device.brand` contient "siemens" et `device.model` contient "logo 8.4". Extrait les clés depuis `state.reported.{key}.value`.

3. **Generic Reported Value Adapter** (fallback) : match si le payload contient `state.reported`. Même logique d'extraction que le LOGO adapter.

### 7.3 RealtimeHub

Le `RealtimeHub` est un singleton en mémoire qui gère les abonnements WebSocket :

- **Canaux** : format `deviceId::key` (ex: `e3c2a1b4-7f8d-4c2e-9a6b-5d1f3e2c4b7a::tension`)
- **Subscribe/Unsubscribe** : via messages WebSocket `{ action: "subscribe", devicesAnsKeys: [{deviceId, key}] }`
- **Nettoyage** : automatique à la fermeture du socket (`drop(ws)`)

### 7.4 Route WebSocket Charts

Endpoint : `GET /ws/charts` (défini dans `charts.ws.ts`)

- **Authentification** : JWT via cookie `access_token` (vérifié par `verifyJwtFromCookie`)
- **Origin check** : whitelist d'origins autorisées
- **Commandes** : `subscribe` et `unsubscribe` avec liste de paires `{deviceId, key}`

---

## 8. Tâches CRON liées à MQTT

Configurées dans `cron-setup.ts`, déclenchées si `CRON_JOB=true` :

| Cron | Fréquence | Fonction | Impact MQTT |
|---|---|---|---|
| `logoPublishJob` | Toutes les 5 secondes | Publie temp min vers LOGO PLC | 1 publish vers `{logo.publish}` |
| `pingPongMqttDevices` | Toutes les 5 secondes | Vérifie la présence de chaque device | N publishes + N subscribes (1 par device actif) |

**Constat critique** : le `pingPongMqttDevices` appelle `registerSubscription()` à chaque exécution (toutes les 5s). Cela **écrase le callback** existant à chaque cycle, potentiellement en conflit avec les callbacks de routage du `MqttService.on("message")`. De plus, chaque appel crée un nouveau `setTimeout` sans annuler les précédents si le device n'a pas encore répondu. (**Sévérité 4**)

---

## 9. Analyse de compatibilité zigbee2mqtt

### 9.1 Qu'est-ce que zigbee2mqtt ?

zigbee2mqtt est un pont qui convertit les messages Zigbee en MQTT. Il publie les données des devices Zigbee sous des topics de la forme :

```
zigbee2mqtt/{friendly_name}
zigbee2mqtt/{friendly_name}/set
zigbee2mqtt/{friendly_name}/get
zigbee2mqtt/{friendly_name}/availability
zigbee2mqtt/bridge/state
zigbee2mqtt/bridge/info
zigbee2mqtt/bridge/devices
zigbee2mqtt/bridge/groups
zigbee2mqtt/bridge/event
```

Les payloads sont des JSON "plats" ou légèrement imbriqués :
```json
{ "temperature": 22.5, "humidity": 45.3, "battery": 87, "linkquality": 120 }
```

### 9.2 Compatibilité avec l'architecture actuelle

| Aspect | État actuel | Compatibilité zigbee2mqtt | Action requise |
|---|---|---|---|
| **Convention de topics** | `unifyIots/{type}/{id}/{get\|set}` | zigbee2mqtt utilise `zigbee2mqtt/{name}` | **Adaptation nécessaire** |
| **Format payload capteurs** | `{ payload: { temperature: 22.5 } }` | `{ temperature: 22.5, humidity: 45 }` (plat) | **Adaptation nécessaire** |
| **Format payload LOGO** | `{ state: { reported: { ... } } }` | N/A (LOGO reste en direct) | Compatible |
| **Routage par pattern** | Match sur `/controller/`, `/sensor/`, `/API/` | Topics `zigbee2mqtt/...` ne matchent aucun pattern | **Adaptation nécessaire** |
| **Stockage topic en DB** | Colonne `subscribe` par device | Peut contenir `zigbee2mqtt/{name}` | Compatible |
| **KeyMapper DSL** | Règles JSONPath dans `metadata` | Peut mapper les payloads plats zigbee2mqtt | **Compatible** ✅ |
| **Subscription dynamique** | `reloadAllSubscribers` + `ensureSubscription` | Fonctionne avec n'importe quel topic | **Compatible** ✅ |
| **Ping-Pong** | Publie `"isOnline"` sur `{device.publish}` | zigbee2mqtt expose `/availability` pour le statut | **Adaptation nécessaire** |
| **isSecure flag** | Détermine port 1883 vs 8883 | zigbee2mqtt se connecte typiquement en local (1883) | Compatible |

### 9.3 Points de blocage identifiés

#### 9.3.1 Routage des messages (Sévérité 4 — Critique)

Le handler `on("message")` dans `mqtt.service.ts` route les messages uniquement selon ces patterns :

```typescript
if(topic.includes("/controller/reader-nfc-")) → handleReaderAccess
else if(topic.includes("/sensor/") || topic.includes("/API/")) → processTemperature
```

Les topics zigbee2mqtt (`zigbee2mqtt/salon_temperature`, etc.) ne matcheront **aucun** de ces patterns. Les messages seront quand même :
- Sauvegardés dans `DataHistoriesRepository` ✅
- Transformés par `KeyMapper` et publiés vers WebSocket ✅

Mais `processTemperature()` ne sera jamais appelé, donc la logique de cache température → publication vers LOGO ne fonctionnera pas pour les capteurs Zigbee.

**Recommandation** : Refactorer le routage pour utiliser un système de handlers enregistrables plutôt que des `if/else` hardcodés. Possibilité d'utiliser le champ `type` du device en DB ou un pattern matching configurable.

#### 9.3.2 Format de payload capteur (Sévérité 3 — Majeur)

Actuellement, `extractTemperatureFromMessage()` cherche la température dans `payload.temperature`, `payload.temp`, ou `payload.t` (imbriqué dans un objet `payload`).

zigbee2mqtt publie des payloads plats : `{ "temperature": 22.5, "humidity": 45 }`.

**Recommandation** : Ce point est partiellement résolu par le `KeyMapper` DSL qui peut mapper des payloads plats via JSONPath (`$.temperature`). Cependant, `processTemperature()` est appelé avant `saveDataHistory()` et utilise sa propre logique d'extraction, contournant le KeyMapper.

#### 9.3.3 Ping-Pong non adapté à zigbee2mqtt (Sévérité 3 — Majeur)

Le système ping-pong publie `"isOnline"` sur le topic `publish` de chaque device et attend une réponse `{ isOnline: "true" }`. Les devices Zigbee ne répondent pas à ce type de commande. zigbee2mqtt expose plutôt :

- `zigbee2mqtt/{name}/availability` → `{ "state": "online" }` ou `{ "state": "offline" }`
- `zigbee2mqtt/bridge/devices` → liste complète avec statut

**Recommandation** : Implémenter un système de monitoring alternatif basé sur les topics `availability` pour les devices zigbee2mqtt, en parallèle du ping-pong existant.

#### 9.3.4 Convention de topics (Sévérité 2 — Mineur)

Les topics actuels suivent `unifyIots/{type}/{id}/{get|set}`. zigbee2mqtt impose sa propre convention `zigbee2mqtt/{friendly_name}`.

**Recommandation** : Ce n'est pas un bloquant — les topics en DB sont libres. Mais il faudra documenter la convention pour les devices Zigbee et potentiellement adapter les patterns de routage.

### 9.4 Ce qui fonctionne déjà pour zigbee2mqtt

1. **Abonnement dynamique** : il suffit d'ajouter un device en DB avec `subscribe = "zigbee2mqtt/salon_temp"` et il sera automatiquement abonné au démarrage.

2. **KeyMapper DSL** : le système de mapping par règles JSONPath dans `metadata` est parfaitement adapté pour mapper les payloads zigbee2mqtt. Exemple :
   ```json
   {
     "format": "jsonpath",
     "rules": [
       { "out": "temperature", "path": "$.temperature", "cast": "float" },
       { "out": "humidity", "path": "$.humidity", "cast": "float" },
       { "out": "battery", "path": "$.battery", "cast": "int" }
     ]
   }
   ```

3. **Persistance + temps réel** : les messages seront sauvegardés et poussés vers les WebSocket clients via le RealtimeHub, permettant l'affichage temps réel dans les charts.

4. **Gestion CRUD devices** : les routes `addDevice` et `updateDevice` gèrent déjà l'abonnement/désabonnement MQTT automatique.

---

## 10. Constats et recommandations

### 10.1 Constats architecturaux

| # | Constat | Sévérité | Fichier(s) |
|---|---|---|---|
| C1 | **Routage hardcodé** : le handler `on("message")` utilise des `if/else` avec `topic.includes()` — non extensible | 4 | mqtt.service.ts:59-63 |
| C2 | **Ping-pong écrase les callbacks** : `registerSubscription()` est appelé toutes les 5s et remplace le callback existant | 4 | ping-pong.service.ts:21 |
| C3 | **Fuite de timeouts** : le ping-pong crée un `setTimeout` par device toutes les 5s sans annuler les précédents | 4 | ping-pong.service.ts:16-19 |
| C4 | **rejectUnauthorized: false** sur TLS — la vérification du certificat serveur est désactivée | 3 | mqtt.service.ts:90 |
| C5 | **Port LOGO hardcodé** : `MqttService.getInstance(1883)` dans logo-publisher — le port est en dur | 3 | logo-publisher.service.ts:104 |
| C6 | **Port NFC hardcodé** : `MqttService.getInstance(8883)` dans handleReaderAccess — le port est en dur | 3 | handleReaderAccess.ts:35 |
| C7 | **heatSetpoint hardcodé** : la valeur `1850` (18.50°C) est en dur dans le code | 2 | logo-publisher.service.ts:100 |
| C8 | **Pas de gestion QoS** : aucun message n'est publié avec un QoS explicite (défaut: 0) | 2 | mqtt.service.ts:163 |
| C9 | **Pas de Last Will and Testament (LWT)** : en cas de déconnexion du backend, le broker ne notifie personne | 2 | server.ts |
| C10 | **reloadAllSubscribers sur les DEUX clients** : tous les devices sont abonnés sur plain ET TLS, sans respecter `isSecure` | 3 | server.ts:102 |
| C11 | **Pas de validation de payload** : les messages MQTT reçus ne sont pas validés contre un schéma avant traitement | 2 | mqtt.service.ts:53-77 |
| C12 | **Pas de gestion de reconnexion explicite** : la lib mqtt.js gère la reconnexion auto, mais aucune stratégie de backoff ou de notification n'est implémentée | 2 | mqtt.service.ts |
| C13 | **Typo dans WebSocket** : `devicesAnsKeys` au lieu de `devicesAndKeys` — API publiée avec une faute | 1 | charts.ws.ts:35,46 |

### 10.2 Recommandations pour l'intégration zigbee2mqtt

| # | Recommandation | Priorité | Impact |
|---|---|---|---|
| R1 | **Refactorer le routage MQTT** : remplacer les `if/else` par un système de handlers enregistrables (pattern → handler), configurable par type de device | Haute | Débloque l'intégration zigbee2mqtt |
| R2 | **Unifier l'extraction de données** : supprimer `processTemperature()` au profit du `KeyMapper` DSL pour toutes les transformations de payload | Haute | Élimine la duplication de logique |
| R3 | **Implémenter monitoring Zigbee** : ajouter un handler pour les topics `zigbee2mqtt/+/availability` en parallèle du ping-pong | Haute | Monitoring devices Zigbee |
| R4 | **Corriger le ping-pong** : stocker les timeouts/callbacks et les nettoyer avant chaque cycle ; ou utiliser un `Map<deviceId, timeout>` | Haute | Corrige fuites mémoire et conflits |
| R5 | **Respecter isSecure dans reloadAllSubscribers** : filtrer les devices par `isSecure` pour n'abonner chaque device que sur le bon client MQTT | Moyenne | Séparation correcte des flux |
| R6 | **Ajouter QoS configurable** : au moins QoS 1 pour les messages critiques (accès NFC, alertes) | Moyenne | Fiabilité des commandes |
| R7 | **Configurer LWT** : publier un message d'état backend via Last Will pour notifier les devices en cas de crash | Moyenne | Résilience système |
| R8 | **Externaliser les constantes** : ports, heatSetpoint, timeouts dans des variables d'environnement ou config | Faible | Maintenabilité |

---

## 11. Synthèse

### 11.1 Résumé quantitatif

| Métrique | Valeur |
|---|---|
| Topics subscribe identifiés (seeders) | 4 |
| Topics publish identifiés (seeders) | 3 |
| Topics publish hardcodés dans le code | 1 (`unifyIots/API/logo/set`) |
| Messages/payloads distincts documentés | 15 |
| Services MQTT backend | 6 (mqtt, reload, ping-pong, handleReader, logo-publisher, monitorController) |
| Adapters KeyMapper | 3 (DSL, Siemens LOGO, Generic Reported) |
| Constats identifiés | 13 |
| Recommandations | 8 |

### 11.2 Score de maturité MQTT

| Critère | Score | Commentaire |
|---|---|---|
| Architecture connexion | 3/5 | Dual connexion OK, mais reloadAll ne respecte pas isSecure |
| Convention de topics | 4/5 | Convention claire `unifyIots/{type}/{id}/{dir}`, cohérente |
| Routage des messages | 2/5 | Hardcodé, non extensible, bloquant pour zigbee2mqtt |
| Sécurité | 2/5 | mTLS présent mais rejectUnauthorized=false, QoS 0, pas de LWT |
| Extensibilité | 3/5 | KeyMapper DSL excellent, mais routage et processTemperature rigides |
| Temps réel (WS bridge) | 4/5 | RealtimeHub bien conçu, adapters propres, auth cookie |
| Prêt pour zigbee2mqtt | 2/5 | Fondations solides (KeyMapper, subscribe dynamique) mais 3 points bloquants |

### 11.3 Verdict global

L'architecture MQTT existante est **fonctionnelle pour le périmètre actuel** (NFC, capteurs propriétaires, LOGO PLC) mais **nécessite des adaptations pour intégrer zigbee2mqtt**. Les trois points bloquants majeurs sont le routage hardcodé des messages, l'extraction de température non unifiée, et le monitoring de présence non compatible avec zigbee2mqtt. Le `KeyMapper` avec son système DSL constitue un excellent point d'entrée pour l'intégration — il suffit de le placer plus tôt dans la chaîne de traitement et de refactorer le routage pour le rendre dynamique.
