# Prompt de reprise (résumé compact pour continuer dans une nouvelle section)
Tu es un·e dev/ops qui finalise la mise en production d’une app web sur **Raspberry Pi** (Raspberry OS 64‑bit) dans un **réseau fermé** (pas d’Internet). Architecture : **frontend React (Vite)** servi par **Nginx** sur `:80`, **backend Node** sur `:3001` (Fastify, WebSocket sous `/ws`), **PostgreSQL 15** sur `:5432`.

Le Pi expose **uniquement le front en :80** ; le back et la DB restent dans le réseau Docker.
Le **registre privé Docker** vit sur le Pi en **TLS** à `192.168.3.100:5000`.
On construit les images **sur une machine avec Internet** (Mac Apple Silicon ARM64), on exporte en `.tar`, on bascule sur le réseau fermé, on **load** et on **push** sur le registre du Pi via `docker --context tfe`, puis on déploie avec **docker compose** (fichier dans `production/`).

Versionning : schéma `vX.Y.Z`. Un script `production/tag.sh` enregistre la version dans `production/VERSION`, met à jour les tags d’images dans `production/docker-compose.production.yml`, et *optionnellement* bump les `package.json` (sans le `v`).

WebSocket : le front utilise **react-websocket** en **même origine** sur `ws(s)://<host>/ws` ; **Nginx** proxifie `/ws` vers `backend:3001/ws` avec les bons en‑têtes `Upgrade/Connection`.

Points déjà traités : ports (`:80` parfois occupé), `production.env` injecté via `env_file` (ne pas lire un fichier depuis le code), CORS évité grâce au proxy Nginx, build ARM64, push vers registre local, vérification DB avec `psql`, seeders exécutable via `compose run` ou service profilé.

---

# Arborescence cible
```
<repo>/
├─ packages/
│  ├─ backend/
│  │  ├─ Dockerfile
│  │  └─ (src, dist, rollup, etc.)
│  └─ frontend/
│     ├─ Dockerfile
│     └─ nginx.conf
└─ production/
   ├─ docker-compose.production.yml
   ├─ production.env
   ├─ db.env
   ├─ VERSION
   ├─ tag.sh
   ├─ build.sh
   ├─ push_deploy.sh
   └─ (artifacts/*.tar générés)
```

---

# Fichiers finaux (définitifs)

## 1) `production/docker-compose.production.yml`
```yaml
# production/docker-compose.production.yml
# Compose v2 : pas de clé 'version:'
name: myapp

services:
  db:
    image: 192.168.3.100:5000/postgres:15-alpine
    env_file:
      - ./db.env
    volumes:
      - dbdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U $$POSTGRES_USER -d $$POSTGRES_DB -h 127.0.0.1"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 15s
    restart: always
    networks: [appnet]

  backend:
    image: 192.168.3.100:5000/backend:vX.Y.Z  # ← tag injecté par production/tag.sh
    env_file:
      - ./production.env
    environment:
      NODE_ENV: production
      PORT: 3001
    depends_on:
      db:
        condition: service_healthy
    restart: always
    networks: [appnet]

  frontend:
    image: 192.168.3.100:5000/frontend:vX.Y.Z  # ← tag injecté par production/tag.sh
    depends_on:
      - backend
    ports:
      - "80:80"  # seul point d'entrée LAN
    restart: always
    networks: [appnet]

  # (optionnel) seeder — à lancer avec --profile seed
  seeder:
    image: 192.168.3.100:5000/backend:vX.Y.Z
    depends_on:
      db:
        condition: service_healthy
    env_file:
      - ./production.env
    command: ["node", "dist/seed.js"]
    restart: "no"
    profiles: ["seed"]
    networks: [appnet]

volumes:
  dbdata:

networks:
  appnet:
    driver: bridge
```

---

## 2) `packages/backend/Dockerfile`
```dockerfile
# packages/backend/Dockerfile
FROM node:20-alpine AS build
WORKDIR /app
RUN npm i -g pnpm
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml ./
RUN pnpm install --frozen-lockfile
COPY . .
# Build via rollup : dist/server.js & dist/seed.js
RUN pnpm build

FROM node:20-alpine AS production
WORKDIR /app
ENV NODE_ENV=production
RUN npm i -g pnpm
COPY --from=build /app/package.json /app/pnpm-lock.yaml ./
RUN pnpm install --prod --frozen-lockfile
COPY --from=build /app/dist ./dist
COPY --from=build /app/certs ./certs
EXPOSE 3001
# ⚠️ adapte la route si nécessaire
HEALTHCHECK --interval=10s --timeout=3s --start-period=15s --retries=3 \
  CMD wget -qO- http://127.0.0.1:3001/health || exit 1
CMD ["node","dist/server.js"]
```

---

## 3) `packages/frontend/Dockerfile`
```dockerfile
# packages/frontend/Dockerfile
FROM node:20-alpine AS build
WORKDIR /app
RUN npm i -g pnpm
COPY package.json pnpm-lock.yaml ./
RUN pnpm install --frozen-lockfile
COPY . .
# En prod, le front parle à /api (proxy nginx) et /ws en same-origin
ENV VITE_API_BASE_URL=/api
RUN pnpm build

FROM nginx:alpine AS production
WORKDIR /usr/share/nginx/html
RUN rm -rf ./*
COPY --from=build /app/dist .
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
HEALTHCHECK --interval=10s --timeout=3s --start-period=5s --retries=3 \
  CMD wget -qO- http://127.0.0.1/ || exit 1
```

---

## 4) `packages/frontend/nginx.conf`
```nginx
# packages/frontend/nginx.con.tamap $http_upgrade $connection_upgrade { default upgrade; '' close; }

server {
  listen 80;
  server_name _;

  root /usr/share/nginx/html;
  index index.html;

  # SPA
  location / { try_files $uri /index.html; }

  # API HTTP -> backend
  location /api/ {
    proxy_pass http://backend:3001/;
    proxy_http_version 1.1;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_read_timeout 3600s;
  }

  # WebSocket (vanilla)
  location /ws/ {
    proxy_pass http://backend:3001/ws/;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $connection_upgrade;
    proxy_set_header Host $host;
    proxy_read_timeout 3600s;
  }
}
```

---

## 5) `production/tag.sh`
```sh
#!/usr/bin/env sh
# usage: ./production/tag.sh vX.Y.Z
set -e

VERSION_FILE="./production/VERSION"
COMPOSE_FILE="./production/docker-compose.production.yml"
REG="192.168.3.100:5000"

if [ -z "$1" ]; then
  echo "❌ Tu dois donner une version (ex: ./production/tag.sh v1.2.3)"
  exit 1
fi
NEW_VERSION="$1"

echo "$NEW_VERSION" | grep -Eq '^v[0-9]+\.[0-9]+\.[0-9]+$' || {
  echo "❌ Mauvais format. Attendu: vX.Y.Z (ex: v1.2.3)"; exit 1; }

if [ -f "$VERSION_FILE" ]; then
  CURRENT_VERSION=$(cat "$VERSION_FILE")
  [ "$NEW_VERSION" = "$CURRENT_VERSION" ] && {
    echo "❌ La version $NEW_VERSION est déjà enregistrée"; exit 1; }
fi

echo "$NEW_VERSION" > "$VERSION_FILE"
echo "✅ VERSION enregistrée → $NEW_VERSION"

# Remplace toute ancienne valeur après backend:/frontend:
sed -i.bak -E "s|(image:[[:space:]]*${REG}/backend:)[^"'[:space:]]+|\\1${NEW_VERSION}|g"  "$COMPOSE_FILE"
sed -i.bak -E "s|(image:[[:space:]]*${REG}/frontend:)[^"'[:space:]]+|\\1${NEW_VERSION}|g" "$COMPOSE_FILE"
rm -f "$COMPOSE_FILE.bak"
echo "✅ $COMPOSE_FILE mis à jour → $NEW_VERSION"

# (Optionnel) bump package.json (sans le 'v')
NUM_VERSION="${NEW_VERSION#v}"
bump_pkg() {
  PKG_JSON="$1/package.json"; [ -f "$PKG_JSON" ] || return 0
  node -e "const fs=require('fs');const p='$PKG_JSON';const j=JSON.parse(fs.readFileSync(p,'utf8'));j.version='${NUM_VERSION}';fs.writeFileSync(p, JSON.stringify(j,null,2));console.log('✅ bump',p,'→ ${NUM_VERSION}');"
}
# adapte selon ta structure
bump_pkg . || true
bump_pkg ./packages/backend || true
bump_pkg ./packages/frontend || true

echo "🎉 Tagging terminé."
```

---

## 6) `production/build.sh`
```sh
#!/usr/bin/env sh
# Build images (ARM64) + export .tar
# Usage: ./production/build.sh
set -e

SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
ROOT_DIR="$(cd "${SCRIPT_DIR}/.." && pwd)"
VERSION_FILE="${ROOT_DIR}/production/VERSION"
ARTIFACTS_DIR="${ROOT_DIR}/production/artifacts"

BACK_CTX="${ROOT_DIR}/packages/backend"
FRONT_CTX="${ROOT_DIR}/packages/frontend"
BACK_DOCKERFILE="${BACK_CTX}/Dockerfile"
FRONT_DOCKERFILE="${FRONT_CTX}/Dockerfile"

REG="${REG:-192.168.3.100:5000}"

[ -f "${VERSION_FILE}" ] || { echo "❌ ${VERSION_FILE} introuvable. Lance ./production/tag.sh vX.Y.Z"; exit 1; }
VERSION="$(tr -d '\r\n' < "${VERSION_FILE}")"
[ -f "${BACK_DOCKERFILE}" ]  || { echo "❌ Dockerfile backend introuvable"; exit 1; }
[ -f "${FRONT_DOCKERFILE}" ] || { echo "❌ Dockerfile frontend introuvable"; exit 1; }

mkdir -p "${ARTIFACTS_DIR}"

echo "ℹ️  VERSION = ${VERSION}"
echo "ℹ️  REG     = ${REG}"

docker buildx create --name prod_builder --use >/dev/null 2>&1 || true
docker buildx inspect --bootstrap >/dev/null 2>&1 || true

echo "▶️  Build backend (linux/arm64)…"
docker buildx build --platform linux/arm64 \
  -t "${REG}/backend:${VERSION}" \
  -f "${BACK_DOCKERFILE}" \
  --load \
  "${BACK_CTX}"

echo "▶️  Build frontend (linux/arm64)…"
docker buildx build --platform linux/arm64 \
  -t "${REG}/frontend:${VERSION}" \
  -f "${FRONT_DOCKERFILE}" \
  --load \
  "${FRONT_CTX}"

echo "▶️  Prépare postgres:15-alpine (linux/arm64)…"
docker pull --platform linux/arm64 postgres:15-alpine
docker tag postgres:15-alpine "${REG}/postgres:15-alpine"

echo "💾 Export des images…"
docker save "${REG}/backend:${VERSION}"  -o "${ARTIFACTS_DIR}/backend_${VERSION}.tar"
docker save "${REG}/frontend:${VERSION}" -o "${ARTIFACTS_DIR}/frontend_${VERSION}.tar"
docker save "${REG}/postgres:15-alpine"  -o "${ARTIFACTS_DIR}/postgres_15-alpine.tar"

echo "✅ Build terminé. Artifacts dans ${ARTIFACTS_DIR}"
```

---

## 7) `production/push_deploy.sh`
```sh
#!/usr/bin/env sh
# Charge les .tar sur le daemon du Pi (context tfe), push vers le registre local, et déploie avec compose
# Usage: ./production/push_deploy.sh
set -e

SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
ROOT_DIR="$(cd "${SCRIPT_DIR}/.." && pwd)"
VERSION_FILE="${ROOT_DIR}/production/VERSION"
ARTIFACTS_DIR="${ROOT_DIR}/production/artifacts"
COMPOSE_FILE="${ROOT_DIR}/production/docker-compose.production.yml"
REG="${REG:-192.168.3.100:5000}"

[ -f "${VERSION_FILE}" ] || { echo "❌ ${VERSION_FILE} introuvable"; exit 1; }
VERSION="$(tr -d '\r\n' < "${VERSION_FILE}")"

for f in "backend_${VERSION}.tar" "frontend_${VERSION}.tar" "postgres_15-alpine.tar"; do
  [ -f "${ARTIFACTS_DIR}/$f" ] || { echo "❌ Manque ${ARTIFACTS_DIR}/$f (as-tu lancé build?)"; exit 1; }
done

echo "▶️  Load images dans le daemon du Pi (context tfe)…"
docker --context tfe load -i "${ARTIFACTS_DIR}/backend_${VERSION}.tar"
docker --context tfe load -i "${ARTIFACTS_DIR}/frontend_${VERSION}.tar"
docker --context tfe load -i "${ARTIFACTS_DIR}/postgres_15-alpine.tar"

echo "▶️  Push vers registre ${REG}…"

 "${REG}/backend:${VERSION}"
docker --context tfe push "${REG}/frontend:${VERSION}"
docker --context tfe push "${REG}/cp"

echo "▶️  Déploiement compose…"
docker --context tfe compose -f "${COMPOSE_FILE}" up -d

echo "✅ Déploiement terminé."
docker --context tfe ps
```

---

# Résumé des lignes de commande (workflow complet)

1) **Choisir/imbriquer la version** (met à jour compose + `package.json`) :
```bash
./production/tag.sh v1.0.0
```
2) **Construire les images (sur machine avec Internet) & exporter** :
```bash
./production/build.sh
```
3) **Basculer sur le réseau fermé, charger/pusher/déployer** :
```bash
docker context use tfe
./production/push_deploy.sh
```
4) **Accéder à l’app** : `http://192.168.3.100/`

5) **Logs / Diagnostique** :
```bash
docker --context tfe ps
docker --context tfe logs backend --tail=100
docker --context tfe logs frontend --tail=100
```
6) **Seeder (optionnel, une fois)** :
```bash
# DB seule, puis seeder via profile
docker --context tfe compose -f production/docker-compose.production.yml up -d db
docker --context tfe compose -f production/docker-compose.production.yml --profile seed up --abort-on-container-exit seeder
# App normale
docker --context tfe compose -f production/docker-compose.production.yml up -d
```
7) **psql rapide (lire une table)** :
```bash
docker --context tfe ps --format '{{.Names}}\t{{.Image}}' | grep postgres
# puis
docker --context tfe exec -it <NOM_CONTENEUR_DB> sh -lc 'PGPASSWORD="$POSTGRES_PASSWORD" psql -U "$POSTGRES_USER" -d "$POSTGRES_DB" -c "SELECT * FROM public.users LIMIT 50;"'
```

---

*Fin du récap — tu peux repartir d’ici pour poursuivre la configuration, les scripts, ou le troubleshooting.*

