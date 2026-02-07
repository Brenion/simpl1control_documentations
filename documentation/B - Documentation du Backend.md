# Documentation du Backend

## Objectifs

Le backend a pour but de réceptionner, traiter et redistribuer des données et commandes provenant de différentes sources :

- Un **dashboard frontend** pour la supervision et le contrôle.
- Des **IoT connectés**, utilisant le protocole **MQTT** via **Mosquitto**.
- Un **automate programmable** qui communique avec le backend via API ou MQTT.
## Technologies utilisées

- **Fastify** (Framework backend)
- **PostgreSQL** (Base de données)
- **Mosquitto MQTT** (Communication avec les IoT et l'automate programmable)
- **RBAC (Role-Based Access Control)** pour la gestion des rôles et autorisations.
## Packages à installer

- `fastify` : Framework backend.
- `mqtt` : Gestion des communications MQTT avec les IoT et l'automate Siemens.
- `@fastify/jwt` : Gestion des authentifications via JWT.
- `@fastify/auth` : Middleware d'authentification.
- `casl` : Gestion avancée des rôles et autorisations.
- `dotenv` : Gestion des variables d'environnement.
- `cron-schedule` : Planification des tâches récurrentes.
- `date-fns` : Manipulation et formatage des dates.
- `yup` : Validation des schémas de données.
- `jsonwebtoken` : Génération et vérification des tokens JWT.
- `reflect-metadata` : Utilisation des métadonnées pour la gestion avancée des classes et des décorateurs.
- `vitest` : Framework de test pour valider le bon fonctionnement du backend.
## Fonctionnalités principales

### 1. Gestion des équipements et des topics MQTT

- Enregistrement de nouveaux équipements.
- Création et gestion des topics de communication pour les échanges MQTT.

### 2. Communication avec le frontend

- Fourniture des données en temps réel ou en différé.
- Réception des commandes de découverte de nouveaux capteurs/actionneurs IoT.
- Enregistrement et gestion des automatisations venant du frontend.
- Génération de statistiques d’utilisation (consommation énergétique, température, ouverture de porte, etc.).

### 3. Communication avec l’automate programmable

- Utilisation de l’API **Siemens Logo! 8.4**.
- Privilégier la communication via **MQTT**, sauf pour la transmission de données volumineuses comme la consommation énergétique.
- Récupération des données depuis l’API de l’automate.
- Envoi de commandes à l’automate pour exécuter certaines actions.

### 4. Gestion des IoT

- Réception des données des capteurs IoT (température, valve électrostatique, etc.).
- Envoi de commandes aux IoT.
- Gestion des accès et autorisations pour les badges Mifare programmables.
- Détection et appairage des nouveaux équipements via un système sécurisé de gestion des messages (ex. RabbitMQ).

### 5. Historisation des modifications

- Archivage des modifications majeures de l’application.
- Suivi des événements et logs d’activité.

## Cas d’usage principaux

1. **Consommation énergétique** :
    
    - Récupérer à intervalle régulier (toutes les **5 minutes**) la consommation énergétique d’un automate **Siemens Logo**.
    - L'automate stocke une donnée toutes les **3 secondes** et transmettra entre **100 et 1000 entrées** à chaque envoi.
    - Stocker ces données dans la base de données pour analyse et reporting.
        
2. **Capteurs de température et valves électrostatiques** :
    
    - Réception et enregistrement des données.
    - Mise à disposition de ces informations pour exploitation dans le frontend.
        
3. **Gestion des autorisations d’ouverture de porte** :
    
    - Contrôle d’accès via badges **Mifare programmables**.
    - Enregistrement des accès et suivi des autorisations.

## Gestion des rôles et permissions

- Les rôles seront préalablement définis par les développeurs du backend.
- Identification des rôles nécessaires : **Admin, Responsable, Utilisateur simple** (à détailler ultérieurement).
- Gestion avancée des rôles avec **CASL**.

## Statistiques et reporting

- Suivi de la **consommation énergétique** sur plusieurs périodes : **5 minutes, moyenne journalière, moyenne mensuelle, moyenne annuelle**.
- Moyenne des **températures** sur différentes périodes.
- Suivi des **ouvertures de portes** et des accès autorisés/refusés.

Ces statistiques seront utilisées pour générer des rapports et optimiser la gestion des équipements.

## Structure de base  du projet 

```
/Users/kavan/www/tfe/domotyk/
├── packages/
│   └── backend/
│       ├── src/
│       │   ├── features/
│       │   │   ├── auth.js
│       │   │   ├── data-histories/
│       │   │   │   ├── data-histories.repository.js
│       │   │   │   └── data-histories.route.js
│       │   │   ├── devices/
│       │   │   │   ├── device.repository.js
│       │   │   │   └── device.route.js
│       │   │   └── users/
│       │   │       └── users.route.js
│       │   ├── plugins/
│       │   │   ├── error-handler.js
│       │   │   └── mqtt.ts
│       │   ├── services/
│       │   │   └── cron.service.ts
│       │   ├── utils/
│       │   │   └── logger.js
│       │   ├── cron-setup.js
│       │   ├── data-source.js
│       │   ├── register.ts
│       │   └── server.ts
│       └── tests/
│           └── integrations/
│               └── devices-integration.test.ts
```


#  Installation et Configuration du Backend

## Création du projet

Dans un terminal, exécuter la commande suivante pour initialiser le projet :

```
mkdir backend-fastify && cd backend-fastify
pnpm init -y  # ou npm init -y / yarn init -y
```

---

##  Installation des dépendances

Installer Fastify et les outils essentiels :

```
pnpm add fastify @fastify/jwt @fastify/auth dotenv reflect-metadata
pnpm i --save-dev @types/node
```

|Package|Description|
|---|---|
|`fastify`|Framework backend|
|`@fastify/jwt`|Gestion des authentifications via JWT|
|`@fastify/auth`|Middleware d'authentification|
|`dotenv`|Gestion des variables d'environnement|
|`reflect-metadata`|Gestion avancée des métadonnées|
|`typescript`|Compilation TypeScript|
|`ts-node`|Exécution de TypeScript sans compilation explicite|
|`@types/node`|Types pour Node.js|

---

##  Configuration de TypeScript

Créer un fichier `**tsconfig.json**` à la racine du projet :

```
{
  "compilerOptions": {
	"types": ["node"],
    "target": "ES6",
    "module": "CommonJS",
    "strict": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "esModuleInterop": true,
    "resolveJsonModule": true
  },
  "include": ["src"],
  "exclude": ["node_modules"]
}
```

---

## Création du dossier source

Créer l’architecture de base :

```
mkdir src
touch src/server.ts
```

---

## Création du fichier `server.ts`

Créer et ajouter le code suivant dans `src/server.ts` :

```
import Fastify from "fastify";
import dotenv from "dotenv";

dotenv.config();

const server = Fastify({ logger: true });

server.get("/", async () => {
  return { message: "Backend Fastify is running!" };
});

const start = async () => {
  try {
    await server.listen({ port: Number(process.env.PORT) || 3000, host: "0.0.0.0" });
    console.log("Server started on port", process.env.PORT || 3000);
  } catch (err) {
    server.log.error(err);
    process.exit(1);
  }
};

start();
```

---

## Ajout de la commande de lancement dans `package.json`

Modifier le fichier `package.json` pour inclure :

```
"scripts": {
  "dev": "ts-node src/server.ts"
}
```

---

## Lancement du serveur

Lancer le serveur avec la commande :

```
pnpm run dev
```

Par défaut, le serveur écoutera sur le port **3000**. Ce port peut être modifié en ajoutant un fichier `**.env**` contenant :

```
PORT=5000
```

---
# Etape 3 Installation des dépendances supplémentaires

Nous allons maintenant installer quelques dépendances supplémentaires utiles pour notre projet :

```
pnpm add date-fns yup
pnpm add -D vitest eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin eslint-plugin-import eslint-plugin-unused-imports
```

|Package|Description|
|---|---|
|`date-fns`|Manipulation et formatage des dates|
|`yup`|Validation des schémas de données|
|`vitest`|Framework de test pour valider le backend|
|`eslint`|Outil d'analyse statique de code TypeScript|
|`@typescript-eslint/parser`|Parser ESLint pour TypeScript|
|`@typescript-eslint/eslint-plugin`|Plugin ESLint pour les bonnes pratiques TypeScript|
|`eslint-plugin-import`|Vérification des bonnes pratiques d'importation|
|`eslint-plugin-unused-imports`|Détection et suppression des imports inutilisés|

---

### Configuration de ESLint

Créer un fichier `**.eslintrc.json**` à la racine du projet avec le contenu suivant :

```
{
  "env": {
    "browser": true,
    "es2021": true,
    "node": true
  },
  "extends": [
    "eslint:recommended",
    "plugin:@typescript-eslint/recommended",
    "plugin:import/errors",
    "plugin:import/warnings",
    "plugin:import/typescript"
  ],
  "parser": "@typescript-eslint/parser",
  "parserOptions": {
    "ecmaVersion": "latest",
    "sourceType": "module"
  },
  "plugins": ["@typescript-eslint", "import", "unused-imports"],
  "rules": {
    "@typescript-eslint/no-unused-vars": "warn",
    "@typescript-eslint/explicit-module-boundary-types": "off",
    "import/order": ["error", { "alphabetize": { "order": "asc" } }],
    "unused-imports/no-unused-imports": "error"
  }
}
```

Créer également un fichier `**.eslintignore**` pour exclure certains fichiers et dossiers :

```
node_modules/
dist/
.env
```

# Étape 4 : Installation des outils complémentaires

Nous allons maintenant installer des dépendances supplémentaires pour la gestion des tâches planifiées, la validation et le support MQTT :

```
pnpm add vite cron-schedule mqtt
pnpm install -S yup
```

|   |   |
|---|---|
|Package|Description|
|`vite`|Outil de build rapide pour le développement|
|`yup`|Validation des schémas de données|
|`cron-schedule`|Planification des tâches récurrentes|
|`mqtt`|Client MQTT.js pour la communication avec Mosquitto|

---

# Etape 5 : Configuration des variables d’environnement

-Nous allons maintenant mettre en place la gestion des variables d’environnement nécessaires au bon fonctionnement du backend.

### 1. Création du fichier `.env`

Créer un fichier `develop.env` et un fichier ```test.env``` à la racine du projet avec les variables de configuration de base :

```
PORT=3000
NODE_ENV=development // pour le ficheir develop
NODE_ENV=test // pour le fichier test
JWT_SECRET=supersecretkey
MQTT_BROKER_URL=mqtt://localhost:1883
DATABASE_URL=postgresql://user:password@localhost:5432/database
```

### 2. Chargement des variables avec `dotenv`

Ajouter `dotenv` pour charger ces variables au démarrage du projet.
Assurez-vous d'importer `dotenv` dans votre fichier principal (`server.ts`) :

```
import dotenv from "dotenv";
dotenv.config();
```

Cela garantit que toutes les variables définies dans `.env` sont accessibles via `process.env` dans l’application.

## 3. Modification du package pour lancer le bonne environnement

```json
// package.json au niveau des scripts
"test": "NODE_ENV=test ts-node src/server.ts && npm run vitest",
"dev": "NODE_ENV=development ts-node src/server.ts"
```

Ces fichiers seront mis a jour au fur et à mesure de la mise en place du backend. 

---

## Étape 6 : Sécurisation du serveur

Nous allons maintenant sécuriser le backend en ajoutant des protections essentielles.

###  1. Installation des dépendances de sécurité

```
pnpm add @fastify/cors @fastify/helmet
```

|                   |                                                                           |
| ----------------- | ------------------------------------------------------------------------- |
| Package           | Description                                                               |
| `@fastify/cors`   | Activation du CORS pour permettre les requêtes cross-origin               |
| `@fastify/helmet` | Ajout de protections contre les attaques courantes (XSS, Clickjacking...) |

---

### 2. Ajout de la configuration de sécurité dans `server.ts`

Modifier le fichier `**server.ts**` pour inclure ces protections :

```
import Fastify from "fastify";
import dotenv from "dotenv";
import cors from "@fastify/cors";
import helmet from "@fastify/helmet";

dotenv.config();

const server = Fastify({ logger: true });

// Activation de CORS
server.register(cors, {
  origin: "*", // À restreindre selon le besoin
});

// Activation de Helmet pour sécuriser les headers
server.register(helmet);

server.get("/", async () => {
  return { message: "Backend Fastify is running with security!" };
});

const start = async () => {
  try {
    await server.listen({ port: Number(process.env.PORT) || 3000, host: "0.0.0.0" });
    console.log("Server started on port", process.env.PORT || 3000);
  } catch (err) {
    server.log.error(err);
    process.exit(1);
  }
};

start();
```

---
# Étape 7 : Configuration de la base de données avec Docker

Nous allons maintenant configurer la base de données PostgreSQL en utilisant Docker et gérer les migrations avec `pg` et `@fastify/postgres` .

### 1. Mise en place du fichier `docker-compose.yml`

Créer un fichier `**docker-compose.yml**` à la racine du projet avec le contenu suivant :

```yaml
version: '3.8'

name: domotyk

services:
	postgres:
		volumes:
		- ./database:/docker-entrypoint-initdb.d
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
	postgres_data:
```

Cela permet de lancer un container.  ```- ./database:/docker-entrypoint-initdb.d```nous permet de lire dans le dossier database et de prendre toute les config sql qu'il trouve. 

pour l'instant dans ce dossier nous allons cree un nouveau fichier ``ìnit.sql

```sql
CREATE DATABASE domotyk_test;
CREATE DATABASE domotyk_dev;
```

et voici les deux ligne qui créerons deux base de donnée distinct. celle de test permettra de faire les test back sans toucher a la version de development.

Lancer la base de données avec la commande :

```
docker compose up -d
```

Cela démarre un conteneur PostgreSQL accessible sur `localhost:5432`.

### ### 2. Installation et configuration de `@fastify/postgres`

Nous allons maintenant intégrer PostgreSQL dans Fastify en utilisant le plugin `@fastify/postgres`.

#### Installation des dépendances

Installer `@fastify/postgres` pour gérer la connexion à PostgreSQL :

```
pnpm add @fastify/postgres pg
```

#### modifications dans `server.ts`

Nous allons modifier `**server.ts**` pour inclure la gestion de la connexion PostgreSQL.

```ts
import Fastify from "fastify";
import dotenv from "dotenv";
import fastifyPostgres from "@fastify/postgres";

dotenv.config();

const server = Fastify({ logger: true });

// Ajout de la configuration PostgreSQL avec `@fastify/postgres`
server.register(fastifyPostgres, {
  connectionString: process.env.DATABASE_URL || "postgres://user:password@localhost:5432/database"
});

// Création d'une route de test pour vérifier la connexion à la base de données
server.get("/db-test", async (request, reply) => {
  const client = await server.pg.connect(); // Connexion à PostgreSQL
  const { rows } = await client.query('SELECT NOW()'); // Exécution d'une requête simple
  client.release(); // Libération du client après utilisation
  return rows;
});

const start = async () => {
  try {
    await server.listen({ port: Number(process.env.PORT) || 3000, host: "0.0.0.0" });
    console.log("Server started on port", process.env.PORT || 3000);
  } catch (err) {
    server.log.error(err);
    process.exit(1);
  }
};

start();
```

#### Explication des modifications apportées :

1. **Ajout de l'importation de **`**@fastify/postgres**` : Permet d’intégrer la gestion de PostgreSQL dans Fastify.
2. **Ajout de **`**server.register(fastifyPostgres, { connectionString })**` : Permet d'établir une connexion persistante avec PostgreSQL.
3. **Création d'une route **`**/db-test**` :
    
    - Cette route établit une connexion à la base de données.
    - Elle exécute une requête SQL (`SELECT NOW()`) pour vérifier que PostgreSQL est accessible.
    - Elle retourne la réponse avec l’heure actuelle depuis PostgreSQL.
    - Le client est libéré après utilisation pour éviter les fuites de connexion.
        

Cette modification permet d'interagir facilement avec PostgreSQL en utilisant `server.pg.query()` dans d'autres parties du backend.

### 3 Requête Postman pour tester la route `/db-test`

Pour tester cette route avec Postman, suivez ces étapes :

1. **Ouvrir Postman**
2. **Créer une nouvelle requête**
3. **Sélectionner la méthode GET**
4. **Entrer l'URL suivante** :
    
    ```
    http://localhost:3000/db-test
    ```
    
5. **Cliquer sur "Send"**
6. **Résultat attendu** :
    
    ```
    [
      {
        "now": "2025-02-07T12:34:56.789Z"
      }
    ]
    ```


# Étape 8 : Configuration de MQTT

Nous allons maintenant configurer la communication MQTT avec le client `mqtt.js` en structurant mieux notre code.

###  1. Installation de `mqtt`

Installer `mqtt` pour gérer la communication MQTT dans le backend :

```
pnpm add mqtt
```

###  2. Déplacement de la connexion MQTT dans un fichier dédié

Créer un fichier `**src/mqttClient.ts**` :

```
import mqtt from "mqtt";
import dotenv from "dotenv";

dotenv.config();

const mqttOptions = {
  clientId: process.env.MQTT_CLIENT_ID || "backend-client",
  username: process.env.MQTT_USERNAME,
  password: process.env.MQTT_PASSWORD,
  keepalive: Number(process.env.MQTT_KEEPALIVE) || 60,
  clean: process.env.MQTT_CLEAN_SESSION === "true"
};

const mqttServer = () => {
  try {
    const mqttClient = mqtt.connect(process.env.MQTT_BROKER_URL || "mqtt://localhost:1883", mqttOptions);

    mqttClient.on("connect", () => {
      console.log("Connecté au broker MQTT");
      mqttClient.subscribe("sensor/temperature", (err) => {
        if (!err) {
          console.log("Abonné au topic sensor/temperature");
        }
      });
    });

    mqttClient.on("message", (topic, message) => {
      console.log(`Message reçu sur ${topic}:`, message.toString());
    });
  } catch (error) {
    console.error("Erreur de connexion au broker MQTT:", error);
  }
};

export default mqttServer;
```

###  3. Utilisation de MQTT dans `server.ts`

Modifier `**server.ts**` pour intégrer le client MQTT :

```
import Fastify from "fastify";
import dotenv from "dotenv";
import fastifyPostgres from "@fastify/postgres";
import mqttServer from "./mqttClient";

dotenv.config();

const server = Fastify({ logger: true });

// Ajout de la configuration PostgreSQL
server.register(fastifyPostgres, {
  connectionString: process.env.DATABASE_URL || "postgres://user:password@localhost:5432/database"
});

// Initialisation de MQTT
mqttServer();

const start = async () => {
  try {
    await server.listen({ port: Number(process.env.PORT) || 3000, host: "0.0.0.0" });
    console.log("Server started on port", process.env.PORT || 3000);
  } catch (err) {
    server.log.error(err);
    process.exit(1);
  }
};

start();
```

###  4. Fonctionnement de MQTT.js dans notre backend

Dans notre architecture, le backend Fastify ne communique pas directement avec les IoT. Il interagit avec **Mosquitto**, qui sert de broker MQTT intermédiaire. Voici comment cela fonctionne :

1. **Les IoT envoient leurs messages au broker Mosquitto.**
    
    - Un capteur IoT publie des données sur un topic spécifique (ex: `sensor/temperature`).
    - Mosquitto reçoit et stocke temporairement ces messages.
        
2. **Notre backend se connecte au broker et s'abonne aux topics MQTT nécessaires.**
    
    - Lorsqu'un nouveau message est publié, Mosquitto le relaie au backend.
    - Le backend traite les données et peut les stocker dans PostgreSQL ou déclencher une action.
        
3. **Le backend peut aussi publier des commandes MQTT vers les IoT.**
    
    - Exemple : Si une commande `toggle/light` est envoyée, Mosquitto la redistribue aux IoT abonnés.
###  5. Explication des modifications dans `server.ts`

1. **Connexion au broker MQTT** via `mqtt.connect(process.env.MQTT_BROKER_URL || "mqtt://localhost:1883")`.
    
2. **Abonnement aux topics MQTT** pour recevoir des données des IoT.
    
3. **Écoute des messages reçus et traitement des données.**
    
    - Par défaut, on affiche simplement les messages reçus.
    - Plus tard, ces données pourront être stockées dans PostgreSQL.
        
4. **Possibilité d'envoyer des commandes aux IoT** en publiant des messages MQTT.


Avec cette structure, notre backend est capable de **recevoir des données IoT via Mosquitto** et d’**envoyer des commandes MQTT aux IoT connectés**.

1. **Connexion au broker MQTT** via `mqtt.connect()`.
2. **Abonnement automatique** au topic `test/topic` dès la connexion.
3. **Écoute des messages reçus** et affichage dans la console.


---

# Étape 9 : Organisation des routes et plugins

Nous allons maintenant structurer le projet de manière modulaire en séparant les routes et en utilisant `fastify-plugin`.

## 1. Installation de `fastify-plugin`

Installer `fastify-plugin` pour organiser les routes et les plugins proprement :

```
pnpm add fastify-plugin
```

## 2. Création de la structure des routes

Créer un dossier `routes/` dans `src/` et ajouter des fichiers de routes spécifiques :

```
src/
 ├── routes/
 │    ├── auth.ts
 │    ├── devices.ts
 │    ├── users.ts
```

Chaque fichier contiendra des routes associées à un domaine spécifique.

#### **Exemple : Route d'authentification **`**routes/auth.ts**`

```
import { FastifyInstance } from "fastify";
import fastifyPlugin from "fastify-plugin";

async function authRoutes(fastify: FastifyInstance) {
  fastify.post("/login", async (request, reply) => {
    return { message: "Authentification réussie" };
  });
}

export default fastifyPlugin(authRoutes);
```

#### **Exemple : Route des utilisateurs **`**routes/users.ts**`

```
import { FastifyInstance } from "fastify";
import fastifyPlugin from "fastify-plugin";

async function userRoutes(fastify: FastifyInstance) {
  fastify.get("/users", async (request, reply) => {
    return { users: [] };
  });
}

export default fastifyPlugin(userRoutes);
```

## 3. Création d'un fichier `register.ts` pour centraliser les routes et plugins

Afin de ne pas alourdir `server.ts`, nous allons centraliser l'enregistrement des routes et plugins dans un fichier dédié.

Créer un fichier `**src/register.ts**` :

```
import authRoutes from "./routes/auth";
import userRoutes from "./routes/users";
import devicesRoutes from "./routes/devices";
import cors from "@fastify/cors";
import helmet from "@fastify/helmet";
import { FastifyInstance } from "fastify";

const register = (server: FastifyInstance) => {
    console.log('🔌 Initialisation des plugins et routes');

    // Activation de CORS
    server.register(cors, {
        origin: "*", // À restreindre selon le besoin
    });

    // Activation de Helmet pour sécuriser les headers
    server.register(helmet);

    // Activation des routes
    server.register(authRoutes, { prefix: "/auth" });
    server.register(userRoutes, { prefix: "/users" });
    server.register(devicesRoutes, { prefix: "/devices" });
};

export default register;
```

## 4. Modification de `server.ts` pour utiliser `register.ts`

Dans `**server.ts**`, nous allons utiliser `register.ts` pour gérer les routes et les plugins:

```
import Fastify from "fastify";
import dotenv from "dotenv";
import fastifyPostgres from "@fastify/postgres";
import mqttServer from "./mqttClient";
import register from "./register";

dotenv.config();

const server = Fastify({ logger: true });

// Ajout de la configuration PostgreSQL
server.register(fastifyPostgres, {
    connectionString: process.env.DATABASE_URL || "postgres://user:password@localhost:5432/database"
});

// Initialisation de MQTT
mqttServer();

// Enregistrement des plugins et routes via register.ts
register(server);

// Ajout de la configuration PostgreSQL avec `@fastify/postgres`
server.register(fastifyPostgres, {
  connectionString: process.env.DATABASE_URL || "postgres://root:test123@localhost:5432/domotyk_dev"
});

const start = async () => {
    try {
        await server.listen({ port: Number(process.env.PORT) || 3000, host: "0.0.0.0" });
        console.log(" Server started on port", process.env.PORT || 3000);
    } catch (err) {
        server.log.error(err);
        process.exit(1);
    }
};

start();
```

## 4. Gestion des nouvelles routes

Lorsque de nouvelles routes doivent être ajoutées au projet, voici les étapes recommandées :

1. **Créer un fichier dans **`**routes/**` :
    
    - Exemple : Pour une route `sensors`, créer `src/routes/sensors.ts`
        
2. **Définir les routes dans le fichier** :
    
    ```
    import { FastifyInstance } from "fastify";
    import fastifyPlugin from "fastify-plugin";
    
    async function sensorRoutes(fastify: FastifyInstance) {
      fastify.get("/", async (request, reply) => {
        return { sensors: [] };
      });
    }
    
    export default fastifyPlugin(sensorRoutes);
    ```
    
3. **Enregistrer la route dans ****register.ts** :
    
    ```
    const register = (server: FastifyInstance) =>{
        console.log('register')
    
    server.register(cors, {
      origin: "*", 
    });
    server.register(helmet);
    server.register(authRoutes, { prefix: "/auth" });
    
    //ajout d'une nouvelle route au registre
    server.register(sensorRoutes, {prefix: "/sensor"}
    ```


# Étape 10 : Mise en place des tâches planifiées

Nous allons maintenant ajouter un système de tâches planifiées pour exécuter automatiquement certaines actions, comme la récupération des données IoT.

## 1. Installation des dépendances nécessaires

Installer `cron-schedule` ainsi que d'autres modules utiles pour gérer les tâches planifiées et la journalisation :

```
pnpm add cron-schedule async-mutex pino pino-pretty
```

## 2. Création du dossier `services/` et du fichier `cron.service.ts`

Créer un dossier `**src/services/**` et ajouter un fichier `**cron.service.ts**` pour gérer l'exécution des tâches planifiées :

```ts
import type { Cron } from 'cron-schedule';
import { IntervalBasedCronScheduler } from 'cron-schedule/schedulers/interval-based.js';
import logger from '../utils/logger.js';
import { Mutex } from 'async-mutex';

const REFRESH_INTERVAL = 1 * 1000; // 1 seconde

export class CronService {
	private scheduler?: IntervalBasedCronScheduler;
	private mutex = new Mutex();
	
	public start() {
		logger.info("Starting Cron service...");
		this.scheduler = new IntervalBasedCronScheduler(REFRESH_INTERVAL);
	}

	public addTask(cron: Cron, fn: () => Promise<unknown>) {
		if (!this.scheduler) {
			throw new Error("Cron service not started!");
		}

		this.scheduler.registerTask(cron, async () => {
			const release = await this.mutex.acquire();
			try {
				logger.info(`cron task execution : ok`);
				await fn();
			} catch (e) {
				logger.error(`error cron task : ${e}`);
			} finally {
				release();
			}
		}, { isOneTimeTask: false });
	}
}
```

### 3. Mise à jour de la configuration du projet

En raison de problèmes de compatibilité avec Node.js, certaines modifications ont été nécessaires :

#### A. Installation de `tsx` pour exécuter les fichiers TypeScript sans compilation préalable

```
pnpm install --save-dev tsx
```

#### B. Modification des scripts dans `package.json`

```
"scripts": {
    "test": "NODE_ENV=test tsx src/server.ts && npm run vitest",
    "dev": "NODE_ENV=development tsx src/server.ts"
}
```

#### C. Mise à jour de `tsconfig.json`

```
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

### 4. Modification du fichier `server.ts`

Ajouter l'initialisation du service de Cron :

```
console.log("process.env.CRON_JOB", process.env.CRON_JOB);

const cronService = new CronService();
cronService.start();

if (process.env.CRON_JOB) {
    setupCronJobs(cronService);
}
```

### 5. Création du fichier `decorator-index.ts`

Créer un fichier dédié pour ajouter `CronService` aux décorateurs de Fastify :

```
import { FastifyInstance } from "fastify";
import { CronService } from "./services/cron.service.js";

const decoratorIndex = (server: FastifyInstance) => {
    server.decorate('cronService', new CronService());
}

export default decoratorIndex;
```

### **À quoi ça sert ?**

Ce fichier sert à **enregistrer** une instance de `CronService` comme un **décorateur** dans Fastify. Les décorateurs permettent d'ajouter des fonctionnalités au contexte Fastify et de les rendre accessibles dans toutes les routes et plugins.

Dans ton cas, ce fichier :

- Crée une instance unique de `CronService`
- Attache cette instance au serveur Fastify sous le nom `cronService`
- Permet aux handlers et plugins d'accéder facilement à `cronService` sans devoir instancier une nouvelle fois l'objet.

### **Quand cela te sera utile ?**

Tu en auras besoin lorsque :

1. **Tu veux centraliser et partager ton service de tâches planifiées (`CronService`) dans toute ton application**
    
    - Sans ce décorateur, chaque module qui veut utiliser `CronService` devrait l'importer et instancier une nouvelle fois, ce qui peut poser des problèmes si tu veux un état partagé.
2. **Tu veux éviter de répéter l'instanciation de `CronService`**
    
    - Par exemple, si ton `CronService` gère des tâches récurrentes (ex : nettoyage de logs, récupération de données IoT toutes les X minutes), une seule instance est suffisante.
3. **Tu veux organiser proprement ton projet en utilisant les bonnes pratiques de Fastify**
    
    - Fastify recommande l'utilisation des décorateurs pour stocker des services qui doivent être accessibles dans plusieurs endroits.
### 6. Création du fichier `cron-setup.ts`

Ce fichier contiendra les tâches à exécuter selon un planning défini :

```
import { CronService } from "./services/cron.service.js";
import testJob from "./cronjobs/test-job.js";
import { parseCronExpression } from "cron-schedule";

const EVERY_FIVE_SECONDS = '*/5 * * * * *';

export function setupCronJobs(cronService: CronService) {
    cronService.addTask(parseCronExpression(EVERY_FIVE_SECONDS), testJob);
}
```

### 7. Création du dossier `cronjobs/` et d'un premier test

Créer un dossier dédié aux tâches planifiées et ajouter un fichier de test :

```
mkdir -p src/cronjobs
```

Créer un fichier `**src/cronjobs/test-job.ts**` :

```
export default async function testJob() {
    console.log("🕒 Test de CronJob : Exécution réussie !");
}
``````

---

# Étape 11 : Ajout d’un gestionnaire d’erreurs global

Nous allons maintenant mettre en place un gestionnaire d’erreurs centralisé afin de standardiser les réponses de l’API et de gérer efficacement les erreurs au sein du backend.

##  1. Création du fichier `errorHandler.ts`

Créer un fichier `**src/plugins/errorHandler.ts**` pour gérer globalement les erreurs dans Fastify :

```
import { FastifyInstance, FastifyReply, FastifyRequest } from "fastify";

export function errorHandler(server: FastifyInstance) {
    server.setErrorHandler((error, request: FastifyRequest, reply: FastifyReply) => {
        console.error("🚨 Erreur interceptée :", error);

        const statusCode = error.statusCode || 500;
        reply.status(statusCode).send({
            success: false,
            message: error.message || "Erreur interne du serveur",
            code: error.code || "INTERNAL_ERROR"
        });
    });
}
```

## 2. Enregistrement du gestionnaire d’erreurs dans `server.ts`

Modifier `**server.ts**` pour enregistrer le gestionnaire d’erreurs global :

```
import { errorHandler } from "./plugins/errorHandler";

// Enregistrement du gestionnaire d’erreurs
errorHandler(server);
```

## 3. Standardisation des réponses API

Créer un fichier `**src/utils/apiResponse.ts**` pour unifier les réponses API :

```
export function successResponse(data: unknown, message = "Succès") {
    return {
        success: true,
        message,
        data
    };
}

export function errorResponse(message = "Une erreur est survenue", code = "ERROR") {
    return {
        success: false,
        message,
        code
    };
}
```

## 4. Exemple d'utilisation dans une route

Dans une route comme `routes/users.ts`, utiliser les fonctions standardisées :

```
import { FastifyInstance } from "fastify";
import { successResponse, errorResponse } from "../utils/apiResponse";

async function userRoutes(fastify: FastifyInstance) {
    fastify.get("/users", async (request, reply) => {
        try {
            const users = [];
            return reply.send(successResponse(users, "Liste des utilisateurs"));
        } catch (error) {
            return reply.status(500).send(errorResponse("Impossible de récupérer les utilisateurs"));
        }
    });
}

export default userRoutes;
```

## 5. Explication des modifications

1. **Création d’un fichier** `**errorHandler.ts**` pour capturer et traiter globalement les erreurs.
2. **Ajout du gestionnaire d’erreurs dans** `**server.ts**` pour une gestion uniforme des erreurs.
3. **Standardisation des réponses API avec** `**apiResponse.ts**`, facilitant la gestion des succès et erreurs.
4. **Application de la standardisation dans les routes**, garantissant une cohérence des réponses.


---

# Étape 12 : Mise en place des entités et des migrations avec TypeORM

Nous allons maintenant configurer **TypeORM** pour la gestion des entités et des migrations dans notre projet.

## 1. Création des entités

Les entités seront placées dans `**./src/entities**`.

#### BaseEntity (classe de base pour les entités)

Créer le fichier `**src/entities/base-entity.ts**` :

```ts
import "reflect-metadata";
import { PrimaryGeneratedColumn, CreateDateColumn, UpdateDateColumn } from "typeorm";

export abstract class BaseEntity {
  @PrimaryGeneratedColumn("uuid")
  id: string;

  @CreateDateColumn({ type: "timestamp", default: () => "CURRENT_TIMESTAMP" })
  createdAt: Date;

  @UpdateDateColumn({ type: "timestamp", default: () => "CURRENT_TIMESTAMP", onUpdate: "CURRENT_TIMESTAMP" })
  updatedAt: Date;
}
```

#### Entité `User`

Créer le fichier `**src/entities/user.ts**` :

```ts
import "reflect-metadata";
import { Entity, Column } from "typeorm";
import { BaseEntity } from "./base-entity";

enum Role {
  ADMIN = "ADMIN",
  USER = "USER",
  MANAGER = "MANAGER",
}

@Entity("users")
export class User extends BaseEntity {
  @Column({ type: "varchar" })
  firstname: string;

  @Column({ type: "varchar" })
  lastname: string;

  @Column({ type: "varchar", unique: true })
  username: string;

  @Column({ type: "varchar", unique: true })
  mail: string;

  @Column({ type: "enum", enum: Role })
  role: Role;
}
```

### 3. Configuration de la connexion TypeORM

Créer un fichier `**src/database/data-source.ts**` pour configurer TypeORM :

```
import "reflect-metadata";
import { DataSource } from "typeorm";
import { User } from "../entities/user";

export const AppDataSource = new DataSource({
  type: "postgres",
  host: "localhost",
  port: 5432,
  username: "root",
  password: "test123",
  database: "domotyk_dev",
  synchronize: false, // Toujours mettre `false` en production
  logging: false,
  entities: [User],
  migrations: ["src/migrations/*.ts"],
});
```

### 4. Création d'un fichier de migration pour l'entité `User`

Les migrations seront placées dans `**./src/migrations**`.

Exécuter la commande suivante pour générer une migration basée sur l'entité `User` :

```
npx typeorm-ts-node-esm migration:generate ./src/migrations/m -d ./src/migrations/data-source.ts
```

Cela créera un fichier **src/migrations/"numerodemigration"-migration.ts** contenant les requêtes SQL nécessaires pour créer la table `users`.

Les migrations seront placées dans `**./src/migrations**`.

### 5. Exécution de la migration pour mettre à jour la base de données

Après la génération du fichier de migration, exécuter :

```
pnpm typeorm migration:run -d src/database/data-source.ts
```

Cela appliquera les changements à la base de données.

### 6. Automatisation de l'exécution des migrations au démarrage du projet

Modifier `**server.ts**` pour exécuter automatiquement les migrations lors du lancement du serveur :

```
import { AppDataSource } from "./database/data-source";

AppDataSource.initialize()
  .then(() => {
    console.log("📦 Connexion à la base de données établie.");
    return AppDataSource.runMigrations();
  })
  .then(() => {
    console.log("Migrations appliquées avec succès.");
  })
  .catch((error) => {
    console.error("Erreur lors de l'initialisation de la base de données :", error);
  });
``````

### 7. Ajout des scripts dans le package.json

```json
"migration:blank": "typeorm migration:create ./src/migrations",
"migration:create": "npx typeorm-ts-node-esm migration:generate ./src/migrations/migration -d ./src/data-source.ts"
"migration:run": "npx typeorm-ts-node-esm migration:run -- -d ./src/data-source.ts"
```


---

# Étape 13 : Mise en place de factories et seeders avec @jorgebodega/typeorm-factory et @jorgebodega/typeorm-seeding

Nous allons maintenant ajouter un système de génération de **données fictives** en utilisant les **factories** et **seeders** avec `@jorgebodega/typeorm-factory` et `@jorgebodega/typeorm-seeding`.

## 1. Installation des dépendances nécessaires

Installer les packages pour gérer les factories et les seeders :

```
pnpm add @jorgebodega/typeorm-factory @jorgebodega/typeorm-seeding faker
```

## 2. Création du dossier `factories` et définition d'une factory pour `User`

Créer un dossier `**src/factories/**` et ajouter un fichier `**user.factory.ts**` pour définir la génération automatique d’utilisateurs.


```ts
import { FactorizedAttrs, Factory } from '@jorgebodega/typeorm-factory'
import { UserEntity } from '../../entities/user-entity.js'
import { AppDataSource as dataSource } from '../../data-source.js'
import { faker } from '@faker-js/faker'
import Role from '../../enums/roles-enum.js'

class UserFactory extends Factory<UserEntity> {
	protected entity = UserEntity
	protected dataSource = dataSource // Imported datasource
	protected attrs(): FactorizedAttrs<UserEntity> {
	
		return {
			firstname: faker.person.firstName(),
			lastname: faker.person.lastName(),
			username: faker.person.middleName(),
			mail: faker.internet.email(),
			role: Role.ADMIN,
		}
	}
}

export default UserFactory
```

## 3. Création d'un fichier `user.seeder.ts` pour insérer des utilisateurs fictifs

Créer un dossier `**src/seeders/**` et ajouter un fichier `**user.seeder.ts**` pour insérer plusieurs utilisateurs fictifs dans la base de données.

```ts
import { Seeder } from '@jorgebodega/typeorm-seeding'
import UserFactory from '../factories/user.factory.js'

class UserSeeder extends Seeder {

	async run() {
		await new UserFactory().createMany(10)
	}
}

export default UserSeeder
```

## 4. Exécution des seeders

Pour exécuter les seeders et insérer les données dans la base de données, utiliser la commande suivante :

```bash
npx typeorm-ts-node-esm migration:generate ./src/migrations/migration -d ./src/data-source.ts
```

## 5. Ajout de la commande dans le package.json

```json
"scripts": {
	"typeorm": "typeorm-ts-node-esm",
	"test": "NODE_ENV=test tsx src/server.ts && npm run vitest",
	"dev": "NODE_ENV=development tsx src/server.ts",
	"seed:run": "NODE_OPTIONS='--loader ts-node/esm' npx typeorm-seeding -d ./src/data-source.ts src/database/seeders/*.ts",
	"migration:blank": "typeorm migration:create ./src/migrations/migration",
	"migration:create": "npx typeorm-ts-node-esm migration:generate ./src/migrations/migration -d ./src/data-source.ts",
	"migration:run": "npx typeorm-ts-node-esm migration:run -- -d ./src/data-source.ts"
},
```

## 6. Explication des modifications

1. **Ajout du package** `**@jorgebodega/typeorm-factory**` **et** `**@jorgebodega/typeorm-seeding**` pour la gestion des seeders et des données fictives.
    
2. **Création d’un fichier** `**user.factory.ts**` pour générer dynamiquement des utilisateurs avec des données réalistes grâce à `faker`.
    
3. **Ajout d’un fichier** `**user.seeder.ts**` pour insérer plusieurs utilisateurs par défaut dans la base de données.
    
4. **Utilisation de** `**createMany(10)**` pour générer plusieurs utilisateurs automatiquement.
    
5. **Exécution des seeders via** `**typeorm-ts-node-esm migration:generate**` pour appliquer les changements de données dans la base.

---

# Étape 14 : Utilisation de `vitest` pour les tests unitaires

Nous allons maintenant configurer et utiliser `**vitest**` pour exécuter des tests unitaires et d'intégration sur nos routes et services.

### 1. Configuration des tests

Les tests seront organisés comme suit :

- **Tests unitaires** dans `src/units/`
    
- **Tests d'intégration** dans `src/integrations/`
    

Créer les dossiers nécessaires :

```
mkdir -p src/units src/integrations
```

### 2. Mise à jour du fichier `package.json`

Modifier la section `scripts` pour assurer que :

1. **Les migrations sont exécutées** avant les tests.
    
2. **Les seeders sont lancés** pour insérer des données de test.
    
3. **Le serveur démarre en mode test** avant d’exécuter `vitest`.
    

```
"scripts": {
    "typeorm": "typeorm-ts-node-esm",
    "test": "NODE_ENV=test pnpm migration:run && pnpm seed:run && tsx src/server.ts && npm run vitest",
    "dev": "NODE_ENV=development tsx src/server.ts",
    "seed:run": "NODE_OPTIONS='--loader ts-node/esm' npx typeorm-seeding -d ./src/data-source.ts src/database/seeders/*.ts",
    "migration:blank": "typeorm migration:create ./src/migrations/migration",
    "migration:create": "npx typeorm-ts-node-esm migration:generate ./src/migrations/migration -d ./src/data-source.ts",
    "migration:run": "npx typeorm-ts-node-esm migration:run -- -d ./src/data-source.ts"
}
```

### 3. Création d'un fichier de test unitaire

Créer un fichier de test `**src/units/user.unit.test.ts**` pour tester la logique métier de l'API `users`.

```
import { describe, it, expect } from "vitest";

// Exemple d'un test unitaire simulé
const userService = {
  getUsers: () => [{ id: 1, name: "John Doe" }],
};

describe("Test du service User", () => {
  it("Devrait retourner une liste d'utilisateurs", () => {
    const users = userService.getUsers();
    expect(users.length).toBeGreaterThan(0);
  });
});
```

### 4. Création d'un fichier de test d'intégration

Créer un fichier de test `**src/integrations/user.integration.test.ts**` pour tester l'API `users`.

```
import { describe, it, expect } from "vitest";
import request from "supertest";

const BASE_URL = "http://localhost:3000";

describe("Test des routes Users", () => {
  it("Devrait récupérer la liste des utilisateurs", async () => {
    const response = await request(BASE_URL).get("/users");
    expect(response.status).toBe(200);
    expect(response.body).toHaveProperty("success", true);
  });
});
```

### 5. Exécution des tests

Pour exécuter les tests unitaires et d'intégration :

```
pnpm test
```

Ce script va :

1. **Exécuter les migrations** (`migration:run`).
2. **Lancer les seeders** (`seed:run`).
3. **Démarrer le serveur en mode test** (`tsx src/server.ts`).
4. **Exécuter les tests avec** `**vitest**` (`npm run vitest`).
