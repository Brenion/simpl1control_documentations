## PHASE 1 – Vérification de la connexion ESP32 au broker MQTT sécurisé

### ✅ 1. Importer le certificat `ca.crt` dans le firmware ESP32

#### Objectif

Disposer du certificat de l'autorité de certification du broker MQTT pour que l'ESP32 puisse établir une connexion TLS vérifiée.

#### Actions attendues

- Identifier le fichier `ca.crt` utilisé par le broker MQTT.
    
- Prévoir son intégration dans le firmware ESP32 (structure, emplacement, nommage).
    
- Préparer sa future utilisation lors de la configuration du client TLS sur l'ESP32.
    

#### Validation de l'étape

- Le certificat est bien localisé, lisible, et prêt à être embarqué dans le firmware.
    

_En attente de validation utilisateur pour passer à l'étape 2._

---

### ⏳ 2. Configurer le firmware pour se connecter en TLS (port 8883)

### ⏳ 3. Connecter l’ESP32 au broker MQTT avec validation TLS

### ⏳ 4. Vérifier la réception des messages MQTT du backend

### ⏳ 5. Vérifier la capacité de l’ESP32 à publier des messages MQTT sur un topic sécurisé

### ⏳ 6. Confirmer la boucle ESP32 ↔ Backend fonctionne sur 8883

### ⏳ 7. Observer les logs côté broker et backend