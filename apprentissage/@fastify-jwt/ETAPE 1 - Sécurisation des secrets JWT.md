### Commandes

```
# DEV (32 chars)
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"

# PROD (64 chars)
node -e "console.log(require('crypto').randomBytes(64).toString('hex'))"
```

### Fichiers modifiés

|Fichier|Action|Raison|
|---|---|---|
|[test.env](vscode-file://vscode-app/Applications/Visual%20Studio%20Code%20-%20Insiders.app/Contents/Resources/app/out/vs/code/electron-browser/workbench/workbench.html)|❌ Aucune|Tests locaux = secrets simples OK|
|[development.env](vscode-file://vscode-app/Applications/Visual%20Studio%20Code%20-%20Insiders.app/Contents/Resources/app/out/vs/code/electron-browser/workbench/workbench.html)|✏️ Secrets 32 chars|Dev local = sécurité moyenne|
|[production.env](vscode-file://vscode-app/Applications/Visual%20Studio%20Code%20-%20Insiders.app/Contents/Resources/app/out/vs/code/electron-browser/workbench/workbench.html)|✏️ Secrets 64 chars|Clients réels = sécurité maximale|

### Principe clé

**Chaque environnement = secrets DIFFÉRENTS** pour isoler la compromission

### **Fichiers modifiés**

#### 1️⃣ test.env - **AUCUNE MODIFICATION**

```
JWT_SECRET=supersecretkey
JWT_REFRESH_SECRET=supersecretrefreshkey
```

**Pourquoi ?** → Environnement de test local, garder simple pour faciliter les tests automatisés

---

#### 2️⃣ development.env - **MODIFIÉ** ✏️

```
JWT_SECRET=<votre_secret_32_caractères>
JWT_REFRESH_SECRET=<votre_secret_32_caractères>
```

**Pourquoi ?** → Votre machine de développement nécessite des secrets moyens (sécurisés mais pratiques)

---

#### 3️⃣ production.env - **MODIFIÉ** ✏️

```
JWT_SECRET=<votre_secret_64_caractères>
JWT_REFRESH_SECRET=<votre_secret_64_caractères>
```


**Pourquoi ?** → Production critique avec utilisateurs réels = secrets ultra-sécurisés (maximum de caractères)

**Résultat :** Production isolée, tokens non-interchangeables entre environnements

## **Ce que vous avez appris**

1. **Séparation des environnements** : Chaque env doit avoir ses propres secrets
2. **Génération sécurisée** : Utiliser `crypto.randomBytes()` au lieu de mots simples
3. **Proportionnalité** : Plus l'environnement est critique, plus le secret doit être fort
4. **Isolation** : Compromettre le dev ne compromet pas la prod