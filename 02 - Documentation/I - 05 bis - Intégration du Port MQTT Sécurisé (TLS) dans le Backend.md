## Intégration du Port MQTT Sécurisé (TLS) dans le Backend

> **Prérequis** : Les certificats doivent être générés avant. Voir [[L - Guide de Mise en Production#2. Génération des Certificats TLS]] ou [[I - 05 - Mise en œuvre technique – Encodeur ESP32#2.2.1 Génération du certificat de l'autorité (CA)]]

Ce document décrit comment configurer le backend pour qu'il écoute à la fois sur le port MQTT non sécurisé (**1883**) et sécurisé (**8883** via TLS/SSL). Les étapes suivantes s'appliquent directement au backend et supposent que les certificats sont déjà générés et que le broker MQTT (ex : Mosquitto) est déjà configuré.

---

### 1. Connexion sécurisée dans le backend

Le backend utilise un fichier `mqtt.ts` qui initialise un client MQTT global, gère les abonnements, la publication et le traitement des messages. L’instance est connectée à un seul port à la fois (via `process.env.MQTT_BROKER_URL`).

Pour supporter **à la fois** le port 1883 (clair) et le port 8883 (TLS), il est nécessaire d'initialiser **deux clients distincts**. Voici la version finale intégrée dans `server.ts` :

```ts
if(process.env.MQTT_BASE_URL || process.env.MQTT_CLIENT_ID || process.env.MQTT_USERNAME || process.env.MQTT_PASSWORD) {
  const createMqttOptions = (port: string, secure: boolean): MqttOptions => ({
    baseUrl: `${port === "8883" ? "mqtts" : "mqtt"}://${process.env.MQTT_BASE_URL}:${port}`,
    clientId: process.env.MQTT_CLIENT_ID ?? (() => { throw new Error("MQTT_CLIENT_ID is required"); })(),
    username: process.env.MQTT_USERNAME,
    password: process.env.MQTT_PASSWORD,
    keepalive: Number(process.env.MQTT_KEEPALIVE) || 60,
    clean: process.env.MQTT_CLEAN_SESSION === "true",
    ...(secure && {
      key: fs.readFileSync("./certs/client_backend.key"),
      cert: fs.readFileSync("./certs/client_backend.crt"),
      ca: fs.readFileSync("./certs/ca.crt"),
      rejectUnauthorized: false,
    }),
  });

  if (process.env.MQTT_PORT_PLAIN) {
    const optionsPlain = createMqttOptions(process.env.MQTT_PORT_PLAIN, false);
    setTimeout(() => {
      mqttServer(optionsPlain, "plain");
    }, 1500);
  }

  if (process.env.MQTT_PORT_TLS) {
    const optionsTls = createMqttOptions(process.env.MQTT_PORT_TLS, true);
    setTimeout(() => {
      mqttServer(optionsTls, "tls");
    }, 1500);
  }
}
```

Cette implémentation permet de lancer deux connexions distinctes vers le broker, avec une configuration TLS pour le port 8883.

> **Important** : L'architecture actuelle de `mqtt.ts` utilise une instance unique (`mqttClientInstance`). Pour permettre les **deux connexions simultanées**, il faut refactorer pour supporter plusieurs clients, par exemple en maintenant une `Map<string, MqttClient>` ou en instanciant deux services.

---

### 2. Gestion simultanée des deux ports

Le backend doit être capable de se connecter aux deux ports **1883 (non sécurisé)** et **8883 (sécurisé)**.

> ⚠️ **Les deux connexions doivent être établies indépendamment.** Le backend doit impérativement initier une connexion vers chacun des deux ports, car ils sont gérés séparément par le broker MQTT.

---

### 3. Préparation des certificats TLS

Pour que la connexion sécurisée fonctionne, créez un dossier `certs` à la racine du projet et placez-y les fichiers suivants :

```bash
mkdir certs
mv chemin/vers/client_backend.key certs/client_backend.key
mv chemin/vers/client_backend.crt certs/client_backend.crt
mv chemin/vers/ca.crt certs/ca.crt
```

Ces fichiers seront lus par le backend pour configurer la connexion TLS.

---

### 4. Test de la connexion sécurisée

Vous pouvez tester l’envoi d’un message MQTT via TLS en utilisant `mosquitto_pub` :

```bash
mosquitto_pub -h localhost -p 8883 --cafile ca.crt \
  --cert client_backend.crt --key client_backend.key \
  -t "test/topic" -m "hello TLS"
```

---

## ✅ Résultat attendu

- Le backend établit deux connexions MQTT : une en clair (1883), une sécurisée (8883)
    
- Les messages reçus sur l’un ou l’autre des ports sont traités de manière équivalente par le broker
    
- Le backend ne dépend pas de l’unique disponibilité d’un port pour fonctionner correctement
    

---

## 📎 Annexe

- Documentation Mosquitto : [https://mosquitto.org/man/mosquitto-conf-5.html](https://mosquitto.org/man/mosquitto-conf-5.html)
    
- MQTT.js TLS Example : [https://github.com/mqttjs/MQTT.js#tls](https://github.com/mqttjs/MQTT.js#tls)