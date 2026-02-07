
cle ssh voir tout 

ssh-keygen -t ed25519
cat raspberry-tfe.pub

test ssh
cat raspberry-tfe.pub

fournir la clé au raspberry

cree un ficheir config sur la machine qui se connectera au raspberry pour facilité 

Host tfe
  Hostname 192.168.3.102
  User <VOTRE_USER>
  IdentityFile ~/.ssh/raspberry-tfe


ecrire
ssh tfe pour se connecte directement

### [Install using the `apt` repository](https://docs.docker.com/engine/install/debian/#install-using-the-repository)

Before you install Docker Engine for the first time on a new host machine, you need to set up the Docker `apt` repository. Afterward, you can install and update Docker from the repository.

1. Set up Docker's `apt` repository.
    

```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

Note

If you use a derivative distribution, such as Kali Linux, you may need to substitute the part of this command that's expected to print the version codename:

- > ```console
    > (. /etc/os-release && echo "$VERSION_CODENAME")
    > ```
    > 
    > Replace this part with the codename of the corresponding Debian release, such as `bookworm`.
    
- Install the Docker packages.
    

To install the latest version, run:

- ```console
     sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    ```
    
- Verify that the installation is successful by running the `hello-world` image:
    

```console
 sudo docker run hello-world
```


cree un registre (faut internet youhou)

$ docker run -d -p 5000:5000 --restart always --name registry registry:3
https://hub.docker.com/_/registry



# Mise en production d’une application web sur Raspberry Pi (réseau fermé)

## 0. Pré-requis (à faire une seule fois)

- Raspberry Pi sous Raspberry OS 64-bit, avec Docker et Docker Compose installés.
    
- Un registre privé Docker déjà lancé sur le Pi :
    
    ```bash
    docker run -d -p 5000:5000 --restart always --name registry registry:3
    ```
    
- Un `docker context` configuré vers le daemon du Pi (nommé `tfe`).
    
- Un routeur fermé (192.168.3.1) ; le Pi a l’adresse fixe `192.168.3.100`.
    

---

## 1. Gestion des versions

1. Créer un fichier `scripts/set-version.sh` qui gère un fichier `VERSION` à la racine du projet.
    
2. Chaque nouvelle version est enregistrée avec le format `vX.Y.Z`.
    
3. Exemple d’usage :
    
    ```bash
    ./scripts/set-version.sh v1.0.0
    ```
    
    → crée ou met à jour le fichier `VERSION`.
    

**But :** éviter d’écraser une version existante et toujours avoir la version courante sauvegardée.

---

## 2. Préparer les Dockerfiles

### Backend (`packages/backend/Dockerfile`)

- Multi-étape (build avec pnpm, puis exécution en Node 20-alpine).
    
- Ne pas inclure `production.env` dans l’image → l’injecter via `env_file`.
    

### Frontend (`packages/frontend/Dockerfile`)

- Étape 1 : build Vite (CSR) avec `VITE_API_BASE_URL=/api`.
    
- Étape 2 : image Nginx qui sert les fichiers statiques + proxy `/api` vers `backend:3001`.
    

### Base de données

- Image officielle `postgres:15-alpine` (taggée et poussée dans le registre du Pi).
    

**But :** avoir des images prêtes pour ARM64, propres et légères.

---

## 3. Préparer docker-compose.production.yml

- Trois services : `db`, `backend`, `frontend`.
    
- Réseau interne `appnet`.
    
- Seul `frontend` expose un port (`80:80`).
    
- `env_file` pour injecter les variables (`production.env` pour back, `db.env` pour Postgres).
    
- `restart: always` sur tous les services.
    

**But :** déploiement reproductible, sécurisé (seul le front est visible sur le LAN).

---

## 4. Construire les images (sur machine avec Internet)

1. Lire la version :
    
    ```bash
    VERSION=$(cat VERSION)
    REG=192.168.3.100:5000
    ```
    
2. Construire les images ARM64 :
    
    ```bash
    docker buildx build --platform linux/arm64 -t ${REG}/backend:${VERSION} -f packages/backend/Dockerfile --load packages/backend
    docker buildx build --platform linux/arm64 -t ${REG}/frontend:${VERSION} -f packages/frontend/Dockerfile --load packages/frontend
    docker pull --platform linux/arm64 postgres:15-alpine
    docker tag postgres:15-alpine ${REG}/postgres:15-alpine
    ```
    

**But :** produire les images compatibles Raspberry, avec les bons tags versionnés.

---

## 5. Exporter les images en archives `.tar`

```bash
mkdir -p artifacts
docker save ${REG}/backend:${VERSION}  -o artifacts/backend_${VERSION}.tar
docker save ${REG}/frontend:${VERSION} -o artifacts/frontend_${VERSION}.tar
docker save ${REG}/postgres:15-alpine  -o artifacts/postgres_15-alpine.tar
```

**But :** transporter les images sans Internet.

---

## 6. Basculer sur le réseau fermé

- Se connecter au Wi-Fi/routeur de démo.
    
- Vérifier le contexte :
    
    ```bash
    docker context use tfe
    docker --context tfe ps
    ```
    

**But :** s’assurer qu’on travaille bien avec le daemon du Pi.

---

## 7. Charger les images dans le Pi

```bash
docker --context tfe load -i production/artifacts/backend_${VERSION}.tar
docker --context tfe load -i production/artifacts/frontend_${VERSION}.tar
docker --context tfe load -i production/artifacts/postgres_15-alpine.tar
```

**But :** rendre les images disponibles localement sur le Pi.

---

## 8. Pousser les images dans le registre privé du Pi

```bash
docker --context tfe push ${REG}/backend:${VERSION}
docker --context tfe push ${REG}/frontend:${VERSION}
docker --context tfe push ${REG}/postgres:15-alpine
```

**But :** enregistrer officiellement les images dans le registre du Pi, source de vérité pour Compose.

---

## 9. Déployer avec Compose sur le Pi

1. Mettre à jour `docker-compose.production.yml` avec les bons tags (`vX.Y.Z`).
    
2. Lancer :
    
    ```bash
    docker --context tfe compose -f docker-compose.production.yml up -d
    docker --context tfe ps
    ```
    

**But :** démarrer les conteneurs versionnés, avec redémarrage auto au boot.

---

## 10. Vérifications

- Navigateur : `http://192.168.3.100/` → front accessible.
    
- Logs :
    
    ```bash
    docker --context tfe logs backend --tail=50
    docker --context tfe logs frontend --tail=20
    ```
    

**But :** confirmer que front et back fonctionnent, DB connectée.

---

## 11. Rollback (si nécessaire)

1. Éditer `docker-compose.production.yml` et remettre une version précédente (`v0.9.0` par ex.).
    
2. Relancer :
    
    ```bash
    docker --context tfe compose -f docker-compose.production.yml up -d
    ```
    

**But :** revenir rapidement à une version stable.

---

## 12. Entretien

- Nettoyer régulièrement les images inutiles :
    
    ```bash
    docker --context tfe image prune -a
    ```
    
- Sauvegarder le fichier `VERSION` à chaque release.
    

**But :** garder le Pi propre et savoir quelle version tourne.

-------------------

Parfait 👍  
On va faire un petit script qui gère un fichier `VERSION` dans ton projet.

### 1. Principe

- Le fichier `VERSION` est créé à la racine de ton projet.
    
- Il contient **la dernière version déployée** (par ex. `v1.0.0`).
    
- Quand tu lances le script, tu donnes une nouvelle version (ex. `v1.0.1`).
    
- Le script vérifie si cette version existe déjà → si oui, il refuse.
    
- Si c’est nouveau, il écrase le fichier `VERSION` et continue.
    

---

### 2. Script `scripts/set-version.sh`

```sh
#!/usr/bin/env sh
# usage: ./scripts/set-version.sh vX.Y.Z

VERSION_FILE="./VERSION"

if [ -z "$1" ]; then
  echo "❌ Tu dois donner une version (ex: ./scripts/set-version.sh v1.0.0)"
  exit 1
fi

NEW_VERSION="$1"

# Vérification du format vX.Y.Z
if ! echo "$NEW_VERSION" | grep -Eq '^v[0-9]+\.[0-9]+\.[0-9]+$'; then
  echo "❌ Mauvais format de version. Utilise vX.Y.Z (ex: v1.2.3)"
  exit 1
fi

# Vérifier si le fichier existe déjà
if [ -f "$VERSION_FILE" ]; then
  CURRENT_VERSION=$(cat "$VERSION_FILE")
  if [ "$NEW_VERSION" = "$CURRENT_VERSION" ]; then
    echo "❌ La version $NEW_VERSION est déjà enregistrée dans $VERSION_FILE"
    exit 1
  fi
fi

# Enregistrer la nouvelle version
echo "$NEW_VERSION" > "$VERSION_FILE"
echo "✅ Nouvelle version enregistrée : $NEW_VERSION"
```

---

### 3. Utilisation

1. Rendre exécutable :
    
    ```bash
    chmod +x scripts/set-version.sh
    ```
    
2. Définir une version :
    
    ```bash
    ./scripts/set-version.sh v1.0.0
    ```
    
    → crée un fichier `VERSION` avec dedans `v1.0.0`.
    
3. Si tu relances avec la **même version** :
    
    ```bash
    ./scripts/set-version.sh v1.0.0
    ```
    
    → le script dit “❌ déjà enregistrée”.
    
4. Si tu relances avec une nouvelle version :
    
    ```bash
    ./scripts/set-version.sh v1.0.1
    ```
    
    → met à jour le fichier `VERSION` et enregistre `v1.0.1`.
    

---

### 4. Exemple du fichier `VERSION`

```
v1.0.1
```

---

👉 Avec ça, tu as une **mémoire permanente** de la version courante.  
Ensuite, tes autres scripts (`build.sh`, `push.sh`) peuvent lire ce fichier pour savoir **quelle version builder/pusher**.

---

Veux-tu que je t’écrive le `build.sh` qui **lit automatiquement la version depuis ce fichier `VERSION`** au lieu de te la demander ?

Parfait — **Étape 2 : préparer/mettre au propre les Dockerfiles (et nginx.conf du front)**.  
Je te donne **les fichiers finaux** à poser dans ton repo, prêts pour ARM64, alignés avec ton `production.env` et ton rollup.

---

# Backend — `packages/backend/Dockerfile`

Points clés : build avec pnpm, **on n’embarque pas** `production.env` dans l’image, exécution en Node 20-alpine, healthcheck HTTP (à adapter si besoin).

```dockerfile
# packages/backend/Dockerfile
FROM node:20-alpine AS build
WORKDIR /app
RUN npm i -g pnpm
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml ./
RUN pnpm install --frozen-lockfile
COPY . .
# Build via rollup (src/server.ts -> dist/server.js, src/seed.ts -> dist/seed.js)
RUN pnpm build

FROM node:20-alpine AS production
WORKDIR /app
ENV NODE_ENV=production
RUN npm i -g pnpm
# Installe uniquement les deps de prod
COPY --from=build /app/package.json /app/pnpm-lock.yaml ./
RUN pnpm install --prod --frozen-lockfile
# Copie des artefacts buildés et éventuels certs
COPY --from=build /app/dist ./dist
COPY --from=build /app/certs ./certs
# Port d'écoute backend (tu as PORT=3001 dans production.env)
EXPOSE 3001
# Healthcheck: assure-toi d'avoir une route /health qui répond 200
HEALTHCHECK --interval=10s --timeout=3s --start-period=15s --retries=3 \
  CMD wget -qO- http://127.0.0.1:3001/health || exit 1
CMD ["node","dist/server.js"]
```

⚠️ Si ta route santé n’est pas `/health`, change la commande du healthcheck.

---

# Frontend — `packages/frontend/Dockerfile`

Points clés : build Vite (CSR), **pas de CORS** car nginx proxifie `/api` (et `/ws`) vers `backend:3001`.  
Tu avais déjà un Dockerfile avec nginx : je le « bétonne » et force `VITE_API_BASE_URL=/api`.

```dockerfile
# packages/frontend/Dockerfile
FROM node:20-alpine AS build
WORKDIR /app
RUN npm i -g pnpm
COPY package.json pnpm-lock.yaml ./
RUN pnpm install --frozen-lockfile
COPY . .
# IMPORTANT: en prod le front appellera /api (proxy nginx), pas l'IP/port du back
ENV VITE_API_BASE_URL=/api
RUN pnpm build

FROM nginx:alpine AS production
WORKDIR /usr/share/nginx/html
# Nettoie le dossier statique
RUN rm -rf ./*
# Copie le build
COPY --from=build /app/dist .
# Conf nginx pour SPA + proxy /api et /ws
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
# Healthcheck simple (retourne index.html)
HEALTHCHECK --interval=10s --timeout=3s --start-period=5s --retries=3 \
  CMD wget -qO- http://127.0.0.1/ || exit 1
```

## `packages/frontend/nginx.conf`

À ajouter (ou remplacer le tien) pour servir la SPA et proxifier l’API/WS vers le **service Docker** `backend:3001` :

```nginx
server {
  listen 80;
  server_name _;

  root /usr/share/nginx/html;
  index index.html;

  # Routes front (SPA)
  location / {
    try_files $uri /index.html;
  }

  # Proxy API -> backend
  location /api/ {
    proxy_pass http://backend:3001/;
    proxy_http_version 1.1;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
  }

  # WebSocket si utilisé
  location /ws/ {
    proxy_pass http://backend:3001/ws/;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
  }
}
```

---

# Notes importantes (pour éviter les pièges)

- **Ne copie pas** `production.env` dans l’image backend. On l’injectera via `env_file` dans Compose (étape 3).
    
- `DATABASE_URL` pointe déjà sur `db:5432` dans ton `production.env` → parfait pour le réseau Docker.
    
- Ton `FRONTEND_URL` en prod doit être `http://192.168.3.100/` (pas `:5173`).
    
- Si tu préfères **sans nginx** côté front, je peux te fournir l’alternative « mini serveur Node (Express) + http-proxy-middleware » — mais comme tu as déjà nginx, on garde ça (plus simple et performant).
    

---

Dis-moi quand c’est posé/commit, et on enchaîne avec **l’étape 3 (docker-compose.production.yml)** pour brancher `env_file`, les images taguées et `restart: always`.

Parfait 👌 tu as maintenant une base claire :

```
ton-projet/
├── packages/
│   ├── backend/
│   │   └── Dockerfile
│   └── frontend/
│       └── Dockerfile
├── production/
│   ├── production.env
│   └── set-version.sh
```

👉 Très bon réflexe de centraliser dans `production/`.

---

### Pourquoi c’est utile ?

- `production.env` est bien séparé des fichiers de dev → Compose pourra l’injecter facilement au back.
    
- `set-version.sh` est au même endroit → tu gardes toutes les infos de mise en prod ensemble.
    
- On pourra aussi mettre dans ce dossier ton futur `docker-compose.production.yml` et ton `db.env`.
    

---

### Prochaine étape (Étape 3)

Créer **`production/docker-compose.production.yml`** qui :

- utilise les images `frontend:vX.Y.Z` et `backend:vX.Y.Z`,
    
- lit `production.env` et `db.env`,
    
- met `restart: always`,
    
- expose uniquement le front (`80:80`),
    
- garde `db` et `backend` internes.
    

---

parfait — voici un `docker-compose.production.yml` prêt à déposer dans ton dossier `production/`.  
Il n’expose **que** le front en `80:80`, utilise tes **env files**, met `restart: always`, et branche le proxy `/api` (service `backend`).

```yaml
# production/docker-compose.production.yml
version: "3.9"

name: myapp

services:
  db:
    image: 192.168.3.100:5000/postgres:15-alpine
    env_file:
      - ./db.env                # <-- à créer dans production/ (voir note plus bas)
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
    image: 192.168.3.100:5000/backend:vX.Y.Z   # <-- remplace vX.Y.Z par ta version (ex: v1.0.0)
    env_file:
      - ./production.env        # <-- ton fichier déjà présent dans production/
    environment:
      NODE_ENV: production
      PORT: 3001                # cohérent avec production.env
    depends_on:
      db:
        condition: service_healthy
    restart: always
    networks: [appnet]

  frontend:
    image: 192.168.3.100:5000/frontend:vX.Y.Z  # <-- même version que backend
    depends_on:
      - backend
    ports:
      - "80:80"                 # seul point d'entrée LAN
    restart: always
    networks: [appnet]

volumes:
  dbdata:

networks:
  appnet:
    driver: bridge
```

## Notes rapides

- **Tags d’images** : remplace `vX.Y.Z` par ta version (ex. `v1.0.0`).
    
- **db.env** (à créer dans `production/`) — adapte aux valeurs que tu utilises déjà :
    
    ```
    POSTGRES_DB=<NOM_PROJET>_dev
    POSTGRES_USER=<NOM_PROJET>
    POSTGRES_PASSWORD=<MOT_DE_PASSE_DB>
    ```
    
- **production.env** (déjà chez toi) : garde `DATABASE_URL=postgresql://<NOM_PROJET>:<MOT_DE_PASSE_DB>@db:5432/<NOM_PROJET>_dev` (host = `db`).
    
- Le **frontend** (nginx) résout le service `backend` via le réseau `appnet` et proxifie `/api` → **pas de CORS** requis.
    
- `restart: always` assure le redémarrage auto au boot.
    

## Utilisation (quand prêt à déployer)

Depuis la **racine** du repo :

```bash
docker --context tfe compose -f production/docker-compose.production.yml up -d
```

ou bien depuis le dossier `production/` :

```bash
docker --context tfe compose up -d
```

oui 👍 très bonne idée → on va enrichir ton `production/set-version.sh` pour :

1. vérifier/écrire la nouvelle version dans le fichier `VERSION`,
    
2. et aussi **mettre à jour automatiquement** la valeur `vX.Y.Z` dans `production/docker-compose.production.yml`.
    

---

### Nouveau contenu de `production/set-version.sh`

```sh
#!/usr/bin/env sh
# usage: ./production/set-version.sh vX.Y.Z

VERSION_FILE="./production/VERSION"
COMPOSE_FILE="./production/docker-compose.production.yml"

if [ -z "$1" ]; then
  echo "❌ Tu dois donner une version (ex: ./production/set-version.sh v1.0.0)"
  exit 1
fi

NEW_VERSION="$1"

# Vérification du format vX.Y.Z
if ! echo "$NEW_VERSION" | grep -Eq '^v[0-9]+\.[0-9]+\.[0-9]+$'; then
  echo "❌ Mauvais format de version. Utilise vX.Y.Z (ex: v1.2.3)"
  exit 1
fi

# Vérifier si la version est déjà enregistrée
if [ -f "$VERSION_FILE" ]; then
  CURRENT_VERSION=$(cat "$VERSION_FILE")
  if [ "$NEW_VERSION" = "$CURRENT_VERSION" ]; then
    echo "❌ La version $NEW_VERSION est déjà enregistrée"
    exit 1
  fi
fi

# Enregistrer la nouvelle version
echo "$NEW_VERSION" > "$VERSION_FILE"
echo "✅ Nouvelle version enregistrée : $NEW_VERSION"

# Mettre à jour docker-compose.production.yml
if [ -f "$COMPOSE_FILE" ]; then
  # Remplace toute ancienne version vX.Y.Z par la nouvelle
  sed -i.bak -E "s|(backend:)(v[0-9]+\\.[0-9]+\\.[0-9]+)|\\1$NEW_VERSION|g" "$COMPOSE_FILE"
  sed -i.bak -E "s|(frontend:)(v[0-9]+\\.[0-9]+\\.[0-9]+)|\\1$NEW_VERSION|g" "$COMPOSE_FILE"
  rm -f "$COMPOSE_FILE.bak"
  echo "✅ docker-compose.production.yml mis à jour avec $NEW_VERSION"
else
  echo "⚠️  $COMPOSE_FILE introuvable, mise à jour ignorée"
fi
```

---

### Ce que ça fait

- Si tu lances :
    
    ```bash
    ./production/set-version.sh v1.0.1
    ```
    
- Alors :
    
    1. Le fichier `production/VERSION` contient maintenant `v1.0.1`.
        
    2. Dans `docker-compose.production.yml`, les lignes :
        
        ```yaml
        image: 192.168.3.100:5000/backend:v0.9.0
        image: 192.168.3.100:5000/frontend:v0.9.0
        ```
        
        deviennent :
        
        ```yaml
        image: 192.168.3.100:5000/backend:v1.0.1
        image: 192.168.3.100:5000/frontend:v1.0.1
        ```
        

---

Parfait — voici **l’étape 4** prête à l’emploi avec un **script unique** qui construit tout sur ta machine (avec Internet), taggue correctement, et exporte les images en `.tar`.

# Script : `production/build.sh`

```sh
#!/usr/bin/env sh
# Build des images en ARM64 + export en .tar
# Usage: ./production/build.sh
# Prérequis: fichier production/VERSION (généré via ./production/tag.sh vX.Y.Z)

set -e

# --- Chemins et constantes ---
ROOT_DIR="$(cd "$(dirname "$0")/.." && pwd)"
VERSION_FILE="${ROOT_DIR}/production/VERSION"
REG="192.168.3.100:5000"

BACK_DOCKERFILE="${ROOT_DIR}/packages/backend/Dockerfile"
FRONT_DOCKERFILE="${ROOT_DIR}/packages/frontend/Dockerfile"
BACK_CTX="${ROOT_DIR}/packages/backend"
FRONT_CTX="${ROOT_DIR}/packages/frontend"

ARTIFACTS_DIR="${ROOT_DIR}/production/artifacts"

# --- Contrôles préalables ---
if [ ! -f "$VERSION_FILE" ]; then
  echo "❌ VERSION introuvable. Lance d'abord: ./production/tag.sh vX.Y.Z"
  exit 1
fi
VERSION="$(cat "$VERSION_FILE")"

if [ ! -f "$BACK_DOCKERFILE" ]; then
  echo "❌ Dockerfile backend introuvable: $BACK_DOCKERFILE"
  exit 1
fi
if [ ! -f "$FRONT_DOCKERFILE" ]; then
  echo "❌ Dockerfile frontend introuvable: $FRONT_DOCKERFILE"
  exit 1
fi

mkdir -p "$ARTIFACTS_DIR"

echo "ℹ️  VERSION: $VERSION"
echo "ℹ️  REG:     $REG"
echo "ℹ️  ROOT:    $ROOT_DIR"
echo

# --- Préparer buildx (idempotent) ---
docker buildx create --name prod_builder --use >/dev/null 2>&1 || true
docker buildx inspect --bootstrap >/dev/null 2>&1 || true

# --- Build BACKEND ---
echo "▶️  Build backend (ARM64)…"
docker buildx build --platform linux/arm64 \
  -t "${REG}/backend:${VERSION}" \
  -f "${BACK_DOCKERFILE}" \
  --load \
  "${BACK_CTX}"

# --- Build FRONTEND ---
echo "▶️  Build frontend (ARM64)…"
docker buildx build --platform linux/arm64 \
  -t "${REG}/frontend:${VERSION}" \
  -f "${FRONT_DOCKERFILE}" \
  --load \
  "${FRONT_CTX}"

# --- Miroir Postgres ---
echo "▶️  Prépare postgres:15-alpine (ARM64)…"
docker pull --platform linux/arm64 postgres:15-alpine
docker tag postgres:15-alpine "${REG}/postgres:15-alpine"

# --- Export en .tar ---
echo "💾 Export des images en ${ARTIFACTS_DIR}"
docker save "${REG}/backend:${VERSION}"   -o "${ARTIFACTS_DIR}/backend_${VERSION}.tar"
docker save "${REG}/frontend:${VERSION}"  -o "${ARTIFACTS_DIR}/frontend_${VERSION}.tar"
docker save "${REG}/postgres:15-alpine"   -o "${ARTIFACTS_DIR}/postgres_15-alpine.tar"

echo
echo "✅ Build OK."
echo "📦 Fichiers générés :"
echo "  - ${ARTIFACTS_DIR}/backend_${VERSION}.tar"
echo "  - ${ARTIFACTS_DIR}/frontend_${VERSION}.tar"
echo "  - ${ARTIFACTS_DIR}/postgres_15-alpine.tar"
```

## Utilisation

```bash
chmod +x production/build.sh
./production/build.sh
```

## Ce que fait ce script (et pourquoi)

- Lit la **version** depuis `production/VERSION` (donc cohérent avec ton `tag.sh`).
    
- Construit **ARM64** (compatible Raspberry Pi) pour **backend** et **frontend**, avec les **bons tags** `192.168.3.100:5000/<service>:vX.Y.Z`.
    
- Prépare une image **Postgres 15-alpine** taggée pour ton registre privé (utile hors-ligne).
    
- Exporte les 3 images en `.tar` dans `production/artifacts/` pour pouvoir les **charger** sur le Pi quand tu seras sur le **réseau fermé**.
    

Quand tu veux, on passe à l’**étape 5 (export OK, bascule réseau fermé & load/push)** avec un script `push_deploy.sh`.


