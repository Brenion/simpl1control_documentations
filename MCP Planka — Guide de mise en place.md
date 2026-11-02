# 🎉 MCP Planka — Guide de mise en place

> **Objectif** : Connecter Claude Desktop à Planka via un serveur MCP custom, sécurisé par Cloudflare Access, tournant sur TrueNAS Scale.

---

## Architecture finale

```
Claude Desktop (Mac)
    └── mcp-remote (npx)
        └── https://mcp-planka.brenion-digit.com  ← Cloudflare Access (Service Token)
            └── Container Docker (TrueNAS Scale)
                └── http://192.168.0.167:30034  ← Planka (interne, sans Cloudflare)
```

---

## Prérequis

- TrueNAS Scale avec Planka installé via le store
- Cloudflare avec Zero Trust activé
- Claude Desktop sur Mac avec Node.js installé
- Accès au shell TrueNAS (web ou SSH)

---

## Étape 1 — Générer le token API Planka

1. Ouvrir Planka dans le navigateur
2. Cliquer sur **ton avatar** → **Edit profile**
3. Section **API access tokens** → **Add token**
4. Nommer le token (ex: `claude-mcp`) et copier la clé

> ⚠️ Note : Ce token n'est pas utilisé directement — Planka utilise en réalité une auth email/password pour les requêtes API machine-to-machine.

---

## Étape 2 — Créer le dataset TrueNAS

1. **Storage** → ton pool principal
2. **Add Dataset** → nom : `mcp-planka`
3. **Preset** : Generic
4. **No** pour l'ACL Manager

---

## Étape 3 — Créer l'app Docker dans TrueNAS

Dans **Apps** → **Discover Apps** → **Custom App** :

|Champ|Valeur|
|---|---|
|Application Name|`mcp-planka`|
|Image Repository|`node`|
|Image Tag|`20-alpine`|
|Container Port|`3333`|
|Host Port|`3333`|
|Protocol|`TCP`|

**Storage Configuration :**

- Type : `ixVolume`
- Dataset Name : `mcp-planka`
- Mount Path : `/app`

**Environment Variables :**

|Variable|Valeur|
|---|---|
|`PLANKA_EMAIL`|ton email Planka|
|`PLANKA_PASSWORD`|ton mot de passe Planka|

**Entrypoint :** `node`  
**Command :** `/app/mpc-planka/server.js`

---

## Étape 4 — Créer le serveur MCP

Trouver le path du volume :

```bash
find /mnt -name "mcp-planka" 2>/dev/null
# Résultat : /mnt/.ix-apps/app_mounts/mcp-planka/mpc-planka
```

Créer le fichier :

```bash
sudo nano /mnt/.ix-apps/app_mounts/mcp-planka/mpc-planka/server.js
```

Contenu du `server.js` :

```javascript
const http = require('http');

const PLANKA_EMAIL = process.env.PLANKA_EMAIL;
const PLANKA_PASSWORD = process.env.PLANKA_PASSWORD;

const state = { token: null };

function authenticate() {
  return new Promise((resolve, reject) => {
    const body = JSON.stringify({ emailOrUsername: PLANKA_EMAIL, password: PLANKA_PASSWORD });
    const options = {
      hostname: '192.168.0.167',  // IP interne TrueNAS
      port: 30034,                 // Port Planka
      path: '/api/access-tokens',
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Content-Length': Buffer.byteLength(body)
      }
    };
    const req = http.request(options, (res) => {
      let data = '';
      res.on('data', chunk => data += chunk);
      res.on('end', () => {
        try {
          const parsed = JSON.parse(data);
          state.token = parsed.item;
          console.log('Token:', state.token ? 'OK' : 'FAILED');
          resolve();
        } catch(e) { reject(new Error('Auth failed: ' + data)); }
      });
    });
    req.on('error', reject);
    req.write(body);
    req.end();
  });
}

function plankaRequest(method, path, body) {
  return new Promise((resolve, reject) => {
    const bodyStr = body ? JSON.stringify(body) : null;
    const options = {
      hostname: '192.168.0.167',
      port: 30034,
      path,
      method,
      headers: {
        'Authorization': `Bearer ${state.token}`,
        'Content-Type': 'application/json',
        ...(bodyStr ? { 'Content-Length': Buffer.byteLength(bodyStr) } : {})
      }
    };
    const req = http.request(options, (res) => {
      let data = '';
      res.on('data', chunk => data += chunk);
      res.on('end', () => {
        try { resolve(JSON.parse(data)); }
        catch(e) { reject(new Error('Invalid JSON: ' + data)); }
      });
    });
    req.on('error', reject);
    if (bodyStr) req.write(bodyStr);
    req.end();
  });
}

function sendJsonRpc(res, id, result) {
  const body = JSON.stringify({ jsonrpc: '2.0', id, result });
  res.writeHead(200, { 'Content-Type': 'application/json', 'Access-Control-Allow-Origin': '*' });
  res.end(body);
}

function sendError(res, id, code, message) {
  const body = JSON.stringify({ jsonrpc: '2.0', id, error: { code, message } });
  res.writeHead(200, { 'Content-Type': 'application/json', 'Access-Control-Allow-Origin': '*' });
  res.end(body);
}

setInterval(authenticate, 23 * 60 * 60 * 1000); // Re-auth toutes les 23h

const server = http.createServer(async (req, res) => {
  res.setHeader('Access-Control-Allow-Origin', '*');
  res.setHeader('Access-Control-Allow-Methods', 'GET, POST, OPTIONS');
  res.setHeader('Access-Control-Allow-Headers', 'Content-Type');

  if (req.method === 'OPTIONS') { res.writeHead(200); res.end(); return; }

  if (req.method === 'GET' && req.url === '/health') {
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({ status: 'ok', authenticated: !!state.token }));
    return;
  }

  if (req.method !== 'POST') { res.writeHead(405); res.end(); return; }

  let body = '';
  req.on('data', chunk => body += chunk);
  req.on('end', async () => {
    let rpc;
    try { rpc = JSON.parse(body); }
    catch(e) { res.writeHead(400); res.end(); return; }

    const { id, method, params } = rpc;

    try {
      if (method === 'initialize') {
        return sendJsonRpc(res, id, {
          protocolVersion: '2024-11-05',
          serverInfo: { name: 'planka-mcp', version: '1.0.0' },
          capabilities: { tools: {} }
        });
      }

      if (method === 'tools/list') {
        return sendJsonRpc(res, id, {
          tools: [
            {
              name: 'get_projects',
              description: 'Récupère tous les projets Planka',
              inputSchema: { type: 'object', properties: {} }
            },
            {
              name: 'get_board',
              description: 'Récupère les listes et cartes d\'un board',
              inputSchema: {
                type: 'object',
                properties: { boardId: { type: 'string', description: 'ID du board' } },
                required: ['boardId']
              }
            },
            {
              name: 'create_card',
              description: 'Crée une carte dans une liste',
              inputSchema: {
                type: 'object',
                properties: {
                  listId: { type: 'string', description: 'ID de la liste' },
                  name: { type: 'string', description: 'Nom de la carte' },
                  description: { type: 'string', description: 'Description' }
                },
                required: ['listId', 'name']
              }
            }
          ]
        });
      }

      if (method === 'tools/call') {
        const { name, arguments: args } = params;

        if (name === 'get_projects') {
          const data = await plankaRequest('GET', '/api/projects');
          return sendJsonRpc(res, id, { content: [{ type: 'text', text: JSON.stringify(data, null, 2) }] });
        }

        if (name === 'get_board') {
          const data = await plankaRequest('GET', `/api/boards/${args.boardId}`);
          return sendJsonRpc(res, id, { content: [{ type: 'text', text: JSON.stringify(data, null, 2) }] });
        }

        if (name === 'create_card') {
          const data = await plankaRequest('POST', `/api/lists/${args.listId}/cards`, {
            name: args.name,
            description: args.description || ''
          });
          return sendJsonRpc(res, id, { content: [{ type: 'text', text: JSON.stringify(data, null, 2) }] });
        }

        return sendError(res, id, -32601, 'Tool not found');
      }

      return sendError(res, id, -32601, 'Method not found');

    } catch(err) {
      return sendError(res, id, -32000, err.message);
    }
  });
});

authenticate().then(() => {
  server.listen(3333, () => console.log('MCP Planka server running on port 3333'));
}).catch(err => {
  console.error('Failed:', err.message);
  process.exit(1);
});
```

---

## Étape 5 — Exposer via Cloudflare Tunnel

Dans le connector TrueNAS de **Cloudflare Zero Trust** → **Networks** → **Connectors** → **truenas** :

Ajouter une **Published Application** :

|Champ|Valeur|
|---|---|
|Subdomain|`mcp-planka`|
|Domain|`brenion-digit.com`|
|Service|`HTTP`|
|URL|`192.168.0.167:3333`|

---

## Étape 6 — Sécuriser avec Cloudflare Access

### Créer un Service Token

**Zero Trust** → **Access** → **Service Auth** → **Service Tokens** → **Create Service Token** :

- Nom : `mcp-planka`
- Duration : **Non-expiring**
- Copier le `Client ID` et `Client Secret`

### Créer une Application Access

**Zero Trust** → **Access** → **Applications** → **Add application** → **Self-hosted** :

- Name : `mcp-planka`
- Domain : `mcp-planka.brenion-digit.com`
- Policy : **Service Auth** avec le token `mcp-planka`

---

## Étape 7 — Connecter à Claude Desktop

Dans `~/Library/Application Support/Claude/claude_desktop_config.json` :

```json
{
  "preferences": { ... },
  "mcpServers": {
    "planka": {
      "command": "npx",
      "args": [
        "mcp-remote",
        "https://mcp-planka.brenion-digit.com",
        "--header",
        "CF-Access-Client-Id:TON_CLIENT_ID.access",
        "--header",
        "CF-Access-Client-Secret:TON_CLIENT_SECRET"
      ]
    }
  }
}
```

Redémarrer Claude Desktop avec **Cmd+Q** puis réouvrir.

---

## Vérification

```bash
# Test local
curl http://localhost:3333/health
# {"status":"ok","authenticated":true}

# Test via Cloudflare
curl https://mcp-planka.brenion-digit.com/health \
  -H "CF-Access-Client-Id: TON_CLIENT_ID" \
  -H "CF-Access-Client-Secret: TON_CLIENT_SECRET"
# {"status":"ok","authenticated":true}

# Test sans auth (doit retourner 403)
curl https://mcp-planka.brenion-digit.com/health
# Forbidden - Cloudflare Access
```

---

## Logs utiles

```bash
# Logs du container MCP
docker logs ix-mcp-planka-mcp-planka-1 --tail 20

# Logs Claude Desktop
tail -f ~/Library/Logs/Claude/mcp-server-planka.log
```

---

## Ce qui reste à faire

- [ ] Ajouter les outils manquants : `update_card`, `delete_card`, `create_list`, `update_list`, etc.
- [ ] Implémenter la gestion complète des boards, listes et cartes

---

## Points clés

- L'auth Planka se renouvelle automatiquement toutes les **23h** — aucune intervention manuelle
- Le MCP communique avec Planka en **interne** (`192.168.0.167:30034`) — pas de double auth Cloudflare
- Le MCP est protégé par **Cloudflare Access Service Token** — inaccessible sans les headers CF
- Ta double auth personnelle sur `planka.brenion-digit.com` n'est **pas affectée**