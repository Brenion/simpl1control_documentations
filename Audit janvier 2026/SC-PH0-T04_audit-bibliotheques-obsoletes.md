# Audit des Bibliothèques Obsolètes — Backend & Frontend

**Projet** : simpl1Control
**Tâche** : SC-PH0-T04 — Identifier les bibliothèques obsolètes backend et frontend
**User Stories** : SC-US-PH0-01 (backend) + SC-US-PH0-02 (frontend)
**Date** : 09/02/2026
**Version auditée** : 1.1.2
**Scoring** : 1 (mineur) → 5 (critique)

---

## Synthèse

| Métrique | Backend | Frontend | Total |
|----------|:---:|:---:|:---:|
| Dépendances totales | 49 | 71 | 120 |
| Dépréciées / abandonnées | 0 | 3 | **3** |
| Inutilisées | 2 | 3 | **5** |
| Redondantes / doublons | 3 | 3 | **6** |
| Mal classées (deps ↔ devDeps) | 4 | 2 | **6** |
| Mises à jour majeures disponibles | 13 | 10 | **23** |
| Mises à jour mineures/patch disponibles | 29 | 47 | **76** |

**Score global de santé des dépendances : 2/5** — Plusieurs packages dépréciés ou inutilisés alourdissent le projet. Les versions installées accusent un retard significatif par rapport aux dernières versions stables.

---

## 1. Packages dépréciés ou abandonnés

### 1.1 Frontend

| Package | Version installée | Statut | Sévérité | Détail |
|---------|:-:|---|:---:|---|
| `@material-ui/core` | 4.12.4 | **DÉPRÉCIÉ** (sept. 2021) | 5 | Ancienne version de Material UI (v4), remplacée par `@mui/material` (v6) qui est déjà installé dans le projet. **Aucune importation trouvée dans le code source.** |
| `@material-vega/core` | 0.1.0 | **ABANDONNÉ** | 4 | Package en v0.x, dernière mise à jour il y a plus d'un an, un seul mainteneur. **Aucune importation trouvée dans le code source.** |
| `@material-vega/material-ui` | 0.2.1 | **ABANDONNÉ** | 4 | Idem que ci-dessus. **Aucune importation trouvée dans le code source.** |

### 1.2 Backend

| Package | Version installée | Statut | Sévérité | Détail |
|---------|:-:|---|:---:|---|
| `modbus-serial` | 8.0.20-no-serial-port | **INUTILISÉ + version non-standard** | 4 | Version épinglée sur un build modifié (`-no-serial-port`), non officielle npm. **Aucune importation trouvée dans le code source** (`grep -r "modbus" src/` : 0 résultats). Package à supprimer. |

---

## 2. Packages inutilisés (aucune importation dans le code)

### 2.1 Frontend

| Package | Version | Type | Sévérité | Preuve |
|---------|:-:|---|:---:|---|
| `dot-prop` | 9.0.0 | dependencies | 3 | `grep -r "dot-prop"` : 0 résultats dans `src/` |
| `date-fns` | 4.1.0 | devDependencies | 3 | `grep -r "date-fns"` : 0 résultats dans `src/`. Ni importé ni utilisé dans le code source. |
| `autoprefixer` | 10.4.20 | devDependencies | 2 | Pas de `postcss.config.js`. Tailwind CSS v4 avec `@tailwindcss/vite` gère l'autoprefixing automatiquement. |

### 2.2 Backend

| Package | Version | Type | Sévérité | Preuve |
|---------|:-:|---|:---:|---|
| `@prisma/client` | 6.3.1 | dependencies | 4 | `grep -r "prisma\|@prisma"` : 0 résultats dans `src/`. Le projet utilise TypeORM exclusivement. |
| `modbus-serial` | 8.0.20-no-serial-port | dependencies | 4 | `grep -r "modbus"` : 0 résultats dans `src/`. Version non-standard pinned, jamais importée. |

---

## 3. Packages redondants ou en doublon

### 3.1 Frontend

| Doublon | Packages | Sévérité | Analyse |
|---------|----------|:---:|---|
| **Material UI v4 + v6** | `@material-ui/core` (4.12.4) + `@mui/material` (6.4.3) | 5 | Deux versions majeures de la même librairie. V4 est dépréciée et non utilisée dans le code. Seul `@mui/material` est importé. |
| **Plugin React Vite** | `@vitejs/plugin-react` (4.3.4) + `@vitejs/plugin-react-swc` (3.5.0) | 3 | Les deux font la même chose (transformation JSX). Seul `@vitejs/plugin-react-swc` est importé dans `vite.config.ts`. L'autre est installé mais jamais utilisé. |
| **Emotion en dep directe** | `@emotion/react` (11.14.0) + `@emotion/styled` (11.14.0) | 2 | Déclarées comme dépendances directes mais jamais importées directement dans le code source. Ce sont des peer dependencies de `@mui/material` — elles devraient rester transitives, pas explicites. |

### 3.2 Backend

| Doublon | Packages | Sévérité | Analyse |
|---------|----------|:---:|---|
| **JWT** | `@fastify/jwt` (9.0.3) + `jsonwebtoken` (9.0.2) | 4 | Les deux gèrent les JWT. `@fastify/jwt` est enregistré comme plugin Fastify mais le code utilise directement `jsonwebtoken` dans les routes. Fonctionnalité dupliquée. |
| **PostgreSQL** | `@fastify/postgres` (6.0.2) + `pg` (8.13.1) via TypeORM | 3 | `@fastify/postgres` est enregistré dans `server.ts` mais les repositories accèdent tous à la BDD via TypeORM (qui utilise `pg` en interne). Double connexion potentielle. |
| **Coverage** | `c8` (10.1.3) + `@vitest/ui` (3.0.5) | 2 | `c8` est un outil de coverage legacy. Vitest intègre son propre coverage (le frontend utilise déjà `@vitest/coverage-v8`). |

---

## 4. Packages mal classés (dependencies ↔ devDependencies)

### 4.1 Backend — En `dependencies` mais devraient être en `devDependencies`

| Package | Version | Raison |
|---------|:-:|---|
| `@faker-js/faker` | 9.4.0 | Utilisé uniquement dans les factories de test/seeding |
| `@jorgebodega/typeorm-factory` | 2.1.0 | Factories de seeding uniquement |
| `@jorgebodega/typeorm-seeding` | 7.1.0 | Script de seeding uniquement |
| `mailgen` | 2.0.29 | Templates d'emails — à vérifier si utilisé en production |

**Sévérité : 3/5** — Augmente inutilement la taille du bundle de production Docker.

### 4.2 Frontend — En `devDependencies` mais devraient être en `dependencies`

| Package | Version | Raison |
|---------|:-:|---|
| `i18next-browser-languagedetector` | 8.0.2 | Plugin de détection de langue au runtime |

**Sévérité : 2/5** — Peut fonctionner grâce au bundler mais la classification est incorrecte.

> **Note** : `date-fns` (4.1.0) a été reclassé en section 2.1 (inutilisé) car aucune importation n'a été trouvée dans le code source frontend.

---

## 5. Mises à jour disponibles — Backend

### 5.1 Mises à jour majeures (breaking changes potentiels)

| Package | Installée | Dernière | Écart | Risque | Notes |
|---------|:-:|:-:|---|:---:|---|
| `zod` | 3.24.4 | **4.3.6** | Majeur | 4 | Refonte API significative. Impact large : validation routes, schémas, type-provider-zod |
| `vite` | 6.1.0 | **7.3.1** | Majeur | 3 | Nouvelle version majeure. Changements config possibles |
| `vitest` | 3.0.5 | **4.0.18** | Majeur | 3 | API de test potentiellement changée |
| `pino` | 9.6.0 | **10.3.1** | Majeur | 3 | Changements de transport/formatage |
| `nodemailer` | 7.0.3 | **8.0.1** | Majeur | 2 | Changements d'API de transport |
| `eslint` | 9.20.0 | **10.0.0** | Majeur | 2 | Nouvelle config flat par défaut |
| `uuid` | 11.1.0 | **13.0.0** | Majeur | 2 | Deux versions majeures de retard |
| `@fastify/jwt` | 9.0.3 | **10.0.0** | Majeur | 2 | Changements API possibles |
| `cron-schedule` | 5.0.4 | **6.0.0** | Majeur | 2 | API de scheduling modifiée |
| `vite-plugin-node` | 4.0.0 | **7.0.0** | Majeur | 2 | Trois versions majeures de retard |
| `fastify-type-provider-zod` | 4.0.2 | **6.1.0** | Majeur | 3 | Lié à la migration Zod 4 |
| `@faker-js/faker` | 9.4.0 | **10.3.0** | Majeur | 1 | Dev/seeding uniquement |
| `@types/node` | 22.13.1 | **25.2.2** | Majeur | 1 | Types uniquement |

### 5.2 Mises à jour mineures et patch notables

| Package | Installée | Dernière | Type | Impact |
|---------|:-:|:-:|---|---|
| `fastify` | 5.2.1 | 5.7.4 | Minor | Patches de sécurité et bugs |
| `typeorm` | 0.3.20 | 0.3.28 | Patch | Corrections de bugs SQL |
| `mqtt` | 5.10.3 | 5.15.0 | Minor | Améliorations de stabilité |
| `pg` | 8.13.1 | 8.18.0 | Minor | Corrections PostgreSQL |
| `argon2` | 0.43.0 | 0.44.0 | Minor | Amélioration performance hashing |
| `dotenv` | 16.4.7 | 17.2.4 | Minor | Nouvelles fonctionnalités |
| `typescript` | 5.7.3 | 5.9.3 | Minor | Améliorations type-checking |
| `@fastify/cors` | 10.0.2 | 11.2.0 | Minor | Corrections CORS |

---

## 6. Mises à jour disponibles — Frontend

### 6.1 Mises à jour majeures (breaking changes potentiels)

| Package | Installée | Dernière | Écart | Risque | Notes |
|---------|:-:|:-:|---|:---:|---|
| `@mui/material` | 6.4.3 | **7.3.7** | Majeur | 4 | Migration MUI v7 : changements de thème, composants, API. Impact large. |
| `@mui/icons-material` | 6.4.7 | **7.3.7** | Majeur | 4 | Doit suivre la version de `@mui/material` |
| `@mui/x-data-grid` | 7.26.0 | **8.27.0** | Majeur | 4 | Changements API de la grille de données |
| `zod` | 3.24.2 | **4.3.6** | Majeur | 4 | Refonte API (même impact que backend) |
| `i18next` | 24.2.2 | **25.8.4** | Majeur | 3 | Changements d'initialisation possibles |
| `react-i18next` | 15.4.0 | **16.5.4** | Majeur | 3 | Lié à la migration i18next |
| `vite` | 6.2.0 | **7.3.1** | Majeur | 3 | Même impact que backend |
| `vitest` | 3.0.7 | **4.0.18** | Majeur | 3 | Même impact que backend |
| `dot-prop` | 9.0.0 | **10.1.0** | Majeur | 1 | Non utilisé — à supprimer |
| `eslint` | 9.19.0 | **10.0.0** | Majeur | 2 | Même impact que backend |

### 6.2 Mises à jour mineures et patch notables

| Package | Installée | Dernière | Type | Impact |
|---------|:-:|:-:|---|---|
| `react` / `react-dom` | 19.0.0 | 19.2.4 | Minor | Bugfixes React 19 |
| `@tanstack/react-router` | 1.105.0 | 1.159.5 | Minor | Améliorations routing |
| `@tanstack/react-form` | 0.41.3 | **1.28.0** | Majeur | ⚠️ Passage de v0 à v1 (stable) |
| `@tanstack/react-query` | 5.66.0 | 5.90.20 | Minor | Bugfixes et perf |
| `@toolpad/core` | 0.12.0 | 0.16.0 | Minor | ⚠️ Encore en v0 (pre-stable) |
| `tailwindcss` | 4.0.6 | 4.1.18 | Minor | Nouvelles utilitaires |
| `@playwright/test` | 1.50.1 | 1.58.2 | Minor | Nouveaux navigateurs |
| `typescript` | 5.7.2 | 5.9.3 | Minor | Améliorations type-checking |
| `happy-dom` | 17.4.3 | 20.5.3 | Minor | Améliorations DOM virtuel |
| `msw` | 2.7.3 | 2.12.9 | Minor | Mock API improvements |
| `react-vega` | 7.7.1 | **8.0.0** | Majeur | Migration Vega 8 |

---

## 7. Packages à surveiller (pre-stable)

| Package | Version | Statut | Risque |
|---------|:-:|---|:---:|
| `@toolpad/core` | 0.12.0 | **v0.x** — API instable, pas de garantie de rétrocompatibilité | 3 |
| `@tanstack/react-form` | 0.41.3 | **v0.x** — la v1.28.0 est disponible (migration recommandée) | 3 |
| `react-auth-kit` | 3.1.3 | Stable, mais la v4 est en alpha (4.0.2-alpha.11) | 2 |
| `vitest-browser-react` | 0.1.1 | **v0.x** — très early, la v2.0.5 est disponible | 2 |

---

## 8. Matrice de décision — Actions recommandées

### Actions immédiates (sans risque de régression)

| Action | Package(s) | Côté | Impact |
|--------|-----------|------|--------|
| **SUPPRIMER** | `@material-ui/core` | Frontend | Déprécié, 0 imports |
| **SUPPRIMER** | `@material-vega/core` | Frontend | Abandonné, 0 imports |
| **SUPPRIMER** | `@material-vega/material-ui` | Frontend | Abandonné, 0 imports |
| **SUPPRIMER** | `dot-prop` | Frontend | 0 imports |
| **SUPPRIMER** | `@prisma/client` | Backend | 0 imports, TypeORM utilisé |
| **SUPPRIMER** | `modbus-serial` | Backend | 0 imports, version non-standard |
| **SUPPRIMER** | `date-fns` | Frontend | 0 imports dans le code source |
| **SUPPRIMER** | `@vitejs/plugin-react` | Frontend | Doublon, seul SWC est configuré |
| **SUPPRIMER** | `autoprefixer` | Frontend | Inutile avec Tailwind CSS v4 |
| **DÉPLACER** vers devDeps | `@faker-js/faker`, `@jorgebodega/typeorm-factory`, `@jorgebodega/typeorm-seeding` | Backend | Seeding/test uniquement |
| **DÉPLACER** vers deps | `i18next-browser-languagedetector` | Frontend | Utilisé au runtime |

### Actions à planifier (nécessitent des tests)

| Priorité | Action | Package(s) | Risque |
|----------|--------|-----------|:---:|
| Haute | Mettre à jour | `fastify` 5.2.1 → 5.7.4 | 2 |
| Haute | Mettre à jour | `typeorm` 0.3.20 → 0.3.28 | 2 |
| Haute | Mettre à jour | `react` / `react-dom` 19.0.0 → 19.2.4 | 2 |
| Haute | Mettre à jour | `@tanstack/react-form` 0.41.3 → 1.28.0 | 3 |
| Haute | Mettre à jour | `typescript` → 5.9.3 (les deux) | 2 |
| Moyenne | Résoudre doublon | `@fastify/jwt` vs `jsonwebtoken` | 3 |
| Moyenne | Résoudre doublon | `@fastify/postgres` vs TypeORM `pg` | 3 |
| Moyenne | Résoudre doublon | `c8` vs `@vitest/coverage-v8` | 1 |
| Moyenne | Évaluer migration | `@emotion/react` + `@emotion/styled` (garder comme peer deps MUI) | 2 |

### Migrations majeures (à planifier sur un sprint dédié)

| Migration | Packages | Impact | Effort estimé |
|-----------|----------|:---:|:---:|
| MUI v6 → v7 | `@mui/material`, `@mui/icons-material`, `@mui/x-data-grid` | 4 | Élevé |
| Zod v3 → v4 | `zod`, `fastify-type-provider-zod` (backend + frontend) | 4 | Moyen |
| Vite v6 → v7 | `vite`, plugins associés | 3 | Moyen |
| Vitest v3 → v4 | `vitest`, `@vitest/*` | 3 | Faible |
| i18next v24 → v25 | `i18next`, `react-i18next` | 3 | Faible |
| Pino v9 → v10 | `pino`, `pino-pretty` | 3 | Faible |

---

## 9. Résumé des sévérités

| Sévérité | Nombre | Détail |
|:---:|:---:|---|
| **5 — Critique** | 2 | `@material-ui/core` déprécié + doublon MUI v4/v6, Zod v3 en fin de vie (v4 disponible) |
| **4 — Majeur** | 7 | Packages abandonnés, `@prisma/client` et `modbus-serial` inutilisés, doublon JWT, migrations MUI/Zod majeures nécessaires |
| **3 — Modéré** | 8 | Doublons fonctionnels, packages mal classés, packages pre-stable, `date-fns` inutilisé |
| **2 — Mineur** | 5 | Emotion redondante, coverage doublon, `autoprefixer`, classification frontend |
| **1 — Info** | 2 | Mises à jour mineures de confort |

---

*Rapport généré dans le cadre de l'audit SC-PH0-T04. Aucune modification n'a été apportée au code source.*
