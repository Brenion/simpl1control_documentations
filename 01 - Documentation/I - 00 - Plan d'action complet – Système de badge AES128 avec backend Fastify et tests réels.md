## 

### 1. Base de données

- Créer les tables `badges` (uid, rôle, clé dérivée, date création) et `access_logs` (uid, horodatage, statut, rôle, source, commentaire).
    
- Implémenter les entités TypeORM correspondantes.
    
- Ajouter un seeder avec deux cartes : une admin, une employé (UID fixes des badges physiques).
    

### 2. Dérivation de clé AES128 (HKDF)

- Ajouter `MASTER_KEY` dans `.env`.
    
- Implémenter `deriveBadgeKey(uid: string): string` avec HKDF SHA256.
    
- Tester la fonction avec un UID fixe et valider la sortie attendue (test unitaire).
    

### 3. API backend (badges)

- Créer la route `POST /badges` avec :
    
    - Paramètres : `uid`, `role`
        
    - Dérivation de la clé AES128 via `deriveBadgeKey`
        
    - Insertion en base de données
        
    - Publication sur le topic MQTT `access/encoder`
        
- Ajouter les tests unitaires de cette route avec mocks de DB et MQTT.
    

### 4. Intégration MQTT backend (lecteur)

- S'abonner au topic `access/reader`.
    
- À réception :
    
    - Vérifier que l’UID est en base
        
    - Récupérer et dériver la clé AES128 attendue
        
    - Comparer avec celle lue
        
    - Identifier le rôle
        
    - Déterminer le droit d’accès (refus ou autorisé)
        
    - Publier la réponse sur `access/reader/ack`
        
    - Insérer un log dans `access_logs`
        
- Écrire des tests unitaires pour tous les cas : UID inconnu, employé, admin, UID connu + mauvaise clé.
    

### 5. Tests unitaires backend

- Tester `deriveBadgeKey` (résultat stable pour un même UID).
    
- Tester `POST /badges` (validation, insertion, MQTT publish).
    
- Tester handler `access/reader` :
    
    - Inconnu : refus
        
    - Employé : refus
        
    - Admin : autorisé
        
    - UID correct + clé fausse : refus
        
- Tester insertion correcte dans `access_logs`.
    

### 6. Encodeur ESP32 (matériel réel)

- Lire l’UID via MFRC522.
    
- Attendre message MQTT `access/encoder`.
    
- Écrire la clé dérivée dans un bloc de données (ex. bloc 8).
    
- Modifier les clés A/B, configurer les bits d’accès.
    
- Publier l’ACK sur `access/encoder/ack`.
    
- Affichage du statut sur LED et LCD (Nokia 5110).
    
- Tester encodage d’une carte admin + lecture OK par le lecteur.
    

### 7. Lecteur ESP32 (matériel réel)

- Lire UID + clé dérivée dans le bloc utilisateur (via clé B).
    
- Publier la donnée sur `access/reader`.
    
- Attendre réponse `access/reader/ack`.
    
- LED WS2812 : vert (OK), rouge (erreur), jaune (employé), bleu (reconnect).
    
- LCD : afficher "ACCÈS", "REFUS", "INCONNU".
    
- Tester lecture carte admin (OK), employé (refus), inconnue (refus), clé incorrecte (refus).
    

### 8. Sécurisation MQTT (TLS)

- Générer certificats pour encodeur, lecteur, backend.
    
- Configurer `mosquitto.conf` avec `require_certificate true`.
    
- Vérifier la connexion de chaque ESP32 via MQTT TLS.
    

### 9. Tests réels (avec matériel)

- Préparer 4 badges :
    
    - Carte admin (OK)
        
    - Carte employé (refusé)
        
    - Carte inconnue (refusé)
        
    - Carte avec clé modifiée (refusé)
        
- Scanner chaque carte avec le lecteur.
    
- Observer le comportement des LEDs, affichage LCD et logs en base.
    
- Vérifier que le backend publie bien la bonne réponse.
    

### 10. Logs et debug

- Activer les logs MQTT pour le suivi des échanges.
    
- Activer logs backend sur les décisions d’accès.
    
- Intégrer messages LCD clairs sur les ESP32.
    

### 11. Scénarios de validation

- Lancer l’encodage d’une carte depuis le backend (`POST /badges`).
    
- Scanner cette carte avec le lecteur.
    
- Comparer le retour backend, état LED et affichage LCD.
    
- Vérifier l’enregistrement correct en base (dans `access_logs`).
    
- Répéter pour chaque rôle + carte falsifiée.
    

### 12. Finalisation

- Vérifier stabilité du système : reconnexion, cohérence UID, logs complets.
    
- Exporter les logs, capturer les tests et préparer un rapport synthétique des résultats.