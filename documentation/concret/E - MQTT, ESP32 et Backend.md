
# A - Problème avec la DTH et solution 

Ayant eu plusieurs problèmes, j'ai du pour l'instant créer un code permettant de simuler un capteur DHT22

```c
/**
* @file DS3132-DHT22-V2.ino
* @version 2.0
* @date 2025-02-18
* @brief Programme permettant de lire la température et l'humidité relative à
* l'aide d'une sonde DHT22 envoyer en MQTT
*/

#include <WiFi.h>
#include <PubSubClient.h>
#include <Adafruit_Sensor.h>
#include <DHT.h>
#include <Arduino_JSON.h>

// Structure de données
struct SensorData {
	float temperature;
	float humidity;
	int errorCode; // 0 pour succès, autre valeur pour erreur
};

struct ingoingData {
	float temperature;
	float humidity;
};

// Paramètres WiFi
const char* ssid = "WhiteSideOfTheWifi";
const char* password = "1Lb3b@ck!";

// Définition des pins
#define DHT_PIN 13 // Pin de la sonde DHT22
#define DHT_TYPE DHT22

// Définition des variables
DHT dht(DHT_PIN, DHT_TYPE);
const int arraySize = 36;
int i = 0;
ingoingData* dataArray;

// Paramètres MQTT
const char* mqtt_server = "192.168.0.102";
const int mqtt_port = 1883;
const char* mqtt_user = "admin";
const char* mqtt_password = "password";
const char* topic = "test/esp32_dht22";

// Création des objets WiFi et MQTT
WiFiClient espClient;
PubSubClient client(espClient);  

void setup() {
	Serial.begin(115200);
	
	// Génération des données et stockage dans dataArray
	
	dataArray = generateData(arraySize);
	Serial.println("--------------------- Données générées : ---------------------");
	displayData(dataArray);
	Serial.println("-------------------------------------------------------------");
	
	setup_wifi();
	client.setServer(mqtt_server, mqtt_port);
	delay(300);
	Serial.println("\n\n Lecture de l'humidité et de la température \n\n ");
	delay(2700);

}

void loop() {

	if (!client.connected()) {
		reconnect();
	}
	client.loop();

	// Publier un message toutes les 5 secondes
	static unsigned long lastMsg = 0;
	if (millis() - lastMsg > 5000) {
		if(i==36){
			i=0;
		}
		
		lastMsg = millis();
		SensorData sensorData;
		sensorData = CheckSensor(dataArray[i]);
		JSONVar doc;
	
		if (sensorData.errorCode == 0) {
	
			// Créer un objet JSON
			doc["temperature"] = String(sensorData.temperature, 2);
			doc["humidity"] = String(sensorData.humidity, 2);
		
			// Convertir l'objet JSON en chaîne de caractères
			String jsonString = JSON.stringify(doc);

			Serial.print("Envoi du message: ");
			Serial.println(jsonString);
			
			client.publish(topic, jsonString.c_str());
		} else {
			
			doc["Error"] = sensorData.errorCode;
			String jsonString = JSON.stringify(doc);
			Serial.println("Erreur de lecture de la sonde, message non envoyé.");
			client.publish(topic, jsonString.c_str());
		}
		i++;
	}
}

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
			delay(2000);
		}
	}
}

/**
* @brief
* Fonction permettant de vérifier le bon fonctionnement de la sonde et d'afficher
* les données si tout est ok sinon renvoie un message d'erreur
* @return SensorData Structure contenant la température, l'humidité et un code d'erreur
*/

SensorData CheckSensor(ingoingData d) {

	SensorData data;
	data.temperature = d.temperature;
	data.humidity = d.humidity;

	if (isnan(data.humidity) || isnan(data.temperature)) {
		Serial.println("Erreur lors de la lecture de la sonde DHT22");
		data.errorCode = 1;
	} else {
		Serial.println("Température: ");
		Serial.println(String(data.temperature, 2));
		Serial.println("Humidité: ");
		Serial.println(String(data.humidity, 2));
		data.errorCode = 0;
	}
	return data;
}

ingoingData* generateData(int arraySize) {

	static ingoingData data[36]; // Utilisation d'un tableau statique
	int index = 0;
	int ascendElements = (arraySize - 12) / 2; // Exclure les cycles fixes et les éléments vides
	int descendElements = ascendElements;
	float tempIncrement = (22.99 - 15.01) / (ascendElements - 1); // Pas entre chaque valeur
	
	// 2 occurrences de 15.00°C au début
	for (int i = 0; i < 2 && index < arraySize; i++) {
		data[index].temperature = 15.00;
		data[index].humidity = random(10, 51);
		index++;
	}
	// Températures montantes
	float temp = 15.01;
	for (int i = 0; i < ascendElements && index < arraySize; i++) {
		data[index].temperature = temp;
		data[index].humidity = random(10, 51);
		temp += tempIncrement;
		index++;
	}
	// 4 occurrences de 23.00°C au sommet
	for (int i = 0; i < 4 && index < arraySize; i++) {
		data[index].temperature = 23.00;
		data[index].humidity = random(10, 51);
		index++;
	}
	// 4 cycles vides
	for (int i = 0; i < 4 && index < arraySize; i++) {
		data[index].temperature = 0; // Valeur par défaut
		data[index].humidity = 0;
		index++;
	}
	// Températures descendantes
	temp = 22.99;
	for (int i = 0; i < descendElements && index < arraySize; i++) {
		data[index].temperature = temp;
		data[index].humidity = random(10, 51);
		temp -= tempIncrement;
		index++;
	}
	// 2 occurrences de 15.00°C à la fin
	for (int i = 0; i < 2 && index < arraySize; i++) {
		data[index].temperature = 15.00;
		data[index].humidity = random(10, 51);
		index++;
	}
	// Vérification de la taille du tableau
	if (index != arraySize) {
		Serial.println("Erreur : Le tableau généré n'a pas la bonne taille.");
	}
	return data;
}

void displayData(ingoingData* data) {

	for (int i = 0; i < arraySize; i++) {
		if (data[i].temperature == 0 && data[i].humidity == 0) {
			Serial.print("Élément ");
			Serial.print(i);
			Serial.println(": Cycle vide - Données non disponibles");
		} else {
		Serial.print("Élément ");
		Serial.print(i);
		Serial.print(": Température = ");
		Serial.print(data[i].temperature);
		Serial.print("°C, Humidité = ");
		Serial.print(data[i].humidity);
		Serial.println("%");
		}
	}
}
```


# B - Communication avec le backend

Dans le dossier plugins, dans le fichier mqtt.ts

On modifie la partie serveur 

```ts
import mqtt from "mqtt";

export interface MqttOptions {
	clientId: string;
	username?: string;
	password?: string;
	keepalive: number;
	clean: boolean;
}

const mqttServer = (mqttOptions: MqttOptions) => {
	console.log(mqttOptions);
	
	try {
		const mqttClient = mqtt.connect(process.env.MQTT_BROKER_URL || "mqtt://localhost:1883", mqttOptions);

		mqttClient.on("connect", () => {
			console.log("Connected to broker MQTT");
			mqttClient.subscribe("test/esp32_dht22", (err) => {
			if (!err) {
				console.log("Abonné au topic sensor/temperature");
			}
		});

		mqttClient.publish("testy/backend", "Hello from backend");
		});

		mqttClient.on("message", (topic, message) => {
			console.log(` Message reçu sur ${topic}:`, message.toString());
		});

	} catch (error) {
		console.error("Erreur de connexion au broker MQTT:", error);
	}
}

export default mqttServer;
```

# C - Création des entités nécéssaire

01 - Entité device et dataHistory

```ts
import "reflect-metadata";
import { Entity, Column, OneToMany } from "typeorm";
import { BaseEntity } from "./base-entity.js";
import { DataHistoryEntity } from "./data-history-entity.js";

@Entity("users")
export class DeviceEntity extends BaseEntity {

	@Column({ type: "varchar" })
	deviceid: string;

	@Column({ type: "varchar" })
	name: string;
	
	@Column({ type: "varchar" })
	brand: string;
	
	@Column({ type: "varchar" })
	model: string;
	
	@Column({ type: "varchar" })
	type: string;
	
	@Column({ type: "varchar" })
	topic: string;
	
	@Column({ type: 'enum', enum: ['active', 'inactive'], default: 'active' })
	status: 'active' | 'inactive';
	
	@Column({ type: 'json', nullable: true })
	metadata: Record<string, any>;
	
	@OneToMany(() => DataHistoryEntity, (dataHistory) => dataHistory.device)
	histories: DataHistoryEntity[];

}
```

```ts
import { Entity, PrimaryGeneratedColumn, Column, ManyToOne, CreateDateColumn } from 'typeorm';
import { DeviceEntity } from './device-entity.js'; 

@Entity('data_history')
export class DataHistoryEntity {
	@PrimaryGeneratedColumn('uuid')
	id: string;
	
	@ManyToOne(() => DeviceEntity, device => device.histories)
	device: DeviceEntity;
	
	@CreateDateColumn()
	timestamp: Date;
	
	@Column({ type: 'json' })
	payload: Record<string, any>;
}
```

02 - Modification supplémentaire du code de l'esp32 afin d'avoir un model pour save les datas

```c
void loop() {

if (!client.connected()) {
	reconnect();
}

client.loop();

// Publier un message toutes les 5 secondes
static unsigned long lastMsg = 0;
if (millis() - lastMsg > 5000) {
	if(i==36){
		i=0;
	}
	
	lastMsg = millis();
	SensorData sensorData;
	sensorData = CheckSensor(dataArray[i]);
	String jsonString;
	JSONVar payload;
	JSONVar doc;
	doc["brand"] = "custom";
	doc["model"] = "sensor";
	doc["type"] = "dth";
	
	if (sensorData.errorCode == 0) {
		payload["temperature"] = String(sensorData.temperature, 2);
		payload["humidity"] = String(sensorData.humidity, 2);
		
		// Ajouter le sous-objet à l'objet principal
		doc["payload"] = payload;
		
		// Convertir l'objet JSON en chaîne de caractères
		jsonString = JSON.stringify(doc);
		
		Serial.print("Envoi du message: ");
		Serial.println(jsonString);
	} else {
		doc["Error"] = sensorData.errorCode;
		doc["payload"] = JSONVar();
		jsonString = JSON.stringify(doc);
		Serial.println("Erreur de lecture de la sonde, message non envoyé.");
	}
	
	client.publish(topic, jsonString.c_str());
	i++;
	
	}
}
```

03 - Mock du custom afin de pouvoir save les données

afin de pouvoir recevoir les données, nous avons besoin d'avoir le device dans la db. pour ce faire nous allons le seeder afin qu'il soit reconnu.

```ts
//device seeder
import { Seeder } from "@jorgebodega/typeorm-seeding";
import DeviceFactory from "../factories/device.factory.js";
import DeviceStatus from "../../enums/device-status.js";

class DeviceSeeder extends Seeder {

async run(){
	await new DeviceFactory().create({
		deviceid: 'custom-sensor-dth-01',
		brand: 'custom',
		model: 'sensor',
		status: DeviceStatus.ACTIVE,
		type: 'dth',
		topic: 'zigbee2mqtt/custom-sensor-dth-01',
		})
	}
}

export default DeviceSeeder

```

A partir de maintenant, nous allons pouvoir faire le code permettant d'enregistré les donnée dans notre base de donnée. 

# D - La table data-history

cette table, nous l'utiliserons comme une table intermédiaire. nous allons enregistré les données sur 1 semaine et nous allons agréger les données par la suite dans des tables final.

La raison est simple comme les devices telque les capteurs vont fournir des donnée en masse, exemple avecl e capteur de température qui enverra toute les 5 seconde des informations cela fera sur une semaine 120.960 entrées le but sera que ca ne depasse pas ce nombre et donc nous fonctionnerons comme ca : 
 engrange des données sur la journée et a minuit tout les donnée du meme jour de la semaine précedente seront engrangé en moyenne de 15 minutes
 lundi 1 -> lundi 7 a minuit (au passage entre dimanche et lundi évidemment) 
 mardi 2 -> mardi 7 a minuit (idem lundi a mardi)
 ...etc
 ca nous donnera 672 entrées 

dans la finalité il sera peut etre interessant de cree des base intermediaire pour les temps plus long (il est clair qu'un utilisateur ne regardera pas une journée particulier 6 mois plus tard et donc la moyenne de la journée sera suffisante pour ne pas surcharger les DB)


                           +--------------------+
                           |     PostgreSQL     |
                           |   (Historique DB)  |
                           +--------------------+
                                   ▲
                                   |
                        +----------+-----------+
                        |   Fastify Backend    |
                        | (Fastify + MQTT.js)  |
                        +----------+-----------+
                                   ▲ ▲
                Publie vers       | |         Reçoit de
    +-------------------+        Pub|Sub      +----------------------+
    |     IoT Device(s) |<------------------->|    API / Automate(s) |
    +-------------------+                    |  (ex: Siemens LOGO!)  |
                                             +----------------------+


rezfactore et mise en place des regle 

# LOGO API Communication – Spécification Backend

## ✅ CONTEXTE GLOBAL

- Le backend communique avec une **API spécifique** (le **LOGO**, un automate programmable) via MQTT.
    
- Ce comportement **n'est pas dynamique** : il est **spécifique au LOGO**.
    
- Le **backend reçoit les données des IoT** (capteurs de température), les analyse, puis **publie vers le LOGO**.
    
- Le **backend agit comme transmetteur**, pas comme décideur :
    
    - Il **ne prend aucune décision métier**.
        
    - Le **LOGO (API)** est seul responsable de l’interprétation et des actions à effectuer à partir des données reçues.
        

---

## ✅ RAISON POUR LAQUELLE CE N’EST PAS DYNAMIQUE

> Oui, un **API** (ici un automate programmable type LOGO!) a un fonctionnement propre, **car il attend des données structurées de manière rigide**, avec des **protocoles, formats et comportements bien définis**.

---

## ✅ STRUCTURE DES DONNÉES À ENVOYER AU LOGO (API)

```ts
state: {
  AUTO: { value: [0 or 1] },         // Mode automatique (boolean)
  PROGRAM: { value: [0 or 1] },      // Programme actif ? (boolean)
  RAZ: { value: [0 or 1] },          // Reset (boolean)
  temperature: { value: [number] }, // Température mesurée la plus basse
  errorStatus: { value: [0+] },     // Code d’erreur (0 = pas d’erreur)
  heatSetpoint: { value: [number] } // Température consigne (manuelle ou auto)
}
```

---

## ✅ SCÉNARIO À METTRE EN PLACE

### 🔁 Récupération des températures

- Le backend reçoit régulièrement des données MQTT provenant de différents **IoT**
    
- Chaque IoT peut utiliser **une clé différente** pour indiquer la température (ex : `"temperature"`, `"temp"`, `"t"`, etc.)
    
- Il faut **deviner dynamiquement** quelle propriété est la température
    
- Il faut **conserver la température la plus basse parmi tous les IoT actifs**
    

---

### 📤 Publication vers le LOGO

- Publier vers le topic de l’API (LOGO) uniquement :
    
    - **si une valeur change** (ex : `heatSetpoint`, `AUTO`, etc.)
        
    - ou si une **erreur** est détectée
        
- Ne jamais envoyer les mêmes données deux fois si elles n’ont pas changé
    
- Toujours structurer le message selon l’objet `state` ci-dessus
    

---

### ❗ Cas spécifiques

- **Erreur si aucune température reçue** → envoyer `errorStatus: 1` (ou autre code selon logique)
    
- **Mode AUTO vs MANUEL** :
    
    - `AUTO = 1` → comportement automatique
        
    - `AUTO = 0` → comportement forcé (manuel)
        
- `heatSetpoint` est utilisé **dans tous les cas pour l’instant**, même si ce sera affiné plus tard
    

---

## ✅ OBJECTIF TECHNIQUE

Créer un **module spécifique au LOGO** (non dynamique) qui :

1. Observe les données des IoT
    
2. Extrait les températures
    
3. Compare avec l’état actuel envoyé au LOGO
    
4. Envoie un **nouveau message MQTT** **seulement si une valeur a changé**