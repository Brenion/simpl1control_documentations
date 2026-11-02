
# SC-PH0-T16 — Rapport initial de couverture de tests (baseline)

> **User Story** : SC-US-PH0-08 — En tant que développeur, je veux mettre en place les outils d'analyse de couverture de tests **Tâche** : Générer un rapport initial de couverture de tests (baseline) **Projet** : simpl1Control v1.1.2 **Date** : 15 février 2026 **Méthode** : Exécution de la suite de tests avec couverture V8/c8

---

## Table des matières

1. [Résumé exécutif](https://claude.ai/chat/73c643dd-479d-4c32-8656-fe1cf286b152#1-r%C3%A9sum%C3%A9-ex%C3%A9cutif)
2. [Couverture globale](https://claude.ai/chat/73c643dd-479d-4c32-8656-fe1cf286b152#2-couverture-globale)
3. [Analyse détaillée — Backend](https://claude.ai/chat/73c643dd-479d-4c32-8656-fe1cf286b152#3-analyse-d%C3%A9taill%C3%A9e--backend)
4. [Analyse détaillée — Frontend](https://claude.ai/chat/73c643dd-479d-4c32-8656-fe1cf286b152#4-analyse-d%C3%A9taill%C3%A9e--frontend)
5. [Modules critiques à 0%](https://claude.ai/chat/73c643dd-479d-4c32-8656-fe1cf286b152#5-modules-critiques-%C3%A0-0)
6. [Modules les mieux couverts](https://claude.ai/chat/73c643dd-479d-4c32-8656-fe1cf286b152#6-modules-les-mieux-couverts)
7. [Écart par rapport aux objectifs Phase 7](https://claude.ai/chat/73c643dd-479d-4c32-8656-fe1cf286b152#7-%C3%A9cart-par-rapport-aux-objectifs-phase-7)
8. [Plan d'amélioration prioritaire](https://claude.ai/chat/73c643dd-479d-4c32-8656-fe1cf286b152#8-plan-dam%C3%A9lioration-prioritaire)
9. [Synthèse](https://claude.ai/chat/73c643dd-479d-4c32-8656-fe1cf286b152#9-synth%C3%A8se)

---

## 1. Résumé exécutif

| Métrique         |  Backend   |  Frontend  | Objectif Phase 7 |
| ---------------- | :--------: | :--------: | :--------------: |
| **% Statements** | **39.00%** | **15.43%** |       ≥70%       |
| **% Branches**   | **16.17%** | **14.96%** |       ≥70%       |
| **% Functions**  | **42.15%** | **16.83%** |       ≥70%       |
| **% Lines**      | **39.93%** | **15.94%** |       ≥70%       |

Le backend affiche une couverture de lignes de **~40%**, portée principalement par quelques modules bien testés (auth routes, badges crypto, handleReaderAccess). Le frontend est à **~16%** de couverture de lignes, avec des tests concentrés uniquement sur les composants auth et quelques composants users.

**Écart moyen vers l'objectif ≥70% :**

- Backend : **-30 points** (lignes)
- Frontend : **-54 points** (lignes)

---

## 2. Couverture globale

### 2.1 Backend — Vue d'ensemble

```
Statements : 39.00%  ████████░░░░░░░░░░░░ 
Branches   : 16.17%  ███░░░░░░░░░░░░░░░░░
Functions  : 42.15%  ████████░░░░░░░░░░░░
Lines      : 39.93%  ████████░░░░░░░░░░░░

Fichiers source : ~75
Fichiers de test : 11
```

### 2.2 Frontend — Vue d'ensemble

```
Statements : 15.43%  ███░░░░░░░░░░░░░░░░░
Branches   : 14.96%  ███░░░░░░░░░░░░░░░░░
Functions  : 16.83%  ███░░░░░░░░░░░░░░░░░
Lines      : 15.94%  ███░░░░░░░░░░░░░░░░░

Fichiers source : ~89
Fichiers de test : 6
```

---

## 3. Analyse détaillée — Backend

### 3.1 Couverture par module

| Module                                          | % Stmts | % Branch | % Funcs | % Lines | Verdict                      |
| ----------------------------------------------- | :-----: | :------: | :-----: | :-----: | ---------------------------- |
| **Enums**                                       |   100   |   100    |   100   |   100   | ✅ Complet                    |
| **Base entity**                                 |   100   |   100    |   100   |   100   | ✅ Complet                    |
| **Utils** (crypto, logger)                      |   90    |    0     |   60    |   90    | 🟡 Bon (branches manquantes) |
| **Auth — login.route**                          |   100   |   100    |   100   |   100   | ✅ Complet                    |
| **Auth — refresh-token.route**                  |   100   |   100    |   100   |   100   | ✅ Complet                    |
| **Auth — profile.route**                        |  66.66  |   100    |   50    |  66.66  | 🟡 Partiel                   |
| **Auth — forgot-password**                      |  35.29  |    0     |   40    |  35.29  | 🟠 Insuffisant               |
| **Auth — reset-password**                       |   50    |    0     |  66.66  |   50    | 🟠 Insuffisant               |
| **Auth — auth.service**                         |  5.55   |    0     |  16.66  |  5.55   | 🔴 Critique                  |
| **Badges — crypto utils**                       |   100   |   100    |   100   |   100   | ✅ Complet                    |
| **Badges — add route**                          |   100   |   100    |   100   |   100   | ✅ Complet                    |
| **Badges — delete/update routes**               |   50    |   100    |  66.66  |   50    | 🟠 Insuffisant               |
| **Badges — service**                            |    0    |    0     |    0    |    0    | 🔴 Absent                    |
| **Badges — repository**                         |  6.25   |    20    |  16.66  |  6.25   | 🔴 Critique                  |
| **Badges — validateBadge**                      |    0    |   100    |    0    |    0    | 🔴 Absent                    |
| **Users — add/update routes**                   |   100   |   100    |   100   |   100   | ✅ Complet                    |
| **Users — getAll/getOne/delete**                |  27-37  |  0-100   |  40-66  |  27-37  | 🟠 Insuffisant               |
| **Users — repository**                          |  3.03   |    0     |  16.66  |  3.03   | 🔴 Critique                  |
| **Devices — active route**                      |   100   |   100    |   100   |   100   | ✅ Complet                    |
| **Devices — add/update/getAll/getOne**          |  13-42  |    0     |  33-66  |  13-42  | 🟠 Insuffisant               |
| **Devices — repository**                        |  3.84   |    0     |    0    |  3.84   | 🔴 Critique                  |
| **Charts — entities/schema**                    |   100   |   100    |   100   |   100   | ✅ Complet                    |
| **Charts — repository**                         |   25    |    0     |    0    |   25    | 🔴 Critique                  |
| **Charts — routes**                             |  37-60  |  0-100   |  50-66  |  37-60  | 🟠 Insuffisant               |
| **Charts — WebSocket**                          |    0    |    0     |    0    |    0    | 🔴 Absent                    |
| **Data histories — repository**                 |    0    |    0     |    0    |    0    | 🔴 Absent                    |
| **Data histories — route**                      |  37.5   |   100    |  33.33  |  37.5   | 🟠 Insuffisant               |
| **Access logs — repository**                    |    0    |    0     |    0    |    0    | 🔴 Absent                    |
| **Access logs — route**                         |  37.5   |   100    |   40    |  37.5   | 🟠 Insuffisant               |
| **Services — handleReaderAccess**               |  90.14  |  77.14   |  66.66  |  92.64  | ✅ Très bon                   |
| **Services — mqtt.service**                     |   100   |  68.75   |   100   |   100   | ✅ Bon                        |
| **Services — logo-publisher**                   |  4.16   |    0     |    0    |  4.25   | 🔴 Critique                  |
| **Services — cron/email/hash/ping-pong/reload** |    0    |    0     |    0    |    0    | 🔴 Absent                    |
| **Plugins — error-handler**                     |   100   |    50    |   100   |   100   | ✅ Bon                        |
| **Plugins — websocket**                         |    0    |    0     |    0    |    0    | 🔴 Absent                    |
| **Realtime — key-mapper**                       |  4.22   |    0     |  7.69   |  5.12   | 🔴 Critique                  |
| **Realtime — realtime-hub**                     |  5.47   |    0     |    0    |  6.77   | 🔴 Critique                  |
| **Publishers — monitorControllerPresence**      |    0    |   100    |    0    |    0    | 🔴 Absent                    |
| **Server / cron-setup / seed**                  |    0    |    0     |    0    |    0    | 🔴 Absent                    |
| **Register**                                    |   100   |   100    |   100   |   100   | ✅ Complet                    |

### 3.2 Distribution de couverture backend

|Tranche|Nombre de fichiers|% du total|
|---|:-:|:-:|
|**100%** (complet)|~18|26%|
|**50-99%**|~8|12%|
|**1-49%**|~16|23%|
|**0%** (aucune couverture)|~27|**39%**|

**Constat** : Près de 40% des fichiers backend n'ont aucune couverture. La couverture globale de ~40% est tirée vers le haut par une poignée de fichiers à 100% (entités, schémas, quelques routes).

---

## 4. Analyse détaillée — Frontend

### 4.1 Couverture par module

| Module                                       | % Stmts | % Branch | % Funcs | % Lines | Verdict     |
| -------------------------------------------- | :-----: | :------: | :-----: | :-----: | ----------- |
| **Config**                                   |   100   |   100    |   100   |   100   | ✅ Complet   |
| **Enums — roles**                            |   100   |   100    |   100   |   100   | ✅ Complet   |
| **Users — schemas**                          |   100   |   100    |   100   |   100   | ✅ Complet   |
| **Auth — composants** (Login, Forgot, Reset) |  80.64  |  84.37   |  90.32  |  80.64  | ✅ Bon       |
| **Users — UserForm**                         |  93.75  |  93.75   |  94.44  |  93.75  | ✅ Très bon  |
| **Users — CreateUser**                       |  66.66  |   100    |   50    |  66.66  | 🟡 Partiel  |
| **Auth — hooks**                             |  18.33  |    0     |    0    |  18.96  | 🔴 Critique |
| **Users — hooks**                            |  10.16  |    0     |    8    |  10.34  | 🔴 Critique |
| **Utils** (fetchUtil, debounce, ws)          |  6.97   |    0     |  7.69   |   7.5   | 🔴 Critique |
| **App / Router / main**                      |  3.12   |    0     |    0    |  3.22   | 🔴 Critique |
| **Components** (SideBar, Layout, etc.)       |    0    |    0     |    0    |    0    | 🔴 Absent   |
| **Devices — tous**                           |    0    |    0     |    0    |    0    | 🔴 Absent   |
| **Charts — tous**                            |    0    |    0     |    0    |    0    | 🔴 Absent   |
| **Access Logs — tous**                       |    0    |    0     |    0    |    0    | 🔴 Absent   |
| **Routes** (toutes)                          |    0    |  0-100   |    0    |    0    | 🔴 Absent   |
| **Themes**                                   |    0    |   100    |   100   |    0    | 🔴 Absent   |

### 4.2 Distribution de couverture frontend

|Tranche|Nombre de fichiers|% du total|
|---|:-:|:-:|
|**100%**|4|4%|
|**50-99%**|6|7%|
|**1-49%**|12|13%|
|**0%** (aucune couverture)|**~67**|**76%**|

**Constat** : Plus de 3/4 des fichiers frontend n'ont aucune couverture. Seuls les composants d'authentification et le formulaire utilisateur sont testés de manière significative.

---

## 5. Modules critiques à 0%

### 5.1 Backend — Modules à 0% avec impact élevé

|Module|Fichier|Impact|Risque|
|---|---|---|---|
|**WebSocket Charts**|`charts.ws.ts`|Communication temps réel|Faille auth WS non détectée|
|**Data Histories Repository**|`data-histories.repository.ts`|Pipeline MQTT → DB → WS|Perte de données|
|**Logo Publisher**|`logo-publisher.service.ts` (~4%)|Commande PLC chauffage|Température incorrecte envoyée|
|**Ping-Pong Service**|`ping-pong.service.ts`|Monitoring devices|Devices marqués offline à tort|
|**Cron Service**|`cron.service.ts`|Orchestration tâches|Jobs non exécutés|
|**Reload MQTT**|`reload-mqtt.service.ts`|Réabonnement au démarrage|Topics perdus au redémarrage|
|**Hash Service**|`hash.service.ts`|Hashing Argon2|Mots de passe non hashés|
|**Email Service**|`email.service.ts`|Reset password|Emails non envoyés|
|**Badge Service**|`badge.service.ts`|Workflow NFC complet|Appairage badges cassé|
|**ValidateBadge**|`validateBadge.ts`|Validation accès physique|Accès non autorisé|
|**WebSocket Plugin**|`websocket.plugin.ts`|Configuration WS|Connexions WS refusées|
|**Key Mapper**|`key-mapper.ts` (~5%)|Transformation données temps réel|Données incorrectes|
|**Realtime Hub**|`realtime-hub.ts` (~7%)|PubSub WebSocket|Temps réel cassé|
|**Monitor Controller**|`monitorControllerPresence.ts`|Présence contrôleurs NFC|Timeout non détectés|
|**Server.ts**|`server.ts`|Initialisation application|Démarrage échoué silencieux|

### 5.2 Frontend — Modules à 0% avec impact élevé

|Module|Fichiers|Impact|Risque|
|---|---|---|---|
|**Devices — tous composants et hooks**|9 fichiers|CRUD appareils IoT|Interface devices cassée|
|**Charts — tous**|6 fichiers|Visualisation temps réel|Graphiques non fonctionnels|
|**Access Logs**|3 fichiers|Journaux d'accès|Logs non affichés|
|**Auth — hooks** (~19%)|6 fichiers|Authentification runtime|Tokens non rafraîchis|
|**Components partagés**|6 fichiers|Navigation, layout|App inutilisable|
|**Routes**|18 fichiers|Navigation SPA|Pages inaccessibles|
|**useWs**|1 fichier|WebSocket client|Temps réel cassé|
|**useLiveKeyValues**|1 fichier|Données live|Charts sans données|

---

## 6. Modules les mieux couverts

### 6.1 Backend — Top 10 fichiers

|Fichier|% Lines|Commentaire|
|---|:-:|---|
|`register.ts`|100%|Point d'entrée routes|
|`login.route.ts`|100%|Authentification login|
|`refresh-token.route.ts`|100%|Refresh JWT|
|`badges.add.route.ts`|100%|Ajout badge NFC|
|`deriveBadgeKey.utils.ts`|100%|Crypto dérivation clé|
|`deriveKeysAB.utils.ts`|100%|Crypto clés MIFARE|
|`crypto.ts`|100%|Chiffrement AES|
|`addUser.route.ts`|100%|Création utilisateur|
|`updateUser.route.ts`|100%|Mise à jour utilisateur|
|`ActiveDevice.route.ts`|100%|Activation device|
|`handleReaderAccess.ts`|92.64%|Contrôle d'accès NFC (meilleur test du projet)|
|`mqtt.service.ts`|100%|Service MQTT singleton|
|`error-handler.ts`|100%|Plugin erreurs|

### 6.2 Frontend — Top 5 fichiers

|Fichier|% Lines|Commentaire|
|---|:-:|---|
|`config.ts`|100%|Configuration API URL|
|`roles-enum.ts`|100%|Enum rôles|
|`user.schema.ts`|100%|Validation Zod users|
|`ForgotPasswordForm.tsx`|100%|Formulaire mot de passe oublié|
|`LoginForm.tsx`|96.29%|Formulaire de connexion|
|`ResetPasswordForm.tsx`|96.29%|Formulaire reset password|
|`UserForm.tsx`|93.75%|Formulaire utilisateur|

---

## 7. Écart par rapport aux objectifs Phase 7

L'objectif de la Phase 7 (SC-US-PH7-01) est une couverture backend ≥70%.

### 7.1 Analyse de l'écart

|Métrique|Backend actuel|Frontend actuel|Objectif|Écart backend|Écart frontend|
|---|:-:|:-:|:-:|:-:|:-:|
|Statements|39.00%|15.43%|≥70%|**-31 pts**|**-55 pts**|
|Branches|16.17%|14.96%|≥70%|**-54 pts**|**-55 pts**|
|Functions|42.15%|16.83%|≥70%|**-28 pts**|**-53 pts**|
|Lines|39.93%|15.94%|≥70%|**-30 pts**|**-54 pts**|

### 7.2 Effort estimé pour atteindre 70%

Pour atteindre 70% de couverture de lignes :

**Backend** (de 40% → 70%) :

- ~27 fichiers actuellement à 0% nécessitent des tests
- ~16 fichiers entre 1-49% nécessitent des tests supplémentaires
- Estimation : **~120-150 cas de test supplémentaires** → **8-12 jours**

**Frontend** (de 16% → 70%) :

- ~67 fichiers actuellement à 0% nécessitent des tests
- ~12 fichiers entre 1-49% nécessitent des tests supplémentaires
- Estimation : **~80-100 cas de test supplémentaires** → **8-10 jours**

---

## 8. Plan d'amélioration prioritaire

### 8.1 Quick wins — Impact maximal, effort minimal

Ces modules ont un impact élevé et une logique testable facilement :

|Priorité|Module|% actuel|Effort|Impact|
|:-:|---|:-:|:-:|---|
|🔴 P1|`auth.service.ts` (backend)|5.55%|1 jour|Sécurité authentification|
|🔴 P1|`hash.service.ts` (backend)|0%|0.5h|Hashing passwords|
|🔴 P1|`validateBadge.ts` (backend)|0%|0.5h|Sécurité accès physique|
|🔴 P1|`key-mapper.ts` (backend)|5.12%|1 jour|Transformation données IoT|
|🔴 P1|`realtime-hub.ts` (backend)|6.77%|0.5 jour|PubSub WebSocket|
|🟠 P2|`logo-publisher.service.ts` (backend)|4.25%|1 jour|Commande chauffage PLC|
|🟠 P2|`data-histories.repository.ts` (backend)|0%|1 jour|Pipeline MQTT → DB|
|🟠 P2|`charts.ws.ts` (backend)|0%|0.5 jour|Auth WebSocket|
|🟠 P2|Auth hooks (frontend)|18.96%|1 jour|Tokens, refresh, state|
|🟠 P2|Devices composants (frontend)|0%|1.5 jours|CRUD devices UI|
|🟡 P3|Repositories (backend)|0-6%|2 jours|Accès données|
|🟡 P3|Services restants (backend)|0%|1.5 jours|cron, email, ping-pong, reload|

### 8.2 Seuils de couverture recommandés

À configurer dans `vitest.config.ts` pour le CI :

|Phase|Seuil lignes|Seuil branches|Échéance|
|---|:-:|:-:|---|
|**Phase 0 (baseline)**|—|—|✅ Fait|
|**Phase 2**|45%|25%|Semaine 8|
|**Phase 4**|55%|35%|Semaine 15|
|**Phase 6**|60%|45%|Semaine 21|
|**Phase 7 (objectif)**|**70%**|**60%**|Semaine 24|

---

## 9. Synthèse

### 9.1 Score baseline

|Critère|Backend|Frontend|
|---|:-:|:-:|
|Couverture lignes|**39.93%**|**15.94%**|
|Couverture branches|**16.17%**|**14.96%**|
|Fichiers à 0%|39% (27/69)|76% (67/89)|
|Fichiers à 100%|26% (18/69)|4% (4/89)|
|Modules critiques non testés|15|8|

### 9.2 Indicateurs clés à suivre

|Indicateur|Valeur baseline|Cible Phase 7|
|---|:-:|:-:|
|% Lines backend|39.93%|≥70%|
|% Lines frontend|15.94%|≥60%|
|% Branches backend|16.17%|≥60%|
|Fichiers à 0% (total)|94|<20|
|Tests E2E Playwright|0|≥15|

### 9.3 Résumé quantitatif

|Métrique|Valeur|
|---|---|
|Couverture lignes backend (baseline)|**39.93%**|
|Couverture lignes frontend (baseline)|**15.94%**|
|Couverture branches backend (baseline)|**16.17%**|
|Couverture branches frontend (baseline)|**14.96%**|
|Fichiers source total|~158|
|Fichiers sans couverture|~94 (59%)|
|Fichiers avec couverture complète (100%)|~22 (14%)|
|Écart vers objectif 70% (backend lignes)|-30 points|
|Écart vers objectif 70% (frontend lignes)|-54 points|
|Tests supplémentaires estimés nécessaires|~200-250|
|Effort estimé total pour atteindre 70%|~16-22 jours|

---

_Ce rapport constitue la baseline de référence (Phase 0) pour le suivi de la couverture de tests tout au long du projet. Les prochaines mesures seront prises à chaque fin de phase pour suivre la progression vers l'objectif ≥70%._