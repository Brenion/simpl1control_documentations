### Décision architecturale finale

Mosquitto et Zigbee2MQTT passent en Docker.

docker-compose.yml

├── postgres (déjà en place)

├── mosquitto (nouveau — eclipse-mosquitto:2)

│ ├── port 1883 (plain, interne Docker uniquement)

│ └── port 8883 (TLS, exposé réseau local pour Logo, Arduinos, serrure)

├── zigbee2mqtt (nouveau — dongle USB passé via devices:)

├── backend (Node.js)

└── frontend (Nginx + React)

PKI locale par installation :

- Une CA par installation client — autonomie totale, fonctionne hors ligne
- `setup-pki.sh` — lancé une fois à l'installation, crée la CA + signe les certs initiaux
- `issue-cert.sh` — lancé depuis le container pour tout nouveau device à tout moment
- `ca.key` dans un volume Docker persistant, jamais dans git
- Certs générés dans `./certs/` — dans `.gitignore`

### Distinction dev vs prod

| Élément | Dev (Phase 1) | Prod (Phase 8) |
|---|---|---|
| **Mosquitto** | Docker, ports 1883 + 8883 optionnel | Docker, 1883 interne uniquement, 8883 obligatoire |
| **Certificats** | Générés localement, CA de dev | PKI `setup-pki.sh`, CA par installation client |
| **Users MQTT** | Mots de passe simples | Mots de passe forts distincts |
| **ACL** | Permissives | Strictes par topic |
| **Zigbee2MQTT** | Docker sans dongle physique | Docker + dongle USB monté via `by-id` |
| **Backend** | Node.js local | Docker + restart policy |
| **TLS** | Optionnel | Obligatoire |
| **PKI** | Script simplifié (CA + broker + backend) | `setup-pki.sh` complet + `issue-cert.sh` pour nouveaux devices |
| **Logo Siemens** | Non connecté (pas de matériel en dev) | Connecté via mqtts:8883, cert dédié |
| **Arduinos** | Non connectés | Certs `.der` générés par `issue-cert.sh` |



### Backlog — tâches modifiées

#### Phase 1 — modifications

| Tâche      | Action     | Nouvelle description                                                   |
| ---------- | ---------- | ---------------------------------------------------------------------- |
| SC-PH1-T04 | ✓ Terminée | Vérification faite — broker externe, pas de conf versionnée            |
| SC-PH1-T11 | Modifier   | ~~Installation native Z2M~~ → Préparer service Z2M en Docker           |
| SC-PH1-T15 | Modifier   | ~~Configurer systemd Z2M~~ → Configurer restart policy Docker pour Z2M |

#### Phase 1 — nouvelles tâches remontées de Phase 8

| Tâche      | Description                                                                                      | US           |
| ---------- | ------------------------------------------------------------------------------------------------ | ------------ |
| SC-PH1-T20 | Créer service Mosquitto dans `docker-compose.yml` avec ports 1883/8883                           | SC-US-PH1-01 |
| SC-PH1-T21 | Créer `mosquitto.conf` de développement (plain 1883 + TLS 8883 optionnel, ACL permissives)       | SC-US-PH1-01 |
| SC-PH1-T22 | Générer certificats de développement (version simplifiée `setup-pki.sh` — CA + broker + backend) | SC-US-PH1-01 |

#### Phase 8 — modifications

| Tâche      | Action    | Nouvelle description                                                                                               |
| ---------- | --------- | ------------------------------------------------------------------------------------------------------------------ |
| SC-PH8-T01 | Modifier  | ~~Générer certificats SSL~~ → Implémenter PKI locale complète via `setup-pki.sh` (CA + tous clients)               |
| SC-PH8-T06 | Modifier  | ~~Configurer Mosquitto TLS~~ → Déployer container Mosquitto prod avec TLS strict via certs PKI                     |
| SC-PH8-T07 | Conserver | ACL Mosquitto strictes par topic (prod uniquement)                                                                 |
| SC-PH8-T08 | Modifier  | ~~Créer users MQTT~~ → Créer users MQTT prod forts + script `issue-cert.sh` pour nouveaux devices                  |
| SC-PH8-T26 | Modifier  | ~~Configurer systemd Mosquitto + Z2M~~ → Configurer `docker-compose.production.yml` (Mosquitto + Z2M + dongle USB) |
| SC-PH8-T27 | Modifier  | ~~Configurer PM2 backend~~ → Configurer restart policy Docker pour backend                                         |

#### Phase 8 — nouvelle tâche

| Tâche          | Description                                                                                                         |
| -------------- | ------------------------------------------------------------------------------------------------------------------- |
| SC-PH8-T01-BIS | Créer script `issue-cert.sh` exécutable depuis container pour émettre de nouveaux certificats clients dans le temps |

---

## Ce qui manque dans le résumé

### 1. `rejectUnauthorized = false` — dette technique à corriger

C'est hardcodé dans `mqtt.service.ts` ligne 89. Avec la PKI qu'on met en place, cette ligne doit être passée à `true` sinon toute la sécurité TLS est inutile. C'est une tâche de code à ajouter.

À ajouter dans le backlog :

| Tâche          | Phase   | Description                                                           |
| -------------- | ------- | --------------------------------------------------------------------- |
| SC-PH8-T06-BIS | Phase 8 | Corriger `rejectUnauthorized = false` → `true` dans `mqtt.service.ts` |

Mais elle peut aussi être faite en Phase 1 puisqu'on touche déjà au MQTT.

---

### 2. `test.env` — oublié dans la réflexion

Le fichier `test.env` pointe encore sur `mqtt://192.168.0.102:1883` (une IP différente de dev). Quand le container Mosquitto sera en place, `test.env` doit aussi être mis à jour vers `localhost`.

À ajouter :

| Tâche      | Phase   | Description                                                                            |
| ---------- | ------- | -------------------------------------------------------------------------------------- |
| SC-PH1-T23 | Phase 1 | Mettre à jour `test.env` — `MQTT_BASE_URL` vers `localhost` cohérent avec le container |

---

### 3. `MQTT_START = false` en dev

Actuellement le service MQTT est désactivé par défaut en dev. Une fois le container Mosquitto en place, il faudra passer `MQTT_START=true` dans `development.env`. Sinon le container tourne mais le backend ne s'y connecte jamais.

C'est mineur mais ça bloquera un dev qui ne comprend pas pourquoi MQTT ne fonctionne pas malgré le container qui tourne.

À inclure dans SC-PH1-T20 ou SC-PH1-T21 comme note explicite.

---

## Résumé des ajouts

| #   | Quoi                                            | Où                          |
| --- | ----------------------------------------------- | --------------------------- |
| 1   | Corriger `rejectUnauthorized = false` → `true`  | SC-PH8-T06-BIS (ou Phase 1) |
| 2   | Mettre à jour `test.env` MQTT_BASE_URL          | SC-PH1-T23                  |
| 3   | Passer `MQTT_START=true` dans `development.env` | Note dans SC-PH1-T20        |

Maintenant je suis confiant que le résumé est complet.