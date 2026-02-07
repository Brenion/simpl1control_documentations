1. Structure du topic unifié
    
    - On définit une convention de nommage pour tous les topics, par exemple :  
        • Un préfixe commun : "unifyIots"  
        • Puis le type de l'appareil, par exemple "sensors", "API", "logo", etc.  
        • Puis un identifiant ou nom de l'appareil, par exemple "sensor01"  
        • Enfin une action ou canal, comme "get" (pour recevoir des données).
    - Ainsi, un topic complet ca pourrait être "unifyIots/sensors/sensor01/get".
2. Stockage dans l'entité device
    
    - Dans votre entité device, vous disposez d'une propriété (exemple : "subscribe") qui sera remplie avec ce topic formé selon la convention (pour chaque type d'appareil et son rôle).
    - Pour les nouveaux IoT qui se présentent sans abonnement prédéfini, vous pourrez leur fournir ce topic lors de leur phase de pairage ou via une configuration ultérieure.
3. Au démarrage du serveur
    
    - Avant de lancer les abonnements, une fonction (par exemple reloadAllSubscribers) interrogera la base de données pour retrouver toutes les device entities dont la propriété "subscribe" est définie.
    - Pour chacun, vous appellerez la méthode registerSubscription de votre service MQTT.
    - Ainsi, tous les appareils connus avec leur topic unifié seront mis en écoute.
4. Lorsqu'un nouveau device est ajouté
    
    - Après l'insertion d'un nouvel appareil dans la base de données (ou mise à jour de l'abonnement), vous pouvez soit :  
        • Appeler directement registerSubscription sur l'instance MQTT pour le topic de ce device,  
        • Ou relancer la fonction reloadAllSubscribers qui rebalaye la liste des abonnements et inscrit le nouveau.
    - Cela permet de recharger dynamiquement les abonnements sans arrêter ou redémarrer le backend.
5. Pour les publications
    
    - De la même façon, la structure du topic unifié s'appliquera. Par exemple, pour envoyer une commande au logo, vous publiez sur "unifyIots/API/logo/get" (« logo » étant le nom de l'appareil ou son identifiant configuré).
    - Les topics deviennent donc l’interface de communication entre le backend et les IoT, en suivant la convention définie.
      
    
modif
  
      1. Lors du démarrage du serveur, nous chargeons les variables d’environnement via dotenv et nous initialisons la connexion à la base de données, le serveur Fastify, etc.
    
6. Une fois le serveur démarré, nous préparons un objet d’options MQTT à partir des variables d’environnement et nous appelons MqttService.initAsync(options). Cela permet d’initialiser le client MQTT (en mode singleton) et de le connecter au broker (ici Mosquitto).
    
7. Dès l'initialisation, nous enregistrons (via registerSubscription) les abonnements par défaut. Par exemple, nous pouvons définir un abonnement sur le topic commun "iot/temperature". Ce callback sera appelé automatiquement dès qu’un message sera reçu sur ce topic.
    
8. Pour la gestion unifiée des IoT, nous adoptons une convention de nommage des topics (par exemple, "unifyIots/sensors/sensor01/get" pour la réception de données, ou "unifyIots/API/logo/set" pour l'envoi de commandes).
    
9. Dans nos entités device, nous stockons le topic d’abonnement (généré selon la convention) dans une propriété, par exemple "subscribe". Lors de la phase de démarrage, une fonction (comme reloadSubscribes) parcourt ces entités et enregistre chacun des abonnements auprès du client MQTT via registerSubscription.
    
10. Lorsqu’un nouveau device est ajouté (ou que ses informations changent), nous pouvons soit lancer directement d’enregistrer son abonnement en appelant registerSubscription avec le topic fourni, soit relancer reloadSubscribes afin de recharger l’ensemble des abonnements dynamiquement sans redémarrer le serveur backend.
    
11. Pour les publications, le client MQTT est utilisé via la méthode publish. Par exemple, pour envoyer une commande « set » à un appareil (exemple : "unifyIots/API/logo/set"), on appelle publish sur ce topic au moment voulu pour déclencher l’action sur l’IoT.

/src
 ├── services
 │    ├── mqtt.service.ts          // Le singleton MQTT, avec initAsync, getInstance, registerSubscription, publish, etc.
 │    └── reload-subscribes.service.ts   // Service dédié à charger/recharger les subscriptions depuis la base (ex. device entities)
 ├── features
 │    ├── devices
 │    │    ├── device.entity.ts    // Modèle de données pour les devices (avec propriété "subscribe" contenant le topic)
 │    │    ├── devices.repository.ts  // Logique d’accès aux devices
 │    └── badges
 │         └── add-route
 │              └── badges.add.route.ts
 ├── plugins
 │    ├── mqtt.ts                 // (si nécessaire) pour gérer plusieurs clients (plain, tls, …)
 │    └── error-handler.ts
 ├── seeders
 │    └── ...                   // Vos fichiers de seed pour peupler les données (devices, etc)
 └── server.ts                  // Démarrage du serveur et initialisation globale (base de données, MQTT, etc)