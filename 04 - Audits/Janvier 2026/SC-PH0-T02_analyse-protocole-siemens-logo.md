# SC-PH0-T02 — Analyser le protocole de communication Siemens Logo

> **User Story** : SC-US-PH0-05 — En tant que développeur, je veux analyser l'intégration actuelle du Siemens Logo et sa communication MQTT
> **Tâche** : Analyser protocole communication Siemens Logo
> **Projet** : simpl1Control v1.1.2
> **Date** : 10 février 2026
> **Méthode** : Analyse statique du code source (read-only, aucune modification)

---

## Table des matières

1. [Vue d'ensemble de l'intégration](#1-vue-densemble-de-lintégration)
2. [Architecture matérielle et réseau](#2-architecture-matérielle-et-réseau)
3. [Configuration du device Logo en base de données](#3-configuration-du-device-logo-en-base-de-données)
4. [Protocole MQTT : topics et payloads](#4-protocole-mqtt--topics-et-payloads)
5. [Pipeline de réception (Logo → Backend)](#5-pipeline-de-réception-logo--backend)
6. [Pipeline d'émission (Backend → Logo)](#6-pipeline-démission-backend--logo)
7. [Intégration contrôle d'accès NFC → Logo](#7-intégration-contrôle-daccès-nfc--logo)
8. [Adapter KeyMapper : siemensLogo84Adapter](#8-adapter-keymapper--siemenslogo84adapter)
9. [Pipeline temps réel (Logo → WebSocket → Frontend)](#9-pipeline-temps-réel-logo--websocket--frontend)
10. [Analyse de sécurité](#10-analyse-de-sécurité)
11. [Constats et problèmes identifiés](#11-constats-et-problèmes-identifiés)
12. [Recommandations](#12-recommandations)
13. [Synthèse](#13-synthèse)

---

## 1. Vue d'ensemble de l'intégration

### 1.1 Rôle du Siemens Logo 8.4

Le Siemens LOGO! 8.4 est un automate programmable (PLC micro) utilisé dans le projet simpl1Control comme **actuateur de bâtiment** remplissant deux fonctions principales :

- **Contrôle de porte** : ouverture via commande MQTT déclenchée par validation NFC
- **Régulation thermique** : réception de données de température depuis les capteurs, avec consigne de chauffage (heatSetpoint)

Le Logo remonte également des **données de métrologie électrique** (courant, tension, énergie active) et des statuts opérationnels (HeaterStatus, errorStatus).

### 1.2 Inventaire des fichiers impliqués

| Fichier | Rôle dans l'intégration Logo |
|---|---|
| `services/logo-publisher.service.ts` | Publication périodique de température vers le Logo |
| `cronjobs/logo-publish-job.ts` | Job CRON appelant `scheduleLogoPublishingTick()` |
| `cron-setup.ts` | Enregistrement du job Logo toutes les 5 secondes |
| `realtime/key-mapper.ts` | Adapter `siemensLogo84Adapter` pour transformer les payloads Logo |
| `services/handleReaderAccess.ts` | Envoi de commandes d'ouverture de porte et d'alerte au Logo |
| `services/mqtt.service.ts` | Service MQTT singleton — routage des messages Logo |
| `services/ping-pong.service.ts` | Vérification de présence en ligne du Logo |
| `services/reload-mqtt.service.ts` | Réabonnement aux topics Logo au démarrage |
| `publishers/monitorControllerPresence.ts` | Monitoring de présence des contrôleurs via MQTT |
| `features/devices/device.entity.ts` | Modèle TypeORM du device (subscribe, publish, metadata) |
| `features/devices/device.repository.ts` | Repository avec `findByModel("logo 8.4")` |
| `features/data-histories/data-histories.repository.ts` | Persistance des messages MQTT + bridge temps réel |
| `database/seeders/device.seeder.ts` | Données initiales du device Logo |
| `server.ts` | Initialisation MQTT et CRON au démarrage |

### 1.3 Schéma d'architecture global

```
┌─────────────┐       MQTT (1883/8883)       ┌──────────────────┐
│ Siemens     │ ◄──────────────────────────── │                  │
│ LOGO! 8.4   │  unifyIots/API/logo/set       │   Backend        │
│             │ ────────────────────────────► │   Fastify        │
│  - Porte    │  unifyIots/API/logo/get       │                  │
│  - Chauffage│                               │  ┌────────────┐  │
│  - Métrologie│                              │  │ CRON 5s    │  │
└─────────────┘                               │  │ logoPublish│  │
                                              │  └────────────┘  │
┌─────────────┐  unifyIots/controller/        │  ┌────────────┐  │
│ Reader NFC  │ ────────────────────────────► │  │handleReader│  │
│ (PN532)     │ ◄──────────────────────────── │  │Access      │──┼─► Logo /set
└─────────────┘  reader-nfc-01/get|set        │  └────────────┘  │
                                              │  ┌────────────┐  │
┌─────────────┐  unifyIots/sensor/            │  │ KeyMapper  │  │
│ Capteurs DHT│ ────────────────────────────► │  │ ┌────────┐ │  │
│ température │  custom-sensor-dth-01/get     │  │ │Logo84  │ │  │
└─────────────┘                               │  │ │Adapter │ │  │
                                              │  │ └────────┘ │  │
                                              │  └────────────┘  │
                                              │        │         │
                                              │  ┌─────▼──────┐  │     ┌──────────┐
                                              │  │ RealtimeHub│──┼──►  │ Frontend │
                                              │  │ (WS bridge)│  │ WS  │ React 19 │
                                              │  └────────────┘  │     └──────────┘
                                              └──────────────────┘
```

---

## 2. Architecture matérielle et réseau

### 2.1 Configuration réseau

D'après les fichiers d'environnement et la configuration Docker :

| Paramètre | Valeur (development.env) | Observation |
|---|---|---|
| `MQTT_BASE_URL` | `192.168.3.100` | IP fixe sur LAN — **même IP que le registre Docker** |
| `MQTT_PORT_PLAIN` | `1883` | Connexion MQTT non chiffrée |
| `MQTT_PORT_TLS` | `8883` | Connexion MQTT avec TLS mutuel |
| `MQTT_CLIENT_ID_PLAIN` | `client` | ID client pour la connexion plain |
| `MQTT_CLIENT_ID` | `client-backend` | ID client pour la connexion TLS |
| `MQTT_USERNAME` | `admin` | Authentification broker |
| `MQTT_KEEPALIVE` | `60` | Keepalive en secondes |
| `MQTT_START` | `false` | **MQTT désactivé par défaut en dev** |

**Constat C01** : L'adresse `192.168.3.100` sert à la fois de broker MQTT et de registre Docker (`192.168.3.100:5000`). Cela suggère qu'un seul serveur concentre broker MQTT, registre Docker, et potentiellement le Logo lui-même sur le même réseau local. — **Sévérité : 3/5**

### 2.2 Configuration test

| Paramètre | Valeur (test.env) | Observation |
|---|---|---|
| `MQTT_BASE_URL` | `mqtt://192.168.0.102:1883` | IP différente, format incohérent (inclut le protocole) |
| `MQTT_PORT_PLAIN` | **absent** | Manquant — incompatible avec `MqttService.getInstance(port)` |
| `MQTT_PORT_TLS` | **absent** | Manquant |
| `CRON_JOB` | `false` | Jobs Logo désactivés en test |

**Constat C02** : Le `test.env` utilise un format de `MQTT_BASE_URL` incohérent avec `development.env`. En dev c'est juste `192.168.3.100` (sans protocole), en test c'est `mqtt://192.168.0.102:1883` (avec protocole et port). Le code dans `server.ts` préfixe déjà `mqtt://` ou `mqtts://`, donc la valeur test provoquerait une URL doublée comme `mqtt://mqtt://192.168.0.102:1883`. — **Sévérité : 4/5**

### 2.3 Connexion TLS

Le backend supporte la connexion TLS mutuelle au broker MQTT :

```
// mqtt.service.ts ligne 86-91
if (options.baseUrl.startsWith("mqtts://")) {
  options.key  = await fs.readFile("./certs/client-backend.key");
  options.cert = await fs.readFile("./certs/client-backend.crt");
  options.ca   = await fs.readFile("./certs/ca.crt");
  options.rejectUnauthorized = false;  // ⚠ Désactivé !
}
```

**Constat C03** : `rejectUnauthorized = false` désactive la vérification du certificat serveur, ce qui rend la connexion TLS vulnérable aux attaques man-in-the-middle. — **Sévérité : 4/5**

### 2.4 Configuration du device Logo (seeder)

Le Logo est configuré avec `isSecure: false`, ce qui signifie qu'il communique sur le **port plain 1883** (non chiffré), même si le backend supporte le TLS.

**Constat C04** : Le device Logo est explicitement configuré en mode non sécurisé (`isSecure: false`), mais le service `handleReaderAccess.ts` utilise `MqttService.getInstance(8883)` (port TLS) pour publier les commandes d'ouverture de porte vers le Logo. Il y a **incohérence entre la configuration du device et le port effectivement utilisé** pour les commandes critiques de sécurité physique. — **Sévérité : 5/5**

---

## 3. Configuration du device Logo en base de données

### 3.1 Données du seeder

```typescript
// device.seeder.ts — device Logo
{
  id: 'e3c2a1b4-7f8d-4c2e-9a6b-5d1f3e2c4b7a',
  deviceId: 'logo-01',
  brand: 'siemens',
  model: 'logo 8.4',
  status: DeviceStatus.ACTIVE,
  type: 'API',
  publish: 'unifyIots/API/logo/set',
  subscribe: 'unifyIots/API/logo/get',
  isSecure: false,
  roles: [Role.ADMIN, Role.EMPLOYEE],
  keyValues: ["courant", "tension", "EActivDirect",
              "EActivTotal", "EActivPartDir", "HeaterStatus"],
  metadata: {
    state: {
      AUTO:        { value: [0] },
      PROGRAM:     { value: [0] },
      RAZ:         { value: [0] },
      temperature: { value: [0] },
      errorStatus: { value: [0] },
      humidity:    { value: [0] },
      heatSetpoint:{ value: [0] },
    }
  }
}
```

### 3.2 Analyse du modèle de données

| Champ | Valeur | Analyse |
|---|---|---|
| `type` | `"API"` | Distingue le Logo des capteurs (`sensor`) et contrôleurs (`controller`) |
| `keyValues` | 6 clés | Métrologie électrique + chauffage — valeurs affichées sur le dashboard |
| `metadata.state` | 7 clés | Template de l'état initial du Logo — utilisé par le KeyMapper |
| `subscribe` | `unifyIots/API/logo/get` | Topic d'écoute des données remontées par le Logo |
| `publish` | `unifyIots/API/logo/set` | Topic de commande vers le Logo |

### 3.3 Cards associées (dashboard)

Le seeder crée 3 cards de visualisation liées au Logo, regroupées dans une page "Consomation électrique" :

| Card | `keyValue` | Nom affiché |
|---|---|---|
| Card 0 | `tension` | Tension |
| Card 1 | `courant` | Courant |
| Card 2 | `EActivDirect` | Énergie active, Direct |

**Constat C05** : Le Logo possède 6 `keyValues` mais seulement 3 cards sont créées. Les valeurs `EActivTotal`, `EActivPartDir` et `HeaterStatus` ne sont pas visualisées par défaut dans le dashboard. — **Sévérité : 1/5** (cosmétique)

### 3.4 Écart entre keyValues et metadata

Les `keyValues` (6 clés) et les clés de `metadata.state` (7 clés) ne se recoupent **que partiellement** :

| Présent dans `keyValues` | Présent dans `metadata.state` |
|---|---|
| `courant` | ❌ |
| `tension` | ❌ |
| `EActivDirect` | ❌ |
| `EActivTotal` | ❌ |
| `EActivPartDir` | ❌ |
| `HeaterStatus` | ❌ |
| ❌ | `AUTO` |
| ❌ | `PROGRAM` |
| ❌ | `RAZ` |
| ❌ | `temperature` |
| ❌ | `errorStatus` |
| ❌ | `humidity` |
| ❌ | `heatSetpoint` |

**Constat C06** : Aucun chevauchement entre `keyValues` et `metadata.state`. Les `keyValues` représentent les données remontées par le Logo (métrologie), tandis que `metadata.state` représente les commandes/états de contrôle. Cette séparation est implicite et non documentée. Le rôle du champ `metadata` pour le Logo est ambigu : il ne contient pas de règles DSL (format `jsonpath`), donc le `dslAdapter` ne le reconnaît pas, et le `siemensLogo84Adapter` l'ignore aussi (il lit `payload.state.reported`, pas `device.metadata`). — **Sévérité : 3/5**

---

## 4. Protocole MQTT : topics et payloads

### 4.1 Topics MQTT du Logo

| Direction | Topic | QoS | Retain | Usage |
|---|---|---|---|---|
| **Subscribe** (Logo→Backend) | `unifyIots/API/logo/get` | Non spécifié (défaut 0) | Non spécifié | Réception état du Logo |
| **Publish** (Backend→Logo) | `unifyIots/API/logo/set` | Non spécifié (défaut 0) | Non spécifié | Envoi commandes au Logo |

### 4.2 Format des payloads entrants (Logo → Backend)

Le `siemensLogo84Adapter` (key-mapper.ts) attend le format suivant :

```json
{
  "state": {
    "reported": {
      "courant":      { "value": [12.5] },
      "tension":      { "value": [230] },
      "EActivDirect": { "value": [1500] },
      "EActivTotal":  { "value": [45000] },
      "EActivPartDir":{ "value": [800] },
      "HeaterStatus": { "value": [1] },
      "temperature":  { "value": [2150] },
      "errorStatus":  { "value": [0] }
    }
  }
}
```

**Structure** : `state.reported.{clé}.value[0]` — chaque valeur est encapsulée dans un tableau à un élément.

### 4.3 Format des payloads sortants (Backend → Logo)

#### 4.3.1 Commande température (logo-publisher.service.ts)

```json
{
  "state": {
    "temperature": { "value": [2150] },
    "heatSetpoint": { "value": [1850] }
  }
}
```

- `temperature` : valeur entière = température × 100 (ex: 21.50°C = 2150)
- `heatSetpoint` : **hardcodé à 1850** (18.50°C)

#### 4.3.2 Commande ouverture de porte (handleReaderAccess.ts)

```json
{ "state": { "open": { "value": [1] } } }
```

#### 4.3.3 Commande alerte porte (handleReaderAccess.ts)

```json
{ "state": { "doorAlert": { "value": [1] } } }
```

#### 4.3.4 Commande effacement alerte (handleReaderAccess.ts)

```json
{ "state": { "clearAlert": { "value": [1] } } }
```

### 4.4 Convention de nommage des payloads

Le format `{ "value": [x] }` est une convention propriétaire du projet simpl1Control, inspirée du format AWS IoT Shadow mais simplifiée. Ce n'est **pas un standard Siemens** ni un protocole Logo natif. Le Logo communique normalement via le protocole S7 (TCP/IP), pas MQTT. L'intégration MQTT implique donc un **gateway intermédiaire** (non présent dans le code source) qui traduit entre S7 et MQTT.

**Constat C07** : Le gateway Logo↔MQTT n'est pas documenté ni présent dans le code source. L'intégration repose sur un composant externe non versionné dont le format de payload est implicitement défini par le code de l'adapter. — **Sévérité : 4/5**

---

## 5. Pipeline de réception (Logo → Backend)

### 5.1 Flux complet

```
Logo → [Gateway?] → Broker MQTT → mqtt.service.ts → data-histories.repository.ts
                                       │                      │
                                       │                ┌─────▼─────┐
                                       │                │ keyMapper │
                                       │                │ .map()    │
                                       │                └─────┬─────┘
                                       │                      │
                                       │                ┌─────▼─────┐
                                       ▼                │realtimeHub│
                                  processTemperature()  │.publish() │
                                  (logo-publisher)      └─────┬─────┘
                                       │                      │
                                       ▼                      ▼
                                  temperatureCache       WebSocket
                                  (Map in-memory)        → Frontend
```

### 5.2 Étape 1 : Réception MQTT (mqtt.service.ts)

Le handler `on("message")` de `mqtt.service.ts` (lignes 53-77) décide du traitement selon le topic :

```typescript
if (topic.includes("/controller/reader-nfc-")) {
  await handleReaderAccess({ topic, message: message.toString() });
} else if (topic.includes("/sensor/") || topic.includes("/API/")) {
  await processTemperature({ topic, message: message.toString() });
}
// Puis toujours : saveDataHistory
await this.dataHistoriesRepository.saveDataHistory({ topic, message: message.toString() });
```

**Constat C08** : Le routage est basé sur `topic.includes()` ce qui est fragile. Un topic comme `unifyIots/sensor/controller/reader-nfc-test/get` satisferait les deux conditions mais seule la première branche (`handleReaderAccess`) serait exécutée. De plus, le topic du Logo (`/API/`) est traité par `processTemperature()` qui extrait uniquement la température, ignorant toutes les autres données (courant, tension, etc.) — **Sévérité : 3/5**

### 5.3 Étape 2 : Extraction de température (logo-publisher.service.ts)

La fonction `processTemperature()` tente d'extraire une température du message :

```typescript
function extractTemperatureFromMessage(message: string): number | null {
  const parsed = JSON.parse(message);
  const payload = parsed?.payload;
  // Cherche dans: "temperature", "temp", "t"
  const keys = ["temperature", "temp", "t"];
  // Multiplie par 100 et arrondit
  const temp = Math.round(typeof raw === "string" ? parseFloat(raw) * 100 : raw * 100);
}
```

**Constat C09** : La fonction cherche la température dans `parsed.payload.temperature`, mais le format Logo entrant (défini par le `siemensLogo84Adapter`) est `{ state: { reported: { temperature: { value: [x] } } } }`. Ces deux chemins sont **incompatibles**. Si le gateway Logo envoie au format `state.reported`, alors `extractTemperatureFromMessage` ne trouvera jamais la température car elle cherche dans `payload.temperature`. Inversement, si le gateway envoie au format `{ payload: { temperature: x } }` (format capteur DHT), alors le `siemensLogo84Adapter` ne reconnaîtra pas ce format. — **Sévérité : 5/5**

### 5.4 Étape 3 : Persistance et temps réel (data-histories.repository.ts)

Indépendamment de `processTemperature()`, la méthode `saveDataHistory()` est **toujours appelée** :

1. Recherche le device par topic subscribe
2. Parse le JSON du message
3. Appelle `keyMapper.map(device, payload)` → événements temps réel
4. Publie sur le `RealtimeHub` (WebSocket)
5. Persiste en base de données (`data_histories`)

Cette étape fonctionne correctement pour le Logo car le `siemensLogo84Adapter` est bien adapté au format `state.reported.{key}.value[0]`.

---

## 6. Pipeline d'émission (Backend → Logo)

### 6.1 Publication périodique de température

#### 6.1.1 Job CRON

Le job `logoPublishJob` est exécuté toutes les **5 secondes** via `cron-setup.ts` :

```typescript
cronService.addTask(parseCronExpression('*/5 * * * * *'), logoPublishJob);
```

#### 6.1.2 Logique de `scheduleLogoPublishingTick()`

```
1. deviceRepository.findByModel("logo 8.4")  → trouve le device Logo
2. Parcourt temperatureCache (Map in-memory)
3. Filtre les entrées de moins de 5 secondes
4. Sélectionne la température la plus basse (Math.min)
5. Publie vers logo.publish via MqttService.getInstance(1883)
```

**Constat C10** : La publication utilise `MqttService.getInstance(1883)` (port plain, non sécurisé), ce qui est cohérent avec `isSecure: false` du device Logo. Mais cela signifie que les **commandes de température transitent en clair** sur le réseau. — **Sévérité : 3/5**

#### 6.1.3 Valeur heatSetpoint hardcodée

```typescript
function publishTemperatureToLogo(temp: number, logo: DeviceEntity) {
  const newState = {
    state: {
      temperature: { value: [temp] },
      heatSetpoint: { value: [1850] },  // ← HARDCODÉ
    },
  };
}
```

**Constat C11** : La consigne de chauffage (`heatSetpoint`) est **hardcodée à 1850** (18.50°C). Il n'existe aucune interface (API REST, frontend, configuration) pour modifier cette valeur. Pour changer la consigne, il faut modifier le code source et redéployer. — **Sévérité : 4/5**

#### 6.1.4 Problème de la température la plus basse

La logique sélectionne `Math.min(...validTemps)` parmi toutes les températures reçues dans les 5 dernières secondes. Si plusieurs capteurs rapportent des températures, seule la plus basse est envoyée au Logo.

**Constat C12** : La stratégie "température la plus basse" est un choix métier non documenté. Si un capteur mal calibré envoie une valeur anormalement basse, le Logo recevra cette valeur erronée et pourrait déclencher un chauffage excessif. Il n'y a aucun mécanisme de filtrage d'anomalies. — **Sévérité : 3/5**

### 6.2 Instanciation du repository à chaque tick

```typescript
export async function scheduleLogoPublishingTick() {
  const deviceRepository = new DeviceRepositoryImpl();  // ← Nouvelle instance à chaque appel
  const logo = await deviceRepository.findByModel("logo 8.4");
```

**Constat C13** : Un nouveau `DeviceRepositoryImpl` est créé à chaque tick (toutes les 5 secondes), ce qui instancie un nouveau repository et effectue une requête SQL. Cela représente **~17 280 requêtes par jour** (5s × 60 × 24 = 17 280) juste pour chercher le device Logo. Ce device pourrait être mis en cache. — **Sévérité : 2/5**

---

## 7. Intégration contrôle d'accès NFC → Logo

### 7.1 Flux complet

```
Badge NFC → Reader NFC (PN532) → MQTT → handleReaderAccess() → Logo (porte)
```

### 7.2 Protocole en 2 étapes (state machine)

Le contrôle d'accès utilise un protocole challenge-response en 2 étapes :

#### Étape 0 : `STATE_STEP_VERIFYING_KEY_A`

```
Reader → Backend : { uid: "hex", seed: "hex", step: "0" }
Backend → Reader : { received: true, uid: "hex" }
Backend → DB    : findOne({ cardId: Buffer.from(uid, 'hex') })
Si badge trouvé :
  Backend → Reader : { keyA: "hex" }
Sinon :
  Backend → Reader : { uid, access: 'denied', source: 'badge-not-found' }
  Backend → DB    : AccessLog { outcome: 'denied' }
```

#### Étape 1 : `STATE_STEP_WAITING_ACCESS_GRANTED`

```
Reader → Backend : { derivedKey: "hex", uid: "hex", seed: "hex", step: "1" }
Backend → DB    : findOne({ cardId: Buffer.from(uid, 'hex') })
Si badge trouvé ET !deniedAccessFlag :
  Backend → validateBadgeWithKey(derivedKey, cardId, userId)
  Si validation OK :
    Backend → Reader : { access: 'granted' }
    Backend → Logo   : { state: { open: { value: [1] } } }  ← OUVERTURE PORTE
    Backend → DB     : AccessLog { outcome: 'granted' }
  Sinon :
    Backend → Reader : { access: 'denied' }
    Backend → DB     : AccessLog { outcome: 'denied' }
```

### 7.3 Gestion des alertes NFC

```
Si error === 'nfc_module_absent' :
  Backend → Logo : { state: { doorAlert: { value: [1] } } }
  Après 60 secondes sans nouveau message :
    Backend → Logo : { state: { clearAlert: { value: [1] } } }
```

### 7.4 Analyse du code handleReaderAccess

**Constat C14** : Le topic de publication vers le Logo est **hardcodé** dans `handleReaderAccess.ts` comme `unifyIots/API/logo/set`. Il ne consulte pas le champ `publish` du device Logo en base de données. Si le topic change, il faudra modifier le code source. — **Sévérité : 4/5**

**Constat C15** : Le `handleReaderAccess` utilise `MqttService.getInstance(8883)` (port TLS) pour toutes ses publications, y compris vers le Logo. Or le device Logo est configuré `isSecure: false` (port 1883). Le service publie donc les commandes d'ouverture de porte via l'instance TLS, mais le Logo écoute potentiellement sur l'instance plain. Cela ne fonctionnerait que si le broker route les messages entre les deux connexions, ce qui dépend de la configuration du broker. — **Sévérité : 5/5** (cf. aussi C04)

**Constat C16** : La variable `nfcAbsentTimeout` est un `setTimeout` global (module-level). Si le serveur reçoit plusieurs erreurs `nfc_module_absent` simultanément depuis différents readers, le timeout est écrasé à chaque fois par `clearTimeout(nfcAbsentTimeout)`. De plus, les variables `nfcAbsentTimeout` et `lastNfcAbsent` ne sont pas thread-safe au sens asynchrone (deux `handleReaderAccess` concurrents pourraient interférer). — **Sévérité : 3/5**

**Constat C17** : Lorsqu'un badge a `deniedAccessFlag = true`, la step 1 (`STATE_STEP_WAITING_ACCESS_GRANTED`) ne fait **rien** : ni publication de refus, ni log d'accès, ni retour au reader. Le reader reste en attente sans réponse. — **Sévérité : 4/5**

---

## 8. Adapter KeyMapper : siemensLogo84Adapter

### 8.1 Code de l'adapter

```typescript
const siemensLogo84Adapter: Adapter = {
  name: "siemens-logo-8-4",
  matches: (device) => {
    const brand = (device.brand || "").toLowerCase();
    const model = (device.model || "").toLowerCase();
    return brand.includes("siemens") && model.includes("logo 8.4");
  },
  map: (device, payload) => {
    const reported = payload?.state?.reported;
    if (!reported || typeof reported !== "object") return [];
    for (const [key, node] of Object.entries(reported)) {
      const v = (node as any).value;
      const value = Array.isArray(v) ? (v.length ? v[0] : undefined)
                  : v !== undefined ? v : undefined;
      if (value === undefined) continue;
      out.push({ deviceId: device.id, key, value });
    }
    return out;
  },
};
```

### 8.2 Chaîne d'adaptation

L'ordre des adapters dans le `KeyMapper` est crucial :

```
1. dslAdapter       — si metadata contient des règles JSONPath
2. siemensLogo84Adapter — si brand="siemens" et model="logo 8.4"
3. genericReportedValueAdapter — fallback pour tout payload avec state.reported
```

### 8.3 Analyse

**Constat C18** : Le `siemensLogo84Adapter` et le `genericReportedValueAdapter` ont **exactement le même code** dans leur méthode `map()`. La seule différence est le critère `matches()` : le Logo adapter vérifie brand+model, le generic vérifie juste la structure du payload. Cela signifie que si le Logo adapter n'existait pas, le generic ferait exactement le même travail. L'adapter Logo est redondant dans l'état actuel du code. — **Sévérité : 1/5**

**Constat C19** : Le metadata du Logo dans le seeder n'utilise pas le format DSL (`{ format: "jsonpath", rules: [...] }`). Il contient un template d'état initial `{ state: { AUTO: { value: [0] } } }` qui ne sert à aucun des adapters. Le `dslAdapter` ne matchera pas (pas de `rules`), et le `siemensLogo84Adapter` ignore le metadata. Ce champ metadata est donc **inerte** pour le Logo. — **Sévérité : 2/5**

---

## 9. Pipeline temps réel (Logo → WebSocket → Frontend)

### 9.1 Flux

```
Message MQTT reçu
  → data-histories.repository.saveDataHistory()
    → keyMapper.map(device, payload)
      → siemensLogo84Adapter.map()
        → [ {deviceId, key: "courant", value: 12.5}, ... ]
    → realtimeHub.publish(deviceId, key, value)
      → WebSocket.send({ deviceId, key, value })
        → Frontend : affichage temps réel sur les cards
```

### 9.2 Fonctionnement du RealtimeHub

Le `RealtimeHub` (singleton) maintient un registre de canaux au format `{deviceId}::{key}` :

- `subscribe(ws, pairs)` : enregistre un client WebSocket sur un ou plusieurs canaux
- `publish(deviceId, key, value)` : diffuse à tous les clients abonnés au canal
- `drop(ws)` : nettoie toutes les subscriptions d'un client

### 9.3 Route WebSocket `/ws/charts`

La route `charts.ws.ts` gère les connexions WebSocket pour les graphiques temps réel :
- Authentification par cookie JWT
- Messages `subscribe` / `unsubscribe` avec des paires `{ deviceId, key }`
- Nettoyage automatique à la déconnexion via `realtimeHub.drop()`

### 9.4 Impact pour le Logo

Les valeurs du Logo transitent par le même pipeline que tous les autres devices. Le frontend affiche les cards "Tension", "Courant", "Énergie active, Direct" avec mise à jour en temps réel via WebSocket.

**Constat C20** : Le temps réel fonctionne correctement pour les données Logo, mais uniquement pour les clés présentes dans `state.reported`. Les commandes envoyées au Logo (`open`, `doorAlert`, `clearAlert`, `temperature`, `heatSetpoint`) ne sont jamais reflétées dans le frontend. Il n'y a pas de retour d'état pour confirmer que le Logo a bien exécuté une commande. — **Sévérité : 3/5**

---

## 10. Analyse de sécurité

### 10.1 Points critiques

| Aspect | État | Risque |
|---|---|---|
| Commande d'ouverture de porte | Via MQTT, topic hardcodé | Un accès au broker permet d'ouvrir la porte |
| Chiffrement MQTT | TLS disponible mais Logo en `isSecure: false` | Commandes en clair sur le réseau |
| Vérification certificat | `rejectUnauthorized: false` | MitM possible même sur TLS |
| Authentification broker | Username/password en `.env` | Credentials en clair dans le fichier |
| Topic de commande Logo | Non protégé par ACL (pas visible dans le code) | Tout client MQTT peut publier |
| Validation des commandes | Aucune — le Logo exécute tout ce qu'il reçoit | Pas de vérification d'origine |

### 10.2 Scénario d'attaque

Un attaquant ayant accès au réseau local (192.168.3.0/24) peut :

1. Se connecter au broker MQTT (port 1883, credentials en clair dans le code)
2. Publier sur `unifyIots/API/logo/set` : `{ "state": { "open": { "value": [1] } } }`
3. Ouvrir la porte physique sans badge NFC

**Constat C21** : L'ensemble du protocole de commande Logo repose sur la sécurité du réseau local, sans aucune couche d'authentification ou de signature au niveau applicatif MQTT. Les credentials MQTT sont identiques pour tous les clients. — **Sévérité : 5/5**

---

## 11. Constats et problèmes identifiés

| # | Constat | Sévérité | Fichier(s) concerné(s) |
|---|---|---|---|
| C01 | Serveur unique concentrant broker, registre Docker, et potentiellement le Logo | 3/5 | `development.env`, `docker-compose.production.yaml` |
| C02 | Format `MQTT_BASE_URL` incohérent entre dev et test (protocole inclus/exclus) | 4/5 | `development.env`, `test.env` |
| C03 | `rejectUnauthorized = false` désactive la vérification du certificat TLS serveur | 4/5 | `mqtt.service.ts` |
| C04 | Incohérence `isSecure: false` du Logo vs `MqttService.getInstance(8883)` dans handleReaderAccess | 5/5 | `device.seeder.ts`, `handleReaderAccess.ts` |
| C05 | 6 keyValues mais seulement 3 cards créées pour le Logo | 1/5 | `device.seeder.ts` |
| C06 | Aucun chevauchement entre `keyValues` et `metadata.state` — séparation implicite non documentée | 3/5 | `device.seeder.ts` |
| C07 | Gateway Logo↔MQTT non documenté ni versionné dans le code source | 4/5 | — |
| C08 | Routage MQTT par `topic.includes()` fragile et messages Logo traités par `processTemperature()` uniquement | 3/5 | `mqtt.service.ts` |
| C09 | `extractTemperatureFromMessage` cherche `payload.temperature` mais le format Logo est `state.reported.temperature.value[0]` — incompatibilité de format | 5/5 | `logo-publisher.service.ts`, `key-mapper.ts` |
| C10 | Commandes de température publiées en clair (port 1883) | 3/5 | `logo-publisher.service.ts` |
| C11 | `heatSetpoint` hardcodé à 1850 (18.50°C) sans interface de configuration | 4/5 | `logo-publisher.service.ts` |
| C12 | Stratégie "température la plus basse" sans filtrage d'anomalies | 3/5 | `logo-publisher.service.ts` |
| C13 | Nouveau `DeviceRepositoryImpl` instancié à chaque tick CRON (toutes les 5s) | 2/5 | `logo-publisher.service.ts` |
| C14 | Topic Logo hardcodé `unifyIots/API/logo/set` dans handleReaderAccess au lieu de lire le champ `publish` en DB | 4/5 | `handleReaderAccess.ts` |
| C15 | handleReaderAccess publie sur le port TLS (8883) mais le Logo est configuré `isSecure: false` (1883) | 5/5 | `handleReaderAccess.ts`, `device.seeder.ts` |
| C16 | Variables globales `nfcAbsentTimeout` / `lastNfcAbsent` non isolées par device | 3/5 | `handleReaderAccess.ts` |
| C17 | Badge avec `deniedAccessFlag = true` ne génère ni refus ni log en step 1 | 4/5 | `handleReaderAccess.ts` |
| C18 | `siemensLogo84Adapter` identique au `genericReportedValueAdapter` dans la méthode `map()` — redondant | 1/5 | `key-mapper.ts` |
| C19 | Metadata du Logo inerte — ni format DSL, ni utilisé par le siemensLogo84Adapter | 2/5 | `device.seeder.ts`, `key-mapper.ts` |
| C20 | Pas de retour d'état des commandes Logo (open, alert) dans le frontend | 3/5 | `handleReaderAccess.ts`, `realtime-hub.ts` |
| C21 | Aucune authentification ou signature applicative sur les commandes Logo — sécurité physique repose uniquement sur le réseau | 5/5 | `handleReaderAccess.ts`, `mqtt.service.ts` |

### Distribution par sévérité

| Sévérité | Nombre | Constats |
|---|---|---|
| **5/5 — Critique** | 4 | C04, C09, C15, C21 |
| **4/5 — Majeur** | 6 | C02, C03, C07, C11, C14, C17 |
| **3/5 — Significatif** | 7 | C01, C06, C08, C10, C12, C16, C20 |
| **2/5 — Mineur** | 2 | C13, C19 |
| **1/5 — Cosmétique** | 2 | C05, C18 |

---

## 12. Recommandations

### 12.1 Sécurité (priorité haute)

| # | Recommandation | Constats liés |
|---|---|---|
| R01 | **Résoudre l'incohérence de port MQTT** : soit configurer le Logo en `isSecure: true` et utiliser le port TLS partout, soit modifier `handleReaderAccess.ts` pour utiliser le port cohérent avec la config du device. Actuellement les commandes de porte passent par le mauvais port. | C04, C15 |
| R02 | **Activer `rejectUnauthorized: true`** en production et configurer correctement les certificats CA | C03 |
| R03 | **Implémenter une authentification applicative** sur les commandes Logo critiques (ouverture de porte). Par exemple : signature HMAC des payloads, ou token rotatif partagé entre backend et gateway Logo | C21 |
| R04 | **Configurer des ACL MQTT** sur le broker pour restreindre quels client IDs peuvent publier sur `unifyIots/API/logo/set` | C21 |
| R05 | **Corriger la gestion du `deniedAccessFlag`** en step 1 : publier un message de refus au reader et créer un AccessLog | C17 |

### 12.2 Protocole et intégration (priorité moyenne)

| # | Recommandation | Constats liés |
|---|---|---|
| R06 | **Corriger l'extraction de température** : soit aligner `extractTemperatureFromMessage()` sur le format `state.reported.temperature.value[0]`, soit documenter le format réellement attendu du gateway et l'aligner sur le capteur DHT | C09 |
| R07 | **Rendre le topic Logo dynamique** dans `handleReaderAccess.ts` : lire le champ `publish` du device Logo en DB au lieu de hardcoder `unifyIots/API/logo/set` | C14 |
| R08 | **Rendre `heatSetpoint` configurable** via l'API REST ou le champ metadata du device Logo | C11 |
| R09 | **Documenter le gateway Logo↔MQTT** : spécifier le composant intermédiaire, son format de payload, et le versionner dans le projet | C07 |
| R10 | **Corriger le format de `MQTT_BASE_URL`** dans `test.env` pour être cohérent avec `development.env` | C02 |

### 12.3 Qualité de code (priorité basse)

| # | Recommandation | Constats liés |
|---|---|---|
| R11 | **Mettre en cache le device Logo** dans `scheduleLogoPublishingTick()` au lieu de créer un nouveau repository et requêter la DB à chaque tick | C13 |
| R12 | **Ajouter un filtrage d'anomalies** sur les températures (plage acceptable, écart-type) avant d'envoyer au Logo | C12 |
| R13 | **Isoler les alertes NFC par device** : remplacer les variables globales `nfcAbsentTimeout` / `lastNfcAbsent` par un Map indexé par device ID | C16 |
| R14 | **Documenter la séparation** entre `keyValues` (métrologie remontée) et `metadata.state` (commandes/état) | C06 |
| R15 | **Implémenter un retour d'état** depuis le Logo après exécution d'une commande (confirmation d'ouverture de porte, état du chauffage) | C20 |

---

## 13. Synthèse

### 13.1 Score de maturité de l'intégration Logo

| Critère | Score | Commentaire |
|---|---|---|
| Fonctionnalité de base | 3/5 | Publication température et ouverture porte fonctionnelles, mais format incompatible détecté |
| Sécurité | 1/5 | Commandes de porte sans authentification applicative, TLS mal configuré, incohérence de ports |
| Robustesse | 2/5 | Variables globales, pas de retry, pas de filtrage d'anomalies, pas de confirmation de commande |
| Maintenabilité | 2/5 | Topics hardcodés, heatSetpoint hardcodé, gateway non documenté, adapter redondant |
| Documentation | 1/5 | Aucune documentation du protocole, du gateway, ni de la séparation keyValues/metadata |
| **Score global** | **1.8/5** | Intégration fonctionnelle en conditions nominales mais vulnérable et fragile |

### 13.2 Résumé quantitatif

| Métrique | Valeur |
|---|---|
| Fichiers impliqués dans l'intégration Logo | 14 |
| Topics MQTT utilisés | 2 (subscribe + publish) |
| Formats de payload distincts | 5 (température, open, doorAlert, clearAlert, reported state) |
| Constats identifiés | 21 |
| Constats critiques (5/5) | 4 |
| Recommandations formulées | 15 |
| Tests existants liés au Logo | 1 fichier (handleReaderAccess.test.ts — 23 cas) |
| Tests manquants pour le Logo | logo-publisher.service.ts (0 test), key-mapper adapter Logo (0 test) |
