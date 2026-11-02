# Guide d'apprentissage : hypertable `device_history` avec TimescaleDB

## Contexte du projet

Ton projet utilise déjà :

- TimescaleDB (Postgres 17) via Docker
- TypeORM pour les migrations
- Un package telemetry dédié aux données IoT
- Un backend avec des migrations TypeORM et une table `data_histories`

---

## Phase 1 – Analyse

### 1.1 Analyser le besoin vs l'existant

Spécifications du ticket :

- Table : `device_history`
- Colonnes : `time`, `deviceId`, `key`, `value`, `raw` (jsonb)
- Hypertable pour du time-series

À vérifier dans le code :

1. Structure similaire dans le backend
    
    - Regarde le fichier `data-history.entity.ts` : `packages/backend/src/features/data-histories/data-history.entity.ts`
    - Il montre un pattern proche (timestamp, payload JSON, device) mais différent de ton objectif (EAV : key/value).
2. Modèles de données pour IoT
    
    - Les colonnes `key`, `value`, `raw` (jsonb) permettent :
        - `key` : nom du paramètre (ex: "temperature")
        - `value` : valeur typée ou normalisée
        - `raw` : payload brut si besoin de trace complète
3. Documentation TimescaleDB à lire
    
    - Création d’hypertables :  
        [https://docs.timescale.com/use-timescale/latest/hypertables/create/](https://docs.timescale.com/use-timescale/latest/hypertables/create/)
    - `segment by` : colonnes utilisées dans les filtres (ex: `deviceId`)
    - `order by` : tri habituel (ex: `time DESC`)
    - `create_hypertable()` :  
        [https://docs.timescale.com/docs/api/latest/hypertable/create_hypertable](https://docs.timescale.com/docs/api/latest/hypertable/create_hypertable)

Questions à se poser :

- Quel type pour `time` ? (TIMESTAMPTZ recommandé)
- Quel type pour `value` ? (TEXT vs NUMERIC vs JSONB selon la variabilité des mesures)
- Relations avec d’autres tables (ex: `devices`) ? Clés étrangères ou pas ?

---

## Phase 2 – Programmation / Conception du schéma

### 2.1 Où placer le schéma et la migration

- Le schéma `device_history` est utilisé par le package telemetry.
- La migration doit être dans :  
    `packages/telemetry/src/migrations/`
- Pour l’instant, `migrations` est vide dans `data-source.ts`.

### 2.2 Référence pour les migrations TypeORM

Dans le backend, un exemple complet :

- Structure d’une migration
    
    - Fichier :  
        `packages/backend/src/migrations/1774124132693-migration.ts`
    - Interface : `MigrationInterface` avec `up()` et `down()`
    - `queryRunner.query()` pour exécuter du SQL
- Inscription de la migration
    
    - Fichier :  
        `packages/backend/src/data-source.ts`
    - Import et ajout dans le tableau `migrations`

### 2.3 Particularité TimescaleDB : hypertable

TimescaleDB ajoute une étape après la création de la table classique :

1. Créer une table PostgreSQL classique.
2. Transformer en hypertable avec `create_hypertable()`.

Tu peux soit :

- écrire du SQL pur dans `queryRunner.query()`, ou
- utiliser l’API TypeORM pour la table et du SQL pour `create_hypertable`.

À consulter :

- [https://docs.timescale.com/use-timescale/latest/hypertables/create/](https://docs.timescale.com/use-timescale/latest/hypertables/create/)
- [https://docs.timescale.com/docs/api/latest/hypertable/create_hypertable](https://docs.timescale.com/docs/api/latest/hypertable/create_hypertable)

### 2.4 Scripts npm pour les migrations

Dans le package telemetry :

- `migration:blank` : création d’une migration vide
- `migration:create` : génération automatique à partir des entités
- `migration:run` : exécution des migrations

Pour une migration “à la main” (avec `create_hypertable`), `migration:blank` est adapté.

---

## Phase 3 – Migration

### 3.1 Ordre des opérations dans `up()`

1. Créer l’extension TimescaleDB si nécessaire :  
    `CREATE EXTENSION IF NOT EXISTS timescaledb CASCADE;`
2. Créer la table `device_history` avec les colonnes définies.
3. Appeler `create_hypertable()` sur cette table avec la colonne de temps et éventuellement `segment by` / `order by`.

### 3.2 Documents utiles

- Création de tables hypertables directement :  
    [https://docs.timescale.com/use-timescale/latest/hypertables/create/](https://docs.timescale.com/use-timescale/latest/hypertables/create/)
- Migration d’une table existante :  
    [https://skills.sh/timescale/pg-aiguide/migrate-postgres-tables-to-hypertables](https://skills.sh/timescale/pg-aiguide/migrate-postgres-tables-to-hypertables)
- Paramètres de `create_hypertable()` (intervalle des chunks, etc.) :  
    [https://docs.timescale.com/docs/api/latest/hypertable/create_hypertable](https://docs.timescale.com/docs/api/latest/hypertable/create_hypertable)

### 3.3 Méthode `down()`

Pour une migration réversible, `down()` doit :

- Supprimer la hypertable (ou la table si elle n’est plus hypertable).
- Éventuellement supprimer l’extension TimescaleDB (en général on la laisse pour les autres hypertables).

---

## Phase 4 – Test

### 4.1 Où tester

- Tests dans :  
    `packages/telemetry/tests/integrations/`
- Docker compose :  
    services `timescaledb_dev` et `timescaledb_test` dans `docker-compose.yml`.

### 4.2 Types de vérifications

1. Migration
    - Exécuter `migration:run` sur la base de test.
    - Vérifier que la table existe et qu’elle est bien une hypertable :  
        `SELECT * FROM timescaledb_information.hypertables;`
2. Insertions
    - Insérer quelques lignes (`time`, `deviceId`, `key`, `value`, `raw`).
    - Vérifier que les données sont visibles et que les chunks sont créés.
3. Requêtes
    - Filtrer par `deviceId` et `time` pour valider la performance et le `segment by`.

### 4.3 Documentation TimescaleDB pour les policies (plus tard)

Pour SC-US-PH2-NEW-02 (policies, agrégation) :

- Continuous aggregates :  
    [https://docs.timescale.com/use-timescale/latest/continuous-aggregates/](https://docs.timescale.com/use-timescale/latest/continuous-aggregates/)
- Policies de rétention (avec continuous aggregates) :  
    [https://docs.timescale.com/use-timescale/latest/data-retention/data-retention-with-continuous-aggregates/](https://docs.timescale.com/use-timescale/latest/data-retention/data-retention-with-continuous-aggregates/)
- Refresh policies :  
    [https://docs.timescale.com/docs/use-timescale/latest/continuous-aggregates/refresh-policies](https://docs.timescale.com/docs/use-timescale/latest/continuous-aggregates/refresh-policies)