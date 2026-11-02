## ÉTAPE 3 : Route de Login avec @fastify/jwt

### Fichiers modifiés

#### `types/fastify-jwt.d.ts`
Déclarations TypeScript globales pour @fastify/jwt

```typescript
import '@fastify/jwt';
import 'fastify';

declare module '@fastify/jwt' {
interface FastifyJWT {
	payload: { id: string; firstname?: string; lastname?: string; mail?: string; role?: string };
	user: { id: string; firstname?: string; lastname?: string; mail?: string; role?: string };
}
// Déclarer le namespace 'refresh' sur l'objet jwt
// Le module utilise export = fastifyJwt, donc JWT est directement dans le scope du module
interface JWT {
	refresh: JWT;
	}
}

declare module 'fastify' {
	interface FastifyInstance {
// Hook d'authentification
		authenticate: (request: any, reply: any) => Promise<void>;
	}
}
```

### Fichier

login.route.ts - Création des tokens JWT lors du login
### Import

```ts
import { FastifyInstance, RouteShorthandOptions } from 'fastify';
import { Repository } from 'typeorm';
```

**Raison :** TypeScript doit connaître les méthodes ajoutées par @fastify/jwt (refresh.sign, refreshVerify, authenticate). Fichier global = types disponibles partout sans duplication.
### Login.route.ts  - Création des tokens

**Access Token (courte durée)**

```ts
const accessToken = fastify.jwt.sign({
    id: user.id,
    firstname: user.firstname,
    lastname: user.lastname,
    mail: user.mail,
    role: user.role
});
```

**Raison :** Utilise la config de server.ts (secret + 1h expiration)

**Refresh Token (longue durée)**

```ts
const refreshToken = fastify.jwt.refresh.sign({
    id: user.id
});
```

**Raison :** Utilise le namespace `refresh` (secret différent + 7j expiration)

### Principe

- fastify.jwt.sign() → Access token avec JWT_SECRET
- fastify.jwt.refresh.sign() → Refresh token avec JWT_REFRESH_SECRET
- Configuration centralisée dans server.ts, pas de paramètres ici

### Résultat

Tokens JWT créés automatiquement avec les bons secrets et expirations sans duplication de code.