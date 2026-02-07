# Installation et Test de Mosquitto (MQTT) sur Raspberry Pi

## Vue d'ensemble

Configuration complète d'un système MQTT avec le broker Mosquitto sur Raspberry Pi, incluant l'authentification sécurisée, les tests avec MQTT Explorer (macOS) et l'intégration d'un ESP32.

**Environnement réseau :**
- Raspberry Pi (Mosquitto) : `192.168.0.102`
- Mac (MQTT Explorer) : `192.168.0.104`
- Port MQTT : `1883`

---

## 1. Installation sur Raspberry Pi

### Commandes essentielles

```bash
# Installation
sudo apt update && sudo apt upgrade -y
sudo apt install -y mosquitto mosquitto-clients

# Activation du service
sudo systemctl enable mosquitto
sudo systemctl start mosquitto

# Vérification
mosquitto -v
sudo systemctl status mosquitto
```

---

## 2. Configuration avec authentification

### Fichier de configuration (`/etc/mosquitto/mosquitto.conf`)

```
password_file /etc/mosquitto/passwd
listener 1883
allow_anonymous false
```

### Création d'utilisateur

```bash
sudo mosquitto_passwd -c /etc/mosquitto/passwd admin
sudo systemctl restart mosquitto
```

---

## 3. Tests locaux (Raspberry Pi)

### Souscription à un topic

```bash
mosquitto_sub -h localhost -t "test/topic" -u admin -P "mot_de_passe"
```

### Publication d'un message

```bash
mosquitto_pub -h localhost -t "test/topic" -m "Bonjour, MQTT!" -u admin -P "mot_de_passe"
```

---

## 4. Configuration MQTT Explorer (macOS)

### Paramètres de connexion

- **Name** : Mosquitto Raspberry
- **Broker Address** : `192.168.0.102`
- **Port** : `1883`
- **Username** : `admin`
- **Password** : mot de passe configuré

### Tests bidirectionnels

**Raspberry → MQTT Explorer :**
```bash
mosquitto_pub -h 192.168.0.102 -t "test/raspberry" -m "Message depuis le Raspberry Pi" -u admin -P "mot_de_passe"
```

**MQTT Explorer → Raspberry :**
```bash
mosquitto_sub -h 192.168.0.102 -t "test/mac" -u admin -P "mot_de_passe"
```

---

## 5. Intégration ESP32

### Prérequis Arduino IDE

1. Installation de la carte ESP32 via le gestionnaire de cartes
2. Installation de la bibliothèque `PubSubClient`
3. Pour carte AZ-Delivery : https://www.az-delivery.de/en/blogs/azdelivery-blog-fur-arduino-und-raspberry-pi/esp32-jetzt-mit-boardverwalter-installieren

### Code de test ESP32

**Fonctionnalités :**
- Connexion WiFi automatique
- Reconnexion MQTT automatique
- Publication périodique (5 secondes)
- Topic : `test/esp32`
- Message : "Coucou je suis ESP32, es-tu là?"

**Configuration requise :**
```c
const char* ssid = "Votre_SSID";
const char* password = "Votre_Mot_De_Passe";
const char* mqtt_server = "192.168.0.102";
const char* mqtt_user = "admin";
const char* mqtt_password = "votre_mot_de_passe";
```

---

## Points clés

✓ **Sécurité** : Authentification obligatoire (allow_anonymous false)  
✓ **Tests réussis** : Communication bidirectionnelle Raspberry ↔ Mac  
✓ **ESP32 opérationnel** : Publication MQTT toutes les 5 secondes  
✓ **Architecture** : Broker central (Raspberry) + clients multiples (Mac, ESP32)

---

## Commandes de diagnostic

```bash
# État du service
sudo systemctl status mosquitto

# Logs en temps réel
sudo journalctl -u mosquitto -f

# Redémarrage
sudo systemctl restart mosquitto
```

---
---

# Backend Fastify - Documentation Technique

## Vue d'ensemble

Backend Fastify pour système domotique avec communication MQTT, gestion IoT, intégration automate Siemens Logo et authentification JWT.

**Stack technique :**
- **Framework** : Fastify
- **Base de données** : PostgreSQL + TypeORM
- **Communication** : Mosquitto MQTT
- **Authentification** : JWT + RBAC (CASL)
- **Tests** : Vitest

---

## Architecture du projet

```
backend/
├── src/
│   ├── entities/          # Entités TypeORM
│   ├── migrations/        # Migrations base de données
│   ├── factories/         # Factories pour génération données
│   ├── seeders/          # Seeders pour données de test
│   ├── routes/           # Routes API (auth, users, devices)
│   ├── services/         # Services métier (cron, etc.)
│   ├── plugins/          # Plugins Fastify (MQTT, errorHandler)
│   ├── cronjobs/         # Tâches planifiées
│   ├── utils/            # Utilitaires (logger, apiResponse)
│   ├── data-source.ts    # Configuration TypeORM
│   ├── register.ts       # Enregistrement routes/plugins
│   └── server.ts         # Point d'entrée application
└── tests/
    ├── units/            # Tests unitaires
    └── integrations/     # Tests d'intégration
```

---

## Installation et configuration

### 1. Initialisation projet

```bash
mkdir backend-fastify && cd backend-fastify
pnpm init -y
```

### 2. Dépendances essentielles

```bash
# Core
pnpm add fastify @fastify/jwt @fastify/auth @fastify/cors @fastify/helmet
pnpm add @fastify/postgres pg dotenv reflect-metadata

# MQTT & Cron
pnpm add mqtt cron-schedule async-mutex

# ORM & Validation
pnpm add typeorm date-fns yup

# Factories & Seeders
pnpm add @jorgebodega/typeorm-factory @jorgebodega/typeorm-seeding faker

# Logs & Tests
pnpm add pino pino-pretty vitest

# Dev dependencies
pnpm add -D @types/node typescript tsx eslint
```

### 3. Configuration TypeScript (`tsconfig.json`)

```json
{
  "compilerOptions": {
    "types": ["node"],
    "target": "ESNext",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "strict": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "esModuleInterop": true,
    "resolveJsonModule": true,
    "skipLibCheck": true
  },
  "include": ["src"],
  "exclude": ["node_modules"]
}
```

### 4. Variables d'environnement (`.env`)

**develop.env :**
```env
PORT=3000
NODE_ENV=development
JWT_SECRET=supersecretkey
MQTT_BROKER_URL=mqtt://192.168.0.102:1883
MQTT_USERNAME=admin
MQTT_PASSWORD=mot_de_passe
MQTT_CLIENT_ID=backend-client
DATABASE_URL=postgresql://root:test123@localhost:5432/domotyk_dev
CRON_JOB=true
```

**test.env :**
```env
NODE_ENV=test
DATABASE_URL=postgresql://root:test123@localhost:5432/domotyk_test
```

---

## Configuration base de données

### Docker Compose (`docker-compose.yml`)

```yaml
version: '3.8'
name: domotyk

services:
  postgres:
    image: postgres:latest
    container_name: postgres_db
    restart: always
    environment:
      POSTGRES_USER: root
      POSTGRES_PASSWORD: test123
      POSTGRES_DB: domotykDB
    ports:
      - "5432:5432"
    volumes:
      - ./database:/docker-entrypoint-initdb.d
```

**Initialisation (`database/init.sql`) :**
```sql
CREATE DATABASE domotyk_test;
CREATE DATABASE domotyk_dev;
```

**Démarrage :**
```bash
docker compose up -d
```

### Configuration TypeORM (`src/data-source.ts`)

```typescript
import { DataSource } from "typeorm";
import { User } from "./entities/user";

export const AppDataSource = new DataSource({
  type: "postgres",
  host: "localhost",
  port: 5432,
  username: "root",
  password: "test123",
  database: "domotyk_dev",
  synchronize: false,
  logging: false,
  entities: [User],
  migrations: ["src/migrations/*.ts"],
});
```

---

## Gestion des entités

### BaseEntity (`src/entities/base-entity.ts`)

```typescript
import { PrimaryGeneratedColumn, CreateDateColumn, UpdateDateColumn } from "typeorm";

export abstract class BaseEntity {
  @PrimaryGeneratedColumn("uuid")
  id: string;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;
}
```

### Entité User (`src/entities/user.ts`)

```typescript
import { Entity, Column } from "typeorm";
import { BaseEntity } from "./base-entity";

enum Role { ADMIN = "ADMIN", USER = "USER", MANAGER = "MANAGER" }

@Entity("users")
export class User extends BaseEntity {
  @Column()
  firstname: string;

  @Column()
  lastname: string;

  @Column({ unique: true })
  username: string;

  @Column({ unique: true })
  mail: string;

  @Column({ type: "enum", enum: Role })
  role: Role;
}
```

---

## Migrations et seeders

### Commandes essentielles

```bash
# Générer migration
pnpm migration:create

# Exécuter migrations
pnpm migration:run

# Exécuter seeders
pnpm seed:run
```

### Factory User (`src/factories/user.factory.ts`)

```typescript
import { Factory } from '@jorgebodega/typeorm-factory';
import { faker } from '@faker-js/faker';
import { UserEntity } from '../entities/user-entity';

class UserFactory extends Factory<UserEntity> {
  protected entity = UserEntity;
  protected attrs() {
    return {
      firstname: faker.person.firstName(),
      lastname: faker.person.lastName(),
      username: faker.person.middleName(),
      mail: faker.internet.email(),
      role: Role.ADMIN,
    };
  }
}
```

### Seeder User (`src/seeders/user.seeder.ts`)

```typescript
import { Seeder } from '@jorgebodega/typeorm-seeding';
import UserFactory from '../factories/user.factory';

class UserSeeder extends Seeder {
  async run() {
    await new UserFactory().createMany(10);
  }
}
```

---

## Architecture routes et plugins

### Enregistrement centralisé (`src/register.ts`)

```typescript
import authRoutes from "./routes/auth";
import userRoutes from "./routes/users";
import devicesRoutes from "./routes/devices";
import cors from "@fastify/cors";
import helmet from "@fastify/helmet";

const register = (server: FastifyInstance) => {
  server.register(cors, { origin: "*" });
  server.register(helmet);
  server.register(authRoutes, { prefix: "/auth" });
  server.register(userRoutes, { prefix: "/users" });
  server.register(devicesRoutes, { prefix: "/devices" });
};

export default register;
```

### Exemple route (`src/routes/users.ts`)

```typescript
import { FastifyInstance } from "fastify";
import fastifyPlugin from "fastify-plugin";
import { successResponse } from "../utils/apiResponse";

async function userRoutes(fastify: FastifyInstance) {
  fastify.get("/", async (request, reply) => {
    const users = await UserRepository.find();
    return reply.send(successResponse(users, "Liste des utilisateurs"));
  });
}

export default fastifyPlugin(userRoutes);
```

---

## Communication MQTT

### Client MQTT (`src/mqttClient.ts`)

```typescript
import mqtt from "mqtt";

const mqttOptions = {
  clientId: process.env.MQTT_CLIENT_ID || "backend-client",
  username: process.env.MQTT_USERNAME,
  password: process.env.MQTT_PASSWORD,
  keepalive: 60,
  clean: true
};

const mqttServer = () => {
  const client = mqtt.connect(process.env.MQTT_BROKER_URL, mqttOptions);

  client.on("connect", () => {
    console.log("✅ Connecté au broker MQTT");
    client.subscribe("sensor/temperature");
  });

  client.on("message", (topic, message) => {
    console.log(`📩 Message reçu sur ${topic}:`, message.toString());
    // Traiter et stocker dans PostgreSQL
  });
};

export default mqttServer;
```

**Architecture MQTT :**
- IoT → Mosquitto → Backend
- Backend publie commandes → Mosquitto → IoT

---

## Tâches planifiées (Cron)

### Service Cron (`src/services/cron.service.ts`)

```typescript
import { IntervalBasedCronScheduler } from 'cron-schedule/schedulers/interval-based';
import { Mutex } from 'async-mutex';

export class CronService {
  private scheduler?: IntervalBasedCronScheduler;
  private mutex = new Mutex();

  public start() {
    this.scheduler = new IntervalBasedCronScheduler(1000); // 1 sec
  }

  public addTask(cron: Cron, fn: () => Promise<unknown>) {
    this.scheduler.registerTask(cron, async () => {
      const release = await this.mutex.acquire();
      try {
        await fn();
      } finally {
        release();
      }
    });
  }
}
```

### Configuration (`src/cron-setup.ts`)

```typescript
import { parseCronExpression } from "cron-schedule";
import testJob from "./cronjobs/test-job";

const EVERY_FIVE_SECONDS = '*/5 * * * * *';

export function setupCronJobs(cronService: CronService) {
  cronService.addTask(parseCronExpression(EVERY_FIVE_SECONDS), testJob);
}
```

---

## Gestion des erreurs

### ErrorHandler (`src/plugins/errorHandler.ts`)

```typescript
export function errorHandler(server: FastifyInstance) {
  server.setErrorHandler((error, request, reply) => {
    const statusCode = error.statusCode || 500;
    reply.status(statusCode).send({
      success: false,
      message: error.message || "Erreur interne",
      code: error.code || "INTERNAL_ERROR"
    });
  });
}
```

### Réponses standardisées (`src/utils/apiResponse.ts`)

```typescript
export function successResponse(data: unknown, message = "Succès") {
  return { success: true, message, data };
}

export function errorResponse(message: string, code = "ERROR") {
  return { success: false, message, code };
}
```

---

## Tests avec Vitest

### Configuration tests

**Structure :**
- `src/units/` : Tests unitaires
- `src/integrations/` : Tests d'intégration

**Script package.json :**
```json
{
  "test": "NODE_ENV=test pnpm migration:run && pnpm seed:run && tsx src/server.ts && vitest"
}
```

### Test d'intégration (`src/integrations/user.test.ts`)

```typescript
import { describe, it, expect } from "vitest";
import request from "supertest";

describe("Routes Users", () => {
  it("Récupération liste utilisateurs", async () => {
    const response = await request("http://localhost:3000").get("/users");
    expect(response.status).toBe(200);
    expect(response.body).toHaveProperty("success", true);
  });
});
```

---

## Scripts package.json

```json
{
  "scripts": {
    "dev": "NODE_ENV=development tsx src/server.ts",
    "test": "NODE_ENV=test pnpm migration:run && pnpm seed:run && vitest",
    "migration:blank": "typeorm migration:create ./src/migrations/migration",
    "migration:create": "typeorm-ts-node-esm migration:generate ./src/migrations/migration -d ./src/data-source.ts",
    "migration:run": "typeorm-ts-node-esm migration:run -d ./src/data-source.ts",
    "seed:run": "typeorm-seeding -d ./src/data-source.ts src/database/seeders/*.ts"
  }
}
```

---

## Fonctionnalités principales

### 1. Gestion équipements IoT
- Enregistrement nouveaux équipements
- Gestion topics MQTT pour communication
- Détection et appairage sécurisé

### 2. Communication automate Siemens Logo
- API Siemens Logo! 8.4
- Récupération consommation énergétique (5 min)
- Transmission 100-1000 entrées / envoi
- Commandes via MQTT

### 3. Authentification & autorisations
- JWT pour authentification
- RBAC avec CASL (Admin, Manager, User)
- Contrôle accès badges Mifare

### 4. Statistiques & reporting
- Consommation énergétique (5min, jour, mois, année)
- Moyennes températures multi-périodes
- Suivi ouvertures portes + accès

---

## Points clés

✓ **Architecture modulaire** : Routes, plugins, services séparés  
✓ **Sécurité** : JWT, CORS, Helmet, authentification MQTT  
✓ **ORM robuste** : TypeORM + migrations + seeders + factories  
✓ **Communication IoT** : MQTT via Mosquitto + gestion topics  
✓ **Tâches automatisées** : Cron pour collecte données périodiques  
✓ **Tests complets** : Vitest + migrations/seeders auto  
✓ **Gestion erreurs** : Handler global + réponses standardisées

---

## Commandes utiles

```bash
# Développement
pnpm dev

# Tests
pnpm test

# Base de données
docker compose up -d
pnpm migration:run
pnpm seed:run

# Génération migration
pnpm migration:create

# Vérification API
curl http://localhost:3000/db-test
```

---
---

# Communication MQTT - ESP32 & Backend

## Vue d'ensemble

Intégration complète d'un capteur DHT22 simulé sur ESP32 communiquant avec le backend Fastify via MQTT, incluant la persistance des données et la communication avec l'automate Siemens Logo.

**Architecture :**
```
IoT (ESP32) → Mosquitto → Backend Fastify → PostgreSQL
                          ↓
                    Automate Logo (API)
```

---

## ESP32 - Capteur DHT22 Simulé

### Configuration matérielle

**Composants :**
- ESP32 (AZ-Delivery)
- Capteur DHT22 (pin 13)
- WiFi : 2.4GHz

**Bibliothèques requises :**
```cpp
#include <WiFi.h>
#include <PubSubClient.h>
#include <Adafruit_Sensor.h>
#include <DHT.h>
#include <Arduino_JSON.h>
```

### Structure des données

```cpp
struct SensorData {
  float temperature;
  float humidity;
  int errorCode;  // 0 = succès
};

// Configuration
const char* ssid = "WhiteSideOfTheWifi";
const char* mqtt_server = "192.168.0.102";
const char* topic = "test/esp32_dht22";
```

### Format message MQTT (JSON)

**Message normal :**
```json
{
  "brand": "custom",
  "model": "sensor",
  "type": "dth",
  "payload": {
    "temperature": "21.50",
    "humidity": "45.20"
  }
}
```

**Message erreur :**
```json
{
  "brand": "custom",
  "model": "sensor",
  "type": "dth",
  "Error": 1,
  "payload": {}
}
```

### Caractéristiques simulation

**Génération de 36 données (3 minutes à 5s/msg) :**
- 2 cycles à 15.00°C (début)
- Montée progressive 15.01°C → 22.99°C
- 4 cycles à 23.00°C (plateau)
- 4 cycles vides (pause)
- Descente progressive 22.99°C → 15.01°C
- 2 cycles à 15.00°C (fin)
- Humidité aléatoire : 10-50%

**Intervalle publication :** 5 secondes

---

## Backend - Gestion MQTT

### Plugin MQTT (`src/plugins/mqtt.ts`)

```typescript
import mqtt from "mqtt";

export interface MqttOptions {
  clientId: string;
  username?: string;
  password?: string;
  keepalive: number;
  clean: boolean;
}

const mqttServer = (mqttOptions: MqttOptions) => {
  const mqttClient = mqtt.connect(process.env.MQTT_BROKER_URL, mqttOptions);

  mqttClient.on("connect", () => {
    console.log("✅ Connected to MQTT broker");
    
    // Souscription topic ESP32
    mqttClient.subscribe("test/esp32_dht22");
    
    // Publication vers backend confirmée
    mqttClient.publish("testy/backend", "Hello from backend");
  });

  mqttClient.on("message", (topic, message) => {
    console.log(`📩 Message reçu sur ${topic}:`, message.toString());
    // Traitement et sauvegarde en DB
  });
};

export default mqttServer;
```

---

## Entités base de données

### Device Entity (`src/entities/device-entity.ts`)

```typescript
@Entity("devices")
export class DeviceEntity extends BaseEntity {
  @Column()
  deviceid: string;  // Identifiant unique

  @Column()
  name: string;

  @Column()
  brand: string;

  @Column()
  model: string;

  @Column()
  type: string;

  @Column()
  topic: string;  // Topic MQTT

  @Column({ type: 'enum', enum: ['active', 'inactive'], default: 'active' })
  status: 'active' | 'inactive';

  @Column({ type: 'json', nullable: true })
  metadata: Record<string, any>;

  @OneToMany(() => DataHistoryEntity, (dataHistory) => dataHistory.device)
  histories: DataHistoryEntity[];
}
```

### DataHistory Entity (`src/entities/data-history-entity.ts`)

```typescript
@Entity('data_history')
export class DataHistoryEntity {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @ManyToOne(() => DeviceEntity, device => device.histories)
  device: DeviceEntity;

  @CreateDateColumn()
  timestamp: Date;

  @Column({ type: 'json' })
  payload: Record<string, any>;  // Données flexibles
}
```

---

## Seeding du device

### Device Seeder (`src/seeders/device.seeder.ts`)

```typescript
import { Seeder } from "@jorgebodega/typeorm-seeding";
import DeviceFactory from "../factories/device.factory";
import DeviceStatus from "../../enums/device-status";

class DeviceSeeder extends Seeder {
  async run() {
    await new DeviceFactory().create({
      deviceid: 'custom-sensor-dth-01',
      brand: 'custom',
      model: 'sensor',
      status: DeviceStatus.ACTIVE,
      type: 'dth',
      topic: 'zigbee2mqtt/custom-sensor-dth-01',
    });
  }
}

export default DeviceSeeder;
```

---

## Stratégie de stockage des données

### Table intermédiaire : data_history

**Objectif :** Limiter la croissance de la DB avec agrégation progressive.

**Règle de rétention :**
- Conservation : **7 jours glissants** de données brutes
- Fréquence : 5 secondes → **120 960 entrées/semaine** par capteur

**Agrégation automatique (Cron) :**

1. **Chaque minuit** : agrégation du jour le plus ancien
2. **Moyennes par tranche de 15 minutes**
3. **Résultat : 96 moyennes/jour** (24h × 4)
4. **Rétention finale : 672 entrées/semaine** (96 × 7)

**Exemple planning :**
```
Lundi 1   → Lundi 8 (minuit) : agrégation des données lundi 1
Mardi 2   → Mardi 9 (minuit) : agrégation des données mardi 2
...
```

### Agrégations futures

**Données anciennes (> 1 semaine) :**
- Moyennes journalières (1 entrée/jour)
- Moyennes mensuelles (1 entrée/mois)
- Conservation multi-années optimisée

---

## Communication avec Automate Logo

### Contexte

**Automate Siemens Logo! 8.4 :**
- Communication via **API spécifique** (non dynamique)
- Backend = **transmetteur** (pas de décision métier)
- Logo = **décideur** (interprétation et actions)

### Structure message vers Logo

```typescript
{
  state: {
    AUTO: { value: 0 | 1 },           // Mode automatique
    PROGRAM: { value: 0 | 1 },        // Programme actif
    RAZ: { value: 0 | 1 },            // Reset
    temperature: { value: number },   // Température la + basse
    errorStatus: { value: number },   // 0 = OK, >0 = erreur
    heatSetpoint: { value: number }   // Consigne chauffage
  }
}
```

### Logique backend

**1. Récupération températures :**
- Recevoir données MQTT de **tous les IoT actifs**
- Détection automatique clé température (`temperature`, `temp`, `t`, etc.)
- Conservation **température la plus basse**

**2. Publication vers Logo :**
- Envoi **uniquement si valeur change**
- Éviter doublons (même message)
- Topic : `logo/api/input`

**3. Gestion erreurs :**
- Aucune température reçue → `errorStatus: 1`
- Capteur inactif > seuil → notification

**4. Modes de fonctionnement :**
- `AUTO = 1` : comportement automatique (programme)
- `AUTO = 0` : mode manuel (consigne fixe)
- `heatSetpoint` : utilisé dans tous les cas

### Module spécifique Logo

**Objectifs :**
1. Observer données IoT en continu
2. Extraire températures dynamiquement
3. Comparer avec état actuel Logo
4. Publier **uniquement si changement**

```typescript
class LogoService {
  private lastState: LogoState;
  private activeTemperatures: Map<string, number>;

  processIoTData(deviceId: string, payload: any) {
    const temp = this.extractTemperature(payload);
    if (temp !== null) {
      this.activeTemperatures.set(deviceId, temp);
      this.updateLogoIfNeeded();
    }
  }

  private extractTemperature(payload: any): number | null {
    // Recherche clé température (temp, temperature, t, etc.)
    const keys = ['temperature', 'temp', 't', 'temperatur'];
    for (const key of keys) {
      if (payload[key] !== undefined) return parseFloat(payload[key]);
    }
    return null;
  }

  private updateLogoIfNeeded() {
    const minTemp = Math.min(...this.activeTemperatures.values());
    const newState = { ...this.lastState, temperature: { value: minTemp } };
    
    if (JSON.stringify(newState) !== JSON.stringify(this.lastState)) {
      this.publishToLogo(newState);
      this.lastState = newState;
    }
  }
}
```

---

## Flux complet de données

```
1. ESP32 DHT22 (5s)
   ↓ MQTT publish → test/esp32_dht22
   
2. Mosquitto Broker (192.168.0.102:1883)
   ↓ forward
   
3. Backend Fastify
   ├─ Validation message
   ├─ Identification device (deviceid)
   ├─ Sauvegarde data_history
   ├─ Extraction température
   └─ Transmission Logo (si changement)
   
4. PostgreSQL
   ├─ Table: devices
   └─ Table: data_history (7 jours)
   
5. Cron Service (minuit)
   └─ Agrégation J-7 (moyennes 15min)
   
6. Automate Logo
   └─ Décisions chauffage basées sur températures
```

---

## Points clés

✓ **Simulation DHT22** : 36 données cycliques avec montée/descente  
✓ **Format JSON structuré** : brand, model, type, payload  
✓ **Identification unique** : deviceid pour association DB  
✓ **Stockage intermédiaire** : 7 jours glissants (data_history)  
✓ **Agrégation automatique** : Moyennes 15min (cron minuit)  
✓ **Communication Logo** : Publication si changement uniquement  
✓ **Détection température** : Recherche automatique clé dynamique  
✓ **Gestion erreurs** : errorStatus + logging

---

## Commandes utiles

```bash
# Seeding device
pnpm seed:run

# Monitoring MQTT
mosquitto_sub -h 192.168.0.102 -t "test/esp32_dht22" -u admin -P password

# Vérification données
SELECT * FROM data_history WHERE device_id = 'custom-sensor-dth-01' ORDER BY timestamp DESC LIMIT 10;

# Agrégation manuelle (test)
SELECT device_id, 
       DATE_TRUNC('hour', timestamp) + INTERVAL '15 min' * FLOOR(EXTRACT(MINUTE FROM timestamp) / 15) AS time_bucket,
       AVG((payload->>'temperature')::numeric) AS avg_temp
FROM data_history
WHERE timestamp >= NOW() - INTERVAL '1 day'
GROUP BY device_id, time_bucket
ORDER BY time_bucket;
```
