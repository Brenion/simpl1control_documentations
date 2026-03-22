
---

## Contexte

On a généré les certificats TLS pour sécuriser le port 8883 de Mosquitto. TLS (Transport Layer Security) est un protocole de chiffrement qui garantit trois choses : confidentialité (les données sont chiffrées), intégrité (les données ne sont pas altérées en transit), authentification (le client vérifie qu'il parle bien au bon serveur).

---

## Les concepts fondamentaux

### CA — Certificate Authority (Autorité de Certification)

Une entité qui signe des certificats et garantit leur authenticité. En production, c'est une organisation tierce (Let's Encrypt, DigiCert). En dev, tu es ta propre CA — on parle de CA auto-signée.

La CA possède deux fichiers :

- `ca.crt` — son certificat public, distribué à tous les clients
- `ca.key` — sa clé privée, ne quitte jamais le serveur

### Certificat serveur

Prouve l'identité du serveur Mosquitto auprès des clients. Signé par la CA.

- `server.crt` — certificat public du serveur
- `server.key` — clé privée du serveur, ne quitte jamais le serveur

### CSR — Certificate Signing Request

Fichier temporaire créé par le futur possesseur d'un certificat. Il dit : "voici mon identité, voici ma clé publique, signez-moi." La CA le lit, vérifie, et produit le `.crt` final. Le `.csr` n'est plus utile ensuite.

### Subject Alternative Names (SAN)

Extension d'un certificat qui lui permet d'être valide pour plusieurs identités simultanément. Sans SAN, un certificat avec `CN=localhost` est rejeté si le client se connecte via `192.168.3.100`.

DNS:localhost, IP:127.0.0.1, IP:192.168.3.100

### TLS simple vs mTLS

TLS simple (ce qu'on a configuré) :

- Le serveur présente `server.crt` au client
- Le client vérifie avec `ca.crt` que le serveur est légitime
- Le client ne s'identifie pas

mTLS — TLS mutuel (tâche future, `require_certificate true`) :

- Le serveur présente son certificat au client
- Le client présente aussi son propre certificat au serveur
- Chaque appareil (Arduino, Logo Siemens, backend) a son propre couple `client.crt` + `client.key`
- Le serveur vérifie avec `ca.crt` que ce client est autorisé

---

## Les trois commandes openssl

### 1 — Créer la CA

openssl req -new -x509 -days 3650 \

-keyout ca.key -out ca.crt \

-nodes -subj "/CN=simpl1control-dev-CA"

`-x509` = auto-signé (la CA se signe elle-même). `-nodes` = clé sans mot de passe (Mosquitto doit pouvoir la lire seul au démarrage).

### 2 — Créer la clé serveur et le CSR

openssl req -new \

-keyout server.key -out server.csr \

-nodes -subj "/CN=localhost"

Produit une demande de signature — pas encore un vrai certificat.

### 3 — Signer le certificat serveur avec les SANs

openssl x509 -req -days 3650 \

-in server.csr \

-CA ca.crt -CAkey ca.key -CAcreateserial \

-out server.crt \

-extfile <(printf "subjectAltName=DNS:localhost,IP:127.0.0.1,IP:192.168.3.100")

La CA signe le CSR et produit `server.crt` avec les SANs intégrés. `-CAcreateserial` génère `ca.srl`, un fichier de numéro de série unique par certificat signé.

---

## Distribution des fichiers

|Fichier|Reste sur le serveur|Distribué aux clients|
|---|---|---|
|`ca.crt`|Oui|Oui — seul fichier à transmettre|
|`ca.key`|Oui, ne bouge jamais|Jamais|
|`server.crt`|Oui|Non|
|`server.key`|Oui, ne bouge jamais|Jamais|

Règle absolue : tout fichier `.key` ne quitte jamais la machine sur laquelle il a été généré.

---

## Configuration dans `mosquitto.conf`

listener 8883

cafile /mosquitto/config/certs/ca.crt

certfile /mosquitto/config/certs/server.crt

keyfile /mosquitto/config/certs/server.key

#require_certificate true ← mTLS, tâche future

#use_identity_as_username true ← extrait le CN du cert client comme username

Les chemins sont dans le container (pas sur la machine hôte) car c'est Mosquitto qui lit ce fichier depuis l'intérieur du container via le bind mount.

---

## Test de validation

# Subscriber sur 8883

mosquitto_sub -h localhost -p 8883 \

--cafile /chemin/absolu/ca.crt \

-t "test/hello" -u admin -P Miministry7819

# Publisher sur 8883

mosquitto_pub -h localhost -p 8883 \

--cafile /chemin/absolu/ca.crt \

-t "test/hello" -m "world-tls" \

-u admin -P Miministry7819

`--cafile` fournit la CA au client pour qu'il puisse valider l'identité du serveur. Sans ce flag, le client rejette le certificat serveur car il ne connaît pas la CA qui l'a signé.