## ÉTAPE 4 : Route Refresh Token avec @fastify/jwt

### Fichier
`refresh-token.route.ts` - Renouvellement des access tokens

### Import
```typescript
import { FastifyInstance, RouteShorthandOptions } from 'fastify';
import { Repository } from 'typeorm';
```

### Vérification du refresh token (asynchrone)
```typescript
let payload;
try {
    payload = await fastify.jwt.refresh.verify(refreshToken);
} catch (error) {
    return reply.status(401).send({ error: 'Invalid refresh token' });
}
```
**Raison :** Utilise JWT_REFRESH_SECRET configuré dans server.ts. **Important :** `await` obligatoire car asynchrone.

### Création d'un nouveau access token
```typescript
const accessToken = fastify.jwt.sign({
    id: user.id,
    firstname: user.firstname,
    lastname: user.lastname,
    mail: user.mail,
    role: user.role
});
```
**Raison :** Utilise JWT_SECRET + expiration 1h de server.ts

### Principe
- `fastify.jwt.refresh.verify()` → Vérifie refresh token avec JWT_REFRESH_SECRET
- `fastify.jwt.sign()` → Crée nouveau access token avec JWT_SECRET
- Méthode asynchrone nécessite `await`

### Flux
1. Reçoit refresh token du client
2. Le vérifie (await refreshVerify)
3. Récupère l'utilisateur en DB
4. Génère nouveau access token
5. Le renvoie au client

### Résultat
Renouvellement automatique des tokens avec configuration centralisée.