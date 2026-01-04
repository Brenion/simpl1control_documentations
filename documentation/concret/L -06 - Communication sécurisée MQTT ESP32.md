## 📘 Documentation technique : Communication sécurisée MQTT ESP32 ↔ Backend (Scénario d’écriture de badge)

---

### 🌟 Objectif global

Mettre en place, tester et valider l’ensemble du scénario d’enregistrement sécurisé d’un badge NFC à partir d’un ESP32 connecté via MQTT TLS (port 8883), avec feedback visuel par LED et interactions contrôlées par le backend.

---

## 🗂️ Étapes par phase

---

### 🟩 PHASE 1 – Vérification de la connexion ESP32 au broker MQTT sécurisé

1. Importer le certificat `ca.crt` dans le firmware ESP32
    
2. Configurer le firmware pour se connecter en TLS (port 8883)
    
3. Connecter l’ESP32 au broker MQTT avec validation TLS
    
4. Vérifier la réception des messages MQTT du backend
    
5. Vérifier la capacité de l’ESP32 à publier des messages MQTT sur un topic sécurisé
    
6. Confirmer la boucle ESP32 ↔ Backend fonctionne sur 8883
    
7. Observer les logs côté broker et backend
    

---

### 🟨 PHASE 2 – Implémentation du scénario d’écriture de badge

#### 📌 INITIATION

1. Le backend simule la création d’un nouvel utilisateur
    
2. Le backend envoie une commande au `writer-nfc-01` via MQTT sécurisé
    
3. Le contrôleur ESP32 valide la réception de la commande
    
4. L’ESP32 vérifie la cohérence du `seed` transmis avec celui attendu
    
5. L’ESP32 entre en **mode attente de badge** :
    
    - Démarrage du **compte à rebours de 15 secondes**
        
    - Activation de la LED bleue pour signaler l’attente
        

#### 📌 GESTION DE TIMEOUT

6. Si **aucune carte n’est présentée** dans les 15 secondes :
    
    - L’ESP32 éteint la LED bleue
        
    - La LED passe au **rouge fixe**
        
    - Un message MQTT est renvoyé au backend pour indiquer le timeout
        
    - Aucun log n’est généré côté backend
        
    - Le scénario s’arrête
        

#### 📌 DÉTECTION D’UNE CARTE

7. Si une carte est détectée avant le timeout :
    
    - La LED passe au **jaune respiration lente**
        
    - L’ESP32 envoie un message de présence de carte avec l’UID
        
    - Le backend vérifie de nouveau le `seed`
        
    - Le backend génère la `clé A`, la `clé B` et une `clé dérivée` (non stockée sur la carte)
        
    - Le backend envoie ces données à l’ESP32 pour l’écriture
        

#### 📌 ÉCRITURE SUR LA CARTE

8. À la réception des données :
    
    - L’ESP32 passe la LED à l’**orange clignotant rapide**
        
    - L’ESP32 écrit les données sur la carte
        
    - Si l’écriture échoue :
        
        - La LED passe au **rouge fixe**
            
        - Un message MQTT d’erreur est renvoyé au backend
            
        - Aucun log n’est généré
            
        - Le scénario s’arrête
            
    - Si l’écriture réussit :
        
        - La LED passe au **vert fixe**
            
        - L’ESP32 envoie une confirmation MQTT au backend
            

#### 📌 ENREGISTREMENT FINAL

9. À la réception de la confirmation :
    
    - Le backend vérifie la validité de la réponse
        
    - Un nouveau badge est enregistré en base de données (`BadgeEntity`)
        
        - Seuls `cardId`, `keyA`, `keyB`, et `userId` sont enregistrés
            
    - Le backend génère un log d’écriture réussie (dans la table de logs dédiée)
        
    - Le scénario est terminé
        

---

## 🧪 PHASE 3 – Tests de validation progressive

|Étape|Objectif|
|---|---|
|✅ Backend MQTT TLS|Déjà validé|
|✅ Broker TLS (port 8883)|Déjà en place|
|🖎 Test 1|Vérifier que l’ESP32 se connecte correctement au broker sécurisé|
|🖎 Test 2|Vérifier que l’ESP32 reçoit un message du backend en TLS|
|🖎 Test 3|Vérifier que l’ESP32 peut publier sur un topic et que le backend le reçoit|
|🟨 Test 4|Simulation complète d’un enregistrement de badge avec carte présentée|
|🔴 Test 5|Simulation complète avec carte **non présentée** (timeout LED rouge)|
|🔴 Test 6|Simulation complète avec carte **présentée** mais **écriture échouée**|
|🟩 Test 7|Simulation complète avec carte présentée et écriture réussie, DB et log validés|