[Ignorer et passer au contenu](https://chatgpt.com/g/g-p-67901a987cf88191a7961868800ad358-domotique/c/68dfcf09-7e18-8329-b204-4d7b695bfe25#main)

## Historique de chat

[](https://chatgpt.com/)

##### Vous avez dit :

que peut faire un fortigate 60D

###### ChatGPT a dit :

Le **FortiGate 60D** est un **pare-feu UTM (Unified Threat Management)** de la gamme Fortinet, conçu pour les petites et moyennes entreprises ou agences.  
Voici ce qu’il peut faire :

### 1. **Fonctions de sécurité réseau**

- **Pare-feu (Firewall stateful)** : filtrage du trafic entrant/sortant selon des règles.
    
- **NAT/PAT** : traduction d’adresses réseau (accès Internet, DMZ).
    
- **VPN** :
    
    - IPsec VPN pour interconnexion de sites.
        
    - SSL VPN pour accès distant des utilisateurs.
        
- **Contrôle applicatif** : reconnaissance et filtrage des applications (Facebook, Skype, etc.).
    
- **IPS (Intrusion Prevention System)** : détection et blocage d’attaques réseau.
    
- **Antivirus et antimalware** (via signatures et analyses heuristiques).
    
- **Filtrage Web** : blocage de sites par catégories (ex. contenu adulte, malveillant).
    
- **Antispam** : filtrage des emails indésirables.
    

### 2. **Routage et réseau**

- Routage dynamique (OSPF, BGP, RIP).
    
- VLANs, segmentation réseau.
    
- Haute disponibilité (HA) avec un second FortiGate (actif/passif).
    
- QoS (gestion de la bande passante et priorisation du trafic).
    

### 3. **Gestion & administration**

- Interface Web (FortiOS GUI) et CLI (console/SSH).
    
- Centralisation avec **FortiManager** et reporting via **FortiAnalyzer**.
    
- Logs, alertes et statistiques en temps réel.
    

### 4. **Performance (approximative pour le 60D)**

- **Firewall throughput** : ~1 Gb/s.
    
- **VPN IPsec throughput** : ~100–200 Mb/s.
    
- **IPS throughput** : ~250 Mb/s.
    
- **Nombre max. de sessions simultanées** : ~1 million.
    
- **Utilisateurs SSL VPN** : quelques centaines max.
    

En résumé :  
👉 Le FortiGate 60D est un **pare-feu UTM compact** offrant sécurité avancée (pare-feu, VPN, filtrage web, IPS, antivirus, antispam), routage et segmentation, adapté aux petites structures qui veulent une protection réseau complète et centralisée.

Veux-tu que je te prépare un **schéma simplifié** montrant comment un FortiGate 60D s’intègre dans un réseau typique (Internet ↔ FortiGate ↔ LAN/DMZ) ?

##### Vous avez dit :

imagine que deriere j'ai une application web qui tourne et que moi et plusieurs utilisateur puissions y accéder. combien d'utilisateur et est il prévu pour cela

###### ChatGPT a dit :

Bonne question 👍

Le **FortiGate 60D** n’est **pas** un serveur d’applications, mais un **pare-feu de sécurité** qui filtre et protège le trafic. Donc, ce n’est pas lui qui “héberge” ton application web. L’application tourne sur un **serveur derrière le FortiGate**, et le FortiGate se contente de :

- **Faire passer le trafic** (NAT/port forwarding).
    
- **Protéger ton serveur** (filtrage, IPS, antivirus, etc.).
    
- **Assurer les connexions VPN/SSL** si besoin.
    

### 📌 Capacité en termes d’utilisateurs

Pour ton cas (application web derrière le FortiGate) :

- Le **nombre d’utilisateurs** dépend surtout de **ton serveur web** (CPU, RAM, base de données, etc.) et de la **connexion Internet** (bande passante).
    
- Le FortiGate 60D peut gérer environ **1 million de sessions simultanées** et **plusieurs dizaines de milliers de nouvelles connexions par seconde**, ce qui est largement suffisant pour une application classique de PME/PMI.
    
- En pratique, un 60D peut très bien encaisser **des centaines, voire quelques milliers d’utilisateurs web** selon la charge de ton application et les services UTM activés (IPS, antivirus, filtrage web).
    

👉 Exemple concret :

- Si tu fais tourner un **site interne simple (intranet, ERP, CRM, etc.)**, le FortiGate ne sera pas le facteur limitant.
    
- Si tu fais tourner un site à **fort trafic public (e-commerce, streaming)**, c’est ton **serveur et ta ligne Internet** qui satureront avant le FortiGate 60D.
    

---

Veux-tu que je t’estime plus précisément **combien d’utilisateurs simultanés ton serveur web pourrait supporter** derrière ce FortiGate, si tu me donnes :

1. La **bande passante Internet** que tu as (ex : fibre 500 Mb/s, ADSL, etc.).
    
2. Le **type d’application web** (simple site vitrine, appli métier avec base de données, e-commerce, etc.).
    
3. Les **ressources serveur** (CPU/RAM) où tourne l’app ?
    

##### Vous avez dit :

ce n'est pas la question je parle bie ndes connection ssl pas de lhebergement

###### ChatGPT a dit :

Parfait, je comprends mieux 👍 Tu parles bien de la **capacité SSL VPN** du **FortiGate 60D**, c’est-à-dire combien d’utilisateurs peuvent se connecter **en même temps** via SSL (VPN Web ou Tunnel).

### 📌 Capacité SSL VPN du FortiGate 60D

Selon les spécifications officielles Fortinet :

- **SSL VPN Throughput** : ~35 Mb/s.
    
- **Nombre max. de sessions SSL VPN simultanées** : ~200 (théorique).
    
- **Nombre recommandé d’utilisateurs simultanés** : plutôt **30 à 50 utilisateurs confortablement**, selon l’usage (navigation, RDP, applis internes).
    

En pratique :

- Si tes utilisateurs font juste du RDP ou de la petite appli métier → tu peux monter vers la centaine.
    
- Si c’est de la navigation lourde ou du transfert de fichiers → plutôt rester entre **30–50 simultanés** pour garder de bonnes performances.
    

👉 Donc :  
Le FortiGate 60D peut gérer **quelques dizaines d’utilisateurs SSL VPN simultanés de manière réaliste**, mais il n’est pas dimensionné pour des centaines d’utilisateurs connectés en même temps (il faudrait passer sur un modèle supérieur, type FortiGate 100D ou +).

Veux-tu que je te prépare un **tableau comparatif** (60D vs modèles récents, genre 60F/100F) pour voir l’évolution en capacité SSL VPN ?

##### Vous avez dit :

oui et un schema de principe entre l'applicatio net les user avec le FG 60D

###### ChatGPT a dit :

Parfait 👍 Je vais te donner les deux :

1. **Un tableau comparatif** des capacités SSL VPN du FortiGate 60D et des modèles plus récents.
    
2. **Un schéma de principe** simple qui montre la position du FortiGate 60D entre les utilisateurs distants et ton application interne.
    

---

## 1. Tableau comparatif (SSL VPN)

|Modèle|SSL VPN débit (approx.)|Sessions SSL VPN max|Utilisateurs recommandés (confort)|
|---|---|---|---|
|**FortiGate 60D** (ancien)|~35 Mb/s|200|30–50|
|**FortiGate 60F** (remplaçant du 60D)|~900 Mb/s|500|200–300|
|**FortiGate 100F** (gamme au-dessus)|~1,4 Gb/s|10 000|1 000+|
|**FortiGate 200F** (PME/ETI)|~3 Gb/s|20 000|2 000+|

👉 Tu vois que le 60D est très limité comparé aux modèles récents. Pour quelques dizaines d’utilisateurs c’est OK, mais si tu montes à plusieurs centaines, il faut envisager un **60F ou 100F**.

---

## 2. Schéma de principe

Voici un schéma logique de ton architecture :

 `Utilisateurs distants (PC, mobiles)         |     (connexion SSL VPN)         v    ┌─────────────┐    │ FortiGate   │  (60D)    │ - Pare-feu  │    │ - SSL VPN   │    │ - NAT/IPS   │    └──────┬──────┘           |           |  LAN interne sécurisé           v    ┌─────────────┐    │ Serveur Web │  (ton application interne)    └─────────────┘`

🔹 Les utilisateurs passent par un **tunnel SSL VPN** → le FortiGate authentifie, chiffre/déchiffre, filtre le trafic → puis redirige vers ton **application interne**.  
🔹 Les sessions simultanées sont limitées par la **capacité SSL VPN du FortiGate**.

---

👉 Veux-tu que je te génère une **vraie image schématique** (type diagramme réseau graphique) pour illustrer ça, au lieu du schéma texte ?

##### Vous avez dit :

comment ca marche il doivent se connecter en vpn puis ensuite a l'application ou il y a possibilité via l'application de se connecter directement avec le compte user applicatif

###### ChatGPT a dit :

Très bonne question 👌 Tu touches à la différence entre :

1. **Accès via SSL VPN classique**
    
2. **Accès direct à l’application (publishing)**
    

---

## 🔹 1. Connexion via SSL VPN classique

- L’utilisateur ouvre un **client VPN SSL** (ex : FortiClient, ou via le portail Web FortiGate).
    
- Il s’authentifie avec ses **identifiants réseau/VPN** (compte local FortiGate, LDAP/AD, RADIUS…).
    
- Une fois connecté, il est **virtuellement dans le réseau interne** (comme s’il était au bureau).
    
- Ensuite, il accède à l’application **comme un utilisateur interne** (avec ses **identifiants applicatifs**).
    

👉 Donc : **2 étapes d’authentification** :

1. Authentification VPN (accès au réseau).
    
2. Authentification applicative (accès à l’app).
    

---

## 🔹 2. Accès direct à l’application via FortiGate (sans VPN complet)

Le FortiGate peut aussi être configuré pour exposer une application web via :

- **SSL VPN Web Portal** → L’utilisateur se connecte au portail SSL (auth VPN) et voit directement un **lien vers l’application**.
    
- **Reverse Proxy / Virtual Server** (via le FortiGate ou un FortiWeb) → Tu exposes ton application sur Internet, protégée par le FortiGate.
    
- Avec un **SSO (Single Sign-On)** (par exemple via FortiAuthenticator ou intégration LDAP/AD), tu peux éviter que l’utilisateur saisisse 2 fois son mot de passe.
    

👉 Dans ce cas, le FortiGate fait office de **passerelle d’accès applicatif**, et l’utilisateur peut n’avoir qu’une seule authentification (selon la config).

---

### 🚦 En résumé :

- **Solution la plus simple/sûre sur un 60D** → Connexion VPN SSL → puis accès normal à l’app (2 identifiants).
    
- **Solution plus confortable** → Utiliser le portail SSL ou un reverse proxy + SSO, mais cela demande plus de config (et parfois des produits Fortinet complémentaires comme FortiAuthenticator/FortiWeb).
    

---

Veux-tu que je te fasse un **schéma comparatif** des 2 approches (VPN complet vs accès direct via portail/proxy) pour bien visualiser la différence ?

##### Vous avez dit :

faut il un abonnement supplémentaire?

###### ChatGPT a dit :

Bonne question 👌

### 🔹 Ce qui est inclus de base avec ton **FortiGate 60D** :

- Tu peux déjà **créer des comptes SSL VPN** (locaux ou liés à ton Active Directory) **sans abonnement**.
    
- Le **VPN SSL/IPsec** fait partie du système (FortiOS).  
    👉 Donc, pour un simple accès VPN vers ton application, **pas besoin d’abonnement supplémentaire**.
    

---

### 🔹 Ce qui nécessite un abonnement (licences FortiGuard) :

- **Services de sécurité avancés** (antivirus, IPS, web filtering, antispam, etc.).
    
- **Support/firmware updates** (contrat FortiCare).
    
- **FortiAuthenticator** (optionnel) si tu veux du SSO, du MFA ou une gestion centralisée des identités.
    
- **FortiWeb** (optionnel) si tu veux publier ton application sur Internet avec un vrai WAF (Web Application Firewall).
    

---

### 🚦 En résumé

- **VPN SSL pour accéder à ton appli** → déjà inclus, pas besoin d’abonnement.
    
- **Si tu veux plus de sécurité ou du confort (SSO, MFA, publication web sécurisée)** → ça demande des produits ou licences en plus.
    

---

👉 Veux-tu que je te fasse une **checklist claire** : _“Ce que tu peux faire gratuitement avec ton 60D”_ vs _“Ce qui nécessite un abonnement/licence”_ ?

##### Vous avez dit :

quel sont les tarif par utilisateur avec authenticator

###### ChatGPT a dit :

Le coût par utilisateur avec FortiAuthenticator dépend du **type de licence**, du **pack d’utilisateurs** et des **options de support (FortiCare, mises à jour, etc.)**. Voici ce que j’ai pu trouver comme indications (à titre indicatif) — cela peut varier selon le pays, le revendeur, les remises, les taxes, etc.

---

## 🔍 Exemples de tarifs repérés

- Pour un **FortiAuthenticator VM – licence de base pour 100 utilisateurs** : environ **1 553,99 USD** selon un revendeur. [cdw.com](https://www.cdw.com/product/fortiauthenticator-vm-license-100-users-unlimited-vcpus/2926310?utm_source=chatgpt.com)
    
- Pour une **extension / upgrade de licence de 100 utilisateurs (VM)** : ~$968,20 USD pour ajouter 100 utilisateurs supplémentaires. [corporatearmor.com](https://www.corporatearmor.com/product/fortinet-fortiauthenticator-vm-upgrade-license-100-users-fac-vm-100-ug/?srsltid=AfmBOoqb56LP02hJqyMv6-4l6g5fJMDwQ_TBhTIm4ann3ZnULjKTei4c&utm_source=chatgpt.com)
    
- Pour une licence de mise à niveau “upgrade” pour 1 000 utilisateurs sur VM : ~$2 723,49 USD. [Network Hardwares](https://www.networkhardwares.com/products/fortinet-fac-vm-1000-ug-fortinet-adds-1000-users-to-fortiauthenticator-vm-fac-vm-1000-ug?pr_prod_strat=e5_desc&pr_rec_id=ac0b4bc33&pr_rec_pid=7258935558349&pr_ref_pid=7258902626509&pr_seq=uniform&srsltid=AfmBOopLHYrwA5BKJfddzl-iyVoCiAG_pVn3T8Y-lGa7sJSrs0X0YCf1&utm_source=chatgpt.com)
    
- Dans le guide officiel FortiAuthenticator : la licence de base supporte jusqu’à 40 000 utilisateurs, et on peut l’étendre jusqu’à 240 000 utilisateurs en achetant des licences d’extension. [Fortinet](https://www.fortinet.com/content/dam/fortinet/assets/data-sheets/og-fortiauthenticator.pdf?utm_source=chatgpt.com)
    

---

## ⚙️ Estimation “par utilisateur”

Prenons l’exemple de la licence de base 100 utilisateurs à ~1 553,99 USD :

- Coût total : ~1 553,99 USD
    
- Divisé par 100 utilisateurs → ≈ **15,54 USD par utilisateur** (licence simple, sans support additionnel)
    

Si tu veux ajouter 100 utilisateurs via upgrade (~968,20 USD) :

- Ajout de 100 utilisateurs → ~968,20 USD
    
- Cela donne ~9,68 USD par utilisateur dans ce cas d’extension
    

Mais ce coût “par utilisateur” peut fortement varier selon :

- Les **remises accordées** par le revendeur
    
- Les **options de support (FortiCare, mises à jour, SLA)**
    
- Le **pays / devise / TVA locale**
    

---

Si tu veux, je peux vérifier les **tarifs en Belgique / UE** actuels pour FortiAuthenticator, pour avoir une estimation localisée – ça t’irait ?

##### Vous avez dit :

est ce que ce tarif est annuel ou definitif

###### ChatGPT a dit :

Bonne question — ça dépend du type de licence que tu achètes. Voici les détails :

---

## 🔍 Types de licences FortiAuthenticator

1. **Licence perpétuelle (“perpetual license”)**
    
    - C’est une licence “définitive” : une fois achetée, tu peux l’utiliser indéfiniment (ou tant que le matériel est opérationnel).
        
    - Toutefois, cela ne couvre **pas nécessairement** les mises à jour futures, le support, ou les services complémentaires (fortiCare, etc.).
        
    - Cette licence peut souvent être complétée par des contrats de maintenance ou de support annuels.
        
2. **Licence sous forme d’abonnement / licence basée sur le temps**
    
    - Tu paies sur une durée (annuelle ou pluriannuelle), et tant que tu payes, tu bénéficies de la licence + mises à jour + support.
        
    - Si tu ne renouvelles pas, tu perds certains droits (accès aux mises à jour, support, parfois accès aux extensions d’utilisateurs).
        
3. **Modèle mixte**
    
    - Parfois, le composant “licence de base” est perpétuel, mais les “services avancés / fonctionnalités additionnelles / mises à jour / support” sont en abonnement.
        
    - Par exemple, tu peux garder la licence de base, mais si tu veux continuer à bénéficier des nouvelles versions du firmware ou du support, tu dois souscrire une extension annuelle.
        

---

## 📄 Ce que la documentation suggère

- Dans le guide FortiAuthenticator, on mentionne que « votre licence perpétuelle se convertit en licence d’abonnement annuel » pour certaines fonctions (notamment pour les mises à jour de certificats, etc.). [Fortinet Web](https://fortinetweb.s3.amazonaws.com/docs.fortinet.com/v2/attachments/008c49c5-40b1-11ef-bfe5-fa163e15d75b/FortiAuthenticator-6.6.2-Administration_Guide.pdf?utm_source=chatgpt.com)
    
- On retrouve également des contrats **“1 Year FortiCare / support”** liés à FortiAuthenticator — ce qui suggère que le support / les services sont effectivement annuels. [Amazon](https://www.amazon.com/FORTINET-FC1-10-0ACVM-248-02-12-FortiAuthenticator-FortiCare-Contract/dp/B07BT2TKPP?utm_source=chatgpt.com)
    
- Dans les forums, certains utilisateurs parlent de frais de licence / renouvellement après un certain temps pour maintenir les fonctionnalités ou l’extension d’utilisateurs. [Reddit](https://www.reddit.com/r/fortinet/comments/17s8j77/fortiauthenticator_license/?utm_source=chatgpt.com)
    

---

## ✅ En résumé

- Le **coût par utilisateur** que je t’ai donné précédemment est très probablement une **licence de base “perpétuelle”** ou une licence “standard” (achat unique).
    
- Mais tu dois **payer chaque année** pour continuer à avoir **le support / mises à jour / services** (par exemple via FortiCare ou équivalent).
    
- Si tu veux, je peux vérifier les conditions exactes en Belgique / pour les revendeurs de ta région, pour savoir ce qui est inclus avec la licence et ce qu’il faut renouveler chaque année ?
    

##### Vous avez dit :

pour une application RGPD compliant qu'elle serait la bonne solution

###### ChatGPT a dit :

Très bonne question 👌 et très importante car le **RGPD** impose des exigences fortes en matière de sécurité, de traçabilité et de protection des données personnelles.

---

## 🔹 Besoin de base

Tu veux que tes **utilisateurs distants** accèdent à ton application interne, de manière **sécurisée** et **conforme RGPD**.  
Cela implique :

- Confidentialité (chiffrement des communications).
    
- Authentification forte (éviter juste un mot de passe).
    
- Gestion des identités (qui a accès à quoi, et pourquoi).
    
- Traçabilité (logs d’accès, audit).
    
- Protection contre les attaques web (si l’app est exposée).
    

---

## 🔹 Scénarios possibles

### 1. **Accès via SSL VPN + authentification forte**

- Utilisateurs se connectent au **SSL VPN du FortiGate**.
    
- Authentification avec **MFA (multi-factor authentication)** → ex. mot de passe + OTP (via FortiToken Mobile ou FortiAuthenticator).
    
- Une fois dans le réseau, ils accèdent à l’application.  
    ✅ Avantage : simple à mettre en place avec ton FortiGate 60D + FortiAuthenticator.  
    ❌ Inconvénient : double login (VPN + application), pas toujours ergonomique.
    

---

### 2. **Accès via portail applicatif (reverse proxy / Web SSO)**

- Tu publies l’application via le **portail SSL du FortiGate** ou via un **FortiWeb (WAF)**.
    
- Les utilisateurs se connectent au portail avec MFA.
    
- Une fois validés, ils sont redirigés automatiquement vers l’application (Single Sign-On).  
    ✅ Avantage : une seule authentification, meilleure expérience utilisateur, toujours conforme RGPD.  
    ❌ Inconvénient : demande plus de configuration (et parfois un FortiWeb en plus du FortiGate).
    

---

### 3. **Accès direct sur Internet (reverse proxy + WAF + MFA)**

- L’application est exposée via un **reverse proxy sécurisé** (FortiWeb ou autre).
    
- Authentification centralisée avec **FortiAuthenticator** (LDAP/AD, SAML, OAuth2).
    
- MFA obligatoire pour tout utilisateur.  
    ✅ Avantage : pas besoin de VPN, solution moderne et scalable, RGPD friendly.  
    ❌ Inconvénient : plus coûteux, nécessite une architecture robuste (sécurité web, durcissement serveur).
    

---

## 🔹 Solution RGPD la plus cohérente (pour toi)

Vu que tu es parti d’un **FortiGate 60D** :  
👉 La solution réaliste et RGPD compliant serait :

- **FortiGate 60D** pour le SSL VPN.
    
- **FortiAuthenticator** pour gérer l’authentification centralisée et activer le **MFA** (OTP ou push via FortiToken).
    
- **Logging activé** sur le FortiGate (et idéalement un FortiAnalyzer) pour conserver la traçabilité des accès.
    
- Si l’application doit être **publique** (pas seulement pour du personnel interne), envisager un **FortiWeb** en plus (WAF) pour sécuriser contre les attaques web (injection, XSS, etc.).
    

---

✅ Cette approche coche les cases RGPD :

- **Chiffrement** → SSL VPN.
    
- **MFA** → via Authenticator + FortiToken.
    
- **Gestion des droits / IAM** → comptes centralisés.
    
- **Traçabilité / logs** → FortiGate + éventuellement FortiAnalyzer.
    

---

Veux-tu que je te propose un **schéma d’architecture RGPD compliant** (avec VPN, Authenticator, MFA, logs) pour visualiser comment tout s’emboîte ?

##### Vous avez dit :

oui fait moi un schema sur le point 2 les autre ne sont pas interessant. un schema pas une image

###### ChatGPT a dit :

Parfait — voici **un schéma texte (ASCII)** du **point 2 : accès via portail applicatif (reverse proxy / Web SSO)** avec FortiGate 60D + FortiAuthenticator (MFA) + (optionnel) FortiWeb en frontal de l’app.

`Utilisateurs distants (PC/Mobile)         |         | 1) HTTPS vers portail SSL (MFA requis)         v +-----------------------------+ |        FortiGate 60D        | |  - Portail SSL (Reverse)    | |  - Accès applicatif         | |  - Policy + IPS + Logs      | +-------------+---------------+               |               | 2) AuthN/AuthZ (SAML/RADIUS/LDAP)               v +-----------------------------+ |     FortiAuthenticator      | |  - IdP / Proxy d’identité   | |  - MFA (FortiToken, TOTP)   | |  - Groupes / RBAC / SSO     | +-------------+---------------+               |           (Assertion SSO)               |               | 3) Rejeu SSO vers l’app (header/SAML/OIDC)               v +-----------------------------+        (Optionnel, si app exposée publiquement/ |          FortiWeb           |-------> ou besoin WAF avancé) |  - WAF / Reverse Proxy      | |  - Anti-OWASP Top 10        | |  - Offload TLS + HSTS       | +-------------+---------------+               |               | 4) HTTP(S) interne (mTLS/strict TLS recommandé)               v +-----------------------------+ |       Application Web       | |  - Trust SSO (SAML/OIDC)    | |  - Session interne          | +-----------------------------+              (5) Logs/Audit                 |                 v         +------------------+         |  FortiAnalyzer   |         |  - Central logs  |         |  - Rapports/Audit|         +------------------+`

### Flux (résumé)

1. L’utilisateur va sur le **portail SSL** du FortiGate (HTTPS).
    
2. **Authentification** via FortiAuthenticator (MFA).
    
3. Le FortiGate (ou FortiWeb si présent) **fait le SSO** vers l’application (SAML/OIDC/headers).
    
4. L’application **fait confiance** à l’assertion SSO et ouvre la session.
    
5. **Journaux centralisés** (FortiGate/FortiWeb/Authenticator) vers FortiAnalyzer pour la traçabilité RGPD.
    

### Points clés RGPD (intégrés au schéma)

- **Chiffrement bout-en-bout** : HTTPS externe, TLS interne (idéalement **mTLS** entre reverse proxy et app).
    
- **MFA obligatoire** via FortiAuthenticator.
    
- **Moindre privilège / RBAC** : groupes synchronisés (AD/LDAP) côté Authenticator.
    
- **Traçabilité** : logs d’authentification, d’accès, et d’événements de sécurité (rétention paramétrée).
    
- **Durcissement TLS** : HSTS, TLS1.2+ (ou 1.3), ciphers fortes, désactivation des suites faibles.
    

Souhaites-tu la **variante sans FortiWeb** (FortiGate seul en reverse) ou une **version “détaillée par ports/protos”** (ex: 443/HTTPS externe, 8443/SSO, 443 interne, mTLS) ?

  

ChatGPT peut commettre des erreurs. Il est recommandé de vérifier les informations importantes. Voir les préférences en matière de cookies.