# SC-PH0-T06 — Documenter l'architecture actuelle avec diagrammes

> **User Story** : SC-US-PH0-06 — En tant que développeur, je veux mettre à jour les dépendances vulnérables
> **Tâche** : Documenter architecture actuelle avec diagrammes
> **Projet** : simpl1Control v1.1.2
> **Date** : 13 février 2026
> **Méthode** : Analyse statique du code source (read-only, aucune modification)

---

## Table des matières

1. [Vue d'ensemble du système](#1-vue-densemble-du-système)
2. [Diagramme d'architecture globale (déploiement)](#2-diagramme-darchitecture-globale-déploiement)
3. [Diagramme de la stack technologique](#3-diagramme-de-la-stack-technologique)
4. [Architecture backend — structure des modules](#4-architecture-backend--structure-des-modules)
5. [Architecture frontend — structure des modules](#5-architecture-frontend--structure-des-modules)
6. [Diagramme entité-relation (base de données)](#6-diagramme-entité-relation-base-de-données)
7. [Diagramme des routes API REST](#7-diagramme-des-routes-api-rest)
8. [Diagramme des routes frontend](#8-diagramme-des-routes-frontend)
9. [Diagramme de flux MQTT et temps réel](#9-diagramme-de-flux-mqtt-et-temps-réel)
10. [Diagramme de séquence — contrôle d'accès NFC](#10-diagramme-de-séquence--contrôle-daccès-nfc)
11. [Diagramme de séquence — authentification](#11-diagramme-de-séquence--authentification)
12. [Diagramme de dépendances des services backend](#12-diagramme-de-dépendances-des-services-backend)
13. [Inventaire des dépendances externes](#13-inventaire-des-dépendances-externes)
14. [Constats sur l'architecture](#14-constats-sur-larchitecture)
15. [Synthèse](#15-synthèse)

---

## 1. Vue d'ensemble du système

### 1.1 Description

simpl1Control est un **dashboard IoT monorepo** pour le contrôle de bâtiment. Il combine :

- Un **backend Fastify** (API REST + WebSocket + MQTT) pour la gestion des devices, l'authentification et le temps réel
- Un **frontend React 19** (SPA) pour la visualisation des données et l'administration
- Une communication **MQTT bidirectionnelle** avec des automates Siemens LOGO 8.4, des capteurs de température et des lecteurs NFC
- Un système de **contrôle d'accès physique** par badges MIFARE via NFC

### 1.2 Chiffres clés

| Métrique | Valeur |
|---|---|
| Version | 1.1.2 |
| Gestionnaire de paquets | pnpm (monorepo) |
| Packages | 2 (backend + frontend) |
| Fichiers source backend | ~100 fichiers |
| Fichiers source frontend | ~120 fichiers |
| Entités base de données | 7 |
| Endpoints API REST | 17 |
| Routes frontend | 14 |
| Dépendances backend (prod) | 33 |
| Dépendances frontend (prod) | 26 |

---

## 2. Diagramme d'architecture globale (déploiement)

```mermaid
graph TB
    subgraph "Réseau local 192.168.3.0/24"
        subgraph "Serveur principal 192.168.3.100"
            subgraph "Docker (docker-compose)"
                NGINX["Nginx<br/>:80"]
                BACKEND["Backend Fastify<br/>:3001"]
                DB["PostgreSQL 15<br/>:5432"]
                REGISTRY["Docker Registry<br/>:5000"]
            end
            BROKER["Broker MQTT<br/>:1883 (plain)<br/>:8883 (TLS)"]
        end

        subgraph "Devices IoT"
            LOGO["Siemens LOGO! 8.4<br/>PLC — Porte & Chauffage"]
            SENSOR["Capteur DHT<br/>Température / Humidité"]
            NFC_R["Reader NFC<br/>PN532"]
            NFC_W["Writer NFC<br/>PN532"]
        end

        CLIENT["Navigateur Web<br/>Client (LAN)"]
    end

    CLIENT -->|"HTTP/WS :80"| NGINX
    NGINX -->|"Proxy /api/*<br/>→ :3001"| BACKEND
    NGINX -->|"Proxy /ws/*<br/>→ :3001 (upgrade)"| BACKEND
    BACKEND -->|"TypeORM"| DB
    BACKEND <-->|"MQTT :1883"| BROKER
    BACKEND <-->|"MQTT :8883 (TLS)"| BROKER
    BROKER <-->|"MQTT"| LOGO
    BROKER <-->|"MQTT"| SENSOR
    BROKER <-->|"MQTT"| NFC_R
    BROKER <-->|"MQTT"| NFC_W

    style NGINX fill:#4CAF50,color:#fff
    style BACKEND fill:#2196F3,color:#fff
    style DB fill:#FF9800,color:#fff
    style BROKER fill:#9C27B0,color:#fff
    style LOGO fill:#607D8B,color:#fff
    style SENSOR fill:#607D8B,color:#fff
    style NFC_R fill:#607D8B,color:#fff
    style NFC_W fill:#607D8B,color:#fff
    style CLIENT fill:#03A9F4,color:#fff
```

### 2.1 Notes de déploiement

- Le frontend est servi comme SPA statique par Nginx
- Nginx agit en reverse proxy vers le backend pour `/api/*` et `/ws/*`
- Le backend se connecte au broker MQTT sur deux ports (plain 1883, TLS 8883) au démarrage (`MQTT_START=true`)
- Le service discovery Docker utilise le nommage `myapp-backend-1` dans la config Nginx
- Le serveur `192.168.3.100` concentre tous les services (Docker, broker MQTT, registre Docker)

---

## 3. Diagramme de la stack technologique

```mermaid
graph LR
    subgraph "Frontend"
        direction TB
        REACT["React 19"]
        TANSTACK_R["TanStack Router"]
        TANSTACK_Q["TanStack React Query"]
        MUI["MUI 6 + Tailwind CSS 4"]
        VEGA["Vega / Vega-Lite"]
        ZOD_F["Zod"]
        I18N["i18next"]
        PWA["Service Worker (PWA)"]
        AUTH_KIT["react-auth-kit"]
        DND["dnd-kit"]
        VITE_F["Vite 6"]
    end

    subgraph "Backend"
        direction TB
        FASTIFY["Fastify 5"]
        TYPEORM["TypeORM 0.3"]
        JWT["@fastify/jwt + Passport"]
        ARGON["Argon2"]
        MQTT_LIB["mqtt.js 5"]
        WEBSOCKET["@fastify/websocket"]
        PINO["Pino (logger)"]
        ZOD_B["Zod"]
        NODEMAILER["Nodemailer + Mailgen"]
        CRON["cron-schedule"]
        TRUE_MYTH["true-myth (Result)"]
        MODBUS["modbus-serial"]
        VITE_B["Vite 6 + vite-plugin-node"]
    end

    subgraph "Infrastructure"
        direction TB
        PG["PostgreSQL 15"]
        DOCKER["Docker"]
        NGINX_I["Nginx"]
        MQTT_BROKER["Broker MQTT"]
    end

    subgraph "Test"
        direction TB
        VITEST["Vitest 3"]
        PLAYWRIGHT["Playwright 1.50"]
        MSW["MSW 2.7"]
        C8["c8 (coverage)"]
    end

    REACT --> TANSTACK_R
    REACT --> TANSTACK_Q
    REACT --> MUI
    FASTIFY --> TYPEORM
    FASTIFY --> JWT
    TYPEORM --> PG
    FASTIFY --> MQTT_LIB
    MQTT_LIB --> MQTT_BROKER
```

---

## 4. Architecture backend — structure des modules

```mermaid
graph TB
    subgraph "Entry Point"
        SERVER["server.ts<br/>Initialisation Fastify + MQTT + CRON"]
    end

    subgraph "Plugins"
        ERROR["error-handler.ts"]
        WS_PLUGIN["websocket.plugin.ts"]
    end

    subgraph "Features (feature-based architecture)"
        subgraph "auth/"
            AUTH_SVC["auth.service.ts"]
            LOGIN["login.route.ts"]
            REFRESH["refresh-token.route.ts"]
            FORGOT["forgot-password.route.ts"]
            RESET["reset-password.route.ts"]
            PROFILE["profile.route.ts"]
        end

        subgraph "users/"
            USER_ENTITY["user.entity.ts"]
            USER_REPO["user.repository.ts"]
            ADD_USER["addUser.route.ts"]
            GET_USERS["getAllUsers.route.ts"]
            GET_USER["getUser.route.ts"]
            DEL_USER["deleteUser.route.ts"]
            UPD_USER["updateUser.route.ts"]
        end

        subgraph "devices/"
            DEV_ENTITY["device.entity.ts"]
            DEV_REPO["device.repository.ts"]
            ADD_DEV["addDevice.route.ts"]
            GET_DEVS["getAllDevices.route.ts"]
            GET_DEV["getDevice.route.ts"]
            UPD_DEV["updateDevice.route.ts"]
            ACT_DEV["ActiveDevice.route.ts"]
        end

        subgraph "badges/"
            BADGE_ENTITY["badges.entity.ts"]
            BADGE_SVC["badge.service.ts"]
            BADGE_REPO["badges.repository.ts"]
            ADD_BADGE["badges.add.route.ts"]
            UPD_BADGE["badges.update.route.ts"]
            DEL_BADGE["badges.delete.route.ts"]
            VALIDATE["validateBadge.ts"]
            DERIVE_KEY["deriveBadgeKey.utils.ts"]
            DERIVE_AB["deriveKeysAB.utils.ts"]
        end

        subgraph "charts/"
            PAGE_ENTITY["page.entity.ts"]
            CARD_ENTITY["cards.entity.ts"]
            PAGE_REPO["page.repository.ts"]
            GET_PAGES["getAllPages.route.ts"]
            GET_PAGE["getOnePage.route.ts"]
            CHARTS_WS["charts.ws.ts"]
        end

        subgraph "data-histories/"
            DH_ENTITY["data-history.entity.ts"]
            DH_REPO["data-histories.repository.ts"]
            DH_ROUTE["data-histories.route.ts"]
        end

        subgraph "access-log/"
            AL_ENTITY["access-log.entity.ts"]
            AL_REPO["access-logs.repository.ts"]
            AL_ROUTE["getAllAccessLogs.route.ts"]
        end
    end

    subgraph "Services transverses"
        MQTT_SVC["mqtt.service.ts<br/>(singleton par URL)"]
        HASH_SVC["hash.service.ts<br/>(Argon2)"]
        EMAIL_SVC["email.service.ts<br/>(Nodemailer)"]
        CRON_SVC["cron.service.ts<br/>(Mutex)"]
        LOGO_PUB["logo-publisher.service.ts"]
        PING_PONG["ping-pong.service.ts"]
        RELOAD_MQTT["reload-mqtt.service.ts"]
        READER_ACCESS["handleReaderAccess.ts"]
    end

    subgraph "Realtime"
        KEY_MAPPER["key-mapper.ts<br/>DSL + Logo84 + Generic"]
        RT_HUB["realtime-hub.ts<br/>(pub/sub WebSocket)"]
    end

    subgraph "CRON Jobs"
        CRON_SETUP["cron-setup.ts"]
        LOGO_JOB["logo-publish-job.ts<br/>*/5s"]
        PING_JOB["ping-pong<br/>*/5s"]
    end

    SERVER --> ERROR
    SERVER --> WS_PLUGIN
    SERVER --> CRON_SETUP
    SERVER --> MQTT_SVC
    SERVER --> RELOAD_MQTT
    CRON_SETUP --> LOGO_JOB
    CRON_SETUP --> PING_JOB
    LOGO_JOB --> LOGO_PUB
    PING_JOB --> PING_PONG
    MQTT_SVC --> DH_REPO
    MQTT_SVC --> READER_ACCESS
    MQTT_SVC --> LOGO_PUB
    DH_REPO --> KEY_MAPPER
    DH_REPO --> RT_HUB
    CHARTS_WS --> RT_HUB
    READER_ACCESS --> MQTT_SVC
    READER_ACCESS --> VALIDATE
    AUTH_SVC --> HASH_SVC
    AUTH_SVC --> EMAIL_SVC
    BADGE_SVC --> DERIVE_KEY
    BADGE_SVC --> DERIVE_AB
    LOGO_PUB --> MQTT_SVC
    LOGO_PUB --> DEV_REPO
    PING_PONG --> MQTT_SVC
    PING_PONG --> DEV_REPO
    RELOAD_MQTT --> DEV_REPO
    RELOAD_MQTT --> MQTT_SVC
```

### 4.1 Pattern architectural backend

Le backend suit une **architecture feature-based** (par domaine métier) :

```
features/
├── auth/           → Authentification (login, refresh, forgot, reset, profile)
├── users/          → Gestion des utilisateurs CRUD
├── devices/        → Gestion des devices IoT CRUD
├── badges/         → Gestion des badges NFC MIFARE
├── charts/         → Pages et cartes de visualisation + WebSocket
├── data-histories/ → Historique des données MQTT
└── access-log/     → Journaux d'accès physique
```

Chaque feature contient : `entity.ts` (modèle TypeORM), `repository.ts` (accès données), `schema.ts` (validation Zod), et des sous-dossiers par route (`add/`, `getAll/`, etc.).

---

## 5. Architecture frontend — structure des modules

```mermaid
graph TB
    subgraph "Entry"
        MAIN["main.tsx"]
        APP["App.tsx"]
        ROUTER_PROVIDER["TanstackRouterAppProvider.tsx"]
    end

    subgraph "Routes (file-based TanStack Router)"
        ROOT["__root.tsx"]
        AUTH_LAYOUT["auth/__auth.tsx"]
        LOGIN_PAGE["auth/login.tsx"]
        LOGOUT_PAGE["auth/logout.tsx"]
        FORGOT_PAGE["auth/forgot-password.tsx"]
        RESET_PAGE["auth/reset-password.tsx"]
        AUTH_GUARD["_authenticated.tsx"]
        DASH_LAYOUT["_authenticated/dashboard.tsx"]
        DASH_INDEX["dashboard/index.tsx"]
        USERS_PAGE["dashboard/users/"]
        DEVICES_PAGE["dashboard/devices/"]
        CHARTS_PAGE["dashboard/charts/"]
        ACCESS_PAGE["dashboard/accessLogs/"]
        LIGHT_PAGE["dashboard/lightings.tsx"]
    end

    subgraph "Features"
        subgraph "auth (feature)"
            AUTH_COMPS["LoginForm<br/>ForgotPasswordForm<br/>ResetPasswordForm<br/>Logout"]
            AUTH_HOOKS["useAuth<br/>useAuthRefresh<br/>useAuthStore<br/>useAuthToken<br/>useWs"]
        end

        subgraph "users (feature)"
            USER_COMPS["UsersList<br/>CreateUser<br/>UpdateUser<br/>UserForm<br/>BadgesList"]
            USER_HOOKS["useUsers<br/>addUser<br/>getUser<br/>updateUser<br/>deleteUser<br/>createBadge<br/>updateBadge<br/>deleteBadge"]
        end

        subgraph "devices (feature)"
            DEV_COMPS["DevicesList<br/>CreateDevice<br/>UpdateDevice<br/>DeviceForm"]
            DEV_HOOKS["getAllDevices<br/>getDevice<br/>addDevice<br/>updateDevice<br/>activeDevice"]
        end

        subgraph "charts (feature)"
            CHART_COMPS["Charts<br/>DynamicNavbar<br/>DynamicPage"]
            CHART_HOOKS["getAllPagesName<br/>getPage<br/>useLiveKeyValues"]
        end

        subgraph "accessLogs (feature)"
            AL_COMPS["AccessLogsList"]
            AL_HOOKS["getAllAccessLogs"]
        end
    end

    subgraph "Shared"
        COMPONENTS["HeaderActions<br/>InputSearch<br/>LanguageSwitcher<br/>SideBar<br/>DashboardLayout"]
        UTILS["fetchUtil<br/>randomUuid<br/>useDebounce<br/>ws (WebSocket)"]
        THEME["blackDashboardTheme"]
        I18N_CFG["i18n.ts"]
    end

    MAIN --> APP
    APP --> ROUTER_PROVIDER
    ROUTER_PROVIDER --> ROOT
    ROOT --> AUTH_LAYOUT
    ROOT --> AUTH_GUARD
    AUTH_LAYOUT --> LOGIN_PAGE
    AUTH_GUARD --> DASH_LAYOUT
    DASH_LAYOUT --> USERS_PAGE
    DASH_LAYOUT --> DEVICES_PAGE
    DASH_LAYOUT --> CHARTS_PAGE
    DASH_LAYOUT --> ACCESS_PAGE
    LOGIN_PAGE --> AUTH_COMPS
    USERS_PAGE --> USER_COMPS
    DEVICES_PAGE --> DEV_COMPS
    CHARTS_PAGE --> CHART_COMPS
    ACCESS_PAGE --> AL_COMPS
    AUTH_COMPS --> AUTH_HOOKS
    USER_COMPS --> USER_HOOKS
    DEV_COMPS --> DEV_HOOKS
    CHART_COMPS --> CHART_HOOKS
    AL_COMPS --> AL_HOOKS
```

### 5.1 Pattern architectural frontend

Le frontend suit le même pattern **feature-based** avec TanStack Router en routing par fichier :

```
src/
├── routes/          → File-based routing (génère routeTree.gen.ts)
│   ├── auth/        → Routes publiques (login, forgot, reset)
│   └── _authenticated/dashboard/  → Routes protégées
├── features/        → Modules métier
│   ├── auth/        → components/ + hooks/ + types/ + mocks/
│   ├── users/       → components/ + hooks/ + types/ + schemas/
│   ├── devices/     → components/ + hooks/ + types/ + schemas/
│   ├── charts/      → components/ + hooks/ + schemas/
│   └── accessLogs/  → components/ + hooks/ + types/
├── components/      → Composants partagés (Sidebar, Layout)
├── utils/           → Utilitaires (fetch, WS, debounce)
└── themes/          → Thème MUI
```

---

## 6. Diagramme entité-relation (base de données)

```mermaid
erDiagram
    users {
        uuid id PK
        varchar firstname
        varchar lastname
        varchar username
        varchar password "Argon2 hash"
        varchar mail UK
        enum role "admin | employee | security"
        varchar refreshToken "nullable"
        varchar resetToken "nullable"
        timestamp resetTokenExpiry "nullable"
        timestamp lastLoginAt "nullable"
        timestamp createdAt
        timestamp updatedAt
    }

    devices {
        uuid id PK
        varchar deviceId
        varchar name
        varchar brand
        varchar model
        varchar type "sensor | controller | API"
        varchar subscribe "nullable - topic MQTT"
        varchar publish "nullable - topic MQTT"
        varchar seed "nullable"
        varchar description "nullable"
        boolean isOnline "default true"
        boolean isActive "default true"
        boolean isSecure "default true"
        text[] keyValues "nullable"
        enum[] roles
        enum status "active | inactive"
        json metadata "nullable"
        timestamp createdAt
        timestamp updatedAt
    }

    badges {
        uuid id PK
        bytea cardId "MIFARE UID"
        uuid userId UK
        boolean deniedAccessFlag "default false"
        bytea keyA "MIFARE Key A"
        bytea keyB "MIFARE Key B"
        timestamp createdAt
        timestamp updatedAt
    }

    data_histories {
        uuid id PK
        uuid device_id FK
        timestamp timestamp
        json payload
        timestamp createdAt
        timestamp updatedAt
    }

    access_logs {
        uuid id PK
        bytea cardId "nullable"
        uuid userId "nullable"
        varchar accessOutcome "granted | denied"
        varchar source "reader ID"
        timestamp createdAt
        timestamp updatedAt
    }

    pages {
        uuid id PK
        varchar name
        uuid[] cardIds
        timestamp createdAt
        timestamp updatedAt
    }

    cards {
        uuid id PK
        varchar deviceId
        varchar name
        int index
        varchar keyValue
        timestamp createdAt
        timestamp updatedAt
    }

    users ||--o{ badges : "1 user → 0..1 badge"
    devices ||--o{ data_histories : "1 device → N histories"
    devices ||--o{ cards : "1 device → N cards (via deviceId)"
    pages ||--o{ cards : "1 page → N cards (via cardIds[])"
```

### 6.1 Notes sur le modèle

- **7 entités** au total, toutes avec UUID comme clé primaire (via `BaseEntity`)
- Les relations `pages ↔ cards` et `devices ↔ cards` passent par des champs `cardIds[]` et `deviceId` (varchar) plutôt que par des FK TypeORM — ce sont des **relations logiques non contraintes** en base
- La relation `badges ↔ users` est liée par `userId` (unique) mais sans FK TypeORM déclarée
- Les `access_logs` référencent `cardId` et `userId` sans FK — journalisation découplée
- `data_histories` est la seule entité avec une vraie relation TypeORM (`@ManyToOne`)

---

## 7. Diagramme des routes API REST

```mermaid
graph LR
    subgraph "Fastify Server :3001"
        subgraph "/ (racine)"
            ROOT_GET["GET / → health check"]
            STATUS["GET /api/v1/status"]
        end

        subgraph "/api/v1 — Auth"
            LOGIN_POST["POST /login"]
            PROFILE_GET["GET /profile 🔒"]
            REFRESH_POST["POST /refresh"]
            FORGOT_POST["POST /forgot-password"]
            RESET_POST["POST /reset-password"]
        end

        subgraph "/api/v1 — Users 🔒"
            USERS_GET["GET /users"]
            USER_GET["GET /users/:id"]
            USER_POST["POST /users"]
            USER_PUT["PUT /users/:id"]
            USER_DEL["DELETE /users/:id"]
        end

        subgraph "/api/v1 — Devices 🔒"
            DEVS_GET["GET /devices"]
            DEV_GET["GET /devices/:id"]
            DEV_POST["POST /devices"]
            DEV_PUT["PUT /devices/:id"]
            DEV_ACTIVE["POST /devices/active"]
        end

        subgraph "/api/v1 — Badges 🔒"
            BADGE_POST["POST /badges"]
            BADGE_PUT["PUT /badges/:id"]
            BADGE_DEL["DELETE /badges/:id"]
        end

        subgraph "/api/v1 — Pages 🔒"
            PAGES_GET["GET /pages"]
            PAGE_GET["GET /pages/:id"]
        end

        subgraph "/api/v1 — Autres 🔒"
            AL_GET["GET /access-logs"]
            DH_GET["GET /data-histories"]
            DH_POST["POST /data-histories"]
        end

        subgraph "WebSocket"
            WS_CHARTS["WS /ws/charts 🍪"]
        end
    end
```

### 7.1 Détail des endpoints

| Méthode | Route | Auth | Description |
|---|---|---|---|
| `GET` | `/` | Non | Health check |
| `GET` | `/api/v1/status` | Non | Statut API |
| `POST` | `/api/v1/login` | Non | Connexion (username/email + password) |
| `POST` | `/api/v1/refresh` | JWT | Renouvellement access token |
| `POST` | `/api/v1/forgot-password` | Non | Demande de réinitialisation |
| `POST` | `/api/v1/reset-password` | Non | Réinitialisation avec token |
| `GET` | `/api/v1/profile` | JWT | Profil utilisateur connecté |
| `GET` | `/api/v1/users` | JWT | Liste paginée (page, size) |
| `GET` | `/api/v1/users/:id` | JWT | Détail utilisateur |
| `POST` | `/api/v1/users` | JWT | Création utilisateur |
| `PUT` | `/api/v1/users/:id` | JWT | Mise à jour utilisateur |
| `DELETE` | `/api/v1/users/:id` | JWT | Suppression utilisateur |
| `GET` | `/api/v1/devices` | JWT | Liste paginée (page, size, includeActived) |
| `GET` | `/api/v1/devices/:id` | JWT | Détail device |
| `POST` | `/api/v1/devices` | JWT | Création device |
| `PUT` | `/api/v1/devices/:id` | JWT | Mise à jour device |
| `POST` | `/api/v1/devices/active` | JWT | Toggle actif/inactif |
| `POST` | `/api/v1/badges` | JWT | Ajout badge NFC |
| `PUT` | `/api/v1/badges/:id` | JWT | Modification badge |
| `DELETE` | `/api/v1/badges/:id` | JWT | Suppression badge |
| `GET` | `/api/v1/pages` | JWT | Liste des pages dashboard |
| `GET` | `/api/v1/pages/:id` | JWT | Détail page avec cards |
| `GET` | `/api/v1/access-logs` | JWT | Liste des logs d'accès |
| `GET` | `/api/v1/data-histories` | JWT | Historique données |
| `POST` | `/api/v1/data-histories` | JWT | Sauvegarde données |
| `WS` | `/ws/charts` | Cookie JWT | Temps réel (subscribe/unsubscribe) |

---

## 8. Diagramme des routes frontend

```mermaid
graph TB
    subgraph "Routes publiques"
        AUTH_LOGIN["/auth/login<br/>LoginForm"]
        AUTH_FORGOT["/auth/forgot-password<br/>ForgotPasswordForm"]
        AUTH_RESET["/auth/reset-password<br/>ResetPasswordForm"]
        AUTH_LOGOUT["/auth/logout<br/>Logout"]
    end

    subgraph "Routes protégées (_authenticated)"
        subgraph "Dashboard Layout"
            DASH_HOME["/dashboard<br/>Index"]
            subgraph "Users"
                USERS_LIST["/dashboard/users<br/>UsersList"]
                USERS_CREATE["/dashboard/users/create<br/>CreateUser"]
                USERS_EDIT["/dashboard/users/$id<br/>UpdateUser"]
            end
            subgraph "Devices"
                DEVS_LIST["/dashboard/devices<br/>DevicesList"]
                DEVS_CREATE["/dashboard/devices/create<br/>CreateDevice"]
                DEVS_EDIT["/dashboard/devices/$id<br/>UpdateDevice"]
            end
            subgraph "Charts"
                CHARTS_INDEX["/dashboard/charts<br/>DynamicPage + Vega"]
            end
            subgraph "Access Logs"
                AL_INDEX["/dashboard/accessLogs<br/>AccessLogsList"]
            end
            LIGHTINGS["/dashboard/lightings<br/>(stub)"]
        end
    end

    ROOT_REDIRECT["/ → /dashboard"]
    ROOT_REDIRECT --> DASH_HOME
```

---

## 9. Diagramme de flux MQTT et temps réel

```mermaid
sequenceDiagram
    participant SENSOR as Capteur DHT
    participant LOGO as Siemens LOGO
    participant NFC as Reader NFC
    participant BROKER as Broker MQTT
    participant MQTT as mqtt.service.ts
    participant DH as data-histories.repository
    participant KM as key-mapper.ts
    participant RT as realtime-hub.ts
    participant WS as WebSocket /ws/charts
    participant FE as Frontend React
    participant LOGO_PUB as logo-publisher.service
    participant READER as handleReaderAccess.ts

    Note over SENSOR,BROKER: FLUX ENTRANT (Subscribe)

    SENSOR->>BROKER: unifyIots/sensor/custom-sensor-dth-01/get
    BROKER->>MQTT: on("message")
    MQTT->>LOGO_PUB: processTemperature()
    Note right of LOGO_PUB: Cache température<br/>Map<string, {value, timestamp}>
    MQTT->>DH: saveDataHistory()
    DH->>KM: keyMapper.map(device, payload)
    KM-->>DH: [{deviceId, key, value}, ...]
    DH->>RT: realtimeHub.publish(deviceId, key, value)
    RT->>WS: ws.send({deviceId, key, value})
    WS->>FE: Mise à jour temps réel

    LOGO->>BROKER: unifyIots/API/logo/get
    BROKER->>MQTT: on("message")
    MQTT->>LOGO_PUB: processTemperature()
    MQTT->>DH: saveDataHistory()
    DH->>KM: siemensLogo84Adapter.map()
    KM-->>DH: [{deviceId, "courant", 12.5}, ...]
    DH->>RT: realtimeHub.publish()
    RT->>FE: Temps réel cards Logo

    Note over LOGO_PUB,BROKER: FLUX SORTANT — CRON */5s

    LOGO_PUB->>LOGO_PUB: scheduleLogoPublishingTick()
    Note right of LOGO_PUB: Math.min(temperatures valides)
    LOGO_PUB->>BROKER: unifyIots/API/logo/set
    BROKER->>LOGO: {state:{temperature:{value:[T]},heatSetpoint:{value:[1850]}}}

    Note over NFC,BROKER: FLUX CONTRÔLE D'ACCÈS

    NFC->>BROKER: unifyIots/controller/reader-nfc-01/get
    BROKER->>MQTT: on("message") — topic.includes("/controller/reader-nfc-")
    MQTT->>READER: handleReaderAccess()
    READER->>BROKER: unifyIots/API/logo/set → {state:{open:{value:[1]}}}
    BROKER->>LOGO: Ouverture porte
```

### 9.1 Topics MQTT par device

| Device | Type | Subscribe (get) | Publish (set) |
|---|---|---|---|
| custom-sensor-dth-01 | sensor | `unifyIots/sensor/custom-sensor-dth-01/get` | — |
| reader-nfc-01 | controller | `unifyIots/controller/reader-nfc-01/get` | `unifyIots/controller/reader-nfc-01/set` |
| writer-nfc-01 | controller | `unifyIots/controller/writer-nfc-01/get` | `unifyIots/controller/writer-nfc-01/set` |
| logo-01 | API | `unifyIots/API/logo/get` | `unifyIots/API/logo/set` |

### 9.2 Convention des topics

```
unifyIots/{type}/{deviceId}/{get|set}
```

- `get` : topic subscribe (device → backend)
- `set` : topic publish (backend → device)
- `{type}` : `sensor`, `controller`, ou `API`

---

## 10. Diagramme de séquence — contrôle d'accès NFC

```mermaid
sequenceDiagram
    actor USER as Utilisateur
    participant NFC as Reader NFC (PN532)
    participant BROKER as Broker MQTT
    participant BACKEND as handleReaderAccess.ts
    participant DB_DEVICE as DeviceEntity
    participant DB_BADGE as BadgeEntity
    participant DB_LOG as AccessLogEntity
    participant LOGO as Siemens LOGO (Porte)

    USER->>NFC: Présente badge MIFARE

    Note over NFC,BACKEND: Étape 0 — STATE_STEP_VERIFYING_KEY_A

    NFC->>BROKER: {uid, seed, step:"0"}
    BROKER->>BACKEND: handleReaderAccess()
    BACKEND->>DB_DEVICE: findOne({seed})
    DB_DEVICE-->>BACKEND: reader device
    BACKEND->>BROKER: {received: true, uid}
    BROKER->>NFC: ACK

    BACKEND->>DB_BADGE: findOne({cardId: uid})

    alt Badge non trouvé
        BACKEND->>BROKER: {uid, access:"denied", source:"badge-not-found"}
        BACKEND->>DB_LOG: save({outcome:"denied"})
    else Badge trouvé
        BACKEND->>BROKER: {keyA: "hex"}
        BROKER->>NFC: Clé d'authentification

        Note over NFC,BACKEND: Étape 1 — STATE_STEP_WAITING_ACCESS_GRANTED

        NFC->>NFC: Authentification MIFARE avec keyA
        NFC->>BROKER: {derivedKey, uid, seed, step:"1"}
        BROKER->>BACKEND: handleReaderAccess()

        BACKEND->>DB_BADGE: findOne({cardId: uid})
        BACKEND->>BACKEND: validateBadgeWithKey(derivedKey, cardId, userId)

        alt Validation OK
            BACKEND->>BROKER: {access:"granted"} → Reader
            BACKEND->>BROKER: {state:{open:{value:[1]}}} → Logo
            BROKER->>LOGO: OUVERTURE PORTE
            BACKEND->>DB_LOG: save({outcome:"granted"})
        else Validation échouée
            BACKEND->>BROKER: {access:"denied"} → Reader
            BACKEND->>DB_LOG: save({outcome:"denied"})
        end
    end
```

---

## 11. Diagramme de séquence — authentification

```mermaid
sequenceDiagram
    actor USER as Navigateur
    participant FE as Frontend React
    participant NGINX as Nginx :80
    participant BE as Backend Fastify
    participant DB as PostgreSQL

    Note over USER,DB: LOGIN

    USER->>FE: Saisie username + password
    FE->>NGINX: POST /api/v1/login
    NGINX->>BE: Proxy → :3001
    BE->>DB: findOne({username}) OR findOne({mail})
    DB-->>BE: UserEntity
    BE->>BE: Argon2.verify(password, hash)

    alt Credentials valides
        BE->>BE: JWT sign accessToken (1h)
        BE->>BE: JWT sign refreshToken (7d)
        BE->>DB: save({refreshToken})
        BE-->>NGINX: 200 {accessToken, refreshToken, user} + Set-Cookie httpOnly
        NGINX-->>FE: Response
        FE->>FE: react-auth-kit stocke tokens
    else Credentials invalides
        BE-->>FE: 401 Unauthorized
    end

    Note over USER,DB: REFRESH TOKEN

    FE->>BE: POST /api/v1/refresh {refreshToken}
    BE->>BE: JWT verify refreshToken
    BE->>DB: findOne({refreshToken})

    alt Token valide et match DB
        BE->>BE: JWT sign new accessToken (1h)
        BE-->>FE: 200 {accessToken}
    else Token invalide
        BE-->>FE: 401 → redirect /auth/login
    end

    Note over USER,DB: WEBSOCKET (Charts temps réel)

    FE->>NGINX: WS /ws/charts
    NGINX->>BE: Upgrade WebSocket
    BE->>BE: Extraire JWT depuis cookie
    BE->>BE: JWT verify

    alt Cookie JWT valide
        BE-->>FE: WebSocket OPEN
        FE->>BE: {action:"subscribe", pairs:[{deviceId, key}]}
        BE->>BE: realtimeHub.subscribe(ws, pairs)
    else Cookie invalide
        BE-->>FE: WebSocket CLOSE 4401
    end
```

---

## 12. Diagramme de dépendances des services backend

```mermaid
graph TB
    subgraph "Couche HTTP (Routes)"
        R_AUTH["Auth routes"]
        R_USERS["Users routes"]
        R_DEVICES["Devices routes"]
        R_BADGES["Badges routes"]
        R_PAGES["Pages routes"]
        R_DH["Data-histories routes"]
        R_AL["Access-logs routes"]
    end

    subgraph "Couche Service"
        S_AUTH["AuthService"]
        S_BADGE["BadgesService"]
        S_LOGO["LogoPublisherService"]
        S_CRON["CronService"]
        S_READER["handleReaderAccess"]
        S_PING["PingPongService"]
    end

    subgraph "Couche Infrastructure"
        I_MQTT["MqttService<br/>(singleton)"]
        I_HASH["HashService<br/>(static)"]
        I_EMAIL["EmailService"]
        I_CRYPTO["crypto.ts"]
    end

    subgraph "Couche Data"
        D_USER["UserRepository"]
        D_DEVICE["DeviceRepository"]
        D_BADGE["BadgesRepository"]
        D_PAGE["PageRepository"]
        D_DH["DataHistoriesRepository"]
        D_AL["AccessLogsRepository"]
    end

    subgraph "Couche Realtime"
        RT_MAPPER["KeyMapper"]
        RT_HUB["RealtimeHub"]
    end

    subgraph "Base de données"
        PG["PostgreSQL<br/>(TypeORM)"]
    end

    R_AUTH --> S_AUTH
    R_BADGES --> S_BADGE
    S_AUTH --> I_HASH
    S_AUTH --> I_EMAIL
    S_AUTH --> D_USER
    S_BADGE --> I_MQTT
    S_BADGE --> D_BADGE
    S_BADGE --> D_DEVICE
    S_LOGO --> I_MQTT
    S_LOGO --> D_DEVICE
    S_READER --> I_MQTT
    S_READER --> D_BADGE
    S_READER --> D_DEVICE
    S_READER --> D_AL
    S_PING --> I_MQTT
    S_PING --> D_DEVICE
    S_CRON --> S_LOGO
    S_CRON --> S_PING
    I_MQTT --> D_DH
    D_DH --> RT_MAPPER
    D_DH --> RT_HUB
    R_USERS --> D_USER
    R_DEVICES --> D_DEVICE
    R_PAGES --> D_PAGE
    R_DH --> D_DH
    R_AL --> D_AL
    D_USER --> PG
    D_DEVICE --> PG
    D_BADGE --> PG
    D_PAGE --> PG
    D_DH --> PG
    D_AL --> PG
```

### 12.1 Observations sur les couches

Le backend utilise implicitement une **architecture en 4 couches** :

| Couche | Rôle | Fichiers |
|---|---|---|
| **HTTP** | Routes Fastify, validation Zod, sérialisation | `*.route.ts` |
| **Service** | Logique métier, orchestration | `*.service.ts`, `handleReaderAccess.ts` |
| **Infrastructure** | Services techniques transverses | `mqtt.service.ts`, `hash.service.ts`, `email.service.ts` |
| **Data** | Accès base de données | `*.repository.ts` |
| **Realtime** | Transformation de données et diffusion WS | `key-mapper.ts`, `realtime-hub.ts` |

Toutefois, cette séparation n'est **pas strictement appliquée** : certaines routes accèdent directement aux repositories sans passer par un service, et le `MqttService` instancie directement un `DataHistoriesRepository` dans son handler de message.

---

## 13. Inventaire des dépendances externes

### 13.1 Backend — dépendances de production (33)

| Catégorie | Package | Version |
|---|---|---|
| **Framework** | `fastify` | ^5.2.1 |
| | `fastify-plugin` | ^5.0.1 |
| | `fastify-type-provider-zod` | ^4.0.2 |
| **Auth** | `@fastify/auth` | ^5.0.2 |
| | `@fastify/jwt` | ^9.0.3 |
| | `@fastify/passport` | ^3.0.2 |
| | `@fastify/cookie` | ^11.0.2 |
| | `argon2` | ^0.43.0 |
| | `jsonwebtoken` | ^9.0.2 |
| **DB/ORM** | `typeorm` | ^0.3.20 |
| | `pg` | ^8.13.1 |
| | `@fastify/postgres` | ^6.0.2 |
| | `reflect-metadata` | ^0.2.2 |
| | `@prisma/client` | ^6.3.1 |
| **Communication** | `mqtt` | ^5.10.3 |
| | `@fastify/websocket` | ^11.2.0 |
| | `modbus-serial` | 8.0.20-no-serial-port |
| | `nodemailer` | ^7.0.3 |
| | `mailgen` | ^2.0.29 |
| **Sécurité** | `@fastify/cors` | ^10.0.2 |
| | `@fastify/helmet` | ^13.0.1 |
| **Utilitaires** | `zod` | ^3.24.4 |
| | `dotenv` | ^16.4.7 |
| | `pino` | ^9.6.0 |
| | `pino-pretty` | ^13.0.0 |
| | `uuid` | ^11.1.0 |
| | `date-fns` | ^4.1.0 |
| | `async-mutex` | ^0.5.0 |
| | `cron-schedule` | ^5.0.4 |
| | `true-myth` | ^9.0.1 |
| **Factory/Seed** | `@jorgebodega/typeorm-factory` | ^2.1.0 |
| | `@jorgebodega/typeorm-seeding` | ^7.1.0 |
| | `@faker-js/faker` | ^9.4.0 |

### 13.2 Frontend — dépendances de production (26)

| Catégorie | Package | Version |
|---|---|---|
| **Framework** | `react` | ^19.0.0 |
| | `react-dom` | ^19.0.0 |
| **Routing** | `@tanstack/react-router` | ^1.105.0 |
| **CSS** | `@tailwindcss/vite` | ^4.0.9 |
| **UI** | `@mui/material` | ^6.4.3 |
| | `@mui/icons-material` | ^6.4.7 |
| | `@mui/x-data-grid` | ^7.26.0 |
| | `@material-ui/core` | ^4.12.4 |
| | `@emotion/react` | ^11.14.0 |
| | `@emotion/styled` | ^11.14.0 |
| | `@toolpad/core` | ^0.12.0 |
| **Charts** | `vega` | ^6.1.2 |
| | `vega-lite` | ^6.2.0 |
| | `react-vega` | ^7.7.1 |
| | `@material-vega/core` | ^0.1.0 |
| | `@material-vega/material-ui` | ^0.2.1 |
| **DnD** | `@dnd-kit/core` | ^6.3.1 |
| | `@dnd-kit/sortable` | ^10.0.0 |
| | `@dnd-kit/utilities` | ^3.2.2 |
| **Auth** | `react-auth-kit` | ^3.1.3 |
| **i18n** | `i18next` | ^24.2.2 |
| | `react-i18next` | ^15.4.0 |
| **WebSocket** | `react-use-websocket` | ^4.13.0 |
| **Utilitaires** | `zod` | ^3.24.2 |
| | `dot-prop` | ^9.0.0 |
| | `react-helmet-async` | ^2.0.5 |

### 13.3 Dépendances notables à surveiller

| Package | Observation |
|---|---|
| `@prisma/client` ^6.3.1 | Installé mais **non utilisé** dans le code — seul TypeORM est actif |
| `@material-ui/core` ^4.12.4 | MUI v4 **obsolète** — coexiste avec MUI v6 (`@mui/material`) |
| `modbus-serial` 8.0.20-no-serial-port | Version patchée sans port série — installée mais aucune utilisation trouvée dans le code |
| `@jorgebodega/typeorm-factory` + `@faker-js/faker` | En `dependencies` (prod) au lieu de `devDependencies` |
| `@fastify/postgres` ^6.0.2 | Plugin PostgreSQL natif installé en parallèle de TypeORM — double ORM potentiel |

---

## 14. Constats sur l'architecture

| # | Constat | Sévérité | Impact |
|---|---|---|---|
| C01 | **Architecture feature-based cohérente** entre backend et frontend — même découpage par domaine métier | ✅ Positif | Facilite la navigation et la maintenance |
| C02 | **Séparation des couches incomplète** — certaines routes accèdent directement aux repositories sans service intermédiaire (ex: `getAllUsers.route.ts` → `UserRepository`) | 3/5 | Logique métier dispersée, testabilité réduite |
| C03 | **Double ORM installé** — `@prisma/client` (non utilisé) coexiste avec TypeORM | 2/5 | Confusion, dépendance inutile à maintenir |
| C04 | **Double MUI installé** — `@material-ui/core` v4 et `@mui/material` v6 en parallèle | 3/5 | Bundle size inutilement alourdi, API inconsistantes |
| C05 | **`modbus-serial` installé mais non utilisé** — aucun import trouvé dans le code | 2/5 | Dépendance orpheline |
| C06 | **Dépendances de test en production** — `@faker-js/faker`, `@jorgebodega/typeorm-factory`, `@jorgebodega/typeorm-seeding` sont dans `dependencies` au lieu de `devDependencies` | 3/5 | Image Docker plus lourde, risque en production |
| C07 | **Serveur unique** — broker MQTT, registre Docker, DB, backend, frontend sur la même machine `192.168.3.100` | 3/5 | SPOF (Single Point of Failure) |
| C08 | **Relations DB non contraintes** — la plupart des relations (pages↔cards, badges↔users) sont logiques sans FK TypeORM | 3/5 | Intégrité référentielle non garantie |
| C09 | **`synchronize: true` en production** — auto-synchronisation du schéma DB activée (data-source.ts) | 4/5 | Risque de perte de données en production |
| C10 | **Pas de validation de rôles sur les routes** — les routes protégées vérifient l'authentification JWT mais pas les rôles (admin/employee/security) | 4/5 | Tout utilisateur authentifié a accès à toutes les routes |
| C11 | **Route `/dashboard/lightings`** stub — page référencée dans le routeur mais sans contenu implémenté | 1/5 | Fonctionnalité incomplète |
| C12 | **PWA configurée** avec service worker et precaching — bonne pratique pour une app dashboard IoT locale | ✅ Positif | Utilisable hors-ligne |
| C13 | **Pas de rate limiting** sur les routes d'authentification (login, forgot-password) | 3/5 | Vulnérable au brute-force |
| C14 | **Config proxy Vite pointe vers la prod** — `vite.config.ts` frontend proxy vers `192.168.3.100:3000` (IP prod) au lieu de localhost | 2/5 | Développement couplé à l'infra de production |

---

## 15. Synthèse

### 15.1 Score d'architecture

| Critère | Score | Commentaire |
|---|---|---|
| Organisation du code | 4/5 | Feature-based cohérent, nommage clair, structure prévisible |
| Séparation des responsabilités | 3/5 | Couches identifiables mais pas toujours respectées |
| Modèle de données | 3/5 | 7 entités bien définies mais relations faiblement contraintes |
| Sécurité | 2/5 | JWT + Argon2 présents mais pas de RBAC, TLS partiel, pas de rate-limiting |
| Scalabilité | 2/5 | Serveur unique, pas de message queue, CRON en-process |
| Maintenabilité | 3/5 | Monorepo pnpm, TypeScript strict, mais double ORM et dépendances orphelines |
| Observabilité | 2/5 | Pino logger mais pas de métriques, pas de health checks MQTT |
| **Score global** | **2.7/5** | Architecture fonctionnelle et bien structurée pour un projet IoT local, mais avec des lacunes de sécurité et d'infrastructure à combler pour la production |

### 15.2 Résumé quantitatif

| Métrique | Valeur |
|---|---|
| Diagrammes produits | 11 (architecture, ER, séquence, dépendances, flux) |
| Entités base de données | 7 |
| Endpoints API REST | 17 + 1 WebSocket |
| Routes frontend | 14 |
| Services backend | 9 (auth, badge, logo-publisher, cron, mqtt, hash, email, reader-access, ping-pong) |
| Dépendances backend (prod) | 33 |
| Dépendances frontend (prod) | 26 |
| Constats architecturaux | 14 (2 positifs, 12 à améliorer) |
