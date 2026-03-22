
## Architecture choisie

|Besoin|Solution|
|---|---|
|Accès local|`http://192.168.0.167:30034` directement|
|Accès extranet sécurisé|Cloudflare Tunnel + Cloudflare Access|
|Sécurité|Seul toi peux accéder via ton email|

**Pourquoi Cloudflare Tunnel ?**

- Pas besoin d'ouvrir de ports sur le routeur
- Pas besoin de NPM ni de reverse proxy
- HTTPS automatique géré par Cloudflare
- Cloudflare Access protège l'accès par email
- Application disponible dans le catalogue TrueNAS
- Documenté officiellement par TrueNAS

---

## Phase 1 - Nettoyage préalable

### 1.1 Ports TrueNAS

- Laisser TrueNAS sur 8080/8443, aucun changement nécessaire

### 1.2 Fermer les ports NAT du routeur

- Supprimer les règles NAT 80 et 443 créées précédemment
- Elles ne sont plus nécessaires

### 1.3 Désinstaller NPM si encore présent

- **Apps → Installed Apps → Nginx Proxy Manager → Uninstall**

### 1.4 Nettoyer Cloudflare DNS

- Aller sur **cloudflare.com → DNS → Records**
- Supprimer l'enregistrement `A` de `planka`
- On le recréera automatiquement via le tunnel

### 1.5 Corriger BASE_URL de Planka

- **Apps → Planka → Edit**
- Variable `BASE_URL` → `http://192.168.0.167:30034`
- Sauvegarder

---

## Phase 2 - Création du Cloudflare Tunnel

### 2.1 Créer le tunnel sur Cloudflare

1. Aller sur **cloudflare.com**
2. Cliquer sur **Zero Trust** dans le menu gauche
3. Aller dans **Networks → Connectors → Cloudflare Tunnels**
4. Cliquer sur **Create a Tunnel**
5. Choisir **Cloudflared** → cliquer **Next**
6. Nommer le tunnel : `truenas`
7. Cliquer **Save tunnel**
8. Sur la page suivante, cliquer sur **Docker**
9. **Copier uniquement le token** (la longue chaîne après `--token`)
10. Garder cette page ouverte

### 2.2 Installer Cloudflared sur TrueNAS

1. Aller dans **Apps → Discover Apps**
2. Chercher **Cloudflared**
3. Cliquer **Install**
4. Dans le champ **Tunnel Token** → coller le token copié
5. Tout le reste → laisser par défaut
6. Cliquer **Install**
7. Attendre que le statut passe à **Running**

### 2.3 Vérifier que le tunnel est actif

- Retourner sur Cloudflare → **Networks → Tunnels**
- Le tunnel `truenas` doit afficher le statut **HEALTHY**

---

## Phase 3 - Configuration du tunnel pour Planka

### 3.1 Ajouter Planka comme service public

1. Sur Cloudflare → **Networks → Tunnels**
2. Cliquer sur les 3 points du tunnel `truenas` → **Configure**
3. Aller dans l'onglet **Public Hostname**
4. Cliquer **Add a public hostname**
5. Remplir :

|Champ|Valeur|
|---|---|
|Subdomain|`planka`|
|Domain|`brenion-digit.com`|
|Type|`HTTP`|
|URL|`192.168.0.167:30034`|

6. Cliquer **Save hostname**

### 3.2 Vérifier le DNS Cloudflare

- Cloudflare crée automatiquement un enregistrement de type **Tunnel** pour `planka.brenion-digit.com`
- Aller dans **DNS → Records** pour confirmer

---

## Phase 4 - Sécurisation avec Cloudflare Access

### 4.1 Créer l'application Access

1. Sur Cloudflare → **Zero Trust → Access → Applications**
2. Cliquer **Add an application**
3. Choisir **Self-hosted**
4. Remplir :
    - Application name : `Planka`
    - Session Duration : `24h`
    - Domain : `planka.brenion-digit.com`
5. Cliquer **Next**

### 4.2 Créer la politique d'accès

1. Policy name : `Moi uniquement`
2. Action : **Allow**
3. Dans **Include** → choisir **Emails**
4. Entrer **ton adresse email uniquement**
5. Cliquer **Next** puis **Add application**

---

## Phase 5 - Test final

### 5.1 Test local

- Ouvrir `http://192.168.0.167:30034`
- Planka doit s'afficher et fonctionner normalement ✅

### 5.2 Test extranet

- Depuis ton téléphone en 4G (sans Wi-Fi)
- Ouvrir `https://planka.brenion-digit.com`
- Cloudflare Access demande ton email
- Tu reçois un code par email → tu le saisis
- Planka s'affiche ✅

---

## Récapitulatif final

|Élément|État|
|---|---|
|Ports ouverts sur le routeur|Aucun|
|NPM|Désinstallé|
|Tailscale|Conservé pour accès TrueNAS|
|Cloudflared|Installé sur TrueNAS|
|Cloudflare Access|Protège `planka.brenion-digit.com`|
|Accès local|`http://192.168.0.167:30034`|
|Accès extranet|`https://planka.brenion-digit.com`|