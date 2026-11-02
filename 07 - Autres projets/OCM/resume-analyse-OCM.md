# Analyse transversale — Documents OCM

---

## 1. Documents de référence (sur le shared)

| Référence | Nom | Description |
|---|---|---|
| **XLSX** | `EVOLUTIONS_FUTURES_OCM.xlsx` | Backlog des évolutions futures de la plateforme OCM, regroupées par utilisateur avec estimations en heures |
| **PDF** | `demande-controle-clients-z-1.pdf` | Spécification fonctionnelle du formulaire de demande de contrôle pour les sociétés de type Z — flux, modélisation DB, paiement, timing estimatif |
| **MD** | `custom-certificate.md` | Analyse technique approfondie de l'entité custom certificate — flux métier, modèle de données, architecture trans-applicative, comparaison Option A vs Option B |

---

## 2. Liste des tâches

### 2.1. Tâches du XLSX correspondant au périmètre PDF et MD

| ID | Tâche | Concrète ? | Pertinence | Provenance |
|---|---|---|---|---|
| 8 | Interface de prise de RDV via site web ocm.be | ✅ Très concrète | Cœur du périmètre — couvre flux 1, réconciliation, notification, rotation salles | XLSX |
| 9 | Configuration des plages horaires par centre et par salle | ✅ Très concrète | Prérequis direct au flux 1 — sans ça la grille de créneaux ne peut pas être générée | XLSX |
| 1 | Template d'horaires médecin appliqué à une salle | ✅ Concrète | Prérequis à la règle de visibilité salle + date côté médecin | XLSX |
| 10 | Upload et gestion des certificats (RGPD) | ✅ Concrète | Englobé dans ID 8 — téléversement write-only flux 1 et flux 2 | XLSX |

### 2.2. Tâches du PDF non couvertes par le XLSX

| Tâche | Concrète ? | Pertinence | Provenance |
|---|---|---|---|
| Intégration passerelle de paiement (Mollie ou Stripe) + configuration compte marchand | ✅ Très concrète | Indispensable — sans ça le flux société Z ne peut pas fonctionner | PDF |
| Implémentation des webhooks de paiement | ✅ Très concrète | Indispensable pour la gestion des statuts de paiement en temps réel | PDF |
| Génération du proforma (basée sur la facture de prestations existante) | ✅ Concrète | Indispensable — preuve de paiement envoyée à la société | PDF |
| Création de la table proforma en DB | ✅ Concrète | Prérequis technique direct à la génération du proforma | PDF |
| Email de confirmation + facture proforma à la société | ✅ Concrète | Indispensable pour la communication avec la société Z | PDF |
| Email de notification au secrétariat à chaque nouvelle demande | ✅ Concrète | Nécessaire au workflow admin | PDF |
| Configuration du montant facturable par les admins | ⚠️ Question ouverte | Pertinente mais non arbitrée — bloquante si le montant n'est pas fixe | PDF |
| Email de suivi post-validation vers la société | ⚠️ Question ouverte | Pertinente mais non arbitrée — à décider avec OCM | PDF |
| Gestion des données incorrectes si identifiant société reconnu | ⚠️ Question ouverte | Pertinente — cas limite non résolu, risque d'incohérence en DB | PDF |

### 2.3. Tâches du MD non couvertes par le XLSX ni le PDF

| Tâche | Concrète ? | Pertinence | Provenance |
|---|---|---|---|
| Nouvelle entité `custom_certificate` en DB (table, migrations, modèle) | ✅ Très concrète | Fondation de tout le périmètre — sans ça rien ne fonctionne | MD |
| États du custom (`en_attente`, `validé`, `refusé`) + champ `state` enum | ✅ Très concrète | Indispensable pour le workflow secrétariat | MD |
| Cron de purge des customs `refusé` au-delà de la durée de rétention | ✅ Concrète | Indispensable pour la conformité RGPD | MD |
| App publique — route `/formulaire/patient` (flux 1) | ✅ Très concrète | Indispensable — formulaire de prise de RDV patient | MD |
| App publique — route `/formulaire/societe` (flux 2) | ✅ Très concrète | Indispensable — formulaire de demande société Z | MD |
| Protections anti-spam (captcha, rate-limiting, honeypot) | ✅ Concrète | Indispensable — route exposée sur Internet sans authentification | MD |
| Recherche fuzzy / approximative pour la réconciliation société | ✅ Concrète | Indispensable pour la qualité des données et le travail du secrétariat | MD |
| Deux onglets distincts côté admin (certificats / customs) | ✅ Concrète | Exigence métier explicite — indispensable pour le secrétariat | MD |
| Visibilité du custom côté médecin (règle salle + date) | ✅ Concrète | Indispensable pour le flux 4 — le médecin doit voir le RDV | MD |
| Restrictions visibilité champs paiement côté extranet docteur | ✅ Concrète | Données financières non destinées au médecin | MD |
| Transformation atomique custom → certificat (`@Transactional`) | ✅ Concrète | Indispensable — risque de corruption de données si non atomique | MD |
| Verrou / indicateur de concurrence d'édition sur le custom | ⚠️ Hors scope strict | Pertinente mais déclarée hors scope dans le MD — à traiter de manière transverse | MD |
| Flux 3 — bouton création custom côté extranet docteur (walk-in) | ⚠️ Optionnel | Pertinent mais économie quasi nulle si retiré (~5-10%) | MD |
| Q-OCM.1 — société non-Z passant par le formulaire Z | ⚠️ Question ouverte | Pertinente — cas limite non arbitré, peut bloquer l'implémentation | MD |
| Q-DIFF.1 — finalisation matrice des champs obligatoires | ⚠️ Question ouverte | **Bloquante avant implémentation** — les validations frontend ne peuvent pas être codées sans ça | MD |
| Q-DIFF.2 — durée des créneaux de RDV (X minutes) | ⚠️ Question ouverte | **Bloquante avant implémentation** — la grille de RDV ne peut pas être générée sans ça | MD |

> **Légende** : ✅ Concrète = tâche clairement définie, implémentable directement — ⚠️ = nécessite un arbitrage ou une décision avant développement

---

## 3. Estimations par document

### XLSX — 4 tâches correspondantes au périmètre

| ID | Tâche | Estimation |
|---|---|---|
| 8 | Interface de prise de RDV | 174h |
| 9 | Configuration des plages horaires | 30h |
| 1 | Template d'horaires médecin | 15h |
| 10 | Upload certificats (englobé dans ID 8) | 0h |
| **Total XLSX** | | **219h (~27 jours)** |

### PDF — éléments non couverts par le XLSX

| Élément | Estimation basse | Estimation haute |
|---|---|---|
| Passerelle de paiement (Mollie/Stripe) + webhooks | 24h | 40h |
| Génération du proforma | 8h | 16h |
| Table proforma en DB | 4h | 8h |
| Email de confirmation + notification secrétariat | 2h | 4h |
| **Total PDF restant** | **38h** | **68h** |

### MD — éléments non couverts par le XLSX ni le PDF

| Élément                                              | Estimation basse | Estimation haute |
| ---------------------------------------------------- | ---------------- | ---------------- |
| Cron de purge des customs `refusé`                   | 2h               | 2h               |
| Protections anti-spam                                | 8h               | 16h              |
| Recherche fuzzy réconciliation société               | 8h               | 16h              |
| Restrictions visibilité champs paiement côté docteur | 1h               | 1h               |
| **Total MD restant**                                 | **19h**          | **39h**          |

---

## 4. Estimation globale et croisement entre documents

Le principe de calcul retenu est le suivant : le **XLSX est la base**, et on ajoute uniquement ce que le PDF et le MD apportent **en plus**, en évitant tout double comptage.

Une correction a été appliquée : les "questions ouvertes" du PDF recoupent partiellement la recherche fuzzy et les restrictions de visibilité paiement déjà comptées dans le MD. Après déduction de cet overlap, le MD restant est réduit de 22h-44h à **10h-20h** et la sous-estimation PDF est ajustée en conséquence.

| Source | Temps estimé |
|---|---|
| XLSX — 4 tâches correspondantes | 219h |
| PDF — non couvert par le XLSX | 38h - 68h |
| MD — non couvert par le XLSX ni le PDF | 10h - 20h |
| **Total global** | **267h - 307h** |

**Soit environ 33 à 38 jours de développement.**

### Proportions entre documents

| Comparaison | Résultat |
|---|---|
| MD en % du total XLSX + PDF | +8.5% à +15.5% |
| MD en % du PDF brut (112h-136h) | +20% à +32% |
| Impact du retrait du flux 3 (walk-in médecin) | Économie de ~5-10% sur le MD uniquement — quasi négligeable |

---

## 5. Sous-estimation probable du PDF

> ⚠️ **Cette section contient une estimation calculé par claude basée sur l'analyse des documents, et non des données chiffrées fournies par les documents eux-mêmes.**

Le timing estimé dans le PDF (14-17 jours / 112h-136h) est vraisemblablement **sous-estimé** pour les raisons suivantes :

| Élément sous-estimé | Estimation basse | Estimation haute | Justification |
|---|---|---|---|
| Passerelle de paiement — cas d'échec, remboursements, sandbox → prod | 8h | 16h | La phase 3 alloue 5-8 jours pour paiement + proforma entiers, insuffisant pour la seule intégration paiement en conditions réelles |
| Tests sécurité + conformité RGPD sur le flux paiement | 8h | 16h | Formulaire public exposé sur Internet, non explicitement budgété dans le PDF |
| Marge phase 3 — fourchette trop large traduisant une incertitude réelle | 8h | 16h | Une fourchette de 3 jours sur une seule phase indique que la complexité n'est pas encore bien cernée |
| **Total sous-estimation estimée** | **24h** | **48h** | *(les questions ouvertes déjà comptées dans le MD ne sont pas recomptées ici)* |

Cela porterait le PDF brut de **112h-136h** à **136h-184h**, soit un dépassement potentiel de **+21% à +35%** par rapport à l'estimation initiale.

### Estimation globale corrigée avec la sous-estimation PDF

| Source | Temps estimé |
|---|---|
| XLSX — 4 tâches correspondantes | 219h |
| PDF — non couvert par le XLSX | 38h - 68h |
| PDF — sous-estimation probable *(estimation personnelle)* | 24h - 48h |
| MD — non couvert par le XLSX ni le PDF | 10h - 20h |
| **Total global corrigé** | **291h - 355h** |

**Soit environ 36 à 44 jours de développement en tenant compte des risques identifiés.**

> ⚠️ Les questions encore ouvertes (Q-DIFF.1, Q-DIFF.2, Q-OCM.1) ne sont pas chiffrées — leur réponse peut générer des développements supplémentaires imprévisibles à ce stade. Q-DIFF.1 et Q-DIFF.2 sont **bloquantes avant le démarrage de l'implémentation**.

---

## 6. Tâches couvertes uniquement par le MD

Le MD étant un document d'architecture technique, il descend à un niveau de détail absent du PDF (fonctionnel) et du XLSX (backlog haut niveau). Les tâches suivantes n'apparaissent **ni dans le XLSX ni dans le PDF** :

| Tâche | Justification |
|---|---|
| Nouvelle entité `custom_certificate` en DB (table, migrations, modèle) | Architecture technique — ni le PDF ni le XLSX ne descendent à ce niveau |
| États du custom (`en_attente`, `validé`, `refusé`) + champ `state` enum | Le PDF parle de "validé/non validé" uniquement, sans le statut `refusé` ni l'enum |
| Cron de purge des customs `refusé` | Absent des deux autres documents |
| Protections anti-spam (captcha, rate-limiting, honeypot) | Absent des deux autres documents |
| Recherche fuzzy réconciliation société | Le PDF mentionne la réconciliation mais pas la tolérance aux fautes de frappe |
| Transformation atomique custom → certificat (`@Transactional`) | Détail technique absent des deux autres |
| Restrictions visibilité champs paiement côté extranet docteur | Absent des deux autres documents |
| Deux onglets distincts côté admin (certificats / customs) | Absent des deux autres documents |
| Visibilité du custom côté médecin (règle salle + date) | Le XLSX mentionne la visibilité médecin mais pas cette règle précise |
| Flux 3 — création custom par le médecin (walk-in) | Absent des deux autres documents |
| Flux 4 — prise en charge d'un custom flux 1 par le médecin | Absent des deux autres documents |

### Conclusion

Le MD couvre effectivement **beaucoup plus** que les deux autres documents car il joue un rôle différent : c'est un **document d'architecture** qui traduit les spécifications fonctionnelles en implications techniques concrètes. Le PDF décrit **ce que** le système doit faire, le MD décrit **comment** le construire sans casser l'existant.

Cette différence de nature explique pourquoi les **219h du XLSX** et les **112-136h du PDF** ne suffisent pas à couvrir tout ce que le MD implique. Les tâches exclusivement dans le MD représentent la **couche technique invisible** des deux autres documents — elle est indispensable mais systématiquement sous-estimée car non visible dans les spécifications fonctionnelles.

En pratique, toute estimation basée uniquement sur le PDF ou le XLSX **sans tenir compte du MD** est structurellement incomplète.
