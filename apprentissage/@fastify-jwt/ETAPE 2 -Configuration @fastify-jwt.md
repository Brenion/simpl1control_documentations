##  ÉTAPE 2 : Configuration @fastify/jwt

### Fichier modifié

[server.ts](vscode-file://vscode-app/Applications/Visual%20Studio%20Code%20-%20Insiders.app/Contents/Resources/app/out/vs/code/electron-browser/workbench/workbench.html) - Configuration centrale JWT

### Modifications

**1. Import (ligne 2)**

```ts
import fastifyJwt from '@fastify/jwt';
```

**2. Plugin Access Token (après création serveur)**

```ts
await server.register(fastifyJwt, {
  secret: process.env.JWT_SECRET!,
  sign: { expiresIn: '1h' }
});
```

**Raison :** Crée [fastify.jwt.sign()](vscode-file://vscode-app/Applications/Visual%20Studio%20Code%20-%20Insiders.app/Contents/Resources/app/out/vs/code/electron-browser/workbench/workbench.html) et [fastify.jwt.verify()](vscode-file://vscode-app/Applications/Visual%20Studio%20Code%20-%20Insiders.app/Contents/Resources/app/out/vs/code/electron-browser/workbench/workbench.html) pour tokens courte durée

**3. Plugin Refresh Token (namespace)**

```ts
await server.register(fastifyJwt, {
  secret: process.env.JWT_REFRESH_SECRET!,
  namespace: 'refresh',
  sign: { expiresIn: '7d' }
});
```

**Raison :** Crée fastify.jwt.refresh.sign() et [fastify.refreshVerify()](vscode-file://vscode-app/Applications/Visual%20Studio%20Code%20-%20Insiders.app/Contents/Resources/app/out/vs/code/electron-browser/workbench/workbench.html) pour tokens longue durée

**4. Hook d'authentification**

```ts
server.decorate('authenticate', async function(request, reply) {
  try {
    await request.jwtVerify();
  } catch (err) {
    reply.status(401).send({ error: 'Unauthorized' });
  }
});
```

**Raison :** Protège les routes avec [onRequest: [fastify.authenticate]](vscode-file://vscode-app/Applications/Visual%20Studio%20Code%20-%20Insiders.app/Contents/Resources/app/out/vs/code/electron-browser/workbench/workbench.html)

### Résultat

Les décorateurs JWT sont maintenant disponibles partout dans l'application :

- [fastify.jwt.sign()](vscode-file://vscode-app/Applications/Visual%20Studio%20Code%20-%20Insiders.app/Contents/Resources/app/out/vs/code/electron-browser/workbench/workbench.html) / [fastify.jwt.verify()](vscode-file://vscode-app/Applications/Visual%20Studio%20Code%20-%20Insiders.app/Contents/Resources/app/out/vs/code/electron-browser/workbench/workbench.html)
- [fastify.refresh.sign()](vscode-file://vscode-app/Applications/Visual%20Studio%20Code%20-%20Insiders.app/Contents/Resources/app/out/vs/code/electron-browser/workbench/workbench.html) / [fastify.refreshVerify()](vscode-file://vscode-app/Applications/Visual%20Studio%20Code%20-%20Insiders.app/Contents/Resources/app/out/vs/code/electron-browser/workbench/workbench.html)
- [fastify.authenticate](vscode-file://vscode-app/Applications/Visual%20Studio%20Code%20-%20Insiders.app/Contents/Resources/app/out/vs/code/electron-browser/workbench/workbench.html)