# Architecture Telemetry — simpl1Control

Date : 21 mars 2026

---

## Contexte et positionnement

Service dédié à l'historisation des données IoT. Conçu pour le profil **PME / bureau / étude** (1-50 devices, usage horaires de bureau) mais architecturé pour tenir à l'échelle industrielle sans refonte.

**Nom du service :** `telemetry` (`packages/telemetry`)
**Stack :** Fastify v5 + TypeORM + fastify-type-provider-zod (identique au backend principal)
**DB :** TimescaleDB (extension PostgreSQL)
**Communication entrante :** MQTT — topic `s1c/internal/telemetry/`

---

## Structure du package `packages/telemetry`

Créé from scratch à partir de la structure du backend principal, en reprenant uniquement le socle technique. Pas de copie de code métier.

### Ce qui est repris du backend
- Structure de dossiers (`src/`, `migrations/`, `features/`)
- `app.ts` — init Fastify + plugins de base (sans JWT/cookie)
- `data-source.ts` — TypeORM configuré vers TimescaleDB
- `server.ts` — point d'entrée
- `tsconfig.json`, `package.json`, config eslint/vitest
- Scripts `migration:run`, `migration:create`, `dev`, `build`

### Ce qui est exclu (non nécessaire)
| Exclu | Raison |
|---|---|
| `@fastify/jwt` / `@fastify/cookie` | Pas d'auth propre — délègue au backend principal |
| `@fastify/passport` | Idem |
| Seeders / factories | Pas de données fictives — flux temps réel uniquement |
| Features existantes (users, devices, badges...) | Pas du ressort de telemetry |
| `argon2`, `nodemailer`, `mailgen` | Pas de gestion utilisateur |

### Ce qui est spécifique à `telemetry`
- Client MQTT — abonné à `s1c/internal/telemetry/#`
- Feature `device_history` — entity + migration hypertable + SQL brut TimescaleDB
- Lecture des `telemetry_policies` depuis le backend principal (API REST ou cache local)
- Endpoints de lecture historique pour le frontend
- `@fastify/websocket` — push d'agrégats temps réel si nécessaire

### Docker
- `Dockerfile` dédié dans `packages/telemetry/`
- Nouvelle entrée dans `docker-compose.yml` à la racine du monorepo
- Variables d'env propres : `TIMESCALE_HOST`, `TIMESCALE_PORT`, `TIMESCALE_DB`, `TIMESCALE_USER`, `TIMESCALE_PASSWORD`, `MQTT_BROKER_URL`

---

## Principe de fonctionnement TimescaleDB

### Table principale — Hypertable

Une seule table `device_history` reçoit tous les messages en continu. TimescaleDB la découpe automatiquement en **chunks** par intervalle de temps (7 jours par défaut) — transparent pour l'application.

```
device_history (vue unifiée)
  ├── chunk_2026_03_01  (données du 1 au 7 mars)
  ├── chunk_2026_03_08  (données du 8 au 14 mars)
  └── chunk_2026_03_15  (données en cours)
```

La suppression de vieux chunks via Retention Policy est instantanée (suppression d'une sous-table entière, pas de DELETE ligne par ligne).

### Continuous Aggregates

Vues matérialisées qui se recalculent automatiquement en **delta** (uniquement les nouveaux buckets, pas un full-scan). Chaque niveau agrège le niveau précédent.

```
device_history (brut)
    └──► agg_15min   (agrège brut)       → 1 an
              └──► agg_1d  (agrège 15min) → indéfini
```

**Ce que contient un bucket agrégé :**
```
Données brutes : 30 valeurs sur 15 minutes
Bucket résultant :
  bucket : 14:00:00
  avg    : 21.45     ← moyenne
  min    : 21.3      ← minimum
  max    : 21.6      ← maximum
  count  : 30        ← nombre de mesures (indicateur d'activité)
```
Les données brutes sont perdues après leur rétention — seul le bucket statistique reste.

### Ce que le frontend interroge selon l'échelle

| Timerange demandé | Table interrogée |
|---|---|
| Dernières 24h | `device_history` (brut) |
| Dernière semaine / mois | `agg_15min` |
| Année et au-delà | `agg_1d` |

L'API `telemetry` choisit automatiquement la bonne table selon le `timeWindow` — le frontend ne sait pas laquelle est interrogée.

---

## Schéma DB — TimescaleDB

### Table `device_history` — hypertable principale

```
device_history  ← hypertable (partition sur time, chunk 7 jours)
├── time        timestamp   partition key — toujours NOT NULL
├── deviceId    uuid        loose ref → devices.id (backend principal)
├── key         varchar     "temperature" | "power" | "energy" | "state" ...
├── value       float       valeur numérique (nullable si non numérique)
└── raw         jsonb       payload brut complet (purge rapide)
```

**Schema-less dans l'esprit :** un nouveau device ou une nouvelle clé s'insère sans modification de schéma. Brancher un deuxième compteur = nouvelles lignes, zéro migration.

### Continuous Aggregates

```sql
-- Agrégat 15 minutes
CREATE MATERIALIZED VIEW agg_15min
WITH (timescaledb.continuous) AS
SELECT
    time_bucket('15 minutes', time) AS bucket,
    deviceId,
    key,
    AVG(value)   AS avg,
    MIN(value)   AS min,
    MAX(value)   AS max,
    COUNT(*)     AS count
FROM device_history
WHERE value IS NOT NULL
GROUP BY bucket, deviceId, key;

-- Agrégat 1 jour (agrège agg_15min)
CREATE MATERIALIZED VIEW agg_1d
WITH (timescaledb.continuous) AS
SELECT
    time_bucket('1 day', bucket) AS bucket,
    deviceId,
    key,
    AVG(avg)     AS avg,
    MIN(min)     AS min,
    MAX(max)     AS max,
    SUM(count)   AS count
FROM agg_15min
GROUP BY time_bucket('1 day', bucket), deviceId, key;
```

### Stratégie de rétention

| Niveau | Résolution | Rétention | Cas d'usage |
|---|---|---|---|
| `device_history` | Brut (variable) | 30 jours | "Cette semaine" — courbes temps réel |
| `agg_15min` | 15 minutes | 1 an | Courbes mensuelles, comparaisons |
| `agg_1d` | 1 jour | Indéfinie | Bilans énergétiques annuels, tendances long terme |

### Cas des données non numériques (state, action)

Les valeurs `ON/OFF`, `single_right`, etc. ne se moyennent pas. Stratégie :
- `value` = NULL pour ces clés
- `raw` jsonb conserve le payload complet pendant 30 jours
- Pas inclus dans les Continuous Aggregates (`WHERE value IS NOT NULL`)
- Enregistré uniquement à l'événement (changement de valeur) via `telemetry_policies`

---

## Policies d'historisation — `telemetry_policies`

Table dans le **backend principal** (pas dans TimescaleDB). Permet au super admin de configurer ce qui est historisé, pour quel device, à quelle fréquence — sans toucher au code.

```
telemetry_policies
├── id            uuid       PK
├── createdAt     timestamp
├── updatedAt     timestamp
├── deviceId      uuid       FK → devices.id (null = policy globale)
├── key           varchar    "temperature" | "power" | "*" (tous)
├── enabled       boolean    default true
├── minInterval   int        secondes — fréquence min d'enregistrement (0 = chaque message)
└── onChangeOnly  boolean    si true → enregistre uniquement quand la valeur change
```

### Valeurs recommandées par type de donnée

| Donnée | `minInterval` | `onChangeOnly` | Lignes/jour/device |
|---|---|---|---|
| Puissance électrique (`power`) | 30s | false | 2 880 |
| Énergie cumulée (`energy`) | 60s | false | 1 440 |
| Température | 120s | false | 720 |
| Humidité | 300s | false | 288 |
| État on/off (`state`) | 0 | true | ~10 (événements) |
| Batterie | 3600s | false | 24 |
| Qualité signal (`linkquality`) | 300s | false | 288 |

**Volume estimé installation type PME (20 devices) :**
- Sans policy : ~130 000 lignes/jour → 47M/an
- Avec policy : ~5 000 lignes/jour → 1.8M/an
- Confortable pour TimescaleDB, scalable vers l'industriel sans refonte

### Logique d'application des policies (backend principal)

```
Message MQTT reçu → backend principal
  Pour chaque key du payload :
    1. Cherche policy (deviceId + key) → sinon policy (deviceId + "*") → sinon policy globale
    2. Si enabled = false → skip
    3. Si onChangeOnly = true et value identique à device_states.state[key] → skip
    4. Si minInterval > 0 et lastSent < now - minInterval → skip
    5. Sinon → PUBLISH s1c/internal/telemetry/{deviceId}/{key}
```

---

## Flux complet

```
Message MQTT device
    │
    ▼
Backend principal
    ├── UPSERT device_states          (état courant)
    ├── Évalue telemetry_policies
    │       ├── enabled ?
    │       ├── minInterval respecté ?
    │       └── onChangeOnly → valeur changée ?
    │
    ├── [si policy OK] PUBLISH s1c/internal/telemetry/{deviceId}/{key}
    │
    └── EMIT WebSocket frontend

Service Telemetry (abonné à s1c/internal/telemetry/#)
    └── INSERT device_history
              └── TimescaleDB gère chunks + aggregates + retention
```

---

## Rôles et accès

| Rôle | Accès telemetry |
|---|---|
| `super_admin` | Configure `telemetry_policies`, voit toutes les installations |
| `admin` | Consulte l'historique de son installation, pas de config policy |
| `user` | Voit ce que l'admin lui autorise (tuiles chart dans l'interface) |

---

## Évolutivité vers l'industriel

L'architecture est identique à l'échelle — seules les policies et les rétentions changent :
- Réduire `minInterval` → plus de granularité
- Ajouter des niveaux d'agrégation (ex: `agg_5min`, `agg_1h`)
- Augmenter les rétentions

Aucune migration de schéma nécessaire pour passer du profil PME au profil industriel.
