## DEV

```bash
# 1. Lancer la DB
cd packages/backend && docker compose up -d

# 2. Lancer le backend
pnpm dev

# 3. Lancer le frontend (autre terminal)
cd packages/frontend && pnpm dev
```

Le frontend tourne sur `http://localhost:5173`, Vite proxy forward `/api` et `/ws` vers `localhost:3000` (configurable dans [packages/frontend/.env](vscode-webview://1um6jr25h1e2jmbepn9hfu3mn4ncab5fjmemvdn8m4qckp484ilu/packages/frontend/.env)).

Si le backend tourne sur une autre machine, modifie juste le `.env` :

```
VITE_DEV_PROXY_TARGET=http://192.168.X.X:3000
VITE_DEV_WS_TARGET=ws://192.168.X.X:3000
```

## TEST

```bash
# Frontend
cd packages/frontend && pnpm test

# Backend
cd packages/backend && pnpm test
```

Rien à configurer. Les tests frontend mockent les appels API, les tests backend utilisent la DB test sur `localhost:5433`.

## PRODUCTION

### 1. Configurer [production/deploy.env](vscode-webview://1um6jr25h1e2jmbepn9hfu3mn4ncab5fjmemvdn8m4qckp484ilu/production/deploy.env)

```
SERVER_IP=192.168.3.100        # ← IP du serveur client
REGISTRY=192.168.3.100:5000    # ← Registry Docker
MQTT_HOST=192.168.3.100        # ← Broker MQTT
VERSION=v1.1.2                 # ← Version à déployer
```

Pour un nouveau client, tu changes uniquement ces 3 IPs.

### 2. Build les images

```bash
./production/tag.sh v1.2.0     # bumpe la version
./production/build.sh           # build ARM64 + export .tar
```

### 3. Transférer sur le serveur

```bash
scp production/artifacts/*.tar user@192.168.X.X:/opt/myapp/
scp production/deploy.env production/docker-compose.production.yaml \
    production/production.env production/db.env user@192.168.X.X:/opt/myapp/
```

### 4. Charger les images sur le serveur

```bash
ssh user@192.168.X.X
docker load -i /opt/myapp/backend_v1.2.0.tar
docker load -i /opt/myapp/frontend_v1.2.0.tar
docker load -i /opt/myapp/postgres_15-alpine.tar
```

### 5. Lancer

```bash
docker compose --env-file deploy.env -f docker-compose.production.yaml up -d

# Si premier déploiement (seed la DB) :
docker compose --env-file deploy.env -f docker-compose.production.yaml --profile seed up seeder
```

L'app est accessible sur `http://192.168.X.X` (port 80, nginx).