# Audit Backend — Mediacio

**Date :** 16 février 2026
**Scope :** Code applicatif + tests
**Référentiel :** Best practices communautaires (Node.js, TypeScript, MikroORM, Koa)
**Niveau :** Exhaustif

---

## 1. Vue d'ensemble

| Élément | Valeur |
|---|---|
| Framework web | Koa 2.15 + @triptyk/nfw-core |
| Langage | TypeScript (ESNext, strict mode) |
| ORM | MikroORM 5.8.10 (PostgreSQL) |
| Auth | JWT (HS512) + bcrypt |
| Validation | fastest-validator-decorators |
| Tests | Jest 29.7 (ts-jest ESM) |
| Module system | ES Modules |
| Fichiers TS | ~531 |
| Lignes de code estimées | ~23 000 |

### Stack complète

- **Auth :** jsonwebtoken, bcrypt, @casl/ability
- **Documents :** docxtemplater, pizzip
- **Microsoft :** @azure/msal-node, @microsoft/microsoft-graph-client
- **Email :** mailgun.js, mustache (templates)
- **Utilitaires :** date-fns, uuid, mathjs
- **Lint :** ESLint (typescript-eslint), jscpd (détection duplication)

---

## 2. Architecture & Structure

```
src/
├── api/
│   ├── controllers/          (28 contrôleurs)
│   ├── models/               (24 entités + embeddables)
│   ├── repositories/         (24 repositories custom)
│   ├── services/             (8 services métier)
│   ├── guards/               (guards d'autorisation)
│   ├── middlewares/           (6 middlewares)
│   ├── decorators/           (8 décorateurs custom)
│   ├── validators/           (25 schémas de validation)
│   ├── serializer/           (22 sérialiseurs JSON:API)
│   ├── deserializer/         (20 désérialiseurs)
│   ├── error-handler/        (gestionnaires d'erreurs)
│   ├── abilities/            (14 fichiers CASL ACL)
│   ├── enums/                (20 énumérations)
│   ├── templates/            (10 utilitaires de templates)
│   ├── query-params-schema/  (schémas de paramètres)
│   ├── utils/                (11 utilitaires)
│   └── areas/                (enregistrement des zones)
├── json-api/                 (implémentation JSON:API)
├── calendar/                 (intégration Microsoft Graph)
├── calendarimpl/             (implémentation calendrier)
├── database/                 (factories + seeders)
└── zip-importer/             (traitement ZIP)
```

### Points forts architecturaux

- Séparation claire controllers / services / repositories
- Injection de dépendances cohérente via décorateurs
- Conformité à la spécification JSON:API
- Système ACL complet basé sur CASL avec granularité par rôle
- Filtrage automatique par rôle sur les entités (soft delete + filters MikroORM)
- Pattern factory pour les données de test

### Points faibles architecturaux

- **Logique métier dans les contrôleurs** : les contrôleurs `beneficiary`, `file`, et `auth` contiennent de la logique de transformation/sérialisation qui devrait être dans la couche service ou serializer (voir §5)
- **Couche service sous-utilisée** : seulement 8 services pour 28 contrôleurs — beaucoup de logique est directement dans les contrôleurs
- **Pas de couche DTO explicite** : les désérialiseurs utilisent `any` comme type d'entrée et de sortie

---

## 3. Sécurité

### 3.1 Gestion JWT — CRITIQUE

**Fichier :** `src/api/middlewares/current-user.middleware.ts`

**Problème 1 — Absence de vérification null sur le header Authorization :**

Le middleware fait `context.header.authorization.split(' ')` sans vérifier que `authorization` existe. Si le header est absent, cela provoque un crash.

**Problème 2 — Pas de try-catch autour de `Jwt.verify()` :**

Si le token est malformé, expiré ou signé avec une mauvaise clé, `Jwt.verify()` lance une exception non rattrapée. Le middleware ne distingue pas les cas (token expiré vs token invalide vs token malformé).

**Problème 3 — Erreur générique :**

Le middleware lance `new Error('Invalid token')` au lieu d'un `createHttpError(401, ...)`. Cela contourne le gestionnaire d'erreurs HTTP et peut produire un 500 au lieu d'un 401.

**Recommandation :**
```typescript
// Exemple de correction
const authHeader = context.header.authorization;
if (!authHeader?.startsWith('Bearer ')) {
  return undefined; // ou throw createHttpError(401)
}
try {
  const token = authHeader.slice(7);
  const decoded = Jwt.verify(token, secret, { complete: true });
  // ...
} catch (err) {
  if (err instanceof Jwt.TokenExpiredError) {
    throw createHttpError(401, 'Token expired');
  }
  throw createHttpError(401, 'Invalid token');
}
```

### 3.2 Récupération de mot de passe — Timing attack

**Fichier :** `src/api/controllers/auth.controller.ts` (lignes 69-92)

Le endpoint de password recovery retourne toujours un objet vide `{}` que l'utilisateur existe ou non. C'est une bonne pratique pour ne pas révéler l'existence d'un compte. **Cependant**, le temps de réponse diffère : si l'utilisateur existe, un email est envoyé (opération I/O lente). Un attaquant peut mesurer la latence pour déterminer si un email est inscrit.

**Recommandation :** Envoyer l'email de manière asynchrone (fire-and-forget) pour uniformiser le temps de réponse, ou ajouter un délai artificiel.

### 3.3 Rate Limiting — Insuffisant pour la production

**Fichier :** `src/api/middlewares/rate-limit.middleware.ts`

| Problème | Impact |
|---|---|
| Store en mémoire (`new Map()`) | Ne survit pas aux redémarrages, inutilisable en multi-instances |
| Limitation par IP uniquement (`ctx.ip`) | Vulnérable aux réseaux partagés (proxy, VPN d'entreprise) |
| Pas de gestion X-Forwarded-For | Derrière un reverse proxy, toutes les requêtes ont la même IP |
| Désactivé en test (`Infinity`) | Acceptable mais non testé en conditions réelles |

**Recommandation :** Migrer vers un store Redis (ex. `koa-ratelimit` avec Redis adapter) et ajouter le support X-Forwarded-For.

### 3.4 Input Sanitization

Seul `sanatize-query.ts` effectue un nettoyage (pour le full-text search PostgreSQL). Les requêtes SQL utilisent des paramètres bindés (protection injection SQL via MikroORM), ce qui est correct. Cependant, il n'y a pas de validation explicite de la longueur des entrées au niveau middleware, ni de protection XSS globale (le JSON:API atténue ce risque mais ne l'élimine pas).

### 3.5 Secrets & Configuration

- **Pas de `.env.example`** : aucun fichier ne documente les variables d'environnement requises
- **`production.env` dans le dépôt** : contient des informations de connexion base de données et clé Mailgun
- **Secrets dans les fichiers `.js`** : JWT secret, clés API chargés via des fichiers JS importés dynamiquement — pas de validation que toutes les clés requises sont présentes au démarrage

**Recommandation :** Créer un `.env.example`, valider les variables requises au boot (fail-fast), et s'assurer que `production.env` est dans `.gitignore`.

---

## 4. Base de données & ORM

### 4.1 Modèle de données

24 entités avec une hiérarchie claire :

```
BaseModel<T>  (abstract)
  ├── PrimaryKey: UUID v4
  ├── createdAt: Date
  └── updatedAt: Date (auto-updated)

SoftDeletableModel<T> extends BaseModel<T>
  └── deletedAt: Date (nullable) + filtre automatique
```

Les rôles disposent de filtres automatiques MikroORM (`admin_access`, `user_access`, `animator_access`, etc.) qui restreignent les données visibles selon le rôle de l'utilisateur connecté. C'est une bonne pratique.

### 4.2 Risques N+1

La stratégie `LoadStrategy.SELECT_IN` est utilisée globalement, ce qui est un bon choix pour éviter les jointures massives. Cependant :

| Contrôleur | Relations chargées | Risque |
|---|---|---|
| `BeneficiaryController.placeholdersFile` | 7 relations dont 2 nestées | Élevé si relations volumineuses |
| `FileController.populateTemplate` | 10 relations avec nesting profond | Élevé |
| `UserRepository.replaceUserOnResources` | 6 relations | Moyen — charge tout en mémoire pour mise à jour |

**Problèmes identifiés :**
- Chargement d'entités complètes alors que seuls quelques champs sont nécessaires (pas de `fields` sélectif dans les `populate`)
- Les relations soft-deleted sont potentiellement chargées par les `populate` (sauf si le filtre est actif globalement)
- Pas d'opérations batch pour les mises à jour massives (ex. `replaceUserOnResources` itère et met à jour un par un)

**Recommandation :** Utiliser des projections (`fields`) dans les `populate`, implémenter des opérations `UPDATE ... WHERE` en batch via `QueryBuilder` pour les mises à jour massives.

### 4.3 Migrations

17+ migrations de octobre 2023 à juillet 2025. Un fichier snapshot `.snapshot-mediacio.json` est maintenu. Pattern standard MikroORM, pas de problème identifié.

---

## 5. Qualité du code

### 5.1 Logique métier dans les contrôleurs — DETTE MAJEURE

C'est le problème le plus important du backend. Les contrôleurs `BeneficiaryController` et `FileController` contiennent chacun **~100 lignes de transformation de données** pour la génération de templates :

```typescript
// beneficiary.controller.ts — lignes 76-180
contacts: beneficiaryContext.contacts.getItems().map((e) => ({
  title: translateTitle(e.title) ?? '',
  firstName: e.firstName ?? '',
  lastName: e.lastName ?? '',
  // ... 11 champs supplémentaires
})),
```

Ce même code est **dupliqué quasi à l'identique** dans `FileController` (lignes 156-235).

**Impact :**
- Violation du Single Responsibility Principle
- Code dupliqué difficile à maintenir (modifier un mapping = modifier 2+ fichiers)
- Contrôleurs de 250-300 lignes au lieu de 50-80
- Logique non testable unitairement (liée au contexte HTTP)

**Recommandation :** Extraire dans un `TemplatePlaceholderService` dédié qui reçoit les entités et retourne les mappings. Cela permet aussi de le tester unitairement.

### 5.2 Duplication de code

Au-delà du mapping de templates, on observe :
- Chaque contrôleur répète les mêmes décorateurs (`@UseErrorHandler`, `@UseGuard`)
- Les patterns de `populate` sont dupliqués entre les endpoints GET (list vs detail)
- Les traductions (translateTitle, translateIncomeType, etc.) sont appelées à de multiples endroits

### 5.3 Type safety — Utilisation de `any`

27 occurrences de `: any` identifiées :

| Zone | Occurrences | Risque |
|---|---|---|
| Désérialiseurs (entrée/sortie) | 20 | Élevé — aucune vérification de type à la désérialisation |
| ConfigurationService (defaultValue) | 1 | Moyen |
| AclService (cast forcé) | 1 | Faible — nécessaire pour l'appel de méthode statique |
| Divers | 5 | Variable |

La règle ESLint `@typescript-eslint/no-explicit-any` est configurée en **warn** au lieu de **error**. Cela permet aux `any` de passer sans bloquer le build.

**Recommandation :** Passer la règle en `error`, typer les désérialiseurs avec des génériques ou au minimum `unknown`, et corriger les 27 occurrences.

### 5.4 Nommage — Incohérences

| Problème | Fichier/Zone |
|---|---|
| Double extension `.ts.ts` | `src/api/controllers/contact.controller.ts.ts` |
| Orthographe britannique vs américaine | `location.deserialiser.ts` vs `*.serializer.ts` (tous les autres) |
| Faute de frappe dans le nom de fichier | `sanatize-query.ts` au lieu de `sanitize-query.ts` |
| Commentaire/test en anglais approximatif | `"Replace old user by new one on ressources"` (resources) |

---

## 6. Gestion des erreurs

### 6.1 Structure

Deux gestionnaires d'erreurs :
- `DefaultErrorHandler` : erreurs génériques (validation, HTTP, internes)
- `JsonApiErrorHandler` : erreurs formatées selon la spécification JSON:API

### 6.2 Problèmes identifiés

**Fuite de stack trace en log :**

Les deux handlers font `this.loggerService.trace(error)` qui log l'objet erreur complet (incluant le stack trace) dans des fichiers via `tracer` avec rotation (`dailyfile`, maxLogFiles: 10). En production, les logs peuvent contenir des informations sensibles (chemins de fichiers, noms de variables internes, etc.).

**Pas de distinction d'erreurs internes :**

Le handler catch-all ne distingue pas les types d'erreurs internes. Une erreur de connexion DB et une erreur de logique métier produisent le même "Internal server error" en production.

**Recommandation :** Ajouter un niveau de classification des erreurs (opérationnelles vs programmation), sanitiser les logs en production, et ajouter du contexte structuré aux logs (request ID, user ID, route).

---

## 7. Tests

### 7.1 Vue d'ensemble

| Métrique | Valeur |
|---|---|
| Fichiers de test | ~20 |
| Lignes de test | ~1 535 |
| Framework | Jest 29.7 (ts-jest ESM) |
| Pattern | Tests d'acceptation HTTP (fetch) + tests unitaires |
| Setup | Seeders dédiés par domaine + factories |
| Timeout | 30 secondes |

### 7.2 Points forts

- `repayment-plan.test.ts` (801 lignes) : test exhaustif de la logique métier d'amortissement avec ~100+ assertions, 5 scénarios majeurs, et utilisation de factories
- Seeders dédiés par rôle (admin, animator, user, support, energy) permettant des tests contextuels
- Factories basées sur @ngneat/falso pour la génération de données réalistes
- Isolation correcte avec `beforeEach`/`afterEach` et nettoyage DB

### 7.3 Faiblesses

| Problème | Impact |
|---|---|
| **Pas de tests d'erreurs** | Les chemins d'erreur (token invalide, permissions refusées, données invalides) ne sont pas testés |
| **Pas de tests d'authentification/autorisation** | Le système CASL entier n'a pas de tests dédiés |
| **Tests superficiels pour certains domaines** | `user.test.ts` = 32 lignes, 1 seul test case |
| **Pas de tests d'intégration pour les error handlers** | Le comportement en cas d'erreur n'est pas vérifié |
| **Pas de rapport de couverture** | Aucune métrique de couverture configurée ou publiée |
| **Pas de test du middleware JWT** | Le middleware `current-user` n'est jamais testé directement |

### 7.4 Recommandations tests

1. Ajouter des tests d'erreur pour chaque contrôleur (401, 403, 404, 422)
2. Tester le système CASL avec des scénarios par rôle
3. Configurer Jest avec `--coverage` et définir des seuils minimaux
4. Tester le middleware JWT (token expiré, invalide, absent)
5. Ajouter des tests de non-régression pour les bugs corrigés

---

## 8. Infrastructure & DevOps

### 8.1 Docker

**Problèmes identifiés :**
- Pas de directive `USER` → le conteneur tourne en root
- Pas de `HEALTHCHECK`
- `npm pkg delete scripts.postinstall` sans explication

**Recommandation :** Ajouter un utilisateur non-root, un healthcheck, et documenter le script de build.

### 8.2 CI/CD

**Workflow de test :** existe (`.github/workflows/test.yml`)

**Problèmes :**
- Utilise MySQL 5.7 alors que la production est PostgreSQL — incohérence potentielle
- Credentials hardcodées dans le workflow (`test123*`)
- Pas d'étape de linting dans le pipeline CI
- Pas de publication de rapport de couverture
- Pas de build de vérification

### 8.3 Cron Tasks

Deux tâches enregistrées :
1. **file-number** (annuel) : reset du numéro de séquence de fichiers
2. **follow-ups** (chaque minute) : envoi des emails de relance en attente

**Problème potentiel :** la tâche follow-ups s'exécute chaque minute et itère sur tous les follow-ups en attente. Si l'envoi d'un email échoue, le follow-up reste en statut "Waiting" et sera retentée chaque minute indéfiniment. Il n'y a pas de mécanisme de retry limité ni de dead-letter.

---

## 9. Synthèse des dettes techniques

### Critiques (à traiter en priorité)

| # | Problème | Fichier(s) | Effort estimé |
|---|---|---|---|
| 1 | Crash possible du middleware JWT (null check manquant) | `current-user.middleware.ts` | 1h |
| 2 | Logique métier dans les contrôleurs (~200 lignes dupliquées) | `beneficiary.controller.ts`, `file.controller.ts` | 1-2j |
| 3 | Désérialiseurs typés en `any` (20 fichiers) | `src/api/deserializer/*.ts` | 1j |
| 4 | Pas de tests auth/ACL | `tests/` | 2-3j |

### Importantes (à planifier)

| # | Problème | Fichier(s) | Effort estimé |
|---|---|---|---|
| 5 | Rate limiting en mémoire, pas adapté au multi-instances | `rate-limit.middleware.ts` | 0.5j |
| 6 | Pas de `.env.example` ni validation des variables au boot | Config | 0.5j |
| 7 | N+1 queries — relations chargées sans projection | Repositories / Controllers | 1-2j |
| 8 | Pas de rapport de couverture de tests | `jest.config.cjs` | 0.5j |
| 9 | Follow-up cron sans retry limité ni dead-letter | `cron-tasks.ts` | 1j |
| 10 | Docker : conteneur root, pas de healthcheck | `Dockerfile` | 0.5j |

### Mineures (à corriger au fil de l'eau)

| # | Problème | Fichier(s) |
|---|---|---|
| 11 | Fichier `contact.controller.ts.ts` (double extension) | Controllers |
| 12 | `sanatize-query.ts` → `sanitize-query.ts` | Utils |
| 13 | `location.deserialiser.ts` → `location.deserializer.ts` | Deserializer |
| 14 | ESLint `no-explicit-any` en warn → passer en error | `.eslintrc.json` |
| 15 | CI utilise MySQL au lieu de PostgreSQL | `.github/workflows/` |

---

## 10. Points positifs à conserver

- **Injection de dépendances cohérente** : pattern décorateur bien maîtrisé
- **JSON:API compliance** : sérialisation/désérialisation complète avec pagination, links, relationships
- **Système ACL granulaire** : CASL avec abilities par entité et par rôle
- **Soft delete + filtres automatiques** : bonne gestion de la suppression logique
- **Chiffrement AES-256-CBC** : tokens calendrier chiffrés avec IV aléatoire
- **Factories de test** : données réalistes avec @ngneat/falso
- **Pattern repository** : abstraction correcte de l'accès aux données
- **Validation déclarative** : schémas de validation via décorateurs

---

*Rapport généré le 16/02/2026 — Audit read-only, aucune modification apportée au code.*
