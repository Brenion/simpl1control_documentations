# Audit des Conventions de Nommage — Backend

**Projet** : simpl1Control
**Tâche** : SC-PH0-T03 — Vérifier les conventions de nommage
**User Story** : SC-US-PH0-01 — Auditer la structure du code backend pour identifier les dettes techniques
**Date** : 09/02/2026
**Version auditée** : 1.1.2
**Scoring** : 1 (mineur) → 5 (critique)

---

## Synthèse

Le backend **ne dispose pas de conventions de nommage documentées**. Aucun fichier de guidelines (CONTRIBUTING.md, coding-standards, ADR) n'a été trouvé dans le projet. Les conventions appliquées sont implicites et **incohérentes dans 7 des 9 catégories** analysées.

| Catégorie | Verdict | Sévérité |
|-----------|---------|:---:|
| Fichiers & dossiers | Incohérent (kebab-case / camelCase / PascalCase mélangés) | 4 |
| Classes | Partiellement cohérent (suffixes incohérents) | 3 |
| Méthodes exposées par les routes | Incohérent (pas d'interface commune) | 4 |
| Variables & propriétés | Correct (camelCase cohérent) | 1 |
| Endpoints API REST | Incohérent (verbes dans les URL, conventions REST violées) | 4 |
| Tables & colonnes (BDD) | Incohérent (camelCase/snake_case mélangés, singulier/pluriel) | 3 |
| Types & interfaces | Incohérent (suffixes variés, pas de convention d'export) | 3 |
| Enums | Correct | 1 |
| Schémas Zod | Partiellement cohérent | 2 |

---

## 1. Fichiers & Dossiers

### 1.1 Constat

Trois conventions différentes coexistent pour les fichiers de routes :

| Convention | Exemples | Comptage |
|------------|----------|:---:|
| **camelCase** | `addUser.route.ts`, `getAllDevices.route.ts`, `getOnePage.route.ts`, `deleteUser.route.ts` | ~15 |
| **kebab-case** | `forgot-password.route.ts`, `refresh-token.route.ts`, `reset-password.route.ts` | 5 |
| **PascalCase** | `ActiveDevice.route.ts` | 1 |

Les fichiers d'entités, repositories, services et schémas sont plus cohérents (kebab-case + suffixe), mais on note des anomalies dans les noms de base :

| Fichier | Problème |
|---------|----------|
| `badges.entity.ts` | Pluriel (`badges`) alors que toutes les autres entités sont au singulier : `user.entity.ts`, `device.entity.ts`, `access-log.entity.ts` |
| `cards.entity.ts` | Pluriel (`cards`) au lieu de `card.entity.ts` |
| `page.entity.ts` | Singulier — correct, mais le schéma associé s'appelle `pagesSchema` (pluriel) |
| `badges.add.route.ts` | Convention `resource.action.route.ts` alors que les autres utilisent `actionResource.route.ts` (`addUser.route.ts`, `addDevice.route.ts`) |
| `badges.delete.route.ts` | Idem — `badges.delete.route.ts` vs `deleteUser.route.ts` |
| `badges.update.route.ts` | Idem — `badges.update.route.ts` vs `updateUser.route.ts` |
| `validateBadge.ts` | Pas de suffixe (ni `.service.ts`, ni `.utils.ts`) |
| `handleReaderAccess.ts` | Dans `services/` mais sans suffixe `.service.ts` |
| `monitorControllerPresence.ts` | Dans `publishers/` mais sans suffixe `.publisher.ts` |

### 1.2 Dossiers

Les dossiers de routes par action n'ont pas de convention unique :

| Feature | Dossiers d'action | Convention |
|---------|-------------------|------------|
| Users | `add/`, `delete/`, `getAll/`, `getOne/`, `update/` | camelCase partiel |
| Devices | `active/`, `add/`, `getAll/`, `getOne/`, `update/` | camelCase partiel |
| Badges | `add-route/`, `delete-route/`, `update-route/` | kebab-case + suffixe `-route` |
| Charts | `getAllPages/`, `getOnePage/`, `ws/` | camelCase |
| Auth | `forgot-password/`, `login/`, `profile/`, `refresh/`, `reset-password/` | kebab-case |
| Access-log | `get-all-route/` | kebab-case + suffixe `-route` |

**Sévérité : 4/5** — L'incohérence rend difficile de deviner le chemin d'un fichier sans explorer l'arborescence.

### 1.3 Convention recommandée

Adopter **kebab-case** partout pour les fichiers et dossiers, conformément à la convention Node.js/TypeScript dominante. Format : `<nom-ressource>.<type>.ts` (ex: `add-user.route.ts`, `badge.entity.ts`).

---

## 2. Classes

### 2.1 Entités

| Classe | Fichier | Singulier/Pluriel |
|--------|---------|:-:|
| `UserEntity` | `user.entity.ts` | ✅ Singulier |
| `DeviceEntity` | `device.entity.ts` | ✅ Singulier |
| `BadgeEntity` | `badges.entity.ts` | ✅ Singulier (mais fichier pluriel) |
| `AccessLogEntity` | `access-log.entity.ts` | ✅ Singulier |
| `DataHistoryEntity` | `data-history.entity.ts` | ✅ Singulier |
| `PagesEntity` | `page.entity.ts` | ❌ **Pluriel** (fichier singulier) |
| `CardsEntity` | `cards.entity.ts` | ❌ **Pluriel** (fichier aussi pluriel) |
| `BaseEntity` | `base.entity.ts` | ✅ Singulier |

`PagesEntity` et `CardsEntity` sont au pluriel alors que les 5 autres entités sont au singulier.

**Sévérité : 3/5**

### 2.2 Repositories

| Classe | Fichier | Suffixe |
|--------|---------|---------|
| `UserRepositoryImpl` | `user.repository.ts` | `RepositoryImpl` |
| `DeviceRepositoryImpl` | `device.repository.ts` | `RepositoryImpl` |
| `PageRepositoryImpl` | `page.repository.ts` | `RepositoryImpl` |
| `AccessLogsRepositoryImpl` | `access-logs.repository.ts` | `RepositoryImpl` (+ nom pluriel) |
| `BadgesRepository` | `badges.repository.ts` | `Repository` (sans `Impl`) |
| `DataHistoriesRepository` | `data-histories.repository.ts` | `Repository` (sans `Impl`) |

Deux conventions coexistent : `*RepositoryImpl` (4 classes) et `*Repository` (2 classes). De plus, certains noms sont au pluriel (`BadgesRepository`, `AccessLogsRepositoryImpl`, `DataHistoriesRepository`) tandis que d'autres sont au singulier (`UserRepositoryImpl`, `DeviceRepositoryImpl`).

Aucune interface `*Repository` n'est définie pour justifier le suffixe `Impl`. Le pattern Impl sans interface n'a pas de sens.

**Sévérité : 3/5**

### 2.3 Services

| Classe | Fichier |
|--------|---------|
| `AuthService` | `auth.service.ts` |
| `HashService` | `hash.service.ts` |
| `EmailService` | `email.service.ts` |
| `MqttService` | `mqtt.service.ts` |
| `CronService` | `cron.service.ts` |
| `BadgesService` | `badge.service.ts` |

`BadgesService` est au pluriel tandis que les 5 autres sont au singulier. De plus, le fichier s'appelle `badge.service.ts` (singulier) mais la classe `BadgesService` (pluriel).

**Sévérité : 2/5**

### 2.4 Routes

Les classes de routes sont toutes en PascalCase avec suffixe `Route` — c'est cohérent. Cependant, **la méthode exposée pour l'enregistrement** varie :

| Classe | Méthode exposée | Registré via |
|--------|----------------|-------------|
| `LoginRoute` | `routeDefinition` | `server.register(loginRoute.routeDefinition)` |
| `ProfileRoute` | `routeDefinition` | `server.register(profileRoute.routeDefinition)` |
| `ForgotPasswordRoute` | `routeDefinition` | `server.register(forgotPasswordRoute.routeDefinition)` |
| `RefreshTokenRoute` | `routeDefinition` | `server.register(refreshTokenRoute.routeDefinition)` |
| `ResetPasswordRoute` | `routeDefinition` | `server.register(resetPasswordRoute.routeDefinition)` |
| `UsersListRoute` | `getAll` | `server.register(usersListRoute.getAll)` |
| `GetUserRoute` | `getUser` | `server.register(getUserRoute.getUser)` |
| `AddUserRoute` | `createUser` | `server.register(addUserRoute.createUser)` |
| `UpdateUserRoute` | `updateUser` | `server.register(updateUserRoute.updateUser)` |
| `DeleteUserRoute` | `deleteUser` | `server.register(deleteUserRoute.deleteUser)` |
| `GetAllDevicesRoute` | `getAll` | `server.register(getDevicesRoute.getAll)` |
| `GetDeviceRoute` | `getDevice` | `server.register(getDeviceRoute.getDevice)` |
| `AddDeviceRoute` | `createDevice` | `server.register(addDeviceRoute.createDevice)` |
| `UpdateDeviceRoute` | `updateDevice` | `server.register(updateDeviceRoute.updateDevice)` |
| `ActiveDeviceRoute` | `activeDevice` | `server.register(activeDeviceRoute.activeDevice)` |
| `AddBadgeRoute` | `addBadge` | `server.register(addBadgeRoute.addBadge)` |
| `UpdateBadgeRoute` | `updateBadge` | `server.register(updateBadgeRoute.updateBadge)` |
| `DeleteBadgeRoute` | `deleteBadge` | `server.register(deleteBadgeRoute.deleteBadge)` |
| `GetAllPagesRoute` | `getAllPages` | `server.register(getAllPagesRoutes.getAllPages)` |
| `GetPageRoute` | `getPage` | `server.register(getOnePage.getPage)` |
| `AccessLogsListRoute` | `getAll` | `server.register(accessLogsListRoute.getAll)` |
| `DataHistoriesRoutes` | *(export default)* | `server.register(DataHistoriesRoutes, ...)` |

**Problèmes :**
- Les routes auth utilisent `routeDefinition` (5 routes), les autres utilisent le nom de l'action (`getAll`, `createUser`, etc.)
- Incohérence verbe : `createUser` vs `addBadge` vs `createDevice` (create vs add)
- `DataHistoriesRoutes` utilise un export default au lieu d'une classe — pattern totalement différent
- Pas d'interface commune (ex: `IRoute { register(): void }`)

**Sévérité : 4/5** — L'absence d'interface commune rend impossible l'auto-registration.

---

## 3. Méthodes des Repositories

| Opération | UserRepositoryImpl | DeviceRepositoryImpl | BadgesRepository |
|-----------|-------------------|---------------------|-----------------|
| Créer | `create()` | `create()` | `createBadge()` |
| Mettre à jour | `update()` | `update()` | `updateBadge()` |
| Supprimer | `delete()` | — | `deleteBadge()` |
| Trouver par ID | `findById()` | `findById()` | `findByCardId()` |
| Lister | `findAndCount()` | `findAndCount()` | — |

`BadgesRepository` préfixe toutes ses méthodes avec le nom de la ressource (`createBadge`, `updateBadge`, `deleteBadge`) alors que les autres repositories utilisent des noms génériques (`create`, `update`, `delete`). Ce préfixe est redondant puisque le contexte est déjà donné par le nom de la classe.

**Sévérité : 2/5**

---

## 4. Endpoints API REST

### 4.1 Inventaire des routes

| Méthode | Endpoint | Problème |
|---------|----------|----------|
| `POST` | `/users/add` | ❌ Verbe `add` dans l'URL — REST utilise `POST /users` |
| `GET` | `/users` | ✅ Correct |
| `GET` | `/users/:id` | ✅ Correct |
| `PATCH` | `/users/update` | ❌ Verbe `update` dans l'URL — REST utilise `PATCH /users/:id` |
| `DELETE` | `/users/:id` | ✅ Correct |
| `POST` | `/devices/add` | ❌ Verbe `add` |
| `GET` | `/devices` | ✅ Correct |
| `GET` | `/devices/:id` | ✅ Correct |
| `PATCH` | `/devices/update` | ❌ Verbe `update` |
| `PATCH` | `/devices/active/:id` | ❌ Verbe `active` — devrait être `PATCH /devices/:id/active` ou `PATCH /devices/:id` |
| `POST` | `/badges/add` | ❌ Verbe `add` |
| `PATCH` | `/badges/update` | ❌ Verbe `update` |
| `DELETE` | `/badges/:id` | ✅ Correct |
| `POST` | `/login` | ⚠️ Acceptable (pas de ressource) mais `POST /auth/login` serait plus structuré |
| `POST` | `/forgot-password` | ⚠️ Idem — `POST /auth/forgot-password` |
| `POST` | `/reset-password` | ⚠️ Idem |
| `GET` | `/profile` | ⚠️ Idem — `GET /auth/profile` |
| `POST` | `/refresh-token` | ⚠️ Idem — `POST /auth/refresh` |
| `GET` | `/pages` | ✅ Correct |
| `GET` | `/pages/:id` | ✅ Correct |
| `GET` | `/data-histories` | ✅ Correct |
| `POST` | `/data-histories` | ✅ Correct |
| `GET` | `/access-logs` | ✅ Correct |

### 4.2 Problèmes identifiés

**Verbes dans les URL (violation REST)** — 6 endpoints utilisent des verbes (`/add`, `/update`, `/active`) alors que la méthode HTTP devrait suffire. C'est un anti-pattern REST bien connu.

**Routes auth non groupées** — Les 5 routes d'authentification sont enregistrées à la racine (`/login`, `/forgot-password`, etc.) au lieu d'être sous un préfixe `/auth/`.

**Incohérence du placement de l'ID** :
- `DELETE /users/:id` → l'ID est dans l'URL ✅
- `PATCH /users/update` → l'ID est dans le body ❌ (pas RESTful)
- `PATCH /devices/active/:id` → l'ID est dans l'URL mais avec un verbe ❌

**Sévérité : 4/5** — L'API ne respecte pas les conventions REST standard, ce qui complique la documentation et l'intégration frontend.

### 4.3 Convention recommandée

| Opération | Convention REST |
|-----------|---------------|
| Créer | `POST /resources` |
| Lister | `GET /resources` |
| Lire | `GET /resources/:id` |
| Modifier | `PATCH /resources/:id` |
| Supprimer | `DELETE /resources/:id` |
| Action custom | `POST /resources/:id/action` |

---

## 5. Base de données (Tables & Colonnes)

### 5.1 Noms de tables

| Décorateur `@Entity()` | Convention |
|------------------------|------------|
| `@Entity("users")` | ✅ snake_case pluriel |
| `@Entity("devices")` | ✅ snake_case pluriel |
| `@Entity({ name: 'badges' })` | ✅ snake_case pluriel |
| `@Entity({ name: 'access_logs' })` | ✅ snake_case pluriel |
| `@Entity('data_histories')` | ✅ snake_case pluriel |
| `@Entity({ name: 'pages' })` | ✅ snake_case pluriel |
| `@Entity({ name: 'cards' })` | ✅ snake_case pluriel |

Les tables sont toutes en snake_case pluriel — c'est cohérent. Cependant, la **syntaxe du décorateur** varie : parfois `@Entity("name")`, parfois `@Entity({ name: 'name' })`.

### 5.2 Noms de colonnes

Les colonnes TypeORM sont en **camelCase dans le code** (`createdAt`, `deviceId`, `isOnline`, `resetTokenExpiry`, `deniedAccessFlag`). Par défaut, TypeORM mappe le camelCase en camelCase côté PostgreSQL (sans conversion snake_case), ce qui donne des colonnes comme `"createdAt"`, `"isOnline"`, `"deviceId"` en base de données.

C'est un choix potentiellement problématique :
- PostgreSQL est case-insensitive par défaut, mais les identifiants en camelCase sont automatiquement mis entre guillemets par TypeORM
- Toute requête SQL manuelle devra utiliser `"createdAt"` au lieu de `created_at`
- La convention PostgreSQL standard est snake_case

**Sévérité : 3/5** — Choix fonctionnel mais non standard pour PostgreSQL.

### 5.3 Relations

| Relation | Syntaxe |
|----------|---------|
| `@OneToMany("DataHistoryEntity", ...)` | String reference (fragile) |
| `@ManyToOne("DeviceEntity", ...)` | String reference (fragile) |

Les relations utilisent des string references (`"DataHistoryEntity"`) au lieu de classes directes. C'est valide en TypeORM (pour éviter les dépendances circulaires) mais fragile car un renommage de classe ne sera pas détecté par le compilateur.

**Sévérité : 2/5**

---

## 6. Types, Interfaces & Schémas Zod

### 6.1 Types

| Nom | Fichier | Suffixe |
|-----|---------|---------|
| `UserType` | `user.type.ts` | `Type` |
| `AccessLogType` | `access-logs.schema.ts` | `Type` (défini dans le fichier schema) |
| `BadgeUpdate` | `badges.repository.ts` | Pas de suffixe (défini dans le repository) |
| `Pagination` | `user.repository.ts`, `device.repository.ts` | Pas de suffixe (dupliqué dans 2 fichiers) |
| `LoginBody` | `login.route.ts` | `Body` |
| `ForgotPasswordBody` | `forgot-password.route.ts` | `Body` |
| `EmailOptions` | `email.service.ts` | `Options` |
| `MqttOptions` | `mqtt.service.ts` | `Options` |

**Problèmes :**
- Le type `Pagination` est défini de manière identique dans `user.repository.ts` et `device.repository.ts` — duplication
- `BadgeUpdate` est défini dans le repository au lieu d'un fichier dédié
- `AccessLogType` est défini dans le fichier `.schema.ts` au lieu d'un `.type.ts`
- Les suffixes varient : `Type`, `Body`, `Options`, ou rien
- Aucune interface explicite n'existe malgré l'utilisation implicite du pattern (ex: `AuthServiceInterface` est un type, pas une interface TypeScript)

### 6.2 Interfaces

| Nom | Fichier | Déclaration |
|-----|---------|-------------|
| `AuthServiceInterface` | `auth.service.ts` | Déclaré comme `type` (pas `interface`) |
| `EmailServiceInterface` | `email.service.ts` | Déclaré comme `type` (pas `interface`) |

Les "interfaces" du projet sont en réalité des types aliasés. La convention `*Interface` comme suffixe pour un `type` est trompeuse.

**Sévérité : 3/5**

### 6.3 Schémas Zod

| Nom | Fichier |
|-----|---------|
| `userSchema` | `user.schema.ts` |
| `userAndBadgesSchema` | `user.schema.ts` |
| `deviceSchema` | `device.schema.ts` |
| `accessLogSchema` | `access-logs.schema.ts` |
| `pagesSchema` | `page.schema.ts` |
| `onePageSchema` | `page.schema.ts` |

**Problèmes :**
- `pagesSchema` est au pluriel, les autres sont au singulier
- `onePageSchema` utilise un préfixe `one` (peu courant)
- Les schémas ne sont pas systématiquement utilisés dans les routes (certaines routes font `req.body as {...}` au lieu de valider avec le schéma)

**Sévérité : 2/5**

---

## 7. Enums

| Enum | Fichier | Clés | Valeurs | Export |
|------|---------|------|---------|--------|
| `Role` | `roles-enum.ts` | `ADMIN`, `EMPLOYEE`, `SECURITY` | `"admin"`, `"employee"`, `"security"` | `export default` |
| `DeviceStatus` | `device-status.ts` | `ACTIVE`, `INACTIVE`, `BUSY` | `"active"`, `"inactive"`, `"busy"` | `export default` |

Les deux enums suivent la même convention (UPPER_CASE keys, lowercase values, export default). C'est cohérent.

Un point à noter : le fichier s'appelle `roles-enum.ts` au lieu de `role.enum.ts` (pour suivre le pattern `<nom>.<type>.ts` utilisé ailleurs).

**Sévérité : 1/5**

---

## 8. Variables & Propriétés

Les variables suivent globalement le **camelCase** de manière cohérente : `firstName`, `lastName`, `accessToken`, `resetToken`, `isOnline`, `deviceId`, etc.

Les constantes utilisent **UPPER_CASE** : `OPEN` (WebSocket state), les clés d'environnement (`JWT_SECRET`, `DATABASE_URL`).

Un seul point notable : le champ `mail` au lieu de `email` dans `UserEntity`. C'est un choix sémantique (probablement francophone) mais `email` est le terme standard en anglais dans les API.

**Sévérité : 1/5**

---

## 9. Format des réponses API

Bien que ce ne soit pas strictement du "nommage", les clés des réponses JSON sont incohérentes :

| Source | Format de réponse |
|--------|------------------|
| `api-response.ts` (successResponse) | `{ success: true, message, data }` |
| `api-response.ts` (errorResponse) | `{ success: false, message, code }` |
| `error-handler.ts` | `{ success: false, message, code }` |
| `login.route.ts` | `{ status: "success", message, accessToken }` |
| `badges.repository.ts` | `{ status: 200, success: true, message }` |
| Routes users | `{ success, message, data }` ou `{ status, message }` |

Trois formats de réponse différents coexistent. La fonction utilitaire `successResponse()` / `errorResponse()` existe mais n'est pas systématiquement utilisée.

**Sévérité : 4/5** — Le frontend doit gérer plusieurs formats de réponse.

---

## 10. Tableau récapitulatif des incohérences

| # | Catégorie | Incohérence | Exemples | Sévérité |
|---|-----------|-------------|----------|:---:|
| N01 | Fichiers routes | 3 conventions mélangées | `addUser.route.ts` vs `forgot-password.route.ts` vs `ActiveDevice.route.ts` | 4 |
| N02 | Fichiers entités | Singulier/pluriel mélangé | `user.entity.ts` vs `badges.entity.ts` vs `cards.entity.ts` | 3 |
| N03 | Fichiers badges | Convention inversée | `badges.add.route.ts` vs `addUser.route.ts` | 3 |
| N04 | Dossiers d'action | 3 patterns | `add/` vs `add-route/` vs `forgot-password/` | 3 |
| N05 | Fichiers sans suffixe | Pas de type dans le nom | `validateBadge.ts`, `handleReaderAccess.ts`, `monitorControllerPresence.ts` | 2 |
| N06 | Classes entités | Pluriel sur 2/7 | `PagesEntity`, `CardsEntity` vs `UserEntity`, `DeviceEntity` | 3 |
| N07 | Classes repositories | Suffixe `Impl` inconstant | `UserRepositoryImpl` vs `BadgesRepository` | 3 |
| N08 | Classes repositories | Singulier/pluriel | `UserRepositoryImpl` vs `BadgesRepository`, `DataHistoriesRepository` | 2 |
| N09 | Classes services | Pluriel isolé | `BadgesService` vs `AuthService`, `HashService`, etc. | 2 |
| N10 | Méthodes routes | 2 conventions | `routeDefinition` (auth) vs nom d'action (`getAll`, `createUser`) | 4 |
| N11 | Méthodes routes | Verbes incohérents | `createUser` vs `addBadge` vs `createDevice` | 3 |
| N12 | Méthodes repositories | Préfixe redondant | `createBadge()` vs `create()` | 2 |
| N13 | Endpoints REST | Verbes dans les URL | `POST /users/add`, `PATCH /users/update` | 4 |
| N14 | Endpoints REST | ID placement | `DELETE /users/:id` vs `PATCH /users/update` (body) | 4 |
| N15 | Endpoints auth | Pas de préfixe groupe | `/login` au lieu de `/auth/login` | 3 |
| N16 | Colonnes BDD | camelCase au lieu de snake_case | `createdAt`, `deviceId`, `isOnline` en PostgreSQL | 3 |
| N17 | Décorateur Entity | Syntaxe variable | `@Entity("users")` vs `@Entity({ name: 'badges' })` | 1 |
| N18 | Types | Suffixes incohérents | `UserType`, `LoginBody`, `BadgeUpdate`, `Pagination` | 3 |
| N19 | Types | Emplacement incohérent | `.type.ts`, `.schema.ts`, `.repository.ts` (duplication `Pagination`) | 3 |
| N20 | Interfaces | Fausses interfaces | `AuthServiceInterface` est un `type`, pas une `interface` | 2 |
| N21 | Schémas Zod | Singulier/pluriel | `userSchema` vs `pagesSchema` | 2 |
| N22 | Enums fichiers | Pas de suffixe `.enum.ts` | `roles-enum.ts` au lieu de `role.enum.ts` | 1 |
| N23 | Champ email | Terme francophone | `mail` au lieu de `email` (standard API) | 2 |
| N24 | Format réponse API | 3 formats coexistent | `{success, message, data}` vs `{status, message}` vs `{status: 200, success, message}` | 4 |
| N25 | Export enums | `export default` | Les enums utilisent `export default` alors que tout le reste utilise des exports nommés | 2 |

---

## 11. Recommandations

### Priorité HAUTE (Sévérité 4)

1. **Standardiser les noms de fichiers** — Adopter **kebab-case** partout : `add-user.route.ts`, `forgot-password.route.ts`, `active-device.route.ts`.

2. **Définir une interface commune pour les routes** — Créer une interface `IRoute` avec une méthode `register()` unique, éliminant la dualité `routeDefinition` / nom d'action.

3. **Corriger les endpoints REST** — Supprimer les verbes des URL (`/users/add` → `POST /users`), placer systématiquement l'ID dans l'URL pour les opérations unitaires, et grouper les routes auth sous `/auth/`.

4. **Standardiser le format de réponse API** — Utiliser systématiquement `successResponse()` / `errorResponse()` de `api-response.ts` dans toutes les routes et repositories.

### Priorité MOYENNE (Sévérité 2-3)

5. **Harmoniser singulier/pluriel** — Entités et services au singulier (`BadgeEntity`, `BadgeService`), repositories au singulier (`BadgeRepository`), tables en pluriel (`badges`).

6. **Supprimer le suffixe `Impl`** — Soit définir des interfaces `UserRepository` avec implémentations `UserRepositoryImpl`, soit supprimer le suffixe `Impl` partout.

7. **Centraliser les types partagés** — Extraire `Pagination` dans un fichier `src/types/pagination.type.ts` partagé. Déplacer `BadgeUpdate` dans un fichier dédié.

8. **Convertir les colonnes en snake_case** — Configurer TypeORM avec un `namingStrategy` (ex: `SnakeNamingStrategy`) pour mapper automatiquement camelCase → snake_case.

9. **Renommer `mail` en `email`** — Standard international pour les API.

10. **Utiliser `interface` au lieu de `type` pour les contrats** — Renommer `AuthServiceInterface` et utiliser le mot-clé `interface`.

### Priorité BASSE (Sévérité 1)

11. **Harmoniser la syntaxe des décorateurs `@Entity()`** — Choisir `@Entity({ name: 'table' })` ou `@Entity('table')`.

12. **Renommer les fichiers d'enums** — `role.enum.ts` au lieu de `roles-enum.ts`.

13. **Convertir les `export default` en exports nommés** pour les enums.

---

## 12. Proposition de convention standard

Pour référence future, voici la convention recommandée :

| Élément | Convention | Exemple |
|---------|-----------|---------|
| Fichiers | kebab-case + suffixe type | `add-user.route.ts` |
| Dossiers | kebab-case | `forgot-password/` |
| Classes | PascalCase + suffixe | `UserEntity`, `UserRepository`, `UserService` |
| Méthodes | camelCase | `findById()`, `create()` |
| Variables | camelCase | `accessToken`, `isOnline` |
| Constantes | UPPER_SNAKE_CASE | `MAX_RETRIES`, `DEFAULT_PORT` |
| Endpoints | kebab-case, pluriel, sans verbe | `GET /users/:id`, `POST /auth/login` |
| Tables BDD | snake_case pluriel | `users`, `access_logs` |
| Colonnes BDD | snake_case | `created_at`, `device_id` |
| Enums | PascalCase (nom), UPPER_CASE (clés) | `Role.ADMIN` |
| Types/Interfaces | PascalCase | `UserType`, `AuthService` (interface) |
| Schémas Zod | camelCase + Schema | `userSchema`, `deviceSchema` |

---

*Rapport généré dans le cadre de l'audit SC-PH0-T03. Aucune modification n'a été apportée au code source.*
