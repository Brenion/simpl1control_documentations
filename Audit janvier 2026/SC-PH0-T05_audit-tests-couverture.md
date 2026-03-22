# SC-PH0-T05 — Lister les tests existants et analyser la couverture

> **User Story** : SC-US-PH0-04 — En tant que développeur, je veux lister les tests existants et identifier les manques
> **Tâche** : Lister tests existants et analyser couverture
> **Projet** : simpl1Control v1.1.2
> **Date** : 10 février 2026
> **Méthode** : Analyse statique du code source (read-only, aucune modification)

---

## Table des matières

1. [Vue d'ensemble](#1-vue-densemble)
2. [Outillage et configuration de test](#2-outillage-et-configuration-de-test)
3. [Inventaire des tests backend](#3-inventaire-des-tests-backend)
4. [Inventaire des tests frontend](#4-inventaire-des-tests-frontend)
5. [Cartographie de couverture par module](#5-cartographie-de-couverture-par-module)
6. [Analyse qualitative des tests existants](#6-analyse-qualitative-des-tests-existants)
7. [Manques critiques identifiés](#7-manques-critiques-identifiés)
8. [Plan complet des tests à écrire](#8-plan-complet-des-tests-à-écrire)
9. [Recommandations](#9-recommandations)
10. [Synthèse](#10-synthèse)

---

## 1. Vue d'ensemble

### 1.1 Chiffres clés

| Métrique | Backend | Frontend | Total |
|---|---|---|---|
| Fichiers source (hors tests) | 75 | 89 | 164 |
| Fichiers de test | 11 | 6 | 17 |
| Blocs `describe` | 11 | 5 + 1 dummy | 17 |
| Cas de test (`it`/`test`) | ~50 | ~17 | **~67** |
| Ratio fichiers testés / fichiers source | 10/75 (13%) | 5/89 (6%) | 15/164 (**9%**) |

### 1.2 Répartition par type de test

| Type | Backend | Frontend | Total |
|---|---|---|---|
| Tests unitaires | 8 fichiers (~38 tests) | 0 | 8 |
| Tests d'intégration | 3 fichiers (~5 tests) | 0 | 3 |
| Tests de composants | 0 | 5 fichiers (~16 tests) | 5 |
| Tests placeholder/dummy | 1 (user.test.ts) | 1 (dummy.test.tsx) | 2 |
| Tests E2E (Playwright) | 0 | 0 (config présente, 0 test) | **0** |

---

## 2. Outillage et configuration de test

### 2.1 Backend

| Élément | Détail |
|---|---|
| Framework | Vitest 3.0.5 |
| UI | @vitest/ui 3.0.5 |
| Couverture | c8 10.1.3 (non configurée dans vitest.config) |
| Configuration | `vitest.config.ts` — globals: true, alias '@' → './src' |
| Mocking | vi.fn(), vi.spyOn(), vi.mock() (natif Vitest) |

**Constat — Script `test` incorrect (Sévérité 4)** : Le script `"test"` du `package.json` backend est défini comme `"NODE_ENV=test tsx watch src/server.ts"`, ce qui **lance le serveur** au lieu d'exécuter les tests. Il n'existe aucun script pour lancer `vitest` directement. Pour exécuter les tests, il faut appeler `npx vitest` manuellement.

### 2.2 Frontend

| Élément | Détail |
|---|---|
| Framework unit/component | Vitest 3.0.7 |
| Mode navigateur | @vitest/browser (Playwright/Chromium, viewport 800x600) |
| Environment | happy-dom 17.4.3 |
| Couverture | @vitest/coverage-v8 (présent, non configuré dans vitest.config) |
| Mocking HTTP | MSW (Mock Service Worker) 2.7.3 |
| Testing Library | @testing-library/jest-dom 6.6.3 |
| E2E | Playwright 1.50.1 (configuré mais **0 test E2E écrit**) |
| Setup | `vitest.setup.ts` — cleanup automatique après chaque test |

**Scripts NPM** :

| Script | Commande | Statut |
|---|---|---|
| `test` | `vitest` | ✅ Correct |
| `test:debug` | `vitest --inspect-brk --no-file-parallelism` | ✅ Correct |
| `test:e2e` | `playwright test` | ⚠️ 0 tests E2E |
| `test:e2e:ui` | `playwright test --ui` | ⚠️ 0 tests E2E |

**Fichiers de support test** :

| Fichier | Rôle |
|---|---|
| `tests/vitest.setup.ts` | Cleanup DOM après chaque test |
| `tests/test-extend.ts` | Fixture Vitest ajoutant un `worker` MSW aux tests |
| `tests/test-worker.ts` | Initialisation `setupWorker()` MSW (vide) |
| `tests/sleep.ts` | Utilitaire sleep (défaut 180000ms — **3 minutes**, semble excessif) |

---

## 3. Inventaire des tests backend

### 3.1 Tableau détaillé

| # | Fichier | Type | Tests | Assertions | Mocks | Ce qui est testé |
|---|---|---|---|---|---|---|
| B1 | `src/utils/crypto.test.ts` | Unit | 4 | toBe, toBeGreaterThan, toThrow | Aucun | Chiffrement/déchiffrement AES, validation clés |
| B2 | `src/features/badges/deriveBadgeKey.utils.test.ts` | Unit | 4 | toBe, toThrow | Aucun | Dérivation clé AES128 depuis UID + MASTER_KEY |
| B3 | `src/features/badges/deriveKeysAB.utils.test.ts` | Unit | 4 | toHaveLength, toEqual, toThrowError | Aucun | Génération KeyA/KeyB (6 octets) |
| B4 | `src/services/handleReaderAccess.test.ts` | Unit | 23 | toHaveBeenCalled(With), not.toHaveBeenCalled | vi.mock (6 modules) | Accès NFC, machine à états, validation badges |
| B5 | `src/features/badges/add-route/badge.add.route.test.ts` | Integ | 2 | toBe, toHaveBeenCalledTimes | vi.mock, vi.spyOn | Route POST /badges/add, publication MQTT |
| B6 | `src/features/devices/active/ActiveDevice.route.test.ts` | Unit | 2 | toHaveBeenCalledWith | vi.fn() | Activation device |
| B7 | `src/features/users/add/addUser.route.test.ts` | Unit | 3 | toHaveBeenCalledWith | vi.fn() | Création utilisateur |
| B8 | `src/features/users/update/updateUser.route.test.ts` | Unit | 3 | toHaveBeenCalledWith, toMatchObject | vi.fn() | Mise à jour utilisateur |
| B9 | `tests/units/user.test.ts` | Unit | 1 | toBeGreaterThan | Aucun | Placeholder (non fonctionnel) |
| B10 | `tests/integrations/user-integrations.test.ts` | Integ | 2 | toMatchSnapshot | Aucun | Routes GET /users via app.inject |
| B11 | `tests/integrations/devices-integration.test.ts` | Integ | 1 | toMatchSnapshot | vi.clearAllMocks | Route GET /devices via app.inject |

**Total backend** : 11 fichiers, ~49 tests

### 3.2 Problèmes identifiés dans les tests backend

| # | Problème | Sévérité | Fichier |
|---|---|---|---|
| P1 | **Test placeholder non fonctionnel** : `user.test.ts` contient un seul test minimal qui ne teste rien de réel | 3 | B9 |
| P2 | **Test d'erreur incomplet** : `ActiveDevice.route.test.ts` ligne 48 — le corps du test d'erreur est vide, le handler n'est jamais appelé | 4 | B6 |
| P3 | **Message incohérent** : `updateUser.route.test.ts` ligne 74 — le code vérifie "created" au lieu de "updated" | 3 | B8 |
| P4 | **Tests snapshot fragiles** : les tests d'intégration utilisent `toMatchSnapshot()` sans assertions explicites sur les données | 3 | B10, B11 |
| P5 | **handleReaderAccess.test.ts trop long** : 610 lignes, 3 describe imbriqués, setup répétitif dans beforeEach | 2 | B4 |
| P6 | **Script test incorrect** : `"test": "NODE_ENV=test tsx watch src/server.ts"` lance le serveur, pas vitest | 4 | package.json |

---

## 4. Inventaire des tests frontend

### 4.1 Tableau détaillé

| # | Fichier | Type | Tests | Assertions | Mocks | Ce qui est testé |
|---|---|---|---|---|---|---|
| F1 | `features/auth/components/LoginForm.test.tsx` | Component | 3 | toBeInTheDocument, toHaveBeenCalled | useNavigate, useAuth, useTranslation | Rendu form login, soumission, erreur |
| F2 | `features/auth/components/ForgotPasswordForm.test.tsx` | Component | 4 | toBeInTheDocument, toHaveBeenCalledWith | useNavigate, useAuth, useTranslation | Rendu form, envoi email, erreur, cancel |
| F3 | `features/auth/components/ResetPasswordForm.test.tsx` | Component | 4 | toBeInTheDocument, toHaveBeenCalledWith | useNavigate, useAuth, useTranslation | Rendu form, token manquant, reset, erreur |
| F4 | `features/users/components/UserForm.test.tsx` | Component | 2 | toBeNull | QueryClient, Router | Create/update user happy flow |
| F5 | `features/users/components/CreateUser.test.tsx` | Component | 3 | toBeDefined | MSW handlers, QueryClient | Création succès, validation, erreur |
| F6 | `tests/dummy.test.tsx` | Dummy | 1 | aucune | Aucun | Placeholder — render `<div>Hello</div>` + console.log |

**Total frontend** : 6 fichiers, ~17 tests (dont 1 dummy)

### 4.2 Problèmes identifiés dans les tests frontend

| # | Problème | Sévérité | Fichier |
|---|---|---|---|
| P7 | **Timers manuels anti-pattern** : tous les tests auth utilisent `new Promise(res => setTimeout(res, 500))` au lieu de `waitFor` ou `findBy` | 4 | F1, F2, F3 |
| P8 | **querySelector brut** : `UserForm.test.tsx` utilise `container.querySelector('[name="username"]')` au lieu des queries Testing Library (`getByLabelText`, `getByRole`) | 3 | F4 |
| P9 | **Test cassé** : `CreateUser.test.tsx` — le test 3 (erreur création) utilise le même setup que le test 1 (succès) sans mocker d'erreur API côté MSW, il s'attend à "failed to create user" mais le handler retourne un succès | 4 | F5 |
| P10 | **Assertions trop vagues** : `toBeDefined()` ne vérifie pas le contenu, `toBeNull()` ne vérifie que l'absence d'erreur visible | 3 | F4, F5 |
| P11 | **Login non vérifié** : `LoginForm.test.tsx` utilise `toHaveBeenCalled()` sans vérifier les arguments passés au mock `login()` | 3 | F1 |
| P12 | **Pas de tests de validation** : aucun test frontend ne vérifie les erreurs de validation (email invalide, mot de passe faible, champs requis manquants) | 3 | F1-F5 |
| P13 | **Textes i18n hardcodés** : les assertions utilisent des regex en anglais (`/username is required/i`) au lieu des clés de traduction | 2 | F5 |
| P14 | **MSW handlers non resettés** : les handlers MSW ne sont pas réinitialisés entre les tests, risque d'interférences | 3 | F5 |
| P15 | **0 tests E2E** : Playwright est configuré avec scripts, mais aucun fichier de test E2E n'existe | 3 | — |
| P16 | **Test dummy placeholder** : `tests/dummy.test.tsx` rend un `<div>Hello</div>` sans aucune assertion, seulement un `console.log` | 2 | F6 |

---

## 5. Cartographie de couverture par module

### 5.1 Backend — Couverture par feature

| Module / Feature | Fichiers source | Fichiers testés | Couverture | Criticité |
|---|---|---|---|---|
| **Auth** (login, refresh, forgot/reset password, profile) | 6 | **0** | **0%** | 🔴 Critique |
| **MQTT** (service, reload, ping-pong) | 4 | **0** | **0%** | 🔴 Critique |
| **Badges** (entity, repo, service, routes, validation, crypto) | 8 | 4 (utils + add route) | 50% | 🟡 Partiel |
| **Users** (entity, repo, routes CRUD) | 7 | 4 (add, update, integ) | 57% | 🟡 Partiel |
| **Devices** (entity, repo, routes CRUD) | 7 | 2 (active, integ) | 29% | 🟠 Insuffisant |
| **Access Logs** (entity, repo, route) | 3 | **0** | **0%** | 🟠 Insuffisant |
| **Charts** (pages, cards, WS) | 5 | **0** | **0%** | 🟠 Insuffisant |
| **Data Histories** (entity, repo, route) | 3 | **0** | **0%** | 🟠 Insuffisant |
| **Realtime** (key-mapper, realtime-hub) | 2 | **0** | **0%** | 🟡 Important |
| **Services** (handleReaderAccess, logo-publisher, email, hash, cron) | 6 | 1 (handleReaderAccess) | 17% | 🟠 Insuffisant |
| **Plugins** (error-handler, websocket) | 2 | **0** | **0%** | 🟡 Important |
| **Utils** (crypto, logger, api-response) | 3 | 1 (crypto) | 33% | 🟡 Partiel |
| **Infrastructure** (server, register, data-source, cron-setup) | 4 | **0** | **0%** | 🟡 Important |

### 5.2 Frontend — Couverture par feature

| Module / Feature | Fichiers source | Fichiers testés | Couverture | Criticité |
|---|---|---|---|---|
| **Auth** (composants, hooks, types) | 12 | 3 (LoginForm, ForgotPassword, ResetPassword) | 25% | 🟡 Partiel |
| **Users** (composants, hooks, types) | 11 | 2 (UserForm, CreateUser) | 18% | 🟠 Insuffisant |
| **Devices** (composants, hooks, types) | 9 | **0** | **0%** | 🔴 Critique |
| **Charts** (composants, hooks) | 5 | **0** | **0%** | 🟠 Insuffisant |
| **Access Logs** (composants, hooks) | 3 | **0** | **0%** | 🟡 Important |
| **Components** (SideBar, Layout, etc.) | 6 | **0** | **0%** | 🟡 Important |
| **Routes** (TanStack Router) | 18 | **0** | **0%** | 🟡 Important |
| **Utils** (fetchUtil, ws, debounce, etc.) | 5 | **0** | **0%** | 🟡 Important |
| **Hooks auth** (useAuth, useAuthStore, useAuthRefresh, useWs) | 5 | **0** | **0%** | 🔴 Critique |

### 5.3 Diagramme de couverture

```
BACKEND (75 fichiers source, 11 fichiers test)
══════════════════════════════════════════════════

Auth            [░░░░░░░░░░░░░░░░░░░░] 0%    ← 🔴 CRITIQUE
MQTT            [░░░░░░░░░░░░░░░░░░░░] 0%    ← 🔴 CRITIQUE
Badges          [██████████░░░░░░░░░░] 50%
Users           [███████████░░░░░░░░░] 57%
Devices         [██████░░░░░░░░░░░░░░] 29%
Access Logs     [░░░░░░░░░░░░░░░░░░░░] 0%
Charts/Pages    [░░░░░░░░░░░░░░░░░░░░] 0%
Data Histories  [░░░░░░░░░░░░░░░░░░░░] 0%
Realtime        [░░░░░░░░░░░░░░░░░░░░] 0%
Services        [███░░░░░░░░░░░░░░░░░] 17%
Plugins         [░░░░░░░░░░░░░░░░░░░░] 0%
Utils           [███████░░░░░░░░░░░░░] 33%

FRONTEND (89 fichiers source, 5 fichiers test)
══════════════════════════════════════════════════

Auth compos.    [█████░░░░░░░░░░░░░░░] 25%
Auth hooks      [░░░░░░░░░░░░░░░░░░░░] 0%    ← 🔴 CRITIQUE
Users           [████░░░░░░░░░░░░░░░░] 18%
Devices         [░░░░░░░░░░░░░░░░░░░░] 0%    ← 🔴 CRITIQUE
Charts          [░░░░░░░░░░░░░░░░░░░░] 0%
Access Logs     [░░░░░░░░░░░░░░░░░░░░] 0%
Components      [░░░░░░░░░░░░░░░░░░░░] 0%
Routes          [░░░░░░░░░░░░░░░░░░░░] 0%
Utils           [░░░░░░░░░░░░░░░░░░░░] 0%
E2E             [░░░░░░░░░░░░░░░░░░░░] 0%    ← 🔴 CRITIQUE
```

---

## 6. Analyse qualitative des tests existants

### 6.1 Points forts

**Backend** :
- Les tests crypto/badges sont bien écrits : cas nominaux + cas d'erreur + validation de stabilité (même entrée = même sortie)
- `handleReaderAccess.test.ts` est le test le plus complet du projet : 23 cas couvrant la machine à états NFC (step 0 et step 1), les erreurs module absent, les refus d'accès, et le logging
- Le test d'intégration `badge.add.route.test.ts` utilise une vraie base de données et valide le flux MQTT complet (presence → prepare → register)

**Frontend** :
- L'infrastructure MSW est correctement mise en place (Mock Service Worker pour les appels API)
- Le setup Vitest browser mode (Playwright/Chromium) est une bonne approche pour les tests de composants
- La fixture `test-extend.ts` qui injecte automatiquement le worker MSW est un bon pattern

### 6.2 Points faibles

**Backend** :
- Le script `"test"` ne lance pas vitest — il est impossible d'exécuter les tests avec `npm test` ou `pnpm test`
- Un test est un placeholder vide (`user.test.ts`), un autre a un cas d'erreur non implémenté (`ActiveDevice.route.test.ts`)
- Les tests d'intégration (user, devices) utilisent des snapshots sans assertions explicites — fragile et peu informatif
- Aucune configuration de seuil de couverture (c8 est installé mais non configuré)
- Aucune intégration CI/CD détectée pour l'exécution automatique des tests

**Frontend** :
- L'anti-pattern `setTimeout(500)` dans tous les tests auth crée des tests flaky et lents
- Les assertions sont souvent trop permissives (`toBeDefined`, `toBeNull` sans vérifier le contenu réel)
- Un test dupliqué/cassé dans `CreateUser.test.tsx` (test 3 = copie du test 1)
- Aucun test ne vérifie les règles de validation des formulaires
- Playwright est installé et configuré mais **0 test E2E n'a été écrit**

### 6.3 Maturité par pyramide de test

```
                    ┌─────────────┐
                    │    E2E      │  0 tests
                    │  (0%)      │  Playwright configuré, aucun test
                    ├─────────────┤
                    │ Integration │  5 tests (3 backend + 0 frontend)
                    │   (~8%)     │  Snapshots fragiles
                    ├─────────────┤
                    │  Component  │  16 tests (0 backend + 5 frontend)
                    │  (~24%)     │  Timers manuels, assertions vagues
                    ├─────────────┤
                    │   Unitaire  │  ~45 tests (8 fichiers backend)
                    │  (~68%)     │  Bonne qualité crypto/badges
                    └─────────────┘
```

La pyramide est **inversée par rapport aux bonnes pratiques** : la base unitaire est insuffisante (seulement 13% de fichiers couverts), la couche intégration est quasi-absente, et la couche E2E est inexistante.

---

## 7. Manques critiques identifiés

### 7.1 Modules backend sans aucun test

| # | Module | Fichiers | Risque | Priorité |
|---|---|---|---|---|
| M1 | **Authentification** (login, refresh token, forgot/reset password, profile) | 6 fichiers | Faille de sécurité non détectée, régression JWT | 🔴 P1 |
| M2 | **Service MQTT** (connexion, subscription, publication, routage) | 4 fichiers | Perte de messages, fuites mémoire (ping-pong), déconnexion silencieuse | 🔴 P1 |
| M3 | **Logo Publisher** (extraction température, cache, publication LOGO) | 1 fichier | Température incorrecte envoyée au PLC, chauffage défaillant | 🔴 P1 |
| M4 | **Realtime** (key-mapper, realtime-hub) | 2 fichiers | Données temps réel incorrectes, fuites mémoire WebSocket | 🟠 P2 |
| M5 | **Charts WebSocket** (route WS, auth, subscribe/unsubscribe) | 1 fichier | Connexion WS non authentifiée, fuite de données | 🟠 P2 |
| M6 | **Access Logs** (entity, repository, route GET) | 3 fichiers | Logs incorrects, pagination cassée | 🟡 P3 |
| M7 | **Data Histories** (repository avec pipeline KeyMapper → RealtimeHub) | 3 fichiers | Perte de données historiques, bridge MQTT→WS cassé | 🟠 P2 |
| M8 | **Devices routes** (add, update, getAll, getOne) — partiellement testé | 4 non testés | CRUD devices défaillant, subscription MQTT non créée | 🟡 P3 |
| M9 | **Email service** (envoi, templates) | 1 fichier | Emails de reset password non envoyés | 🟡 P3 |
| M10 | **Plugins** (error-handler, websocket) | 2 fichiers | Erreurs non gérées, WebSocket origin bypass | 🟡 P3 |

### 7.2 Modules frontend sans aucun test

| # | Module | Fichiers | Risque | Priorité |
|---|---|---|---|---|
| M11 | **Hooks auth** (useAuth, useAuthStore, useAuthRefresh, useAuthToken, useWs) | 5 fichiers | Tokens non rafraîchis, déconnexion non gérée | 🔴 P1 |
| M12 | **Devices** (composants CRUD, hooks, formulaire) | 9 fichiers | Création/édition device cassée | 🟠 P2 |
| M13 | **Charts** (composants, hooks live, WebSocket) | 5 fichiers | Graphiques temps réel cassés | 🟠 P2 |
| M14 | **SideBar** (412 lignes, composant monolithique) | 1 fichier | Navigation cassée | 🟡 P3 |
| M15 | **Utils** (fetchUtil, ws, debounce) | 5 fichiers | Requêtes API défaillantes | 🟡 P3 |
| M16 | **Routing** (guards auth, redirections) | 18 fichiers | Accès non autorisé à des pages protégées | 🟠 P2 |

### 7.3 Types de tests manquants

| Type de test | État actuel | Impact |
|---|---|---|
| **Tests de sécurité** (JWT, CORS, injection, auth bypass) | **0 tests** | Failles de sécurité non détectées |
| **Tests E2E** (parcours utilisateur complet) | **0 tests** (Playwright configuré mais vide) | Régressions fonctionnelles non détectées |
| **Tests de validation** (Zod schemas, form validation) | **0 tests** | Données invalides acceptées |
| **Tests WebSocket** (connexion, auth, messages) | **0 tests** | Communication temps réel cassée |
| **Tests MQTT** (connexion, subscription, publication) | **0 tests** | IoT non fonctionnel |
| **Tests de performance/charge** | **0 tests** | Dégradation non mesurée |
| **Tests de contrat API** (request/response schemas) | **0 tests** | Breaking changes API non détectés |

---

## 8. Plan complet des tests à écrire

Cette section détaille exhaustivement chaque test à créer pour atteindre une couverture correcte de l'application. Chaque test est spécifié avec son fichier cible, les cas à couvrir et les mocks nécessaires.

### 8.1 Backend — Tests Auth (Priorité 🔴 P1)

#### 8.1.1 `src/features/auth/auth.service.test.ts` — Unit

| # | Cas de test | Description | Mocks |
|---|---|---|---|
| 1 | login — username valide, password valide | Vérifie `Result.ok(user)` retourné, `lastLoginAt` mis à jour | userRepository.findOne, HashService.verify → true |
| 2 | login — login par email (fallback `mail`) | Login avec adresse mail au lieu de username | userRepository.findOne |
| 3 | login — utilisateur non trouvé | Vérifie `Result.err(InvalidUserNameOrPasswordError)` | userRepository.findOne → null |
| 4 | login — mot de passe invalide | Vérifie `Result.err(InvalidUserNameOrPasswordError)` | HashService.verify → false |
| 5 | forgotPassword — utilisateur existe | resetToken (32 bytes hex) généré, expiry +12h, email envoyé | userRepository.findOne/save, emailService.sendMail |
| 6 | forgotPassword — utilisateur inexistant | Retour silencieux, pas d'email envoyé | userRepository.findOne → null |
| 7 | forgotPassword — contenu email | `ForgotPasswordEmail.build()` appelé avec resetToken, firstName, lastName | emailService.sendMail spy |
| 8 | resetPassword — token valide, non expiré | Password hashé, resetToken/Expiry → undefined, `Result.ok()` | userRepository.findOne (MoreThan), HashService.hash |
| 9 | resetPassword — token expiré/invalide | `Result.err(ExpiredTokenError)` | userRepository.findOne → null |

#### 8.1.2 `src/features/auth/login/login.route.test.ts` — Unit

| # | Cas de test | Mocks |
|---|---|---|
| 1 | POST /login — credentials valides → 200 + accessToken + refreshToken + user + cookie set | authService.login → ok |
| 2 | POST /login — credentials invalides → 401 + `{ error }` | authService.login → err |
| 3 | POST /login — JWT contient id, firstname, lastname, mail, role | jwt.sign spy |
| 4 | POST /login — refreshToken sauvé dans user.refreshToken en DB | userRepository.save spy |
| 5 | POST /login — cookie `access_token` avec path="/", httpOnly=true | reply.setCookie spy |

#### 8.1.3 `src/features/auth/refresh/refresh-token.route.test.ts` — Unit

| # | Cas de test | Mocks |
|---|---|---|
| 1 | POST /refresh-token — token valide + user trouvé + tokens matchent → 200 + nouveau accessToken | jwt.verify, userRepository.findOne |
| 2 | POST /refresh-token — token JWT invalide → 401 | jwt.verify → throw |
| 3 | POST /refresh-token — user non trouvé → 401 | userRepository.findOne → null |
| 4 | POST /refresh-token — refreshToken ne matche pas en DB → 401 | userRepository.findOne → user avec token différent |

#### 8.1.4 `src/features/auth/forgot-password/forgot-password.route.test.ts` — Unit

| # | Cas de test | Mocks |
|---|---|---|
| 1 | POST /forgot-password — email valide → 200 + message succès | authService.forgotPassword |
| 2 | POST /forgot-password — email format invalide → erreur validation Zod | — |
| 3 | POST /forgot-password — authService throw → 400 | authService.forgotPassword → throw |

#### 8.1.5 `src/features/auth/reset-password/reset-password.route.test.ts` — Unit

| # | Cas de test | Mocks |
|---|---|---|
| 1 | POST /reset-password — token valide + password ≥8 chars → 200 | authService.resetPassword → ok |
| 2 | POST /reset-password — token expiré → 404 | authService.resetPassword → err |
| 3 | POST /reset-password — password <8 chars → erreur validation Zod | — |

#### 8.1.6 `src/features/auth/profile/profile.route.test.ts` — Unit

| # | Cas de test | Mocks |
|---|---|---|
| 1 | GET /profile — user authentifié → 200 + données user (id, firstname, lastname, mail, role) | request.user = UserEntity |
| 2 | GET /profile — non authentifié (preValidation refuse) → 401 | passport.authenticate reject |

### 8.2 Backend — Tests MQTT (Priorité 🔴 P1)

#### 8.2.1 `src/services/mqtt.service.test.ts` — Unit

| # | Cas de test | Mocks |
|---|---|---|
| 1 | initAsync — crée une instance et se connecte | mqtt.connect |
| 2 | initAsync — singleton : même URL → même instance | mqtt.connect |
| 3 | initAsync — URL TLS → charge certificats (key, cert, ca) | fs.readFile |
| 4 | initAsync — rejectUnauthorized = false pour TLS | — |
| 5 | getInstance — port plain → instance correcte | — |
| 6 | getInstance — port TLS → instance correcte | — |
| 7 | getInstance — port inconnu → throw Error | — |
| 8 | getInstance — non initialisé → throw Error | — |
| 9 | ensureSubscription — nouveau topic → subscribe + callback vide | client.subscribe spy |
| 10 | ensureSubscription — topic existant → idempotent, pas de re-subscribe | client.subscribe spy |
| 11 | registerSubscription — enregistre callback + subscribe au broker | client.subscribe spy |
| 12 | publish — publie payload sur un topic | client.publish spy |
| 13 | unsubscribe — supprime callback + unsubscribe du broker | client.unsubscribe spy |
| 14 | on("message") — topic `/controller/reader-nfc-*` → handleReaderAccess() | handleReaderAccess spy |
| 15 | on("message") — topic `/sensor/*` → processTemperature() | processTemperature spy |
| 16 | on("message") — topic `/API/*` → processTemperature() | processTemperature spy |
| 17 | on("message") — tout message → sauvegardé dans DataHistoriesRepository | saveDataHistory spy |
| 18 | on("message") — callback enregistré appelé si présent | callback spy |
| 19 | on("connect") — réabonnement auto aux topics enregistrés | client.subscribe spy |
| 20 | getPortForSecurity(true) → MQTT_PORT_TLS | — |
| 21 | getPortForSecurity(false) → MQTT_PORT_PLAIN | — |

#### 8.2.2 `src/services/reload-mqtt.service.test.ts` — Unit

| # | Cas de test | Mocks |
|---|---|---|
| 1 | reloadAllSubscribers — charge devices avec subscribe et appelle ensureSubscription | deviceRepository.findSubscribers, mqttService.ensureSubscription |
| 2 | reloadAllSubscribers — aucun subscriber → 0 abonnement | deviceRepository.findSubscribers → [] |

#### 8.2.3 `src/services/ping-pong.service.test.ts` — Unit

| # | Cas de test | Mocks |
|---|---|---|
| 1 | publie "isOnline" sur chaque device avec publish non-null | deviceRepository.findAll, mqttService.publish |
| 2 | device sans publish → ignoré | deviceRepository.findAll |
| 3 | réponse `{ isOnline: "true" }` → device.isOnline = true | registerSubscription callback |
| 4 | timeout (pas de réponse 2s) → device.isOnline = false | setTimeout |
| 5 | réponse `{ isOnline: "1" }` → device.isOnline = true | — |
| 6 | utilise le bon port selon device.isSecure | MqttService.getPortForSecurity spy |

#### 8.2.4 `src/services/logo-publisher.service.test.ts` — Unit

| # | Cas de test | Mocks |
|---|---|---|
| 1 | extractTemperatureFromMessage — `{ payload: { temperature: 22.5 } }` → 2250 | — |
| 2 | extractTemperatureFromMessage — clé `temp` fonctionne | — |
| 3 | extractTemperatureFromMessage — clé `t` fonctionne | — |
| 4 | extractTemperatureFromMessage — payload sans température → null | — |
| 5 | extractTemperatureFromMessage — JSON invalide → null | — |
| 6 | processTemperature — stocke température dans le cache avec timestamp | — |
| 7 | processTemperature — extrait deviceId du dernier segment du topic | — |
| 8 | scheduleLogoPublishingTick — pas de device "logo 8.4" → warn, pas de publish | deviceRepository.findByModel → null |
| 9 | scheduleLogoPublishingTick — aucune temp valide (<5s) → warn, pas de publish | — |
| 10 | scheduleLogoPublishingTick — publie la temp la plus basse vers logo.publish | mqttService.publish spy |
| 11 | scheduleLogoPublishingTick — payload : `{ state: { temperature: { value: [T] }, heatSetpoint: { value: [1850] } } }` | mqttService.publish spy |
| 12 | scheduleLogoPublishingTick — utilise port 1883 | MqttService.getInstance spy |

### 8.3 Backend — Tests Realtime (Priorité 🟠 P2)

#### 8.3.1 `src/realtime/key-mapper.test.ts` — Unit

| # | Cas de test |
|---|---|
| 1 | DSL adapter — metadata avec rules JSONPath → extrait les valeurs correctes |
| 2 | DSL adapter — cast "float", "int", "bool", "string" fonctionnent |
| 3 | DSL adapter — scale multiplie la valeur |
| 4 | DSL adapter — valeur par défaut si path introuvable |
| 5 | DSL adapter — paths (fallback multiples) → prend le premier trouvé |
| 6 | DSL adapter — metadata vide ou sans rules → ne matche pas |
| 7 | Siemens LOGO adapter — brand "siemens" + model "logo 8.4" → extrait state.reported |
| 8 | Siemens LOGO adapter — value est un Array → prend le premier élément |
| 9 | Siemens LOGO adapter — brand différent → ne matche pas |
| 10 | Generic Reported adapter — payload avec state.reported → extrait les clés |
| 11 | Generic Reported adapter — payload sans state.reported → ne matche pas |
| 12 | Priorité : DSL matche en premier si metadata présent |
| 13 | Fallback : aucun adapter ne matche → tableau vide |
| 14 | registerAdapter — ajouté en première position de la chaîne |

#### 8.3.2 `src/realtime/realtime-hub.test.ts` — Unit

| # | Cas de test |
|---|---|
| 1 | subscribe — ajoute un WS au canal `deviceId::key` |
| 2 | subscribe — déduplique les paires identiques |
| 3 | subscribe — ignore les paires sans deviceId ou key |
| 4 | unsubscribe — retire un WS d'un canal |
| 5 | unsubscribe — supprime le canal si plus aucun WS |
| 6 | publish — envoie JSON à tous les WS abonnés |
| 7 | publish — ignore les WS fermés (readyState ≠ 1) et les nettoie |
| 8 | publish — gère l'erreur ws.send sans crash |
| 9 | drop — retire un WS de tous ses canaux |
| 10 | drop — nettoie les canaux vides après drop |

### 8.4 Backend — Tests Data Histories + Charts WS (Priorité 🟠 P2)

#### 8.4.1 `src/features/data-histories/data-histories.repository.test.ts` — Unit

| # | Cas de test | Mocks |
|---|---|---|
| 1 | saveDataHistory — device trouvé par topic → parse JSON → persist | deviceRepository.findOneBy, repository.create/save |
| 2 | saveDataHistory — device non trouvé → warn, pas de persist | deviceRepository.findOneBy → null |
| 3 | saveDataHistory — JSON invalide → error, pas de persist | — |
| 4 | saveDataHistory — keyMapper.map → realtimeHub.publish pour chaque event | keyMapper.map, realtimeHub.publish spy |
| 5 | saveDataHistory — keyMapper échoue → persist en DB quand même (best-effort) | keyMapper.map → throw |
| 6 | saveDataHistory — realtimeHub échoue → persist en DB quand même | realtimeHub.publish → throw |

#### 8.4.2 `src/features/charts/ws/charts.ws.test.ts` — Unit

| # | Cas de test | Mocks |
|---|---|---|
| 1 | GET /ws/charts — auth OK → reçoit `{ type:"ok", event:"connected", userId }` | verifyOrigin → true, verifyJwtFromCookie → { userId } |
| 2 | GET /ws/charts — origin non autorisée → fermeture 1008 | verifyOrigin → false |
| 3 | GET /ws/charts — pas de cookie → fermeture 1008 | verifyJwtFromCookie → null |
| 4 | message subscribe → realtimeHub.subscribe + réponse ok | realtimeHub.subscribe |
| 5 | message unsubscribe → realtimeHub.unsubscribe + réponse ok | realtimeHub.unsubscribe |
| 6 | message invalide → `{ type:"error", code:400 }` | — |
| 7 | JSON invalide → `{ type:"error", code:400 }` | — |
| 8 | fermeture connexion → realtimeHub.drop | realtimeHub.drop spy |

### 8.5 Backend — Tests Routes manquantes (Priorité 🟡 P3)

#### 8.5.1 `src/features/devices/add/addDevice.route.test.ts` — Unit (compléter l'existant)

| # | Cas de test | Mocks |
|---|---|---|
| 1 | POST /devices/add — device avec subscribe → ensureSubscription appelé | repository.create, mqttService.ensureSubscription |
| 2 | POST /devices/add — device sans subscribe → pas d'abonnement MQTT | repository.create |
| 3 | POST /devices/add — erreur DB → 400 | repository.create → throw |
| 4 | POST /devices/add — erreur MQTT → 200 quand même (warn log) | ensureSubscription → throw |

#### 8.5.2 `src/features/devices/update/updateDevice.route.test.ts` — Unit (compléter l'existant)

| # | Cas de test | Mocks |
|---|---|---|
| 1 | PATCH /devices/update — subscribe changé → unsubscribe ancien + subscribe nouveau | unsubscribe, ensureSubscription |
| 2 | PATCH /devices/update — subscribe identique → pas de changement MQTT | — |
| 3 | PATCH /devices/update — erreur DB → 400 | repository.update → throw |

#### 8.5.3 `src/features/devices/getAll/getAllDevices.route.test.ts` — Unit

| # | Cas de test | Mocks |
|---|---|---|
| 1 | GET /devices — retourne liste paginée + meta.total | repository.findAndCount |
| 2 | GET /devices — includeActived=true filtre | repository.findAndCount |
| 3 | GET /devices — mapping via deviceMapperResponse | — |

#### 8.5.4 `src/features/devices/getOne/getDevice.route.test.ts` — Unit

| # | Cas de test | Mocks |
|---|---|---|
| 1 | GET /devices/:id — trouvé → 200 | repository.findById |
| 2 | GET /devices/:id — non trouvé → 404 | repository.findById → null |

#### 8.5.5 `src/features/users/delete/deleteUser.route.test.ts` — Unit

| # | Cas de test | Mocks |
|---|---|---|
| 1 | DELETE /users/delete — id UUID valide → 200 | repository.delete |
| 2 | DELETE /users/delete — erreur → 400 | repository.delete → throw |

#### 8.5.6 `src/features/users/getAll/getAllUsers.route.test.ts` — Unit

| # | Cas de test | Mocks |
|---|---|---|
| 1 | GET /users — liste paginée + meta.total | repository.findAndCount |
| 2 | GET /users — page=2, size=5 | repository.findAndCount |

#### 8.5.7 `src/features/users/getOne/getUser.route.test.ts` — Unit

| # | Cas de test | Mocks |
|---|---|---|
| 1 | GET /users/:id — trouvé → 200 + user + badges | repository.findById |
| 2 | GET /users/:id — non trouvé → null | repository.findById → null |
| 3 | GET /users/:id — badges vides → tableau vide | repository.findById → [user, []] |

#### 8.5.8 `src/features/access-log/get-all-route/getAllAccessLogs.route.test.ts` — Unit

| # | Cas de test | Mocks |
|---|---|---|
| 1 | GET /accessLogs — logs paginés + meta.total | repository.findAndCount |
| 2 | GET /accessLogs — pagination par défaut (page=1, size=10) | — |

#### 8.5.9 `src/features/badges/delete-route/badges.delete.route.test.ts` — Unit

| # | Cas de test | Mocks |
|---|---|---|
| 1 | DELETE /badges/delete — id valide → 200 + success:true | repository.deleteBadge |
| 2 | DELETE /badges/delete — non trouvé → 400 | repository.deleteBadge → erreur |

#### 8.5.10 `src/features/badges/update-route/badges.update.route.test.ts` — Unit

| # | Cas de test | Mocks |
|---|---|---|
| 1 | PATCH /badges/update — toggle deniedAccessFlag → 200 | repository.updateBadge |
| 2 | PATCH /badges/update — non trouvé → 400 | repository.updateBadge → erreur |

#### 8.5.11 `src/features/badges/badge.service.test.ts` — Unit

| # | Cas de test | Mocks |
|---|---|---|
| 1 | add — user inexistant → { status:400, message:"no_user_existed" } | userRepo.findOneBy → null |
| 2 | add — badge déjà existant → { status:400, message:"already_badge_registered" } | badgeRepo.findByCardUserId → badge |
| 3 | add — writer non trouvé (timeout) → { status:400 } | monitorControllerPresenceJob → throw |
| 4 | add — carte déjà enregistrée → { status:400, message:"already_registered" } | badgeRepo.findByCardId → badge |
| 5 | add — flux complet succès → { status:201, success:true } | tous mocks OK |

#### 8.5.12 `src/features/badges/validateBadge.test.ts` — Unit

| # | Cas de test |
|---|---|
| 1 | validateBadgeWithKey — clé dérivée correcte → true |
| 2 | validateBadgeWithKey — clé dérivée incorrecte → false |

#### 8.5.13 `src/features/charts/getAllPages/getAllPages.route.test.ts` — Unit

| # | Cas de test | Mocks |
|---|---|---|
| 1 | GET /pages — retourne toutes les pages | repository.find |
| 2 | GET /pages — aucune page → tableau vide | repository.find → [] |

#### 8.5.14 `src/features/charts/getOnePage/getOnePage.route.test.ts` — Unit

| # | Cas de test | Mocks |
|---|---|---|
| 1 | GET /pages/:id — trouvée → 200 | repository.findById |
| 2 | GET /pages/:id — non trouvée → 404 | repository.findById → null |

### 8.6 Backend — Tests Services/Utils/Plugins restants (Priorité 🟡 P3)

#### 8.6.1 `src/services/hash.service.test.ts` — Unit

| # | Cas de test |
|---|---|
| 1 | hash — retourne un hash Argon2id non vide |
| 2 | hash — deux appels même mot de passe → hash différent (salt) |
| 3 | verify — mot de passe correct → true |
| 4 | verify — mot de passe incorrect → false |

#### 8.6.2 `src/services/email.service.test.ts` — Unit

| # | Cas de test | Mocks |
|---|---|---|
| 1 | sendMail — appelle transporter.sendMail avec from, to, subject, html | nodemailer.createTransport |
| 2 | sendMail — avec pièces jointes → attachments mappées | — |
| 3 | sendMail — **constat** : le destinataire est `process.env.EMAIL_TO` et non `options.to` (bug ?) | — |

#### 8.6.3 `src/services/cron.service.test.ts` — Unit

| # | Cas de test |
|---|---|
| 1 | start — initialise le scheduler |
| 2 | addTask avant start → log error, pas d'enregistrement |
| 3 | addTask après start → enregistre la tâche |
| 4 | mutex empêche exécution parallèle |
| 5 | erreur dans fn → loggée, pas de crash |

#### 8.6.4 `src/plugins/error-handler.test.ts` — Unit

| # | Cas de test |
|---|---|
| 1 | erreur avec statusCode → utilise ce code |
| 2 | erreur sans statusCode → 500 |
| 3 | message et code transmis dans la réponse |

#### 8.6.5 `src/publishers/monitorControllerPresence.test.ts` — Unit

| # | Cas de test | Mocks |
|---|---|---|
| 1 | publie la commande puis attend la réponse | mqttService.publish, registerSubscription |
| 2 | timeout atteint → reject avec erreur | — |
| 3 | réponse reçue avant timeout → resolve avec payload | — |

### 8.7 Backend — Tests de sécurité transversaux

#### 8.7.1 `tests/security/auth-security.test.ts` — Integration

| # | Cas de test |
|---|---|
| 1 | JWT avec secret invalide est rejeté |
| 2 | JWT expiré est rejeté par le middleware |
| 3 | Refresh token ne peut pas servir d'access token |
| 4 | Route protégée sans token → 401 |
| 5 | Route protégée avec token expiré → 401 |

---

### 8.8 Frontend — Tests Hooks Auth (Priorité 🔴 P1)

#### 8.8.1 `src/features/auth/hooks/useAuth.test.ts` — Unit

| # | Cas de test | Mocks |
|---|---|---|
| 1 | login — appel API réussi → stocke tokens + retourne user | MSW POST /login |
| 2 | login — appel API échoué → retourne erreur | MSW 401 |
| 3 | logout — supprime les tokens stockés | — |
| 4 | forgotPassword — appel API /forgot-password | MSW |
| 5 | resetPassword — appel API /reset-password | MSW |

#### 8.8.2 `src/features/auth/hooks/useAuthStore.test.ts` — Unit

| # | Cas de test |
|---|---|
| 1 | état initial — user null, isAuthenticated false |
| 2 | setUser — met à jour user et isAuthenticated |
| 3 | clearUser — remet à l'état initial |

#### 8.8.3 `src/features/auth/hooks/useAuthRefresh.test.ts` — Unit

| # | Cas de test | Mocks |
|---|---|---|
| 1 | refresh automatique avant expiration | MSW POST /refresh-token |
| 2 | refresh échoué → déconnexion | MSW 401 |

#### 8.8.4 `src/features/auth/hooks/useWs.test.ts` — Unit

| # | Cas de test | Mocks |
|---|---|---|
| 1 | connexion WebSocket établie avec bon URL | WebSocket mock |
| 2 | messages reçus sont parsés et retournés | — |
| 3 | reconnexion automatique après déconnexion | — |

### 8.9 Frontend — Tests Composants Devices (Priorité 🟠 P2)

#### 8.9.1 `src/features/devices/components/DevicesList.test.tsx`

| # | Cas de test | Mocks |
|---|---|---|
| 1 | affiche la liste des devices | MSW GET /devices |
| 2 | état vide si aucun device | MSW → [] |
| 3 | pagination (page suivante/précédente) | MSW |
| 4 | indicateur online/offline correct | — |
| 5 | bouton activation toggle | MSW PATCH |

#### 8.9.2 `src/features/devices/components/DeviceForm.test.tsx`

| # | Cas de test |
|---|---|
| 1 | rendu avec tous les champs |
| 2 | soumission avec données valides |
| 3 | validation : champs requis manquants → erreurs |
| 4 | mode édition : pré-remplissage |

#### 8.9.3 `src/features/devices/components/CreateDevice.test.tsx`

| # | Cas de test | Mocks |
|---|---|---|
| 1 | création réussie → message succès | MSW POST /devices/add |
| 2 | erreur API → message erreur | MSW erreur |

#### 8.9.4 `src/features/devices/components/UpdateDevice.test.tsx`

| # | Cas de test | Mocks |
|---|---|---|
| 1 | chargement device → formulaire pré-rempli | MSW GET /devices/:id |
| 2 | mise à jour réussie | MSW PATCH /devices/update |

### 8.10 Frontend — Tests Charts + Access Logs (Priorité 🟠 P2)

#### 8.10.1 `src/features/charts/hooks/useLiveKeyValues.test.ts`

| # | Cas de test | Mocks |
|---|---|---|
| 1 | subscribe envoie le bon message WS | WebSocket mock |
| 2 | réception de valeurs → mise à jour state | onmessage |
| 3 | unsubscribe au démontage | — |
| 4 | reconnexion après perte WS | — |

#### 8.10.2 `src/features/charts/components/Charts.test.tsx`

| # | Cas de test | Mocks |
|---|---|---|
| 1 | rendu avec données | useLiveKeyValues mock |
| 2 | état chargement | — |

#### 8.10.3 `src/features/accessLogs/components/AccessLogsList.test.tsx`

| # | Cas de test | Mocks |
|---|---|---|
| 1 | affiche la liste des logs | MSW GET /accessLogs |
| 2 | pagination fonctionne | MSW |
| 3 | style granted vs denied | — |

### 8.11 Frontend — Tests Composants + Utils restants (Priorité 🟡 P3)

#### 8.11.1 `src/features/users/components/UsersList.test.tsx`

| # | Cas de test | Mocks |
|---|---|---|
| 1 | affiche la liste des utilisateurs | MSW GET /users |
| 2 | lien vers page de détail | — |

#### 8.11.2 `src/features/users/components/UpdateUser.test.tsx`

| # | Cas de test | Mocks |
|---|---|---|
| 1 | chargement user → formulaire pré-rempli | MSW GET /users/:id |
| 2 | mise à jour réussie | MSW PATCH /users/update |

#### 8.11.3 `src/features/users/components/BadgesList.test.tsx`

| # | Cas de test | Mocks |
|---|---|---|
| 1 | affiche badges d'un utilisateur | — |
| 2 | toggle deniedAccessFlag | MSW PATCH /badges/update |
| 3 | suppression badge avec confirmation | MSW DELETE /badges/delete |

#### 8.11.4 `src/utils/fetchUtil.test.ts` — Unit

| # | Cas de test | Mocks |
|---|---|---|
| 1 | GET réussi → retourne données | fetch mock |
| 2 | erreur réseau → throw | fetch reject |
| 3 | statut 401 → gestion non-authentifié | — |

#### 8.11.5 `src/utils/useDebounce.test.ts` — Unit

| # | Cas de test |
|---|---|
| 1 | valeur retournée après le délai |
| 2 | reset du timer si valeur change avant délai |

### 8.12 Frontend — Tests E2E Playwright (Priorité 🟠 P2)

#### 8.12.1 `e2e/auth.spec.ts`

| # | Parcours |
|---|---|
| 1 | Login credentials valides → redirection dashboard |
| 2 | Login credentials invalides → message erreur |
| 3 | Forgot password → confirmation envoi email |
| 4 | Logout → redirection page login |
| 5 | Page protégée sans auth → redirection login |

#### 8.12.2 `e2e/devices.spec.ts`

| # | Parcours |
|---|---|
| 1 | Login → naviguer devices → voir liste |
| 2 | Créer device → vérifier apparition dans liste |
| 3 | Modifier device → vérifier changements |
| 4 | Activer/désactiver device |

#### 8.12.3 `e2e/users.spec.ts`

| # | Parcours |
|---|---|
| 1 | Login → naviguer users → voir liste |
| 2 | Créer utilisateur |
| 3 | Modifier utilisateur |
| 4 | Supprimer utilisateur |

#### 8.12.4 `e2e/charts.spec.ts`

| # | Parcours |
|---|---|
| 1 | Login → naviguer charts → pages chargées |
| 2 | Sélectionner page → graphiques affichés |

---

### 8.13 Résumé quantitatif des tests à écrire

| Catégorie | Fichiers | Cas de test | Effort |
|---|---|---|---|
| Backend Auth | 6 | 28 | 2-3 jours |
| Backend MQTT/Services | 4 | 33 | 2 jours |
| Backend Realtime/Data/Charts WS | 4 | 28 | 1.5 jours |
| Backend Routes manquantes | 14 | 40 | 2 jours |
| Backend Services/Utils/Plugins | 5 | 16 | 1 jour |
| Backend Sécurité | 1 | 5 | 0.5 jour |
| Frontend Hooks Auth | 4 | 12 | 1 jour |
| Frontend Composants Devices | 4 | 13 | 1.5 jours |
| Frontend Charts + Access Logs | 3 | 9 | 1 jour |
| Frontend Composants + Utils restants | 5 | 11 | 1 jour |
| Frontend E2E Playwright | 4 | 15 | 2-3 jours |
| **TOTAL** | **54 fichiers** | **~210 cas de test** | **~16-19 jours** |

---

## 9. Recommandations

### 9.1 Actions immédiates (Sprint 0)

| # | Action | Priorité | Effort |
|---|---|---|---|
| R1 | **Corriger le script `test` backend** : remplacer par `"vitest"` | 🔴 P0 | 5 min |
| R2 | **Configurer la couverture** : ajouter `coverage: { provider: 'v8', reporter: ['text', 'lcov'], thresholds: { statements: 30 } }` dans vitest.config | 🔴 P0 | 30 min |
| R3 | **Supprimer les tests placeholder** `tests/units/user.test.ts` (backend) et `tests/dummy.test.tsx` (frontend) | 🔴 P0 | 5 min |
| R4 | **Corriger le test cassé** `ActiveDevice.route.test.ts` (implémenter le cas d'erreur) | 🟠 P1 | 15 min |
| R5 | **Corriger le test cassé** `CreateUser.test.tsx` (mocker une erreur API MSW pour le test 3) | 🟠 P1 | 15 min |
| R6 | **Remplacer les setTimeout** dans les tests frontend auth par `waitFor()` ou `findBy*` | 🟠 P1 | 1h |

### 9.2 Améliorations structurelles (Sprint 2+)

| # | Amélioration | Détail |
|---|---|---|
| R7 | **Tests de contrat API** | Valider que les schémas Zod backend et frontend sont cohérents |
| R8 | **Tests de sécurité** | JWT forgery, CORS bypass, SQL injection, XSS |
| R9 | **CI/CD** | Intégrer `vitest run --coverage` dans le pipeline de déploiement |
| R10 | **Seuil de couverture** | Définir un seuil minimum (ex: 60%) et le faire respecter en CI |

---

## 10. Synthèse

### 10.1 Score de maturité testing

| Critère | Score | Commentaire |
|---|---|---|
| Quantité de tests | 1/5 | ~66 tests pour 164 fichiers source (9% de couverture fichiers) |
| Qualité des tests existants | 3/5 | Tests crypto/badges très bons, mais anti-patterns dans les tests frontend |
| Couverture des modules critiques | 1/5 | Auth, MQTT, Logo Publisher à 0% |
| Outillage | 3/5 | Vitest + MSW + Playwright installés, mais mal configurés (script test, coverage) |
| Tests E2E | 0/5 | Playwright configuré mais 0 test écrit |
| Intégration CI/CD | 0/5 | Aucune intégration détectée |
| **Score global** | **1.5/5** | Infrastructure présente mais largement sous-exploitée |

### 10.2 Résumé quantitatif final

| Métrique | Valeur |
|---|---|
| Total fichiers de test | 17 (dont 2 placeholders) |
| Total cas de test | ~67 |
| Modules sans aucun test (backend) | 10/13 |
| Modules sans aucun test (frontend) | 6/9 |
| Problèmes identifiés dans les tests existants | 16 |
| Recommandations formulées | 16 |
| Tests à écrire en priorité (Sprint 1) | ~50-80 tests estimés |
| Tests E2E à écrire | Minimum 5-10 parcours |
