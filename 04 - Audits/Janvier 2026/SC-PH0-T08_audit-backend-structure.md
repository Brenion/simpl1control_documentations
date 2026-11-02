# Audit de la Structure du Code Backend

**Projet** : simpl1Control
**Tâche** : SC-PH0-T08 — Analyser la structure du code backend existant
**User Story** : SC-US-PH0-01 — Auditer la structure du code backend pour identifier les dettes techniques
**Date** : 09/02/2026
**Version auditée** : 1.1.2

---

## Stack technique identifié

| Composant | Technologie | Version |
|-----------|------------|---------|
| Runtime | Node.js + TypeScript (ESM) | TS 5.7.3 |
| Framework HTTP | Fastify | 5.2.1 |
| ORM | TypeORM | 0.3.20 |
| Base de données | PostgreSQL | via `pg` 8.13.1 |
| Validation | Zod | 3.24.4 |
| Authentification | JWT (`jsonwebtoken` + `@fastify/jwt`) | 9.0.2 / 9.0.3 |
| Hashing | Argon2 | 0.43.0 |
| Logging | Pino | 9.6.0 |
| Tests | Vitest | 3.0.5 |
| Build | Rollup | 4.50.0 |
| Protocoles IoT | MQTT, Modbus, WebSocket | mqtt 5.10.3 |
| Déploiement | Docker (node:20-alpine) | Multi-stage |
| Package manager | pnpm | — |

---

## 1. Architecture & Organisation

### 1.1 Arborescence du code source

```
src/
├── server.ts              # Point d'entrée principal
├── app.ts                 # Factory Fastify (non utilisé en production)
├── register.ts            # Enregistrement centralisé des routes (God File)
├── data-source.ts         # Configuration TypeORM
├── cron-setup.ts          # Configuration des tâches planifiées
├── seed.ts                # Script de seeding
├── features/              # Modules métier
│   ├── auth.ts            # Fichier auth mort (doublon)
│   ├── base.entity.ts     # Entité de base
│   ├── access-log/        # Logs d'accès
│   ├── auth/              # Authentification (login, refresh, reset, profile)
│   ├── badges/            # Gestion des badges NFC
│   ├── charts/            # Pages & cartes de visualisation + WebSocket
│   ├── data-histories/    # Historique des données capteurs
│   ├── devices/           # Gestion des appareils IoT
│   └── users/             # Gestion des utilisateurs
├── services/              # Services transversaux
├── plugins/               # Plugins Fastify
├── realtime/              # Hub temps réel (WebSocket PubSub)
├── publishers/            # Publication MQTT
├── cronjobs/              # Tâches cron
├── database/              # Factories & seeders
├── migrations/            # Migrations TypeORM
├── enums/                 # Enums partagés
├── types/                 # Types globaux
└── utils/                 # Utilitaires (crypto, logger, API response)
```

### 1.2 Pattern architectural

Le backend suit un pattern **Layered/Modular** avec un découpage par feature : chaque feature contient ses propres entités, repositories, schémas et routes. Les services transversaux (email, MQTT, hash, cron) sont centralisés dans `src/services/`.

**Points positifs :**
- Découpage par domaine métier dans `features/`
- Utilisation du Repository Pattern pour l'accès aux données
- Utilisation de Zod pour la validation des entrées
- Logging structuré avec Pino
- Pattern PubSub propre dans `realtime/realtime-hub.ts`
- Utilisation du type `Result` (true-myth) dans AuthService pour un error handling explicite

### 1.3 Problèmes architecturaux identifiés

| #   | Problème                                                                                                                                                                                                                                    | Sévérité (1-5) | Fichier(s) concerné(s)                        |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------: | --------------------------------------------- |
| A1  | **God File `register.ts`** — Fichier centralisé de 64 lignes qui instancie manuellement tous les services et enregistre toutes les routes. Toute nouvelle feature nécessite de modifier ce fichier. Viole le principe Open/Closed.          |       4        | `src/register.ts`                             |
| A2  | **Pas d'injection de dépendances** — Les services sont instanciés manuellement avec `new` dans les routes et dans `register.ts`. Pas de DI container (Awilix, TypeDI). `AuthService` est instancié deux fois avec des arguments différents. |       4        | `src/register.ts`, routes                     |
| A3  | **Fichier `auth.ts` mort** — `src/features/auth.ts` contient un endpoint `/login` factice qui retourne toujours `"Authentification réussie"`. Le vrai login est dans `src/features/auth/login/login.route.ts`. Code mort et confus.         |       3        | `src/features/auth.ts`                        |
| A4  | **Logique métier dans les routes** — Les routes contiennent de la logique métier (ex: souscription MQTT dans `addDevice.route.ts`) au lieu de la déléguer au service.                                                                       |       4        | `src/features/devices/add/addDevice.route.ts` |
| A5  | **Effet de bord dans le repository** — `UserRepositoryImpl.create()` envoie un email (via `authService.forgotPassword()`) après la création d'un utilisateur. Le repository ne devrait gérer que la persistance.                            |       4        | `src/features/users/user.repository.ts`       |
| A6  | **Singleton statique pour MQTT** — `MqttService` utilise un pattern ServiceLocator via `static getInstance()` qui cache les dépendances et complique les tests.                                                                             |       3        | `src/services/mqtt.service.ts`                |
| A7  | **Pas de validation centralisée des variables d'environnement** — Les `process.env.*` sont accédés directement dans une vingtaine de fichiers sans validation au démarrage.                                                                 |       3        | Multiples fichiers                            |
| A8  | **Patterns d'enregistrement de routes incohérents** — Certaines routes exposent `.routeDefinition`, d'autres `.getAll`, `.createUser`, `.getAllPages`, etc. Aucune interface commune.                                                       |       3        | `src/register.ts`                             |

---

## 2. Qualité du code

### 2.1 Conventions de nommage

| Aspect | Constat | Sévérité (1-5) |
|--------|---------|:-:|
| **Fichiers** | Incohérent : `addUser.route.ts` (camelCase) vs `ActiveDevice.route.ts` (PascalCase) vs `getAllAccessLogs.route.ts` | 3 |
| **Classes** | Incohérent : `UserRepositoryImpl` vs `BadgesRepository` vs `DeviceRepositoryImpl` | 2 |
| **Entités** | Incohérent : singulier `BadgeEntity`, `UserEntity` vs pluriel `PagesEntity`, `CardsEntity` | 2 |
| **Méthodes de routes** | Incohérent : `createUser` vs `addBadge` vs `getAll` vs `routeDefinition` | 3 |
| **Réponses API** | Incohérent : `{success, message}` vs `{status, message}` vs `{success, message, code}` | 4 |

### 2.2 Duplication de code

| # | Problème | Sévérité (1-5) | Détail |
|---|----------|:-:|---|
| Q1 | **Try-catch boilerplate** — Pattern identique de try-catch avec `reply.status().send()` répété dans les 22+ fichiers de routes. Aucun middleware centralisé. | 4 | ~30 instances |
| Q2 | **Schémas Zod dupliqués** — Définitions de schémas répétées dans les routes au lieu d'être centralisées par feature. | 3 | Routes multiples |

### 2.3 Type Safety

| # | Problème | Sévérité (1-5) | Détail |
|---|----------|:-:|---|
| Q3 | **Usage de `any`** — 30+ instances de `any` dans le code, notamment dans les handlers WebSocket (`charts.ws.ts`), le plugin WebSocket, le service de badges, et le key-mapper. | 4 | Fichiers WebSocket et MQTT |
| Q4 | **Double type assertion** — `req.body as {...} as DeviceEntity` dans `addDevice.route.ts`. Le cast double contourne la validation Zod. | 3 | `addDevice.route.ts` |
| Q5 | **Repository par string** — `AppDataSource.getRepository('BadgeEntity')` au lieu de `AppDataSource.getRepository(BadgeEntity)`. Fragile et sans type safety. | 3 | `badges.repository.ts` |
| Q6 | **`strictPropertyInitialization: false`** dans tsconfig — Permet les propriétés de classe non initialisées, masque des bugs potentiels. | 2 | `tsconfig.json` |

### 2.4 Gestion des erreurs

| # | Problème | Sévérité (1-5) | Détail |
|---|----------|:-:|---|
| Q7 | **Pas de classes d'erreurs personnalisées** — Seules les erreurs auth (`ExpiredTokenError`, `InvalidUserNameOrPasswordError`) ont des classes dédiées. Le reste utilise des `Error` génériques. | 3 | Global |
| Q8 | **Perte de contexte d'erreur** — `Promise.reject(new Error(\`${error}\`))` dans les repositories perd le stack trace et le type de l'erreur originale. | 4 | `user.repository.ts` |
| Q9 | **Échecs silencieux** — `forgotPassword()` retourne silencieusement si l'email n'existe pas. `monitorControllerPresence` résout toujours, même en erreur. | 3 | `auth.service.ts`, `monitorControllerPresence.ts` |
| Q10 | **Format de réponse d'erreur incohérent** — Le error handler global utilise `{success, message, code}`, mais les routes utilisent `{status, message}` ou `{success, message}`. | 4 | Global |

### 2.5 Divers

| # | Problème | Sévérité (1-5) | Détail |
|---|----------|:-:|---|
| Q11 | **`console.log` au lieu du logger** | 2 | `test-job.ts`, `deriveBadgeKey.utils.ts` |
| Q12 | **Logs commentés** — Plusieurs `logger.info()` commentés dans `mqtt.service.ts` | 2 | `mqtt.service.ts` |
| Q13 | **Valeurs magiques** — Timeouts hardcodés (`5000`), expiration 12h, topics MQTT hardcodés dans la logique métier | 3 | `badge.service.ts`, `auth.service.ts` |
| Q14 | **Interface contrat cassé** — `EmailService.sendMail()` ignore le champ `options.to` et utilise toujours `process.env.EMAIL_TO` | 4 | `email.service.ts` |

---

## 3. Sécurité

### 3.1 Problèmes critiques

| # | Problème | Sévérité (1-5) | Détail |
|---|----------|:-:|---|
| S1 | **Secrets hardcodés et commités dans Git** — Les fichiers `.env` (development.env, test.env, et production/production.env) contiennent des secrets en clair (JWT secrets, mots de passe DB, clés de chiffrement, mot de passe SMTP Mailgun) et sont commités dans le repo. Le JWT_SECRET de production est `supersecretkey` — identique au développement. | 5 | `development.env`, `production.env`, `db.env` |
| S2 | **JWT secrets faibles et identiques dev/prod** — `JWT_SECRET=supersecretkey` et `JWT_REFRESH_SECRET=supersecretrefreshkey` sont des mots du dictionnaire, utilisés en production comme en développement. | 5 | `.env` files, `login.route.ts` |
| S3 | **Cookie `secure: false` en production** — Le cookie d'accès JWT est configuré avec `secure: false`, ce qui le rend vulnérable aux attaques MITM. Le `sameSite` est `lax` au lieu de `strict`. | 4 | `login.route.ts` (lignes 93-98) |
| S4 | **Cookie secret par défaut** — `process.env.COOKIE_SECRET \|\| 'a-secret-string'` dans `server.ts`. Si la variable n'est pas définie, un secret statique est utilisé. | 4 | `server.ts` (ligne 48) |
| S5 | **`synchronize: true` sur TypeORM** — En production, cette option peut provoquer une perte de données en synchronisant automatiquement le schéma. | 5 | `data-source.ts` (ligne 24) |

### 3.2 Problèmes majeurs

| # | Problème | Sévérité (1-5) | Détail |
|---|----------|:-:|---|
| S6 | **Token de reset non hashé** — Le token de réinitialisation de mot de passe est stocké en clair dans la base de données avec une fenêtre d'expiration de 12 heures (trop longue, recommandé 1-2h). | 4 | `auth.service.ts` |
| S7 | **Pas de rate limiting** — Aucune protection contre le brute force sur `/login`, `/forgot-password`, `/refresh-token`. Le package `@fastify/rate-limit` n'est pas installé. | 4 | Global |
| S8 | **Pas de protection CSRF** — Aucun mécanisme de protection CSRF malgré l'utilisation de cookies pour l'authentification. | 4 | Global |
| S9 | **Routes non protégées** — La majorité des routes ne vérifient pas le JWT/session. Seul `/profile` utilise `fastifyPassport.authenticate()`. Pas de RBAC malgré l'existence d'un enum Role. | 4 | Routes multiples |
| S10 | **`rejectUnauthorized: false` sur MQTT TLS** — La validation des certificats TLS est désactivée pour les connexions MQTT, rendant vulnérable aux attaques MITM. | 4 | `mqtt.service.ts` (ligne 90) |
| S11 | **Méthode `hashPassword()` jamais appelée** — `UserEntity.hashPassword()` est définie mais aucun décorateur `@BeforeInsert()` ou `@BeforeUpdate()` ne la déclenche. Le hashing est fait manuellement dans le repository, mais l'incohérence est risquée. | 4 | `user.entity.ts` (lignes 39-43) |
| S12 | **Health check inexistant** — Le Dockerfile configure un health check sur `/health`, mais cette route n'existe pas dans l'application. | 3 | `Dockerfile` (ligne 25) |

### 3.3 Problèmes modérés

| # | Problème | Sévérité (1-5) | Détail |
|---|----------|:-:|---|
| S13 | **Messages d'erreur exposés au client** — Le error handler global envoie `error.message` complet au client, ce qui peut révéler des détails internes (schéma DB, requêtes SQL). | 3 | `error-handler.ts` |
| S14 | **Validation d'entrée insuffisante** — Les schémas Zod des utilisateurs n'ont pas de contraintes de longueur sur les champs texte (`username`, `firstname`, `lastname`). Pas de validation de format pour le username. | 3 | `user.schema.ts` |
| S15 | **Credentials DB dans le code source** — Fallback `"postgres://root:test123@localhost:5432/..."` hardcodé dans `server.ts`. | 3 | `server.ts` (ligne 81) |
| S16 | **Helmet sans configuration custom** — `@fastify/helmet` est enregistré mais sans configuration CSP, HSTS, etc. | 2 | `register.ts` |

---

## 4. Dépendances

### 4.1 Dépendances inutilisées

| Package | Version | Statut | Action recommandée |
|---------|---------|--------|-------------------|
| `@prisma/client` | ^6.3.1 | **Non utilisé** — Aucune référence dans le code source. TypeORM est l'ORM exclusif. | Supprimer |
| `@fastify/passport` | ^3.0.2 | **Usage minimal** — Utilisé uniquement pour `/profile`. JWT est géré manuellement via `jsonwebtoken`. | Évaluer la suppression ou l'adoption complète |

### 4.2 Doublons fonctionnels

| Packages | Problème | Action recommandée |
|----------|----------|-------------------|
| `@fastify/jwt` + `jsonwebtoken` | Les deux sont installés mais seul `jsonwebtoken` est utilisé directement. `@fastify/jwt` est chargé mais ses fonctionnalités ne sont pas exploitées. | Choisir l'un ou l'autre. Recommandé : utiliser `@fastify/jwt` qui s'intègre nativement avec Fastify. |
| `@fastify/postgres` + TypeORM (`pg`) | Les deux accèdent à PostgreSQL. `@fastify/postgres` est enregistré dans `server.ts` mais les repositories utilisent TypeORM directement. | Évaluer si `@fastify/postgres` est nécessaire ou si TypeORM suffit seul. |

### 4.3 Dépendances mal classées

Les packages suivants sont en `dependencies` mais devraient être en `devDependencies` car ils ne sont utilisés qu'en développement/seeding :

| Package | Version | Raison |
|---------|---------|--------|
| `@faker-js/faker` | ^9.4.0 | Utilisé uniquement dans les factories/seeders |
| `@jorgebodega/typeorm-factory` | ^2.1.0 | Factories de test/seeding |
| `@jorgebodega/typeorm-seeding` | ^7.1.0 | Seeding de la base de données |
| `mailgen` | ^2.0.29 | Templates d'emails (à vérifier si utilisé en production) |

### 4.4 Packages potentiellement obsolètes

| Package | Installé | Dernière version | Écart |
|---------|----------|-------------------|-------|
| `fastify` | 5.2.1 | 5.7.4+ | Patches de sécurité manquants |
| `typeorm` | 0.3.20 | 0.3.28+ | Corrections de bugs |
| `zod` | 3.24.4 | 4.x | Version majeure de retard |
| `dotenv` | 16.4.7 | 17.x | Version majeure de retard |
| `mqtt` | 5.10.3 | 5.15+ | Mises à jour de sécurité |
| `pino` | 9.6.0 | 10.x | Version majeure de retard |

---

## 5. Tests

### 5.1 Couverture par fonctionnalité

| Feature | Fichiers de test | Couverture | Verdict |
|---------|:---:|---|---|
| Authentification (login, refresh, reset, forgot, profile) | 0 | Aucun test sur 6 routes | **CRITIQUE** |
| WebSocket (charts.ws.ts) | 0 | Aucun test, validation JWT + origin critique | **CRITIQUE** |
| MQTT Service | 0 | 199 lignes sans test unitaire | **CRITIQUE** |
| Badges — routes | 1 | Test d'intégration partiel (add uniquement) | Partiel |
| Badges — utilitaires crypto | 2 | Bonne couverture des cas limites | Correct |
| Devices — activation | 1 | Test basé sur mocks, flux réel non testé | Faible |
| Users — CRUD | 3 | Routes testées, repositories non testés | Partiel |
| Crypto utils | 1 | Encrypt/decrypt couverts | Correct |
| handleReaderAccess | 1 | 609 lignes, test le plus complet du projet | Bon |
| Email Service | 0 | Aucun test | Manquant |
| Cron Jobs | 0 | Aucun test | Manquant |
| Data Histories | 0 | Aucun test | Manquant |
| Charts/Pages | 0 | Aucun test | Manquant |
| Access Logs | 0 | Aucun test | Manquant |

**Couverture estimée : ~25-30%**

### 5.2 Problèmes de qualité des tests

| # | Problème | Sévérité (1-5) |
|---|----------|:-:|
| T1 | **Organisation incohérente** — Tests répartis entre `/tests/` (3 fichiers) et inline dans `src/` (8 fichiers). Pas de convention claire. | 3 |
| T2 | **Tests d'intégration non isolés** — `badge.add.route.test.ts` utilise `AppDataSource.initialize()` réel au lieu d'une base de test isolée. | 3 |
| T3 | **Snapshot testing faible** — Les tests d'intégration (`devices-integration.test.ts`, `user-integrations.test.ts`) utilisent `toMatchSnapshot()` sans assertions spécifiques sur les valeurs. | 3 |
| T4 | **`as any` dans les tests** — Multiple casts `as any` dans les mocks, contournant le type safety. | 2 |
| T5 | **Script test = serveur dev** — `"test": "NODE_ENV=test tsx watch src/server.ts"` lance le serveur au lieu d'exécuter les tests. Pas de script `vitest` dans package.json. | 4 |

---

## 6. Configuration & DevOps

### 6.1 TypeScript

| Point | Statut | Note |
|-------|--------|------|
| `strict: true` | ✅ | Bonne pratique |
| `experimentalDecorators` | ✅ | Nécessaire pour TypeORM |
| `skipLibCheck: true` | ⚠️ | Peut masquer des erreurs de types dans les dépendances |
| `strictPropertyInitialization: false` | ⚠️ | Permet des propriétés non initialisées |

### 6.2 ESLint

| Point | Statut | Note |
|-------|--------|------|
| Base config | ✅ | `eslint:recommended` + `@typescript-eslint` |
| Import ordering | ✅ | Alphabétique, appliqué |
| Unused imports | ✅ | Plugin configuré |
| `no-unused-vars` | ⚠️ | En `warn` au lieu de `error` |
| Règles manquantes | ❌ | Pas de `no-any`, `complexity`, `max-lines`, `no-console` |

### 6.3 Docker

| Point | Statut | Note |
|-------|--------|------|
| Multi-stage build | ✅ | Build + production séparés |
| Base image alpine | ✅ | Image légère |
| Health check | ❌ | Route `/health` inexistante dans l'application |
| Docker Compose dev | ⚠️ | Credentials hardcodés dans le fichier |

### 6.4 CI/CD

| Point | Statut |
|-------|--------|
| Pipeline CI/CD | ❌ Non détecté — Aucun fichier `.github/workflows/`, `.gitlab-ci.yml`, `Jenkinsfile` ou équivalent trouvé |

---

## 7. Couche données (TypeORM)

| # | Problème | Sévérité (1-5) | Détail |
|---|----------|:-:|---|
| D1 | **`synchronize: true` en production** — Risque de perte de données. Doit être `false` avec migrations. | 5 | `data-source.ts` |
| D2 | **Pas de gestion de transactions** — Les opérations multi-étapes (création user + envoi email) ne sont pas encapsulées dans des transactions. | 4 | `user.repository.ts` |
| D3 | **Relations incohérentes** — Certaines FK sont des colonnes string simples (`badges.userId`) au lieu de relations TypeORM `@ManyToOne`. D'autres utilisent des string references (`"DataHistoryEntity"`) au lieu de classes. | 3 | Entités multiples |
| D4 | **Pas de soft delete** — Les suppressions sont définitives (pas de `@DeleteDateColumn`). | 2 | Toutes les entités |
| D5 | **Une seule migration** — Une seule migration pour tout le schéma (`1747259713078-migration.ts`). Pas d'historique granulaire. | 2 | `src/migrations/` |

---

## 8. Synthèse des dettes techniques

### Par sévérité

| Sévérité | Nombre | Exemples clés |
|:---:|:---:|---|
| **5 — Critique** | 3 | Secrets commités en prod, `synchronize: true`, JWT secrets faibles |
| **4 — Majeur** | 16 | God File, pas de DI, logique métier dans routes, pas de rate limiting, pas de CSRF, routes non protégées, cookie insécure, format d'erreur incohérent |
| **3 — Modéré** | 14 | Nommage incohérent, fichier mort, singleton MQTT, validation insuffisante, tests non isolés |
| **2 — Mineur** | 6 | console.log, logs commentés, entités pluriel/singulier, `skipLibCheck`, no soft delete |

### Par catégorie

| Catégorie | Sévérité max | Nombre de points |
|-----------|:---:|:---:|
| Sécurité | 5 | 16 |
| Architecture | 4 | 8 |
| Qualité du code | 4 | 14 |
| Tests | 4 | 5 |
| Dépendances | 3 | 6+ |
| Configuration | 3 | 5 |
| Couche données | 5 | 5 |

---

## 9. Recommandations prioritaires

### Priorité IMMÉDIATE (Sévérité 5)

1. **Retirer tous les fichiers `.env` du dépôt Git** et les ajouter au `.gitignore`. Fournir un `.env.example` avec des valeurs factices. Rotation immédiate de tous les secrets.
2. **Générer des JWT secrets forts** (128+ caractères aléatoires) distincts par environnement.
3. **Désactiver `synchronize: true`** dans TypeORM et utiliser exclusivement les migrations.

### Priorité HAUTE (Sévérité 4)

4. **Implémenter un rate limiting** (`@fastify/rate-limit`) sur les endpoints critiques.
5. **Protéger toutes les routes** avec un guard JWT + RBAC.
6. **Refactorer `register.ts`** — Adopter un auto-loading par feature ou un DI container (Awilix).
7. **Extraire la logique métier des routes** vers des services dédiés.
8. **Standardiser le format de réponse d'erreur** via un middleware centralisé.
9. **Corriger la configuration des cookies** (`secure: true` en production, `sameSite: strict`).
10. **Ajouter une protection CSRF**.
11. **Supprimer le fichier `src/features/auth.ts`** (code mort).

### Priorité MOYENNE (Sévérité 3)

12. **Supprimer `@prisma/client`** et déplacer les dépendances de seeding vers `devDependencies`.
13. **Choisir entre `@fastify/jwt` et `jsonwebtoken`** (éliminer le doublon).
14. **Centraliser la validation des variables d'environnement** avec Zod au démarrage de l'application.
15. **Standardiser les conventions de nommage** (fichiers, classes, méthodes).
16. **Ajouter des tests pour les chemins critiques** (auth, WebSocket, MQTT).
17. **Mettre en place un pipeline CI/CD**.
18. **Mettre à jour les dépendances** (Fastify, TypeORM, Zod, etc.).

### Priorité BASSE (Sévérité 2)

19. Remplacer `console.log` par le logger Pino.
20. Supprimer les logs commentés.
21. Activer `strictPropertyInitialization` dans tsconfig.
22. Ajouter des règles ESLint pour `no-any`, `complexity`, `no-console`.

---

*Rapport généré dans le cadre de l'audit SC-PH0-T08. Aucune modification n'a été apportée au code source.*
