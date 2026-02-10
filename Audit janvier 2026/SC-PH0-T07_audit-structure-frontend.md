# Audit de la Structure du Code Frontend

**Projet** : simpl1Control
**Tâche** : SC-PH0-T07 — Analyser la structure du code frontend existant
**User Story** : SC-US-PH0-02 — Auditer le frontend pour vérifier la qualité du code
**Date** : 10/02/2026
**Version auditée** : 1.1.2
**Scoring** : 1 (mineur) → 5 (critique)

---

## Stack technique identifié

| Composant | Technologie | Version |
|-----------|------------|---------|
| Framework UI | React | 19.0.0 |
| Langage | TypeScript (strict) | 5.7.2 |
| Routing | TanStack Router (file-based) | 1.105.0 |
| State serveur | TanStack React Query | 5.66.0 |
| Formulaires | TanStack React Form | 0.41.3 |
| Validation | Zod | 3.24.2 |
| UI Components | MUI Material | 6.4.3 |
| Data Grid | MUI X Data Grid | 7.26.0 |
| Styling | Tailwind CSS 4 + MUI sx | 4.0.6 |
| Graphiques | Vega / Vega-Lite / react-vega | 6.1.2 / 6.2.0 |
| Authentification | react-auth-kit | 3.1.3 |
| i18n | i18next + react-i18next | 24.2.2 / 15.4.0 |
| WebSocket | react-use-websocket | 4.13.0 |
| Build | Vite (SWC) | 6.2.0 |
| Tests | Vitest + Playwright | 3.0.7 / 1.50.1 |
| Mocking | MSW (Mock Service Worker) | 2.7.3 |
| PWA | vite-plugin-pwa + Workbox | 1.0.0 |
| Déploiement | Docker (nginx:alpine) | — |
| Package manager | pnpm | — |

---

## 1. Architecture & Organisation

### 1.1 Arborescence du code source

```
src/
├── main.tsx                          # Point d'entrée
├── App.tsx                           # Providers (Auth, QueryClient, Theme)
├── TanstackRouterAppProvider.tsx      # Router + AppProvider (Toolpad)
├── config.ts                         # URL API de base
├── i18n.ts                           # Configuration i18next
├── sw.ts                             # Service Worker PWA
├── routeTree.gen.ts                  # Auto-généré par TanStack Router
├── vite-env.d.ts                     # Types Vite
│
├── assets/icons/                     # Icônes SVG
├── commons/                          # Types partagés (address.type.ts)
├── components/                       # Composants globaux
│   ├── layouts/DashboardLayout.tsx
│   ├── SideBar.tsx
│   ├── HeaderActions.tsx
│   ├── InputSearch.tsx
│   ├── LanguageSwitcher.tsx
│   └── Meta.tsx
├── enums/                            # Enums partagés
├── styles/                           # CSS global
├── themes/                           # Thème MUI
├── utils/                            # Utilitaires (fetch, ws, debounce)
├── mocks/                            # MSW mock handlers
├── i18n/                             # Config i18n pour les tests
│
├── features/                         # Modules métier
│   ├── auth/                         # Authentification
│   │   ├── components/               # LoginForm, ForgotPassword, ResetPassword
│   │   ├── hooks/                    # useAuth, useAuthRefresh, useWs
│   │   ├── mocks/                    # Mock handlers auth
│   │   └── types/                    # AuthUser, LoginCredentials, etc.
│   ├── users/                        # Gestion utilisateurs
│   │   ├── components/               # UsersList, CreateUser, UpdateUser, UserForm
│   │   ├── hooks/                    # CRUD hooks (addUser, getUser, etc.)
│   │   ├── schemas/                  # Zod validation
│   │   └── types/                    # UserType, UserInterface
│   ├── devices/                      # Gestion appareils IoT
│   │   ├── components/               # DevicesList, CreateDevice, UpdateDevice, DeviceForm
│   │   ├── hooks/                    # CRUD hooks
│   │   ├── schemas/                  # Zod validation
│   │   └── types/                    # DeviceType, DeviceInterface
│   ├── charts/                       # Visualisation temps réel
│   │   ├── components/               # Charts, DynamicPage, DynamicNavbar
│   │   ├── hooks/                    # Pages + Live WebSocket data
│   │   └── schemas/                  # Page schema
│   └── accessLogs/                   # Logs d'accès
│       ├── components/               # AccessLogsList
│       ├── hooks/                    # getAllAccessLogs
│       └── types/                    # AccessLog types
│
└── routes/                           # TanStack Router (file-based)
    ├── __root.tsx                    # Layout racine + HelmetProvider
    ├── index.tsx                     # Page d'accueil (redirect)
    ├── _authenticated.tsx            # Guard d'authentification
    ├── _authenticated/dashboard/     # Routes protégées
    │   ├── users/, devices/, charts/, accessLogs/, lightings
    └── auth/                         # Routes publiques (login, forgot, reset)
```

### 1.2 Pattern architectural

Le frontend suit une architecture **Feature-Based** bien structurée. Chaque feature est autonome avec ses propres composants, hooks, types et schémas. Le routing est file-based via TanStack Router avec code-splitting automatique.

**Points positifs :**
- Découpage par feature clair et cohérent (auth, users, devices, charts, accessLogs)
- Séparation hooks/components/types/schemas systématique
- Routing file-based avec code-splitting automatique
- Guard d'authentification via layout route `_authenticated.tsx`
- Internationalisation complète avec namespaces par feature
- Validation Zod avec messages d'erreur i18n
- React Query pour la gestion d'état serveur
- Configuration PWA (Service Worker + Workbox)

### 1.3 Problèmes architecturaux identifiés

| # | Problème | Sévérité | Fichier(s) | Détail |
|---|----------|:---:|---|---|
| A1 | **Pas d'Error Boundary** — Aucun composant Error Boundary dans le projet. Une erreur dans un composant enfant peut crasher toute l'application. | 4 | Global |
| A2 | **`QueryClient` recréé à chaque rendu** — `new QueryClient()` est appelé dans le corps du composant `App` sans `useMemo` ou extraction en dehors du composant. | 4 | `App.tsx` |
| A3 | **Token WebSocket dans l'URL** — Le token JWT est passé en query parameter (`?token=...`) pour l'authentification WebSocket. Les URL apparaissent dans les logs serveur, le navigateur et les proxies. | 4 | `useWs.ts`, `useLiveKeyValues.ts` |
| A4 | **Dualité de thèmes** — Deux définitions de thème distinctes coexistent : `blackDashboardTheme.ts` (MUI ThemeProvider) et un thème inline dans `TanstackRouterAppProvider.tsx` (Toolpad AppProvider) avec des couleurs différentes. | 3 | `blackDashboardTheme.ts`, `TanstackRouterAppProvider.tsx` |
| A5 | **SideBar monolithique** — `SideBar.tsx` fait 413 lignes et combine AppBar, Drawer, menu utilisateur, boîte de dialogue de déconnexion, et navigation. | 3 | `SideBar.tsx` |
| A6 | **Type orphelin** — `commons/address.type.ts` exporte un type `Address` qui n'est importé nulle part dans le projet. | 2 | `commons/address.type.ts` |
| A7 | **Fichier CSS vide** — `styles/button.css` est vide (1 octet). | 1 | `styles/button.css` |

---

## 2. Data Fetching & State Management

### 2.1 Pattern d'appels API

Le projet utilise un **wrapper `fetch` custom** (`fetchUtil.ts`) qui :
- Construit l'URL à partir de `VITE_API_BASE_URL` + `/api/v1` + endpoint
- Injecte automatiquement le Bearer token (sauf si `skipAuth: true`)
- Gère les Content-Type JSON par défaut
- Lance une exception sur les codes 400/401

React Query est utilisé systématiquement pour les hooks de lecture (`useQuery`) et les mutations (`useMutation`).

### 2.2 Problèmes identifiés

| # | Problème | Sévérité | Fichier(s) | Détail |
|---|----------|:---:|---|---|
| D1 | **Pas de gestion centralisée des erreurs API** — Chaque hook de mutation a son propre try-catch avec `notif.show()`. Pas d'intercepteur global pour les erreurs 401 (token expiré), 403, 500. | 4 | Tous les hooks de mutation |
| D2 | **Pas de gestion du refresh token sur les appels API** — Le fetch wrapper ne gère pas le 401 automatiquement (retry après refresh). Le refresh token n'est géré que par un `setInterval` de 59s dans `useAuthRefresh.ts`. | 4 | `fetchUtil.ts`, `useAuthRefresh.ts` |
| D3 | **Credentials sélectif** — `credentials: 'include'` n'est ajouté que pour `/login`. Les autres appels n'incluent pas les cookies, ce qui peut poser problème si le backend s'attend à un cookie. | 3 | `fetchUtil.ts` |
| D4 | **Console.log dans le fetch** — `console.log('dans le fetch util', ...)` laissé en production. | 2 | `fetchUtil.ts:23` |
| D5 | **Pas d'abstraction pour les query keys** — Les clés React Query sont définies en dur dans chaque hook (`['user', ...]`, `['device', ...]`). Pas de constantes centralisées. | 2 | Hooks multiples |

### 2.3 WebSocket

| # | Problème | Sévérité | Fichier(s) | Détail |
|---|----------|:---:|---|---|
| W1 | **Token dans l'URL** — Déjà mentionné en A3. Sévérité sécurité. | 4 | `useWs.ts` |
| W2 | **Message `any`** — Le paramètre `msg` dans `useLiveKeyValues.ts` est typé `any`. | 3 | `useLiveKeyValues.ts:9` |
| W3 | **Pas de gestion de la reconnexion sur changement de token** — Si le token expire et se refresh, le WebSocket garde l'ancien token dans l'URL. | 3 | `useWs.ts` |

---

## 3. Qualité du code

### 3.1 Console statements résiduels

| Fichier | Ligne | Contenu |
|---------|:---:|---|
| `main.tsx` | 8 | `console.log(id)` — log d'un UUID à chaque démarrage |
| `config.ts` | 3 | `console.log('apiUrl contain', apiUrl)` — debug URL |
| `fetchUtil.ts` | 23 | `console.log('dans le fetch util', ...)` — debug fetch |
| `useAuthRefresh.ts` | 27 | `console.error('Refresh token failed:', error)` |
| `TanstackRouterAppProvider.tsx` | 17, 22 | `.catch(console.error)` — erreurs de query invalidation |
| `ForgotPasswordForm.tsx` | 40 | `console.error(error)` |

**Sévérité : 3/5** — 7 console statements dont plusieurs sont des logs de debug en français.

### 3.2 Type Safety

| # | Problème | Sévérité | Fichier | Détail |
|---|----------|:---:|---|---|
| TS1 | **`any` pour les messages WebSocket** | 3 | `useLiveKeyValues.ts:9` | `msg: any` — devrait avoir une interface `WsMessage` |
| TS2 | **`Record<string, any>` pour metadata** | 2 | `device.type.ts:36` | Intentionnel (documenté), mais pourrait être `Record<string, unknown>` |
| TS3 | **`as any` dans routeTree** | 1 | `routeTree.gen.ts` | Auto-généré, non contrôlable |

**Note positive** : `strict: true` est activé dans tsconfig, `noUnusedLocals` et `noUnusedParameters` sont actifs. La couverture de types est globalement bonne.

### 3.3 Duplication de code

| Pattern dupliqué | Occurrences | Impact | Sévérité |
|-----------------|:---:|---|:---:|
| Try-catch + `notif.show()` dans les mutations | ~10 hooks | Boilerplate identique dans chaque hook de mutation | 3 |
| Structure Create/Update identique entre users et devices | 2 features | Composants CreateUser/CreateDevice et UpdateUser/UpdateDevice quasi-identiques | 3 |
| Query wrapper `wrapQueryResult` appelé dans chaque composant de liste | 3 composants | Pattern répétitif | 2 |
| Hooks CRUD (add, update, delete, get, getAll) | 2 features | Structure identique pour users et devices | 2 |

### 3.4 Composants volumineux

| Composant | Lignes | Problème | Sévérité |
|-----------|:---:|---|:---:|
| `SideBar.tsx` | 413 | Combine navigation, menu utilisateur, logout dialog, responsive drawer | 3 |
| `DeviceForm.tsx` | 367 | Grand formulaire (acceptable vu la densité des champs) | 2 |
| `useLiveKeyValues.ts` | 140 | Hook complexe avec gestion WS + state normalization | 2 |

### 3.5 Accessibilité (a11y)

| # | Constat | Sévérité |
|---|---------|:---:|
| Les composants MUI fournissent une base a11y correcte (rôles ARIA, focus management) | — |
| `aria-label` présent sur les éléments interactifs clés (`SideBar.tsx`, `DevicesList.tsx`) | — |
| Boîtes de dialogue avec `aria-labelledby` et `aria-describedby` | — |
| Pas de gestion explicite du focus après navigation ou ouverture de modale | 2 |
| Icônes SVG sans `role="img"` ni `aria-hidden` systématique | 2 |
| Pas de skip-to-content link pour la navigation au clavier | 2 |

**Sévérité globale a11y : 2/5** — Base MUI correcte mais manque de polish pour une conformité WCAG AA.

### 3.6 Internationalisation (i18n)

**Excellent** — L'i18n est implémentée de manière exhaustive :
- `useTranslation()` systématique dans tous les composants
- 7 namespaces : `common`, `auth`, `users`, `devices`, `sidebar`, `validation`, `accessLog`
- 2 langues : français (langue par défaut) et anglais
- Messages de validation Zod internationalisés
- Selector de langue dans la SideBar

**Seule lacune** : 3 messages d'erreur hardcodés en français dans `Charts.tsx` (lignes 41, 54).

**Sévérité : 1/5**

---

## 4. Styling & Theming

### 4.1 Stratégie de styling mixte

Le projet utilise **deux systèmes de styling en parallèle** :

| Système | Usage | Exemples |
|---------|-------|----------|
| **MUI sx props** | Composants MUI, layouts, thème | `sx={{ bgcolor: 'background.default' }}` |
| **Tailwind CSS classes** | Utilitaires rapides, flexbox, spacing | `className="flex justify-between items-center w-full"` |
| **Tailwind `!important`** | Overrides MUI | `className="!bg-white/5"` |

### 4.2 Problèmes identifiés

| # | Problème | Sévérité | Détail |
|---|----------|:---:|---|
| S1 | **Deux thèmes incompatibles** — `blackDashboardTheme.ts` définit primary=#9c27b0 (violet), tandis que `TanstackRouterAppProvider.tsx` redéfinit primary=#37BCF8 (cyan) et secondary=#DDB867 (or). | 4 | Incohérence visuelle entre composants MUI et Toolpad |
| S2 | **Mélange MUI sx + Tailwind** — Les deux systèmes sont utilisés dans les mêmes composants, parfois sur le même élément. Augmente la complexité et les conflits de spécificité. | 3 | `SideBar.tsx:234`, `DeviceForm.tsx:65` |
| S3 | **`!important` Tailwind** — Utilisation de `!` prefix (`!bg-white/5`) pour override MUI, signe de conflits de spécificité. | 3 | `DeviceForm.tsx` |
| S4 | **Couleurs Tailwind custom sous-utilisées** — `tailwind.config.js` définit des couleurs custom (`primary.blue`, `secondary.extranet`) rarement utilisées. | 2 | `tailwind.config.js` |
| S5 | **Thème dark-mode uniquement** — Pas de mécanisme de basculement clair/sombre. Le thème est hardcodé en mode sombre. | 2 | `blackDashboardTheme.ts` |
| S6 | **Fichier CSS vide** — `button.css` est vide. | 1 | `styles/button.css` |
| S7 | **Preflight Tailwind désactivé** — `corePlugins.preflight: false` pour éviter les conflits MUI. Conséquence attendue mais à documenter. | 1 | `tailwind.config.js` |

---

## 5. Tests

### 5.1 Couverture par feature

| Feature | Fichiers de test | Couverture | Verdict |
|---------|:---:|---|---|
| Auth — LoginForm | 1 | Rendu + soumission + erreurs | Correct |
| Auth — ForgotPasswordForm | 1 | Rendu + soumission | Correct |
| Auth — ResetPasswordForm | 1 | Rendu + soumission | Correct |
| Users — UserForm | 1 | Validation des champs | Partiel |
| Users — CreateUser | 1 | Rendu du formulaire | Partiel |
| Users — UsersList | 0 | Aucun test | **Manquant** |
| Users — UpdateUser | 0 | Aucun test | **Manquant** |
| Devices — tous composants | 0 | Aucun test | **Manquant** |
| Charts — tous composants | 0 | Aucun test | **Manquant** |
| AccessLogs | 0 | Aucun test | **Manquant** |
| Hooks (fetch, mutations, queries) | 0 | Aucun test | **Manquant** |
| Utils (fetchUtil, ws, debounce) | 0 | Aucun test | **Manquant** |
| E2E (Playwright) | 0 | Dossier `e2e/` vide | **Non démarré** |

**Couverture estimée : ~10-15%** — 5 fichiers de test sur ~110 fichiers source.

### 5.2 Infrastructure de test

| Point | Statut | Note |
|-------|--------|------|
| Vitest configuré | ✅ | Browser mode avec Playwright/Chromium |
| MSW configuré | ✅ | Mock Service Worker avec handlers |
| Playwright configuré | ⚠️ | Config présente mais dossier `e2e/` vide |
| Test helpers | ✅ | `vitest.setup.ts`, `test-extend.ts`, `i18n-test-config.ts` |
| Coverage tool | ✅ | `@vitest/coverage-v8` installé |

**Sévérité : 4/5** — L'infrastructure de test est en place mais la couverture est très faible. Les chemins critiques (hooks, WebSocket, navigation) ne sont pas testés.

---

## 6. Configuration & DevOps

### 6.1 TypeScript

| Point | Statut | Note |
|-------|--------|------|
| `strict: true` | ✅ | Bonne pratique |
| `noUnusedLocals: true` | ✅ | Détecte le code mort |
| `noUnusedParameters: true` | ✅ | Idem |
| Path aliases | ⚠️ | `@auth/*`, `@components/*` définis mais `@dashboard/*`, `@template/*`, `@resources/*` non utilisés |

### 6.2 ESLint

| Point | Statut | Note |
|-------|--------|------|
| TypeScript recommandé | ✅ | `typescript-eslint` configuré |
| React Hooks | ✅ | Règles recommandées |
| React Refresh | ✅ | Vérification export |
| File naming (check-file) | ✅ | PascalCase composants, camelCase hooks |
| Prettier intégration | ✅ | `eslint-config-prettier` |
| `no-console` | ❌ | Pas configuré — console.logs passent |

### 6.3 IPs et URLs hardcodées

| Fichier | Valeur hardcodée | Sévérité |
|---------|-----------------|:---:|
| `vite.config.ts` | Proxy → `http://192.168.3.100:3000` et `ws://192.168.3.100` | 4 |
| `playwright.config.ts` | baseURL → `http://192.168.3.100:5173` | 3 |
| `Dockerfile` | `VITE_API_BASE_URL=http://192.168.3.100` | 4 |
| `.env` | `VITE_API_BASE_URL=http://localhost:3000` | ✅ OK (dev) |

**Sévérité : 4/5** — Les IPs hardcodées rendent le déploiement fragile. Toutes les URL devraient provenir de variables d'environnement.

### 6.4 Docker

| Point | Statut | Note |
|-------|--------|------|
| Multi-stage build | ✅ | Build + nginx séparés |
| Base alpine | ✅ | Images légères |
| Health check | ✅ | wget sur port 80 |
| Nginx SPA routing | ✅ | `try_files $uri /index.html` |
| Proxy API + WS | ✅ | `/api/` et `/ws/` reverse-proxied |
| Env var à l'exécution | ❌ | `VITE_API_BASE_URL` est injecté au build, pas au runtime |

### 6.5 CI/CD

Aucun pipeline CI/CD détecté (pas de `.github/workflows/`, `.gitlab-ci.yml`, etc.).

**Sévérité : 3/5**

---

## 7. Sécurité frontend

| # | Problème | Sévérité | Détail |
|---|----------|:---:|---|
| SEC1 | **Token JWT dans l'URL WebSocket** — Le token est visible dans les logs réseau, l'historique navigateur, et les proxies. | 4 | `useWs.ts`, `useLiveKeyValues.ts` |
| SEC2 | **Pas de sanitization des entrées** — Les données saisies par l'utilisateur sont validées par Zod mais pas sanitisées contre les XSS. MUI échappe le HTML par défaut dans la plupart des cas. | 2 | Global |
| SEC3 | **localStorage pour l'auth** — `react-auth-kit` stocke les tokens dans `localStorage` (configurable). Vulnérable aux attaques XSS. `httpOnly cookies` serait plus sécurisé. | 3 | `useAuthStore.ts` |
| SEC4 | **Console.log expose des données sensibles** — `fetchUtil.ts:23` log les paramètres de requête qui pourraient contenir des tokens. | 3 | `fetchUtil.ts` |

---

## 8. Synthèse des dettes techniques

### Par sévérité

| Sévérité | Nombre | Exemples clés |
|:---:|:---:|---|
| **5 — Critique** | 0 | — |
| **4 — Majeur** | 8 | Pas d'Error Boundary, QueryClient non memoïzé, token WS dans URL, IPs hardcodées, tests ~10%, pas de gestion 401, thèmes incompatibles |
| **3 — Modéré** | 12 | Console.logs, duplication code, SideBar monolithique, styling mixte MUI/Tailwind, localStorage auth, WS reconnection, CI/CD absent |
| **2 — Mineur** | 10 | a11y polish, type `any` metadata, Tailwind custom sous-utilisé, query keys non centralisées, path aliases inutilisés |
| **1 — Info** | 4 | Fichier CSS vide, preflight désactivé, routeTree `as any`, erreurs i18n Charts |

### Par catégorie

| Catégorie | Sévérité max | Nombre de points |
|-----------|:---:|:---:|
| Architecture | 4 | 7 |
| Data Fetching / State | 4 | 8 |
| Qualité du code | 3 | 10 |
| Styling / Theming | 4 | 7 |
| Tests | 4 | 2 |
| Configuration / DevOps | 4 | 5 |
| Sécurité frontend | 4 | 4 |

---

## 9. Recommandations prioritaires

### Priorité HAUTE (Sévérité 4)

1. **Ajouter un Error Boundary** — Implémenter un composant React Error Boundary au niveau de l'application et au niveau de chaque route pour éviter les crashes globaux.
2. **Mémoriser le QueryClient** — Extraire `new QueryClient()` en dehors du composant `App` ou utiliser `useMemo`/`useRef`.
3. **Sécuriser le token WebSocket** — Utiliser un protocole d'authentification basé sur un ticket temporaire (ticket → WS connect → validate server-side) au lieu du token JWT en query param.
4. **Extraire les IPs hardcodées** — Remplacer toutes les IPs `192.168.3.100` par des variables d'environnement dans `vite.config.ts`, `playwright.config.ts` et `Dockerfile`.
5. **Intercepteur 401 global** — Ajouter un intercepteur dans `fetchUtil.ts` qui déclenche automatiquement un refresh token sur réponse 401 avant de réessayer la requête.
6. **Unifier les thèmes** — Fusionner le thème Toolpad de `TanstackRouterAppProvider.tsx` avec `blackDashboardTheme.ts`.
7. **Augmenter la couverture de tests** — Prioriser les tests des hooks d'authentification, des mutations, et du composant Charts (WebSocket).

### Priorité MOYENNE (Sévérité 3)

8. **Supprimer les console.log** — Nettoyer les 7 instances de console.log/console.error de debug. Ajouter la règle ESLint `no-console`.
9. **Choisir une stratégie de styling** — Documenter les cas d'usage de MUI sx vs Tailwind pour éviter le mélange anarchique. Recommandation : MUI sx pour les composants MUI, Tailwind pour le layout.
10. **Découper `SideBar.tsx`** — Extraire `UserMenu`, `NavigationItems`, `LogoutDialog` en sous-composants.
11. **Factoriser les hooks CRUD** — Créer un hook factory générique `useCrudHooks<T>()` pour éliminer la duplication users/devices.
12. **Centraliser la gestion d'erreurs mutations** — Créer un wrapper `useMutationWithNotification()` pour le pattern try-catch + notif.show().
13. **Mettre en place un pipeline CI/CD**.
14. **Évaluer le remplacement de `localStorage`** par `httpOnly cookies` pour le stockage des tokens.

### Priorité BASSE (Sévérité 1-2)

15. Supprimer `styles/button.css` (vide).
16. Supprimer `commons/address.type.ts` (orphelin).
17. Nettoyer les path aliases TypeScript inutilisés (`@dashboard/*`, `@template/*`, `@resources/*`).
18. Centraliser les query keys React Query dans des constantes.
19. Ajouter `role="img"` ou `aria-hidden` sur les icônes SVG.
20. Externaliser les 3 messages d'erreur hardcodés en français dans `Charts.tsx`.

---

## 10. Points positifs notables

- **Architecture feature-based** bien découpée et cohérente
- **Internationalisation excellente** (i18n) avec 2 langues et 7 namespaces
- **TypeScript strict** activé avec détection du code mort
- **ESLint robuste** avec conventions de nommage de fichiers enforced
- **Validation Zod** avec messages d'erreur internationalisés
- **React Query** bien intégré pour la gestion d'état serveur
- **Code-splitting automatique** via TanStack Router
- **Infrastructure de test** en place (Vitest, MSW, Playwright)
- **PWA configurée** avec Service Worker et Workbox
- **Nginx bien configuré** pour SPA + API + WebSocket reverse proxy
- **Responsive design** avec breakpoints mobile via MUI + Tailwind

---

*Rapport généré dans le cadre de l'audit SC-PH0-T07. Aucune modification n'a été apportée au code source.*
