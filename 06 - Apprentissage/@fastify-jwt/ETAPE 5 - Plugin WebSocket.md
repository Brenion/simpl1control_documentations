## ÉTAPE 5 : Plugin WebSocket avec @fastify/jwt

### Fichiers modifiés

#### 1. `websocket.plugin.ts` - Vérification JWT dans les WebSockets

### Import
```typescript
import websocket from "@fastify/websocket";
import type { FastifyInstance } from "fastify";
import fp from "fastify-plugin";
import logger from "../utils/logger.js";
// Pas d'import JWT nécessaire
```

### Fonction de vérification JWT (asynchrone)
```typescript
fastify.decorate("verifyJwtFromCookie", async (req: any) => {
  try {
    const cookieHeader = req.headers?.cookie || "";
    const m = cookieHeader.match(/(?:^|; )access_token=([^;]+)/);
    const token = m?.[1];
    if (!token) return null;

    const payload: any = await fastify.jwt.verify(token);
    const userId = String(payload?.id ?? payload?.sub ?? "");
    if (!userId) return null;

    return { userId, ...payload };
  } catch {
    return null;
  }
});
```
**Raison :** Vérifie les tokens JWT depuis les cookies pour les connexions WebSocket. Utilise JWT_SECRET de server.ts.

#### 2. `types/web-socket.d.ts` - Déclaration TypeScript

```typescript
verifyJwtFromCookie: (req: any) => Promise<null | { userId: string; [k: string]: any }>;
```
**Raison :** Type mis à jour pour refléter le caractère asynchrone de la fonction (retourne une Promise).

### Principe
- `fastify.jwt.verify()` → Vérifie access token avec JWT_SECRET
- Fonction asynchrone nécessite `async/await`
- Types TypeScript synchronisés avec l'implémentation

### Utilisation
```typescript
// Dans une route WebSocket
const user = await fastify.verifyJwtFromCookie(request);
if (!user) {
  // Accès refusé
}
```

### Résultat
Authentification JWT dans les WebSockets avec configuration centralisée et types corrects.