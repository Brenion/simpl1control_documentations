## Documentation MQTT — Simpl1Control

### Broker

- Mosquitto sur `192.168.3.100` (serveur LAN de production)
- Port 1883 (plain) et 8883 (TLS)
- Aucun fichier de config Mosquitto n'est versionné dans le repo

### Utilisateurs

|Utilisateur|Port|Rôle|
|---|---|---|
|`admin`|1883 + 8883|Backend — client unique avec accès `unifyIots/#`|
|Devices IoT|variable|Credentials firmware (hors repo)|

Le backend initialise deux connexions simultanées :

- `clientId: client` sur `mqtt://` port 1883 (devices `isSecure=false`)
- `clientId: client-backend` sur `mqtts://` port 8883 avec certificats mutuels (devices `isSecure=true`)

### Topics et ACL

Convention : `unifyIots/<type>/<deviceId>/<get|set>`

|Device|Subscribe (backend écoute)|Publish (backend envoie)|isSecure|
|---|---|---|---|
|`custom-sensor-dth-01` (DHT)|`unifyIots/sensor/custom-sensor-dth-01/get`|—|false (1883)|
|`writer-nfc-01`|`unifyIots/controller/writer-nfc-01/get`|`unifyIots/controller/writer-nfc-01/set`|true (8883)|
|`reader-nfc-01`|`unifyIots/controller/reader-nfc-01/get`|`unifyIots/controller/reader-nfc-01/set`|true (8883)|
|`logo-01` (LOGO 8.4)|`unifyIots/API/logo/get`|`unifyIots/API/logo/set`|false (1883)|

### Points d'amélioration identifiés

1. Port 8883 hardcodé dans `monitorControllerPresence.ts` (ligne 10) — devrait utiliser `getPortForSecurity()`
2. `rejectUnauthorized: false` dans `mqtt.service.ts` — à passer à `true` en prod
3. Mot de passe de prod réutilisé en dev dans `development.env`
4. Aucun fichier Mosquitto versionné — config du broker opaque
5. ACL non documentée/vérifiée — l'user `admin` a potentiellement `#` wildcard
6. Pas de broker MQTT local pour le développement — dépendance directe sur la prod