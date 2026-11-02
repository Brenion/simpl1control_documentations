# Forgejo Runner sur TrueNAS — Documentation

## Contexte

L'app Forgejo Runner du catalogue TrueNAS génère un `config.yaml` incorrect et ne fait jamais l'enregistrement. La solution est de passer en custom app avec un yaml modifié.

---

## Prérequis

- Forgejo installé et fonctionnel sur TrueNAS
- Port HTTP de Forgejo : `30142`
- Port SSH de Forgejo : `30143`
- IP locale du NAS : `192.168.0.167`

---

## Étape 1 — Générer le token dans Forgejo

Dans Forgejo : **Site Administration → Actions → Runners → Create new runner**

Copie le token affiché — il ne s'affiche qu'une seule fois.

---

## Étape 2 — Convertir en custom app

Dans TrueNAS : **Apps → forgejo-runner → Edit → Convert to custom app**

---

## Étape 3 — Remplacer le yaml

Dans l'éditeur TrueNAS, efface tout et colle ce yaml en remplaçant `TON_TOKEN` et `TON_UUID` par le token généré à l'étape 1 :

```yaml
configs:
  entrypoint:
    content: |
      echo -n "TON_TOKEN" > /data/runner-token
      forgejo-runner daemon \
        --url http://192.168.0.167:30142 \
        --uuid [TON_UUID] \
        --token-url file:///data/runner-token \
        --label docker:docker://node:lts
services:
  forgejo-runner:
    image: code.forgejo.org/forgejo/runner:12.10.2
    cap_drop:
      - ALL
    configs:
      - mode: 493
        source: entrypoint
        target: /ix-entrypoint.sh
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 4096M
    entrypoint:
      - /bin/sh
      - '-c'
      - /ix-entrypoint.sh
    environment:
      GID: '568'
      GROUP_ID: '568'
      NVIDIA_VISIBLE_DEVICES: void
      PGID: '568'
      PUID: '568'
      TZ: Europe/Brussels
      UID: '568'
      UMASK: '002'
      UMASK_SET: '002'
      USER_ID: '568'
    extra_hosts:
      host.docker.internal: host-gateway
    group_add:
      - 568
      - 999
    healthcheck:
      interval: 30s
      retries: 5
      start_interval: 2s
      start_period: 15s
      test: pgrep forgejo-runner
      timeout: 5s
    network_mode: host
    platform: linux/amd64
    privileged: False
    restart: unless-stopped
    security_opt:
      - no-new-privileges=true
    stdin_open: False
    tty: False
    user: '568:568'
    volumes:
      - bind:
          create_host_path: False
          propagation: rprivate
        read_only: False
        source: /mnt/.ix-apps/app_mounts/forgejo-runner/data
        target: /data
        type: bind
      - bind:
          create_host_path: False
          propagation: rprivate
        read_only: False
        source: /var/run/docker.sock
        target: /var/run/docker.sock
        type: bind
volumes: {}
x-portals: []
```

---

## Étape 4 — Créer le config.yml

En SSH sur le NAS :

```bash
cat > /mnt/.ix-apps/app_mounts/forgejo-runner/data/config.yml << 'EOF'
log:
  level: info

runner:
  file: .runner
  capacity: 1
  timeout: 3h
  insecure: false
  fetch_timeout: 5s
  fetch_interval: 2s
  labels: []

cache:
  enabled: true

container:
  network: "host"
  docker_host: "-"
  force_pull: false
EOF
```

---

## Étape 5 — Démarrer l'app

Dans TrueNAS : sauvegarde et démarre l'app.

Vérifie dans Forgejo **Site Administration → Actions → Runners** — le runner doit apparaître en statut **idle**.

---

## Notes importantes

- Le token Forgejo expire si non utilisé — si le runner ne démarre plus, regénère un token dans Forgejo et mets-le à jour dans le yaml
- L'UUID est généré automatiquement par Forgejo au premier enregistrement
- `network_mode: host` est indispensable pour que le runner joigne Forgejo sur le réseau du NAS
- La v12 du runner utilise `--token-url` et non plus `--token` (déprécié)

---

## Connexion SSH Git via Tailscale

### Configuration

- IP Tailscale du NAS : `100.109.159.25`
- Port SSH Git : `30143`
- Clé SSH dédiée : `~/.ssh/forgejo`

### ~/.ssh/config sur le Mac

```
Host forgejo.brenion-digit.com
  HostName 100.109.159.25
  Port 30143
  User git
  IdentityFile ~/.ssh/forgejo
```

### Cloner un repo

```bash
git clone ssh://git@forgejo.brenion-digit.com/Brenion/NOM_DU_REPO.git
```

### Si la clé n'est pas chargée au démarrage

```bash
ssh-add ~/.ssh/forgejo
```

### Expiration Tailscale

Désactiver l'expiration sur le NAS : **Dashboard Tailscale → Machines → NAS → Disable key expiry**
