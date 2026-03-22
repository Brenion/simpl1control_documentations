# Guide complet — Nextcloud Hub + ONLYOFFICE sur TrueNAS SCALE

## Alternative européenne et self-hosted à Microsoft 365 / OneDrive

---

## Pourquoi Nextcloud Hub ?

Nextcloud Hub est une suite collaborative complète, open source, développée en Allemagne. Elle remplace l'ensemble de Microsoft 365 :

|Microsoft 365|Nextcloud Hub|
|---|---|
|OneDrive|Nextcloud Files|
|Word / Excel / PowerPoint en ligne|ONLYOFFICE intégré|
|App OneDrive desktop|Client Nextcloud desktop|
|Office desktop|ONLYOFFICE Desktop|
|App mobile OneDrive|App mobile Nextcloud|
|Teams (chat + visio)|Nextcloud Talk|
|Outlook (mail, agenda, contacts)|Nextcloud Groupware|

**Avantages clés :**

- 100% gratuit et open source
- Vos données restent chez vous (sur votre TrueNAS)
- Conforme RGPD, aucune dépendance aux serveurs américains
- Accessible depuis n'importe où via votre domaine

---

## Architecture globale

```
Internet
    ↓ HTTPS (cloud.votredomaine.com)
Cloudflare Edge
    ↓ Tunnel chiffré sortant (aucun port ouvert sur le routeur)
Cloudflared (app TrueNAS)
    ↓ HTTP local
Nextcloud (app TrueNAS)
    ↓ HTTP local interne
ONLYOFFICE Document Server (app TrueNAS)
```

> ONLYOFFICE n'est jamais exposé directement sur Internet. Nextcloud fait l'intermédiaire pour l'édition de documents.

---

## Prérequis

- TrueNAS SCALE installé et fonctionnel
- Un domaine géré par Cloudflare
- Un tunnel Cloudflare existant

---

## Partie 1 — Cloudflare : comprendre les deux mécanismes

Cloudflare Zero Trust propose deux choses distinctes qu'il ne faut pas confondre :

### 1. Published Application Routes (dans le tunnel)

Définit **où pointe une URL publique**. C'est le routing pur.

- `cloud.votredomaine.com` → `http://IP-TrueNAS:9001`
- Accessible depuis n'importe quel navigateur, sans logiciel client

### 2. Access → Applications (Cloudflare Access)

Ajoute une **couche d'authentification** devant une URL. L'utilisateur doit s'identifier via Cloudflare avant d'accéder au service.

- Utilisé pour les apps sensibles (Planka, panneau admin...)
- **Non utilisé pour Nextcloud** — les clients mobiles/desktop Nextcloud sont incompatibles avec Cloudflare Access

---

## Partie 2 — Cloudflare Access pour vos apps sensibles

Pour des apps comme Planka ou votre panneau d'administration, ajoutez une double authentification via Cloudflare Access :

1. Cloudflare Zero Trust → **Access → Applications → Add an Application → Self-Hosted**
2. Remplissez :
    - **Application name** : nom de votre app
    - **Domain** : `planka.votredomaine.com`
3. Créez une **Policy** :
    - Policy name : votre choix
    - Action : Allow
    - Rule : Email → votre adresse email autorisée
4. Sauvegardez

> L'utilisateur devra s'authentifier via Cloudflare (email OTP ou SSO) avant d'accéder à l'app. C'est la "double auth" pour vos apps sensibles.

---

## Partie 3 — Cloudflare : créer la route pour Nextcloud

Nextcloud n'utilise **pas** Cloudflare Access — uniquement une route publique.

1. Zero Trust → **Networks → Tunnels** → votre tunnel → **Edit**
2. **Published Application Routes → Add**
3. Remplissez :
    - **Subdomain** : `cloud`
    - **Domain** : votre domaine
    - **Service Type** : `HTTP`
    - **URL** : `IP-TrueNAS:PORT` (à mettre à jour après installation)
    - **Additional Settings → TLS → No TLS Verify** : ✅
4. Sauvegardez

> Le CNAME DNS est créé automatiquement par Cloudflare dans votre domaine.

---

## Partie 4 — Préparer les datasets ZFS sur TrueNAS

Dans TrueNAS → **Datasets**, créez la structure suivante :

```
pool/
└── apps/
    └── nextcloud/
        ├── appdata/     ← configuration Nextcloud, apps, thèmes
        ├── userdata/    ← fichiers des utilisateurs
        └── postgres/    ← base de données
```

**Pourquoi des datasets séparés ?** Chaque dataset peut avoir ses propres snapshots indépendants. Si la base de données est corrompue, on restaure uniquement `postgres` sans toucher aux fichiers utilisateurs.

---

## Partie 5 — Installer Nextcloud sur TrueNAS

1. TrueNAS → **Apps → Discover Apps** → **Nextcloud** → **Install**

### Credentials

|Champ|Valeur|
|---|---|
|**Admin Username**|votre choix|
|**Admin Password**|mot de passe solide|

### Redis

|Champ|Valeur|
|---|---|
|**Redis Password**|mot de passe quelconque (à noter)|

> Redis gère le cache et le verrouillage de fichiers. Obligatoire, transparent pour l'utilisateur.

### Variables d'environnement

|Name|Value|Rôle|
|---|---|---|
|`OVERWRITEHOST`|`cloud.votredomaine.com`|Hostname public de Nextcloud|
|`OVERWRITECLIURL`|`https://cloud.votredomaine.com`|URL de base pour les liens générés|
|`OVERWRITEPROTOCOL`|`https`|Force HTTPS (obligatoire derrière Cloudflare)|

> Ces variables sont nécessaires car Nextcloud est derrière un reverse proxy (Cloudflare Tunnel). Sans elles, Nextcloud génère des liens avec l'IP locale et des erreurs HTTPS apparaissent.

### Host Paths

|Champ|Path|
|---|---|
|**App Data**|`/mnt/pool/apps/nextcloud/appdata`|
|**User Data**|`/mnt/pool/apps/nextcloud/userdata`|
|**PostgreSQL**|`/mnt/pool/apps/nextcloud/postgres`|

### Options importantes

|Option|Valeur|Pourquoi|
|---|---|---|
|**Host**|⚠️ **vide**|Obligatoire sinon OVERWRITEHOST ne fonctionne pas|
|**ACL**|désactivé|Évite les conflits de permissions|
|**Automatic Permissions**|✅ activé|TrueNAS applique automatiquement les bons chown|
|**Cronjobs**|✅ activé|Tâches de maintenance automatiques|
|**Certificate**|vide|Cloudflare gère le HTTPS, pas Nextcloud|

2. **Save** → attendre le démarrage (quelques minutes)
3. Notez le **port attribué** visible dans la fiche de l'app
4. Retournez dans Cloudflare et mettez à jour l'URL de la route avec ce port

---

## Partie 6 — Installer ONLYOFFICE sur TrueNAS

1. TrueNAS → **Apps → Discover Apps** → **ONLYOFFICE Document Server** → **Install**

### Variable d'environnement obligatoire

|Name|Value|
|---|---|
|`JWT_SECRET`|mot de passe long et aléatoire ⚠️ **à noter absolument**|

> Le JWT Secret est la clé partagée entre ONLYOFFICE et Nextcloud pour sécuriser leur communication interne.

### Autres options

|Option|Valeur|
|---|---|
|**Certificate**|vide|
|**PostgreSQL data storage**|par défaut|
|**Network Config**|par défaut|

2. **Save** → attendre le démarrage
3. Notez le **port attribué** (ex: `30134`)

> Pas de route Cloudflare pour ONLYOFFICE — il communique uniquement en HTTP local avec Nextcloud.

---

## Partie 7 — Connecter ONLYOFFICE à Nextcloud

### Installer le connecteur dans Nextcloud

1. Nextcloud → **Avatar → Apps**
2. Recherchez **ONLYOFFICE** → **Télécharger et activer**

### Configurer la connexion

1. Nextcloud → **Avatar → Administration settings → ONLYOFFICE**
2. **Document Editing Service address** : `http://IP-TrueNAS:30134`
3. **JWT Secret** : le secret défini lors de l'installation ONLYOFFICE
4. **Save** → vérifiez la coche verte ✅

---

## Partie 8 — Installer le client desktop

### Mac / PC

1. Téléchargez sur **nextcloud.com/install** → section Desktop
2. Installez et lancez
3. À la connexion :
    - **Adresse** : `https://cloud.votredomaine.com`
    - **Login/mot de passe** : vos identifiants Nextcloud
4. Un dossier `Nextcloud` se crée et se synchronise automatiquement

> Coexiste sans conflit avec OneDrive sur Mac. Deux dossiers distincts, deux services totalement indépendants.

### Mobile (iOS / Android)

Installez l'app **Nextcloud** depuis l'App Store ou Google Play, connectez-vous avec la même adresse et vos identifiants.

---

## Dépannage

|Erreur|Cause|Solution|
|---|---|---|
|"URL ne commence pas par HTTPS"|Variable manquante|Ajouter `OVERWRITEPROTOCOL` = `https`|
|postgres_upgrade exit 1|Mauvaises permissions|Activer **Automatic Permissions**|
|Bloqué en "Deploying" + erreur 403|Champ Host rempli|Vider le champ **Host**|
|Coche rouge dans ONLYOFFICE|Mauvaise IP ou JWT|Vérifier l'IP du serveur et le JWT Secret|

---

## Prochaines étapes recommandées

- [ ] Activer le **2FA** dans Nextcloud (Avatar → Personal settings → Sécurité)
- [ ] Configurer les **snapshots ZFS automatiques** sur les datasets Nextcloud
- [ ] Installer **ONLYOFFICE Desktop** sur Mac/PC pour l'édition locale avec sync
- [ ] Créer des **utilisateurs supplémentaires** dans Nextcloud

---

## Glossaire

**ZFS** — Système de fichiers avancé utilisé par TrueNAS. Offre snapshots, compression, chiffrement et protection contre la corruption des données.

**Dataset** — Partition logique dans un pool ZFS avec ses propres paramètres (quota, compression, snapshots) indépendants des autres datasets.

**Pool** — Ensemble de disques physiques gérés par ZFS. Les datasets vivent à l'intérieur d'un pool.

**Snapshot** — Photo instantanée de l'état d'un dataset. Permet de restaurer des données sans backup complet.

**Redis** — Base de données en mémoire. Utilisée par Nextcloud pour le cache et le verrouillage de fichiers lors d'éditions simultanées.

**PostgreSQL** — Base de données relationnelle. Stocke toutes les métadonnées de Nextcloud (utilisateurs, partages, agenda, contacts...).

**JWT Secret** — Clé partagée entre Nextcloud et ONLYOFFICE pour sécuriser leur communication interne. Jamais exposé publiquement.

**Cloudflare Tunnel** — Connexion sortante chiffrée vers Cloudflare. Permet d'exposer un service sur Internet sans ouvrir de ports ni exposer votre IP publique.

**Published Application Routes** — Section du tunnel Cloudflare qui définit quelle URL publique pointe vers quel service local.

**Cloudflare Access** — Couche d'authentification Cloudflare ajoutée devant une app. Oblige l'utilisateur à s'identifier avant d'accéder au service.

**Reverse Proxy** — Intermédiaire entre Internet et votre serveur. Cloudflare Tunnel joue ce rôle ici.

**OVERWRITEPROTOCOL** — Variable qui force Nextcloud à générer des liens en HTTPS, nécessaire derrière un reverse proxy.