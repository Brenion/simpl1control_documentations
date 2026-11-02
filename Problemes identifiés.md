### Problemes identifiés

1. `DB_PASSWORD=test123` en production (critique) Le mot de passe DB en prod est `test123`, identique à celui de dev. `db.env` confirme `POSTGRES_PASSWORD=test123`. C'est le même password que dans `docker-compose.yml` dev local — clairement un oubli de changement.

2. `MASTER_KEY_HEX` et `DB_ENC_KEY_HEX` identiques dev/prod (critique)

# development.env

MASTER_KEY_HEX=b3b8d71cb547d06e38250b4f8c30dcc9aea1bc07b4165b3a5f052707e3bdd0a0

DB_ENC_KEY_HEX=99b4df078b0da7390b303be6aec7fb42e4baca009ba755f4a2cdec35ad5c6761

# production.env — valeurs IDENTIQUES

MASTER_KEY_HEX=b3b8d71cb547d06e38250b4f8c30dcc9aea1bc07b4165b3a5f052707e3bdd0a0

DB_ENC_KEY_HEX=99b4df078b0da7390b303be6aec7fb42e4baca009ba755f4a2cdec35ad5c6761

Les clés de chiffrement dev et prod sont les mêmes — cela annule toute isolation entre environnements.

3. `DB_NAME=simpl1control_dev` en production Le nom de la base de données contient `_dev` en prod. Cosmétique mais trompeur.

4. `MQTT_START = false` avec espace (mineur) `MQTT_START = false` avec des espaces autour du `=` — certains parseurs dotenv peuvent mal interpréter cette valeur. La prod a la même syntaxe.

---

### Résumé

|Point|Statut|
|---|---|
|Architecture docker-compose / séparation fichiers|Bon|
|`FRONTEND_ORIGINS` injecté dynamiquement|Bon|
|Cookies `SECURE=true` / `SAMESITE=strict`|Bon|
|`DB_SYNCHRONIZE=false`|Bon|
|`DB_PASSWORD=test123` en prod|Critique|
|Clés chiffrement identiques dev/prod|Critique|
|`DB_NAME` contient `_dev`|Mineur|
|`MQTT_START = false` (espace)|Mineur|

Les deux points critiques concernent des secrets — ça sort du scope d'une modification de config automatisée (ce sont des valeurs à changer manuellement sur le serveur de prod). Souhaites-tu qu'on les traite maintenant ?


