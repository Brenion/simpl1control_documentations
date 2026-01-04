
Cette documentation décrit l'installation, la configuration et le test du broker MQTT Mosquitto sur un Raspberry Pi. Elle couvre également l'utilisation de MQTT Explorer sur un Mac pour interagir avec Mosquitto et la configuration d'un ESP32 pour envoyer des messages MQTT.

Les étapes incluent :

1. Installation de Mosquitto sur le Raspberry Pi.
2. Mise en route et configuration avec authentification.
3. Test local de Mosquitto sur le Raspberry Pi.
4. Installation et configuration de MQTT Explorer sur macOS.
5. Connexion entre MQTT Explorer et Mosquitto.
6. Test de communication entre le Raspberry Pi et MQTT Explorer.
7. Préparation et programmation d'un ESP32 pour publier des messages MQTT.

## 1. Installation de Mosquitto sur le Raspberry Pi

### Prérequis

- Un Raspberry Pi avec Raspberry Pi OS installé
- Une connexion Internet active
- Un accès SSH ou un terminal ouvert sur le Raspberry Pi

### Installation de Mosquitto

Mosquitto est un broker MQTT open-source qui permet la communication entre différents appareils.

8. Mettez à jour votre système :

```
sudo apt update && sudo apt upgrade -y
```
    
9. Installez Mosquitto et son client MQTT :

```
sudo apt install -y mosquitto mosquitto-clients
```
    
10. Activez et démarrez le service Mosquitto :

```
sudo systemctl enable mosquitto
sudo systemctl start mosquitto
```

11. Vérifiez l'installation avec :
    
```
mosquitto -v
```

Cette commande devrait afficher la version de Mosquitto et confirmer qu'il fonctionne.

---

## 2. Mise en Route du Serveur Mosquitto

### Vérifier que Mosquitto est actif

Utilisez la commande suivante pour vérifier l'état du service Mosquitto :

```
sudo systemctl status mosquitto
```

Si le service est actif, vous devriez voir une sortie indiquant qu'il est en cours d'exécution.

### Configurer Mosquitto pour l'accès distant (optionnel)

Par défaut, Mosquitto écoute uniquement sur `localhost`. Pour permettre l'accès depuis d'autres machines :

12. Éditez le fichier de configuration :
 ```
 sudo nano /etc/mosquitto/mosquitto.conf
```
    
13. Ajoutez les lignes suivantes à la fin du fichier :
    
```
password_file /etc/mosquitto/passwd
listener 1883
allow_anonymous false
```
    
![[Capture d’écran 2025-02-04 à 19.09.17.png]]
    
14. Créez un utilisateur administrateur pour Mosquitto :
    
```
sudo mosquitto_passwd -c /etc/mosquitto/passwd admin
```
    
    Entrez un mot de passe sécurisé lorsque demandé.
    
15. Redémarrez Mosquitto pour appliquer les modifications :
    
```
sudo systemctl restart mosquitto
```

---

## 3. Test de Mosquitto sur le Raspberry Pi

### Publier et souscrire à un message MQTT localement

Nous allons utiliser les clients Mosquitto installés pour tester la communication MQTT en local.

16. Ouvrez un premier terminal et exécutez la commande suivante pour écouter les messages publiés sur un topic :
    
    ```
    mosquitto_sub -h localhost -t "test/topic" -u admin -P "votre_mot_de_passe"
    ```
    
    - `mosquitto_sub` est la commande utilisée pour souscrire à un topic.
    - `-h localhost` indique que nous nous connectons au broker Mosquitto exécuté localement.
    - `-t "test/topic"` spécifie le topic auquel nous nous abonnons.
    - `-u admin -P "votre_mot_de_passe"` sont les identifiants d'authentification nécessaires pour accéder au broker.
    
17. Dans un second terminal, publiez un message sur le même topic :
    
```
mosquitto_pub -h localhost -t "test/topic" -m "Bonjour, MQTT!" -u admin -P "votre_mot_de_passe"
```

    - `mosquitto_pub` est la commande utilisée pour publier un message sur un topic.
    - `-h localhost` indique que nous nous connectons au broker Mosquitto exécuté localement.
    - `-t "test/topic"` spécifie le topic sur lequel nous publions le message.
    - `-m "Bonjour, MQTT!"` est le message envoyé au topic.
    - `-u admin -P "votre_mot_de_passe"` sont les identifiants d'authentification nécessaires pour publier sur le broker
    
![[Capture d’écran 2025-02-04 à 19.28.14.png]]

---

## 4. Installation de MQTT Explorer sur macOS

### Contexte réseau

- **Adresse IP du Raspberry Pi (serveur Mosquitto)** : `192.168.0.102`
- **Adresse IP du Mac (client MQTT Explorer)** : `192.168.0.104`

### Installation de MQTT Explorer

MQTT Explorer est une application graphique permettant de visualiser et tester les communications MQTT.

18. Téléchargez MQTT Explorer depuis [le site officiel](https://mqtt-explorer.com/).
19. Installez l'application en suivant les instructions fournies.
20. Ouvrez MQTT Explorer une fois l'installation terminée.

---

## 5. Création d'une Connexion entre MQTT Explorer et Mosquitto

### Présentation de MQTT Explorer

MQTT Explorer est un client graphique avancé permettant d'explorer les topics MQTT, de visualiser les messages échangés et de publier des messages facilement.

#### Écrans principaux :

- **Écran principal** : Affiche la liste des topics souscrits et les messages reçus en temps réel.
- **Zone de publication** : Permet d'envoyer des messages à un topic spécifique.
- **Paramètres de connexion** : Zone permettant de configurer les détails du broker MQTT.
- **Affichage des logs** : Historique des messages envoyés et reçus pour le débogage.

### Configuration de la connexion dans MQTT Explorer

21. Ouvrez MQTT Explorer sur votre Mac.
22. Cliquez sur `+ Add new connection` pour ajouter un nouveau serveur MQTT.
23. Renseignez les informations suivantes :
    - **Name** : `Mosquitto Raspberry`
    - **Broker Address** : `192.168.0.102`
    - **Port** : `1883`
    - **Username** : `admin`
    - **Password** : mot_de_passe
24. Cliquez sur `Save` puis sur `Connect` pour établir la connexion.

## 6. Test de la connexion entre le Raspberry Pi et le Mac

### Tester la publication et la souscription MQTT

#### Créer un topic sur le Raspberry Pi et observer la réponse dans MQTT Explorer

25. Sur le Raspberry Pi, ouvrez un terminal et exécutez :
```
mosquitto_pub -h 192.168.0.102 -t "test/raspberry" -m "Message depuis le Raspberry Pi" -u admin -P "votre_mot_de_passe"
```
26. Dans MQTT Explorer, vérifiez que le message apparaît sous le topic `test/raspberry`.
#### Créer un topic dans MQTT Explorer et observer la réponse sur le Raspberry Pi

27. Dans MQTT Explorer, publiez un message sur un nouveau topic, par exemple `test/mac`.
28. Sur le Raspberry Pi, ouvrez un terminal et exécutez :
```
 mosquitto_sub -h 192.168.0.102 -t "test/mac" -u admin -P "votre_mot_de_passe"
```
29. Vérifiez que le message envoyé depuis MQTT Explorer est bien affiché dans le terminal du Raspberry Pi.

## 7. Préparation de l'ESP32 pour l'envoi de messages MQTT

### Installation des packages dans Arduino IDE

30. Ouvrez Arduino IDE.
31. Allez dans `Outils > Gestionnaire de cartes`.
32. Recherchez `ESP32` et installez la dernière version de la carte ESP32.
33. Allez dans `Outils > Gérer les bibliothèques`.
34. Recherchez `PubSubClient` et installez cette bibliothèque, qui permet la gestion de la communication MQTT.

Pour l'installation ayant une carte az-delivery esp32, nous devons l'installer dans l'ide et pour ce faire voir 

https://www.az-delivery.de/en/blogs/azdelivery-blog-fur-arduino-und-raspberry-pi/esp32-jetzt-mit-boardverwalter-installieren

code de test utilisé : 
```c
#include <WiFi.h>
#include <PubSubClient.h>

// Paramètres WiFi
const char* ssid = "Votre_SSID";
const char* password = "Votre_Mot_De_Passe";

// Paramètres MQTT
const char* mqtt_server = "192.168.0.102";
const int mqtt_port = 1883;
const char* mqtt_user = "admin";
const char* mqtt_password = "votre_mot_de_passe";
const char* topic = "test/esp32";

// Création des objets WiFi et MQTT
WiFiClient espClient;
PubSubClient client(espClient);

// Fonction de connexion au WiFi
void setup_wifi() {
    delay(10);
    Serial.println("Connexion au WiFi...");
    WiFi.begin(ssid, password);

    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
    }

    Serial.println("\nWiFi connecté");
    Serial.print("Adresse IP: ");
    Serial.println(WiFi.localIP());
}

// Fonction de connexion au broker MQTT
void reconnect() {
    while (!client.connected()) {
        Serial.println("Connexion au broker MQTT...");
        if (client.connect("ESP32Client", mqtt_user, mqtt_password)) {
            Serial.println("Connecté au broker MQTT");
        } else {
            Serial.print("Échec, rc=");
            Serial.print(client.state());
            Serial.println(" Nouvelle tentative dans 5 secondes...");
            delay(5000);
        }
    }
}

void setup() {
    Serial.begin(115200);
    setup_wifi();
    client.setServer(mqtt_server, mqtt_port);
}

void loop() {
    if (!client.connected()) {
        reconnect();
    }
    client.loop();

    // Publier un message toutes les 5 secondes
    static unsigned long lastMsg = 0;
    if (millis() - lastMsg > 5000) {
        lastMsg = millis();
        const char* message = "Coucou je suis ESP32, es-tu là?";
        Serial.print("Envoi du message: ");
        Serial.println(message);
        client.publish(topic, message);
    }
}
```

