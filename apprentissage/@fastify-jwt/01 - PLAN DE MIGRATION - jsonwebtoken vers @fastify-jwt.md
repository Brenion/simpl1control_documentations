## 📋 PLAN DE MIGRATION : jsonwebtoken → @fastify/jwt

Voici toutes les étapes que nous allons suivre ensemble :

---

### 🎯 **ÉTAPE 1 : Vérifier les secrets (Environnement)**

**Fichier :** development.env

**Objectif :** S'assurer que nos secrets JWT sont sécurisés avant de commencer

**Ce qu'on va faire :**

- Vérifier que JWT_SECRET et JWT_REFRESH_SECRET existent
- Optionnel : Les renforcer si trop simples

**Pourquoi ?** Avant de configurer @fastify/jwt, il faut que les secrets soient en place et sécurisés.

---

### 🔧 **ÉTAPE 2 : Installer et configurer @fastify/jwt (Configuration centrale)**

**Fichier :** server.ts
**Objectif :** Enregistrer le plugin @fastify/jwt et créer les décorateurs

**Ce qu'on va faire :**

- Ajouter import fastifyJwt from '@fastify/jwt'
- Enregistrer le plugin pour les access tokens
- Enregistrer le plugin pour les refresh tokens (namespace séparé)
- Créer le hook d'authentification fastify.authenticate

**Pourquoi ?** C'est le cœur de la migration. Une fois configuré ici, tous les autres fichiers pourront utiliser fastify.jwt.sign() et fastify.jwt.verify().

---

### 🔐 **ÉTAPE 3 : Migrer la route de login**

**Fichier :** login.route.ts

**Objectif :** Remplacer jwt.sign() par fastify.jwt.sign()
**Ce qu'on va faire :**

- Supprimer `import jwt from 'jsonwebtoken'`
- Remplacer jwt.sign(payload, JWT_SECRET, options) par fastify.jwt.sign(payload)
- Remplacer jwt.sign(payload, JWT_REFRESH_SECRET, options) par fastify.refresh.sign(payload)

**Pourquoi ?** C'est là que les tokens sont créés lors du login. C'est la première utilisation de notre nouvelle configuration.

---

### 🔄 **ÉTAPE 4 : Migrer la route de refresh token**

**Fichier :** [refresh-token.route.ts](vscode-file://vscode-app/Applications/Visual%20Studio%20Code%20-%20Insiders.app/Contents/Resources/app/out/vs/code/electron-browser/workbench/workbench.html)

**Objectif :** Remplacer [jwt.verify()](vscode-file://vscode-app/Applications/Visual%20Studio%20Code%20-%20Insiders.app/Contents/Resources/app/out/vs/code/electron-browser/workbench/workbench.html) et [jwt.sign()](vscode-file://vscode-app/Applications/Visual%20Studio%20Code%20-%20Insiders.app/Contents/Resources/app/out/vs/code/electron-browser/workbench/workbench.html) par les versions Fastify

**Ce qu'on va faire :**

- Supprimer `import jwt from 'jsonwebtoken'`
- Remplacer [jwt.verify(token, JWT_REFRESH_SECRET)](vscode-file://vscode-app/Applications/Visual%20Studio%20Code%20-%20Insiders.app/Contents/Resources/app/out/vs/code/electron-browser/workbench/workbench.html) par [await fastify.refreshVerify(token)](vscode-file://vscode-app/Applications/Visual%20Studio%20Code%20-%20Insiders.app/Contents/Resources/app/out/vs/code/electron-browser/workbench/workbench.html)
- Remplacer [jwt.sign(payload, JWT_SECRET, options)](vscode-file://vscode-app/Applications/Visual%20Studio%20Code%20-%20Insiders.app/Contents/Resources/app/out/vs/code/electron-browser/workbench/workbench.html) par [fastify.jwt.sign(payload)](vscode-file://vscode-app/Applications/Visual%20Studio%20Code%20-%20Insiders.app/Contents/Resources/app/out/vs/code/electron-browser/workbench/workbench.html)

**Pourquoi ?** Cette route vérifie les refresh tokens et génère de nouveaux access tokens. Elle utilise les deux secrets.

---

### 🌐 **ÉTAPE 5 : Migrer le plugin WebSocket**

**Fichier :** [websocket.plugin.ts](vscode-file://vscode-app/Applications/Visual%20Studio%20Code%20-%20Insiders.app/Contents/Resources/app/out/vs/code/electron-browser/workbench/workbench.html)

**Objectif :** Remplacer [jwt.verify()](vscode-file://vscode-app/Applications/Visual%20Studio%20Code%20-%20Insiders.app/Contents/Resources/app/out/vs/code/electron-browser/workbench/workbench.html) dans la fonction `verifyJwtFromCookie`

**Ce qu'on va faire :**

- Supprimer `import jwt from 'jsonwebtoken'`
- Remplacer [jwt.verify(token, JWT_SECRET)](vscode-file://vscode-app/Applications/Visual%20Studio%20Code%20-%20Insiders.app/Contents/Resources/app/out/vs/code/electron-browser/workbench/workbench.html) par [await fastify.jwt.verify(token)](vscode-file://vscode-app/Applications/Visual%20Studio%20Code%20-%20Insiders.app/Contents/Resources/app/out/vs/code/electron-browser/workbench/workbench.html)
- Gérer la nature asynchrone de la nouvelle méthode

**Pourquoi ?** Les WebSockets utilisent aussi les tokens pour l'authentification. Il faut qu'ils utilisent la même configuration.

---

### 🧹 **ÉTAPE 6 : Nettoyer [package.json](vscode-file://vscode-app/Applications/Visual%20Studio%20Code%20-%20Insiders.app/Contents/Resources/app/out/vs/code/electron-browser/workbench/workbench.html)**

**Fichier :** [package.json](vscode-file://vscode-app/Applications/Visual%20Studio%20Code%20-%20Insiders.app/Contents/Resources/app/out/vs/code/electron-browser/workbench/workbench.html)

**Objectif :** Supprimer les dépendances inutilisées

**Ce qu'on va faire :**

- Supprimer `"jsonwebtoken": "^9.0.2"` des dependencies
- Supprimer `"@types/jsonwebtoken": "^9.0.9"` des devDependencies
- Exécuter `pnpm install` pour nettoyer

**Pourquoi ?** Une fois la migration terminée, ces packages ne sont plus nécessaires. On garde le projet propre.

---

### ✅ **ÉTAPE 7 : Tester l'application**

**Fichiers :** Tous

**Objectif :** Vérifier que tout fonctionne correctement

**Ce qu'on va faire :**

- Démarrer le serveur
- Tester le login
- Tester une route protégée
- Tester le refresh token
- Tester les WebSockets

### Récapitulatif

```bash
ÉTAPE 1: development.env
         ↓ (Vérifier secrets)
         
ÉTAPE 2: server.ts
         ↓ (Configuration centrale @fastify/jwt)
         
ÉTAPE 3: login.route.ts
         ↓ (Création des tokens)
         
ÉTAPE 4: refresh-token.route.ts
         ↓ (Vérification + création)
         
ÉTAPE 5: websocket.plugin.ts
         ↓ (Vérification dans WebSocket)
         
ÉTAPE 6: package.json
         ↓ (Nettoyage)
         
ÉTAPE 7: Tests
         ✅ (Validation complète)
```
