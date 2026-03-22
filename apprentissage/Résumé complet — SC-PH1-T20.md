
---

## Contexte

On a ajouté Mosquitto, un broker MQTT, dans le `docker-compose.yml` existant de `packages/backend/`. Un broker MQTT est un serveur de messagerie qui reçoit des messages de publishers et les redistribue aux subscribers abonnés aux mêmes topics. C'est le cœur de communication du système domotique.

---

## Étape 1 — Comprendre les types de stockage Docker

Volume nommé (`postgres_data:/var/lib/postgresql/data`) Stockage géré par Docker, opaque, inaccessible directement depuis l'IDE. Utilisé pour des données binaires qu'on ne lit jamais à la main (fichiers internes de PostgreSQL).

Bind mount (`./mosquitto/config:/mosquitto/config`) Un dossier réel sur ta machine mappé dans le container. Bidirectionnel et instantané. Utilisé pour les fichiers qu'on veut lire et éditer directement depuis l'IDE (config, logs).

Règle : si tu as besoin de toucher au fichier depuis ton IDE → bind mount. Sinon → volume nommé.

---

## Étape 2 — Créer la structure de dossiers

mkdir -p mosquitto/config mosquitto/data mosquitto/log

Trois dossiers distincts car Mosquitto sépare config, persistance des messages et logs.

---

## Étape 3 — Écrire `mosquitto.conf`

Concept clé : lecture séquentielle et contexte de listener Mosquitto lit le fichier de haut en bas. Toute directive avant le premier `listener` est globale. Toute directive après un `listener` lui appartient jusqu'au suivant.

# GLOBAL — s'applique à tous les listeners

password_file /mosquitto/config/passwd # chemin dans le container

allow_anonymous false # interdit les connexions sans credentials

persistence true # active la persistance des messages QoS 1/2

persistence_location /mosquitto/data # où stocker les messages persistés

log_dest file /mosquitto/log/mosquitto.log

# LISTENER 1883 — plain MQTT, interne

listener 1883

# LISTENER 8883 — futur TLS, réseau local

listener 8883

#cafile ... # à activer quand les certificats seront générés

#certfile ...

#keyfile ...

#require_certificate true

#use_identity_as_username true

Mosquitto v2 vs v1 : en v1 les connexions anonymes étaient autorisées par défaut. En v2 elles sont refusées. `allow_anonymous false` + `password_file` est donc obligatoire.

`require_certificate true` : active le TLS mutuel (mTLS) — le serveur exige un certificat du client en plus de présenter le sien. `use_identity_as_username true` extrait le CN du certificat client comme username, remplaçant le système password. Commenté car les certificats n'existent pas encore.

---

## Étape 4 — Ajouter le service dans `docker-compose.yml`

mosquitto:

image: eclipse-mosquitto:2 # version 2, majeure pinée

pull_policy: if_not_present # cohérence avec les autres services

container_name: simpl1controlMQTT # convention de nommage du projet

restart: always # redémarre automatiquement (reboot Raspberry Pi)

ports:

- "1883:1883" # plain MQTT

- "8883:8883" # futur TLS

volumes:

- ./mosquitto/config:/mosquitto/config

- ./mosquitto/data:/mosquitto/data

- ./mosquitto/log:/mosquitto/log

Les bind mounts ne se déclarent pas dans la section `volumes:` globale en bas du fichier — seuls les volumes nommés (comme `postgres_data`) y figurent.

---

## Étape 5 — Protéger les secrets avant de les créer

Ajout dans `.gitignore` avant de créer le fichier :

mosquitto/config/passwd

Pourquoi avant : Git n'oublie pas. Si le fichier est commité même une fois, il reste dans l'historique même après suppression. Il faut que Git ne le voie jamais, même une fraction de seconde.

---

## Étape 6 — Générer le fichier `passwd`

Le fichier `passwd` est créé une seule fois manuellement avec `mosquitto_passwd`. Il persiste sur le disque grâce au bind mount. Docker ne le recrée pas à chaque démarrage — il le lit.

docker run --rm \

-v $(pwd)/mosquitto/config:/mosquitto/config \

eclipse-mosquitto:2 \

mosquitto_passwd -b -c /mosquitto/config/passwd admin Miministry7819

|Flag|Rôle|
|---|---|
|`--rm`|Supprime le container après exécution|
|`-b`|Batch mode — password passé en argument, pas interactif|
|`-c`|Create — crée le fichier (écrase si existant)|

Résultat dans le fichier :

admin:$7$1000$salt...==$hash...==

`$7$` = PBKDF2-SHA512. Impossible à inverser sans attaque par force brute. Même si le fichier fuite, le mot de passe n'est pas lisible directement.

---

## Étape 7 — Démarrage et validation

docker compose up mosquitto

Logs confirmés :

Opening ipv4 listen socket on port 1883 ✓

Opening ipv6 listen socket on port 1883 ✓

Opening ipv4 listen socket on port 8883 ✓

Opening ipv6 listen socket on port 8883 ✓

New client connected ... u'admin' ✓

Test pub/sub :

# Terminal 1 — subscriber

mosquitto_sub -h localhost -t "test/hello" -u admin -P Miministry7819

# Terminal 2 — publisher

mosquitto_pub -h localhost -t "test/hello" -m "world" -u admin -P Miministry7819

Le message `world` apparaît dans le terminal du subscriber → broker fonctionnel de bout en bout.