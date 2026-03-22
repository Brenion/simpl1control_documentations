# Rapport d'impact — Plan de travail structuré
## Post-analyse DB Phase 2 (SC-PH2-T01 + SC-PH2-TELEMETRY)

**Date :** 22 mars 2026
**Scope :** Phases 0 à 8 — analyse complète sans modification des fichiers existants
**Base de référence :** SC-PH2-T01-analyse-schema-db.md + SC-PH2-TELEMETRY-architecture.md + Décision architecturale finale.md

---

## 1. Vérification du flux logique global

Le flux des phases reste globalement pertinent dans sa logique de mise en place :

```
Phase 0 (Audit) → Phase 1 (Infra Zigbee + Docker) → Phase 2 (DB + API Devices)
→ Phase 3 (Chauffage) → Phase 4 (Automatisations) → Phase 5 (Scènes)
→ Phase 6 (Frontend) → Phase 7 (Tests + Doc) → Phase 8 (Prod + Installation physique)
```

**Ce qui tient :** les dépendances entre phases sont correctes. On ne peut pas faire Phase 3 (chauffage) sans Phase 2 (devices + schéma DB). On ne peut pas faire Phase 6 (frontend) sans Phase 3-4-5 (backend complet). Phase 7 (tests) après Phase 6 est logique. Phase 8 (prod) en dernier est correct.

**Ce qui a changé structurellement :**

- Phase 1 est terminée — ses impacts résiduels (préfixe `s1c/`, `MQTT_START=true`) se ventilent dans Phase 2+
- `packages/telemetry` est un **prérequis de développement** — il doit être bootstrappé et fonctionnel avant toute feature qui produit des données IoT. Il est donc en tête de Phase 2, pas à la fin.
- Les migrations ne sont plus toutes en Phase 2 — elles accompagnent leur feature naturelle
- Custom MQTT et LOGO suivent le même schéma logique que Zigbee (`capabilities.controls`) — Phase 3 doit en tenir compte dans l'implémentation du driver
- Le concept de deux types de scènes est abandonné — Phase 5 implémente un système de scènes applicatives unifié couvrant tous les protocoles, y compris les groupes Zigbee
- Le système `profiles → pages → cards` (Phase 2 DB) impacte Phase 6 (Frontend) qui n'en tient pas compte
- La table `rooms` disparaît du schéma — plusieurs phases en dépendaient

**Conclusion :** le flux logique est maintenu. Les ajustements sont dans le contenu des phases, pas dans leur ordre. Exception : `packages/telemetry` doit être prêt **avant** que le backend publie sur `s1c/internal/telemetry/` — soit en tout début de Phase 2.

---

## 2. État des User Stories par phase

### PHASE 0 — Audit et préparation : INTACTE

Toutes les US et tâches de Phase 0 sont valides et terminées ou non impactées par les décisions DB.

- **SC-US-PH0-01 à SC-US-PH0-08** : toutes pertinentes, non impactées
- **SC-PH0-T01 à SC-PH0-T16** : toutes valides

Aucune modification requise sur Phase 0.

---

### PHASE 1 — Infrastructure Zigbee et environnement développement : TERMINÉE — VENTILATION REQUISE

La Phase 1 est terminée. Son contenu ne doit pas être retravaillé. En revanche, deux éléments issus de Phase 1 ont un impact sur les phases suivantes et doivent être ventilés :

**1. Convention de topics MQTT avec préfixe `s1c/`** — décidée dans SC-PH2-T01 mais absente du plan Phase 1. Toutes les phases qui définissent ou utilisent des topics MQTT doivent appliquer ce préfixe. À ventiler dans :
- Phase 2 (Section E — MQTT Listener, topics `s1c/zigbee2mqtt/`, `s1c/devices/`, `s1c/internal/telemetry/`)
- Phase 3 (topics chauffage et LOGO)
- Phase 8 (ACL MQTT Mosquitto)

**2. `MQTT_START=true` en dev** — à activer dans `development.env` maintenant que le container Mosquitto est en place. Si ce n'est pas fait, le backend ne se connecte jamais au broker. À vérifier en amont de Phase 2.

---

### PHASE 2 — Modèle de données et API Devices : FORTEMENT IMPACTÉE

C'est la phase la plus touchée. Voir les chapitres Suppression / Mise à jour / Ajout ci-dessous.

---

### PHASE 3 — Gestion du chauffage : FORTEMENT IMPACTÉE

Les US métier tiennent dans l'intention mais pas dans leur implémentation. Deux points fondamentaux changent la nature du travail :

**1. Custom MQTT et LOGO suivent le même schéma logique que Zigbee** — tout device, quel que soit son protocole (`zigbee`, `mqtt`, `logo`), est représenté par `devices.capabilities.controls`. La communication passe par le même flux : MQTT reçu → UPSERT `device_states` → WebSocket. Le LOGO n'est pas un cas particulier — c'est un device avec `protocol=logo` et des `controls` qui correspondent à ses blocs Q/I/M/V. Un device custom MQTT (Arduino, capteur) suit exactement la même logique avec ses propres `controls` déclarés manuellement.

**2. Le schéma `heating_zones` / `heating_schedules` est entièrement à définir** — ces tables ne sont pas dans SC-PH2-T01. Elles doivent être conçues en Phase 3 avant tout code, en tenant compte du fait qu'une zone peut référencer des devices multi-protocole simultanément (TRV Zigbee + blocs LOGO).

---

### PHASE 4 — Automatisations : LÉGÈREMENT IMPACTÉE — RÔLE CLARIFIÉ

Les US tiennent toutes. La distinction avec les tuiles et les scènes est confirmée et structurante :

```
Automatisation (QUAND + SI)
    └── déclenche → Scène (QUOI — état cible multi-device, tous protocoles)
                        └── commande via → Device (COMMENT — publish MQTT sur devices.publish)
```

- **Tuile** = contrôle direct temps réel (action humaine immédiate)
- **Scène** = snapshot d'état nommé et rappelable ("Mode hiver matin", "Mode été")
- **Automatisation** = logique qui décide quand appliquer quoi ("du 1er octobre au 31 mars, à 6h → scène mode chauffe")

**SC-US-PH4-03** ("ajouter des conditions heure, état, température") est la logique temporelle/événementielle — ce n'est pas une scène, c'est ce qui déclenche une scène. US valide.

**SC-US-PH4-05** (historique d'exécution) est la seule US non couverte ailleurs — les tuiles n'ont pas d'historique, les scènes non plus. US valide et unique à Phase 4.

L'impact technique reste : source de vérité des états = `device_states.state` jsonb, pas un champ sur `devices`. La Section B du moteur (T10-T27) est à conserver intégralement.

---

### PHASE 5 — Scènes : FORTEMENT IMPACTÉE — REFONTE DU CONCEPT

Il n'y a plus deux types de scènes séparés. Il y a **un seul système de scènes unifié** qui suit la même logique que les groupes et scènes Zigbee décidés dans SC-PH2-T01.

**Le modèle retenu :**
- Une scène est une collection d'états cibles par device (`targetId` + `key` + `value` cible)
- Elle est stockée en DB (métadonnées + états cibles)
- Son activation publie les commandes MQTT correspondantes sur chaque device via son topic `devices.publish`
- Pour un device Zigbee (dont les groupes), la commande va sur `s1c/zigbee2mqtt/[friendlyName]/set`
- Pour un device LOGO, la commande va sur son topic `devices.publish` avec le payload `{ "Q1": true }`
- Pour un device custom MQTT, idem avec le payload adapté à ses `controls`

**Ce qui disparaît :** la distinction "scènes Zigbee firmware" vs "scènes applicatives" n'est pas pertinente du point de vue de la Phase 5. Les `zigbee_scenes` de Phase 2 sont des métadonnées Z2M de bas niveau (firmware des ampoules). Les scènes Phase 5 sont des scènes **applicatives** qui englobent tous les protocoles via le même mécanisme.

La Phase 5 doit aussi gérer les **groupes** comme entité activable (un groupe est un device `protocol=zigbee_group` — son activation publie une seule commande broadcast Z2M qui touche tous les membres simultanément via radio Zigbee).

---

### PHASE 6 — Frontend complet : FORTEMENT IMPACTÉE + TÂCHES FRONTEND MANQUANTES DANS LES PHASES PRÉCÉDENTES

Deux problèmes distincts :

**1. Architecture UI Phase 6 refondue** — profiles/pages/cards + widget system. Les US métier tiennent mais les tâches sont largement à revoir.

**2. Tâches frontend absentes dans les phases 2, 3, 4, 5** — chaque feature backend développée en Phase 2-5 a une contrepartie frontend (types TypeScript, service API, store, composant). Le plan actuel regroupe tout le frontend en Phase 6, ce qui est trop tard. Le frontend doit accompagner chaque feature backend dès son développement pour que la feature soit testable de bout en bout. Cette ventilation est à intégrer dans le nouveau plan de travail.

---

### PHASE 7 — Tests, optimisation et documentation : LÉGÈREMENT IMPACTÉE

Tests manquants sur le nouveau service telemetry, device_states, telemetry_policies. Le reste tient.

---

### PHASE 8 — Sécurisation et déploiement production : LÉGÈREMENT IMPACTÉE

Ajout de `packages/telemetry` dans docker-compose. ACL MQTT à étendre. Le reste tient.

---

---

# SUPPRESSION

## User Stories à supprimer

### Phase 2

| ID | Texte | Raison |
|---|---|---|
| SC-US-PH2-11 | Je veux organiser mes appareils par pièce | La table `rooms` est supprimée du schéma final. Il n'y a plus de notion de "pièce" comme entité DB. |

### Phase 6

| ID | Texte | Raison |
|---|---|---|
| SC-US-PH6-04 (partielle) | "section pièces (cartes cliquables)" | Le concept rooms n'existe plus. Le dashboard s'organise autour de pages/cards, pas de pièces. |

---

## Tâches à supprimer

### Phase 2 — Section B (migrations devenues caduques ou déplacées)

| ID | Texte | Raison |
|---|---|---|
| SC-PH2-T05 | Créer migration pour table `rooms` | Table supprimée du schéma final |
| SC-PH2-T07 | Créer migration pour table `heating_zones` | Déplacée en Phase 3 — migration accompagne la feature |
| SC-PH2-T08 | Créer migration pour table `heating_schedules` | Déplacée en Phase 3 |
| SC-PH2-T09 | Créer migration pour table `automations` | Déplacée en Phase 4 |
| SC-PH2-T10 | Créer migration pour table `automation_logs` | Déplacée en Phase 4 |
| SC-PH2-T11 | Créer migration pour table `scenes` | Déplacée en Phase 5 (scènes applicatives) |
| SC-PH2-T12 | Créer migration pour table `device_updates` | En suspens — décision non prise dans SC-PH2-T01 |
| SC-PH2-T13 | Implémenter modèle ORM `Room` | Table supprimée |
| SC-PH2-T15 | Implémenter modèle ORM `HeatingZone` | Déplacé Phase 3 |
| SC-PH2-T16 | Implémenter modèle ORM `HeatingSchedule` | Déplacé Phase 3 |
| SC-PH2-T17 | Implémenter modèle ORM `Automation` | Déplacé Phase 4 |
| SC-PH2-T18 | Implémenter modèle ORM `AutomationLog` | Déplacé Phase 4 |
| SC-PH2-T19 | Implémenter modèle ORM `Scene` | Déplacé Phase 5 |
| SC-PH2-T20 | Implémenter modèle ORM `DeviceUpdate` | En suspens |
| SC-PH2-T22 | Seeders `rooms` (5-8 pièces) | Table supprimée |
| SC-PH2-T24 | Seeders `heating_zones` | Déplacé Phase 3 |
| SC-PH2-T25 | Seeders `heating_schedules` | Déplacé Phase 3 |
| SC-PH2-T26 | Seeders `automations` | Déplacé Phase 4 |
| SC-PH2-T27 | Seeders `scenes` | Déplacé Phase 5 |

### Phase 2 — Section D (rooms)

| ID | Texte | Raison |
|---|---|---|
| SC-PH2-T46 | `GET /api/rooms` | Table rooms supprimée |
| SC-PH2-T47 | `POST /api/rooms` | Table rooms supprimée |

### Phase 6 — Dashboard et Gestion appareils

| ID | Texte | Raison |
|---|---|---|
| SC-PH6-T21 | Créer section pièces (cartes cliquables) | Concept rooms supprimé — remplacé par pages/cards |
| SC-PH6-T36 | Drag & drop pour assigner pièce | rooms supprimées |
| SC-PH6-T28 | Créer switch mode liste/grille | Le dashboard n'est plus une liste d'appareils — c'est une page de cards configurables. Le switch liste/grille n'a plus de sens dans ce modèle |

### Phase 2 — Section C (Impact frontend — tâches qui se chevauchent avec Phase 6)

| ID | Texte | Raison |
|---|---|---|
| SC-PH2-T35 | Créer composants frontend temporaires/basiques pour nouvelles entités (devices, rooms) | `rooms` supprimées. Les composants "temporaires" n'ont pas lieu d'être — on fait directement les vrais composants Phase 6 |

---

---

# MISE À JOUR

## User Stories à mettre à jour

### Phase 1 — TERMINÉE, impacts ventilés

Les tâches de Phase 1 ne sont plus à mettre à jour — la phase est close. Les deux points résiduels se reportent sur Phase 2 :
- Préfixe `s1c/` → à appliquer dans les tâches MQTT de Phase 2 (Section E) et Phase 3
- `MQTT_START=true` → à vérifier au démarrage de Phase 2 avant tout développement MQTT

### Phase 2

| ID | Texte actuel | Mise à jour requise |
|---|---|---|
| SC-US-PH2-03 | Créer/modifier les tables nécessaires avec migrations | Reformuler : "migrer uniquement les tables prérequis cross-feature en Phase 2 — les autres migrations suivent leur phase naturelle" |
| SC-US-PH2-08 | Voir tous mes appareils avec leur état actuel en temps réel | Préciser : l'état temps réel vient de `device_states.state` (table dédiée), pas d'un champ sur `devices` |
| SC-US-PH2-11 | Organiser mes appareils par pièce | **À remplacer** par : "En tant qu'utilisateur, je veux organiser mes tuiles dans des pages au sein de mon profil" |
| SC-US-PH2-14 | Écouter les messages MQTT et mettre à jour les états | Préciser le flux complet : UPSERT device_states + évaluation telemetry_policies + PUBLISH s1c/internal/telemetry/ + EMIT WebSocket |

### Phase 3

| ID | Texte actuel | Mise à jour requise |
|---|---|---|
| SC-US-PH3-04 | Associer des vannes thermostatiques Zigbee à chaque zone | Préciser : une zone peut référencer des devices multi-protocole (TRV Zigbee ET/OU blocs LOGO) |
| SC-US-PH3-09 | Communiquer avec le Siemens Logo via MQTT | Enrichir : le LOGO est un `device` avec `protocol=logo` et `capabilities.blocks` — la communication passe par le même flux que tout device |

### Phase 4

| ID | Texte actuel | Mise à jour requise |
|---|---|---|
| SC-US-PH4-06 | Exécuter automatiquement les automatisations selon leurs déclencheurs | Préciser : l'évaluation des conditions lit `device_states.state[key]` comme source de vérité runtime |

### Phase 5

| ID | Texte actuel | Mise à jour requise |
|---|---|---|
| SC-US-PH5-01 | Créer une scène "Cinéma" (lumières tamisées, stores, chauffage) | Préciser : une scène couvre tous les protocoles (Zigbee, MQTT custom, LOGO) via le même mécanisme de commande MQTT |
| SC-US-PH5-03 | Capturer l'état actuel pour créer une scène rapidement | Préciser : le snapshot lit `device_states.state` par deviceId — couvre tous les devices actifs, tous protocoles confondus |
| SC-US-PH5-05 | Activer une scène en appliquant tous les états définis | Reformuler : activer une scène publie les commandes sur le topic `devices.publish` de chaque device ciblé — pour les groupes Zigbee, une seule commande broadcast couvre tous les membres |

### Phase 6

| ID | Texte actuel | Mise à jour requise |
|---|---|---|
| SC-US-PH6-03 | Composants UI cohérents et réutilisables | Enrichir : les composants doivent inclure le widget system générique basé sur `capabilities.controls` |
| SC-US-PH6-04 | Dashboard avec tous mes appareils visibles | Reformuler : le dashboard est organisé en `profiles → pages → cards`, pas en liste d'appareils par pièce |
| SC-US-PH6-05 | État temps réel des appareils | Préciser : les états temps réel viennent de `device_states` via WebSocket, rendus par les widgets des cards |
| SC-US-PH6-06 | Gérer facilement mes appareils | Enrichir : la gestion d'un appareil passe par l'édition de ses `capabilities.controls` (pour les devices custom MQTT et LOGO) et la configuration de ses cards associées |
| SC-US-PH6-08 | Gérer facilement mes zones de chauffage | Enrichir : une zone de chauffage s'appuie sur des devices (`protocol=logo`, `protocol=zigbee` TRV) — l'interface doit permettre d'associer des devices à une zone et de configurer les blocs LOGO correspondants |
| SC-US-PH6-10 | Créer des automatisations de manière visuelle | Enrichir : le builder IF-THEN doit permettre de sélectionner des scènes comme actions (relation automatisation → scène définie en Phase 4-5) |
| SC-US-PH6-11 | Gérer mes scènes facilement | Enrichir : une scène cible des devices tous protocoles via `devices.publish` — l'interface doit afficher le protocole et le type de commande pour chaque device cible |

### Phase 7

| ID | Texte actuel | Mise à jour requise |
|---|---|---|
| SC-US-PH7-01 | Couverture tests backend ≥70% | Étendre : inclure `packages/telemetry` dans la cible de couverture |
| SC-US-PH7-03 | Valider stabilité et performance du système | Préciser : inclure TimescaleDB, hypertable, et service telemetry dans les tests de charge |

### Phase 8

| ID | Texte actuel | Mise à jour requise |
|---|---|---|
| SC-US-PH8-02 | Sécuriser le broker MQTT avec TLS | Ajouter : ACL à étendre pour `s1c/internal/telemetry/#` |
| SC-US-PH8-05 | Déployer le système en production | Ajouter : `packages/telemetry` dans `docker-compose.production.yml` + TimescaleDB comme service Docker |

---

## Tâches à mettre à jour

### Phase 1

| ID | Texte actuel | Mise à jour |
|---|---|---|
| SC-PH1-T13 | Définir base topic `zigbee2mqtt/` | Mettre à jour : le préfixe décidé est `s1c/` — topics = `s1c/zigbee2mqtt/`, `s1c/devices/`, `s1c/internal/telemetry/` |
| SC-PH1-T19 | Vérifier cohabitation topics MQTT | Mettre à jour : valider avec la convention `s1c/` décidée dans SC-PH2-T01 |

### Phase 2 — Section B (modèles ORM)

| ID | Texte actuel | Mise à jour |
|---|---|---|
| SC-PH2-T06 | Créer migration pour table `devices` | Le schéma a changé radicalement : supprimer `ieee_address`, `brand`, `model`, `is_online`, `current_state`, `room_id` — ajouter `protocol` enum, `friendlyName`, `adminStatus`, `capabilities` jsonb |
| SC-PH2-T14 | Implémenter modèle ORM `Device` | Refonte complète selon nouveau schéma SC-PH2-T01 |
| SC-PH2-T21 | Mettre à jour seeders existants | Adapter au nouveau schéma `devices` (capabilities jsonb, protocol enum, sans room_id) |
| SC-PH2-T23 | Seeders `devices` (15-20 devices variés) | Les devices seeders doivent inclure `capabilities.controls` complets par protocole (zigbee, mqtt, logo) |

### Phase 2 — Section D (API)

| ID | Texte actuel | Mise à jour |
|---|---|---|
| SC-PH2-T37 | `GET /api/devices` avec filtres (room, type, category, online) | Filtres à réviser : `room` supprimé, `type` → `protocol`, `online` → basé sur `device_states.technicalStatus` |
| SC-PH2-T40 | `PUT /api/devices/:id` pour modification (rename, room) | Supprimer `room` des champs modifiables, ajouter `adminStatus`, `capabilities` |
| SC-PH2-T45 | `GET /api/devices/:id/state` | Préciser : retourne `device_states` (table dédiée), pas un champ sur `devices` |
| SC-PH2-T48 | Validation des payloads (Zod ou Joi) | Fastify v5 utilise `fastify-type-provider-zod` — préciser l'outil |

### Phase 2 — Section E (MQTT)

| ID | Texte actuel | Mise à jour |
|---|---|---|
| SC-PH2-T51 | Vérifier service MQTT Listener pour `heating/#` | Mettre à jour : le topic est `s1c/devices/` ou `s1c/zigbee2mqtt/` selon le protocole, pas `heating/#` |
| SC-PH2-T52 | Implémenter mise à jour états devices en DB | Préciser le flux complet : UPSERT `device_states` + évaluation `telemetry_policies` + PUBLISH `s1c/internal/telemetry/{deviceId}/{key}` + EMIT WebSocket |
| SC-PH2-T53 | Implémenter détection nouveaux appareils Zigbee | Préciser : mapping `exposes` → `capabilities.controls` selon les règles documentées (numeric/binary/enum → widget déduit) |
| SC-PH2-T54 | Gestion online/offline avec timeout (5 min) | Le status est `technicalStatus` sur `device_states` : `online` / `offline` / `unreachable`. Timeout → job périodique sur `lastSeenAt` |

### Phase 2 — Section F (OTA)

| ID | Texte actuel | Mise à jour |
|---|---|---|
| SC-PH2-T65 | Logging historique updates (table `device_updates`) | En suspens — décision non prise. Peut être stocké en jsonb dans `device_states` ou table dédiée à décider |

### Phase 3

| ID | Texte actuel | Mise à jour |
|---|---|---|
| SC-PH3-T13 | Créer service de régulation (boucle 30s) | Préciser : lit `device_states.state[key]` pour la température courante, pas un champ dédié |
| SC-PH3-T17 | Implémenter communication MQTT avec Siemens Logo | Préciser : le LOGO est un `device` avec `protocol=logo`, `capabilities.blocks` (Q/I/M/V). La commande publie sur `devices.publish` topic avec le payload `{ "Q1": true }` |
| SC-PH3-T19 | Implémenter `SiemensLogoHeatingSource` | Enrichir : encapsule la traduction `heating_zone → blocs LOGO → commande MQTT` via l'abstraction `HeatingSource` |

### Phase 4

| ID | Texte actuel | Mise à jour |
|---|---|---|
| SC-PH4-T14 | Évaluer conditions (heure, jour, état, température) | Préciser : lecture état → `device_states.state[key]` pour tous les devices, toutes clés |
| SC-PH4-T25 | Optimiser performances (debouncing, éviter boucles infinies) | Ajouter : protéger contre le re-déclenchement d'une automatisation sur un device dans la même fenêtre de temps (source = `device_states` updaté par la commande elle-même) |

### Phase 5

| ID | Texte actuel | Mise à jour |
|---|---|---|
| SC-PH5-T10 | Récupération device_states depuis DB | Préciser : lire `device_states.state` par `deviceId` — tous protocoles confondus, pas `devices.current_state` |
| SC-PH5-T11 | Publication commandes MQTT pour tous devices | Préciser : pour chaque device ciblé, publier sur `devices.publish` (champ de la table `devices`). Pour un groupe Zigbee (`protocol=zigbee_group`), une seule commande broadcast suffit via `s1c/zigbee2mqtt/[friendlyName]/set` |
| SC-PH5-T12 | Inclure états zones chauffage dans scènes | Préciser : les zones chauffage sont des devices (`protocol=logo` ou TRV Zigbee) — pas de traitement spécial, même mécanisme que les autres devices |

### Phase 6

| ID | Texte actuel | Mise à jour |
|---|---|---|
| SC-PH6-T23 | Créer composant `DeviceCard` | Renommer en `CardWidget` — une card rend les `selectedKeys` d'une `card` en widgets selon `capabilities.controls[key].widget`. Ce n'est pas une card par device, c'est une card configurable |
| SC-PH6-T26 | Créer filtres (par pièce, type, favoris) | Supprimer filtre "pièce" — remplacer par filtre "page" et "profil" |
| SC-PH6-T27 | Mise à jour temps réel via WebSocket | Préciser : les mises à jour WebSocket portent sur `device_states.state` — le frontend met à jour le widget correspondant selon la `key` reçue |
| SC-PH6-T32 | Graphique historique états (24h) | Préciser : appel au service `packages/telemetry` (pas le backend principal) — routing automatique brut/agg_15min/agg_1d selon `timeWindow` |
| SC-PH6-T40 | Composant carte zone (température, consigne, mode, demande) | Reformuler : ce composant est en réalité un `CardWidget` avec `WidgetThermostat` pour la zone — pas un composant custom. L'affichage passe par le même widget system |
| SC-PH6-T41 | Slider/boutons ajustement température | Déjà couvert par `WidgetSlider` et `WidgetThermostat` du widget system — à fusionner avec les tâches widget system plutôt que de dupliquer |
| SC-PH6-T44 | Vue globale état Logo, graphique multi-zones | Préciser : l'état du LOGO vient de `device_states` comme tout device — pas de traitement spécial. Le graphique multi-zones appelle `packages/telemetry` par deviceId |
| SC-PH6-T55 | Section actions du builder IF-THEN (liste drag & drop) | Enrichir : les actions disponibles incluent "Activer une scène" (relation Phase 4-5) — le picker de scènes doit être intégré au builder |

### Phase 7

| ID | Texte actuel | Mise à jour |
|---|---|---|
| SC-PH7-T06 | Tests MQTT avec mock broker | Étendre : inclure le flux complet `device_states` upsert + publish `s1c/internal/telemetry/` + évaluation policies |
| SC-PH7-T39 | Optimiser requêtes DB (ajouter indexes) | Ajouter : GIN index sur `capabilities` jsonb + index sur `device_states.deviceId` + indexes TimescaleDB |
| SC-PH7-T52 | Configurer PM2 pour gestion process | Mettre à jour : PM2 remplacé par Docker restart policy (décision architecturale finale) |
| SC-PH7-T53 | Configurer watchdog systemd | Mettre à jour : Docker restart policy remplace systemd pour les services applicatifs |

### Phase 8

| ID | Texte actuel | Mise à jour |
|---|---|---|
| SC-PH8-T07 | ACL Mosquitto strictes | Ajouter : topic `s1c/internal/telemetry/#` — accessible uniquement au backend principal (publish) et au service telemetry (subscribe) |
| SC-PH8-T26 | `docker-compose.production.yml` (Mosquitto + Z2M + dongle USB) | Ajouter : `packages/telemetry` + TimescaleDB comme services Docker dans docker-compose.production |
| SC-PH8-T28 | Tests smoke post-déploiement | Ajouter : vérifier TimescaleDB opérationnel + hypertable créée + 1 aggregate calculé |

---

---

# AJOUT

## User Stories à ajouter

### Phase 2 — Nouvelles US issues des décisions DB

| ID proposé | Texte | Section |
|---|---|---|
| SC-US-PH2-NEW-01 | En tant que développeur, je veux bootstrapper le package `packages/telemetry` (Fastify v5 + TypeORM) à partir du socle backend, avec connexion TimescaleDB et client MQTT | Nouveau package telemetry |
| SC-US-PH2-NEW-02 | En tant que système, je veux historiser les données IoT dans TimescaleDB selon les policies configurées, avec agrégation automatique | Telemetry |
| SC-US-PH2-NEW-03 | En tant que super admin, je veux configurer les policies d'historisation par device et par clé de donnée | Telemetry policies |
| SC-US-PH2-NEW-04 | En tant qu'utilisateur, je veux créer et gérer des profils d'interface adaptés à mes appareils (mobile, tablette, desktop) | Profiles |
| SC-US-PH2-NEW-05 | En tant qu'utilisateur, je veux organiser mes tuiles dans des pages au sein de mon profil | Pages |
| SC-US-PH2-NEW-06 | En tant qu'utilisateur, je veux créer des tuiles configurables pointant vers un device, un groupe ou une scène Zigbee | Cards |
| SC-US-PH2-NEW-07 | En tant que système, je veux gérer les groupes Zigbee et les scènes firmware Zigbee via MQTT bridge | Zigbee groups/scenes |
| SC-US-PH2-NEW-08 | En tant qu'administrateur, je veux choisir les keys à exposer dans une tuile parmi les capacités d'un device | Cards/selectedKeys |

### Phase 3 — Custom MQTT + LOGO driver (même logique que Zigbee)

| ID proposé | Texte | Section |
|---|---|---|
| SC-US-PH3-NEW-01 | En tant que développeur, je veux que les devices custom MQTT et LOGO suivent le même flux de traitement que les devices Zigbee (`capabilities.controls`, `device_states`, WebSocket) sans traitement spécifique dans le core | Driver unifié |
| SC-US-PH3-NEW-02 | En tant que développeur, je veux un parser LOGO qui traduit les blocs Q/I/M/V en payload standardisé exploitable par le même flux MQTT que les autres devices | Parser LOGO |
| SC-US-PH3-NEW-03 | En tant que développeur, je veux définir le schéma complet de `heating_zones` et `heating_schedules` avant d'implémenter | Architecture chauffage |

### Phase 6 — Widget system

| ID proposé | Texte | Section |
|---|---|---|
| SC-US-PH6-NEW-01 | En tant que développeur, je veux un système de widgets génériques qui rendent les controls d'un device selon `capabilities.controls[key].widget` | Widget system |
| SC-US-PH6-NEW-02 | En tant qu'utilisateur, je veux voir les données historiques d'un device en modal full-screen avec sélecteur de période | Widget chart |
| SC-US-PH6-NEW-03 | En tant qu'utilisateur, je veux que mon profil soit détecté automatiquement selon mon appareil et chargé sans action manuelle | Profil auto-détection |

---

## Tâches à ajouter

### Phase 1 — TERMINÉE : impacts ventilés dans Phase 2

Ces deux points ne créent plus de nouvelles tâches en Phase 1 mais deviennent des prérequis de démarrage Phase 2 :

| ID proposé | Texte | Ventilé dans |
|---|---|---|
| SC-PH2-PRE-T01 | Vérifier que `MQTT_START=true` dans `development.env` avant tout développement MQTT en Phase 2 | Début Phase 2, avant Section E |
| SC-PH2-PRE-T02 | Vérifier que `test.env` pointe sur `localhost` pour le container Mosquitto (cohérence avec Phase 1 T23) | Début Phase 2, avant tests |
| SC-PH2-PRE-T03 | Documenter et valider la convention topics `s1c/` dans l'architecture MQTT du projet (source de vérité pour toutes les phases suivantes) | Début Phase 2, Section E |

### Phase 2 — Nouveau package telemetry (PRÉREQUIS — à faire EN PREMIER)

| ID proposé | Texte | US |
|---|---|---|
**Note d'ordre :** `packages/telemetry` doit être bootstrappé et fonctionnel **avant** que le backend commence à publier sur `s1c/internal/telemetry/`. Sans service actif pour consommer ces messages, ils sont perdus. Le service doit tourner en dev dès le début de Phase 2, en parallèle du développement backend.

| ID proposé | Texte | US | Ordre |
|---|---|---|---|
| SC-PH2-TEL-T01 | Bootstrapper `packages/telemetry` : copier socle Fastify v5 + TypeORM du backend, retirer auth/users/devices/seeders | SC-US-PH2-NEW-01 | 1er |
| SC-PH2-TEL-T02 | Configurer `data-source.ts` vers TimescaleDB (variables `TIMESCALE_HOST`, `TIMESCALE_PORT`, `TIMESCALE_DB`) | SC-US-PH2-NEW-01 | 2e |
| SC-PH2-TEL-T03 | Ajouter service `timescaledb` + service `telemetry` dans `docker-compose.yml` de dev (avec volumes persistants) | SC-US-PH2-NEW-01 | 3e |
| SC-PH2-TEL-T04 | Créer migration hypertable `device_history` (time, deviceId, key, value, raw jsonb) | SC-US-PH2-NEW-02 | 4e |
| SC-PH2-TEL-T05 | Créer Continuous Aggregate `agg_15min` (rétention 1 an) | SC-US-PH2-NEW-02 | 5e |
| SC-PH2-TEL-T06 | Créer Continuous Aggregate `agg_1d` (rétention indéfinie) | SC-US-PH2-NEW-02 | 6e |
| SC-PH2-TEL-T07 | Configurer Retention Policy : `device_history` 30 jours brut | SC-US-PH2-NEW-02 | 7e |
| SC-PH2-TEL-T08 | Implémenter client MQTT abonné à `s1c/internal/telemetry/#` | SC-US-PH2-NEW-02 | 8e |
| SC-PH2-TEL-T09 | Implémenter INSERT `device_history` à chaque message reçu | SC-US-PH2-NEW-02 | 9e |
| SC-PH2-TEL-T10 | Valider en dev : publier un message test sur `s1c/internal/telemetry/test/temperature` et vérifier l'INSERT dans TimescaleDB | SC-US-PH2-NEW-02 | 10e — validation |
| SC-PH2-TEL-T11 | Implémenter routing automatique `timeWindow` → brut / `agg_15min` / `agg_1d` | SC-US-PH2-NEW-02 | après validation |
| SC-PH2-TEL-T12 | Créer endpoint `GET /telemetry/:deviceId/:key?timeWindow=...` | SC-US-PH2-NEW-02 | après routing |

### Phase 2 — Nouvelle table telemetry_policies (backend principal)

| ID proposé | Texte | US |
|---|---|---|
| SC-PH2-POL-T01 | Créer migration `telemetry_policies` (id, deviceId FK, key, enabled, minInterval, onChangeOnly) | SC-US-PH2-NEW-03 |
| SC-PH2-POL-T02 | Implémenter modèle ORM `TelemetryPolicy` | SC-US-PH2-NEW-03 |
| SC-PH2-POL-T03 | Créer endpoints CRUD `GET/POST/PUT/DELETE /api/telemetry-policies` (super_admin uniquement) | SC-US-PH2-NEW-03 |
| SC-PH2-POL-T04 | Implémenter logique d'évaluation des policies dans le flux MQTT du backend principal | SC-US-PH2-NEW-03 |
| SC-PH2-POL-T05 | Créer seeders policies par défaut (température 120s, state onChangeOnly, batterie 3600s) | SC-US-PH2-NEW-03 |

### Phase 2 — Nouvelles tables device_states, profiles, pages, cards, zigbee_groups, zigbee_scenes

| ID proposé | Texte | US |
|---|---|---|
| SC-PH2-DS-T01 | Créer migration `device_states` (deviceId PK FK, technicalStatus, lastSeenAt, updatedAt, state jsonb) | SC-US-PH2-14 |
| SC-PH2-DS-T02 | Implémenter modèle ORM `DeviceState` | SC-US-PH2-14 |
| SC-PH2-DS-T03 | Implémenter job périodique : passer `technicalStatus=offline` si `lastSeenAt` > seuil configurable | SC-US-PH2-14 |
| SC-PH2-PR-T01 | Créer migration `profiles` (id, userId FK, name, deviceType enum, isDefault) | SC-US-PH2-NEW-04 |
| SC-PH2-PR-T02 | Créer migration `pages` (id, profileId FK, name, position) | SC-US-PH2-NEW-05 |
| SC-PH2-PR-T03 | Créer migration `cards` (id, pageId FK, name, isActive, position, targetType enum, targetId loose, selectedKeys varchar[]) | SC-US-PH2-NEW-06 |
| SC-PH2-PR-T04 | Implémenter modèles ORM `Profile`, `Page`, `Card` | SC-US-PH2-NEW-04/05/06 |
| SC-PH2-PR-T05 | Créer profil `isDefault` automatiquement à la création d'un user (règle applicative) | SC-US-PH2-NEW-04 |
| SC-PH2-PR-T06 | Créer endpoints CRUD profiles/pages/cards | SC-US-PH2-NEW-04/05/06 |
| SC-PH2-PR-T07 | Implémenter logique détection profil : User-Agent → deviceType → profil matching → fallback isDefault | SC-US-PH2-NEW-03 |
| SC-PH2-ZG-T01 | Créer migration `zigbee_groups` (id, deviceId FK, groupId int, friendlyName) | SC-US-PH2-NEW-07 |
| SC-PH2-ZG-T02 | Créer migration `zigbee_scenes` (id, groupId FK, sceneId int, name) | SC-US-PH2-NEW-07 |
| SC-PH2-ZG-T03 | Implémenter modèles ORM `ZigbeeGroup`, `ZigbeeScene` | SC-US-PH2-NEW-07 |
| SC-PH2-ZG-T04 | Créer endpoints CRUD `GET/POST/PUT/DELETE /api/zigbee-groups` | SC-US-PH2-NEW-07 |
| SC-PH2-ZG-T05 | Créer endpoints CRUD `GET/POST/PUT/DELETE /api/zigbee-groups/:id/scenes` | SC-US-PH2-NEW-07 |
| SC-PH2-ZG-T06 | Implémenter appel MQTT bridge `s1c/zigbee2mqtt/bridge/request/scene/...` pour rappel scène firmware | SC-US-PH2-NEW-07 |

### Phase 2 — Seeders manquants

| ID proposé | Texte | US |
|---|---|---|
| SC-PH2-SEED-T01 | Créer seeders `devices` réalistes avec `capabilities.controls` complets par protocole (zigbee, mqtt, logo) | SC-US-PH2-05 |
| SC-PH2-SEED-T02 | Créer seeders `profiles/pages/cards` (1 profil default, 2 pages, 5 cartes variées) | SC-US-PH2-05 |
| SC-PH2-SEED-T03 | Créer seeders `telemetry_policies` par défaut | SC-US-PH2-NEW-03 |

### Phase 3 — Architecture chauffage + driver unifié (LOGO et custom MQTT)

| ID proposé | Texte | US |
|---|---|---|
| SC-PH3-T00 | Produire document `SC-PH3-T00_schema-heating.md` : schéma `heating_zones` / `heating_schedules`, articulation avec `devices` (protocol=logo, protocol=zigbee pour TRV, protocol=mqtt pour capteurs) | SC-US-PH3-NEW-03 |
| SC-PH3-LOGO-T01 | Valider que le flux générique MQTT (UPSERT `device_states` + publish `s1c/internal/telemetry/` + EMIT WebSocket) couvre nativement les devices custom MQTT sans code spécifique | SC-US-PH3-NEW-01 |
| SC-PH3-LOGO-T02 | Implémenter parser payload MQTT LOGO → normaliser en payload clé/valeur → injecter dans le même flux générique que Zigbee et custom MQTT | SC-US-PH3-NEW-02 |
| SC-PH3-LOGO-T03 | Implémenter traduction commande `capabilities.controls` → payload LOGO (`{ "Q1": true }`) → publish sur `devices.publish` | SC-US-PH3-NEW-02 |
| SC-PH3-LOGO-T04 | Implémenter watchdog LOGO : passer `technicalStatus=unreachable` si `lastSeenAt` > 2min (même job périodique que les autres devices) | SC-US-PH3-NEW-02 |
| SC-PH3-LOGO-T05 | Créer migration `heating_zones` (schéma à définir dans SC-PH3-T00) | SC-US-PH3-01 |
| SC-PH3-LOGO-T06 | Créer migration `heating_schedules` (schéma à définir dans SC-PH3-T00) | SC-US-PH3-02 |
| SC-PH3-LOGO-T07 | Implémenter modèles ORM `HeatingZone`, `HeatingSchedule` | SC-US-PH3-01 |
| SC-PH3-LOGO-T08 | Créer seeders `heating_zones` et `heating_schedules` réalistes | SC-US-PH3-01 |

### Phase 2 — Tâches frontend à intégrer avec les features backend

Le principe : chaque feature backend livrée en Phase 2 doit avoir sa contrepartie frontend minimale pour être testable de bout en bout. Pas de composants définitifs — les types, services et stores suffisent à ce stade.

| ID proposé | Texte | Rattaché à |
|---|---|---|
| SC-PH2-FE-T01 | Définir types TypeScript `Device`, `DeviceState`, `Capability`, `Control` conformes au schéma SC-PH2-T01 | Refonte `devices` + `device_states` |
| SC-PH2-FE-T02 | Définir types TypeScript `Profile`, `Page`, `Card` | Nouvelles tables profiles/pages/cards |
| SC-PH2-FE-T03 | Définir types TypeScript `ZigbeeGroup`, `ZigbeeScene`, `TelemetryPolicy` | Nouvelles tables |
| SC-PH2-FE-T04 | Créer service API `devices` (GET liste, GET détail, POST, PUT, DELETE, POST command) | API REST devices Phase 2 Section D |
| SC-PH2-FE-T05 | Créer service API `profiles`, `pages`, `cards` (CRUD complet) | API REST profiles/pages/cards |
| SC-PH2-FE-T06 | Créer service API `zigbee-groups` et `zigbee-groups/:id/scenes` | API REST zigbee groups/scenes |
| SC-PH2-FE-T07 | Créer service API `telemetry` (GET historique avec timeWindow) vers `packages/telemetry` | Endpoint telemetry |
| SC-PH2-FE-T08 | Créer store `devices` (liste, état courant par deviceId, filtre par protocol/adminStatus) | State management |
| SC-PH2-FE-T09 | Créer store `profiles/pages/cards` (profil actif, pages du profil actif, cards par page) | State management |
| SC-PH2-FE-T10 | Implémenter client WebSocket : abonnement aux events `device_states` — mise à jour du store `devices` à chaque message | Flux temps réel Phase 2 Section E |
| SC-PH2-FE-T11 | Implémenter logique de détection de profil : User-Agent → deviceType → appel API → store profil actif | Profiles Phase 2 |

### Phase 3 — Tâches frontend à intégrer avec les features chauffage

| ID proposé | Texte | Rattaché à |
|---|---|---|
| SC-PH3-FE-T01 | Définir types TypeScript `HeatingZone`, `HeatingSchedule` (schéma défini dans SC-PH3-T00) | Migrations Phase 3 |
| SC-PH3-FE-T02 | Créer service API `heating/zones` et `heating/schedules` (CRUD + override + activate) | API REST Phase 3 Section A |
| SC-PH3-FE-T03 | Créer store `heatingZones` (zones actives, température courante depuis `device_states`, demande) | State management chauffage |
| SC-PH3-FE-T04 | Implémenter mise à jour temps réel des zones via WebSocket (température courante = `device_states.state[key]` des TRV/LOGO associés) | Flux temps réel chauffage |

### Phase 4 — Tâches frontend à intégrer avec les automatisations

| ID proposé | Texte | Rattaché à |
|---|---|---|
| SC-PH4-FE-T01 | Définir types TypeScript `Automation`, `AutomationLog`, `Trigger`, `Condition`, `Action` | API automatisations Phase 4 |
| SC-PH4-FE-T02 | Créer service API `automations` (CRUD, toggle, test, logs paginés) | API REST Phase 4 Section A |
| SC-PH4-FE-T03 | Créer store `automations` (liste, état enabled/disabled, dernière exécution) | State management |

### Phase 5 — Tâches frontend à intégrer avec les scènes

| ID proposé | Texte | Rattaché à |
|---|---|---|
| SC-PH5-FE-T01 | Définir types TypeScript `Scene`, `SceneTarget` (deviceId, key, value cible) | API scènes Phase 5 |
| SC-PH5-FE-T02 | Créer service API `scenes` (CRUD, activate, capture) | API REST Phase 5 Section A |
| SC-PH5-FE-T03 | Créer store `scenes` (liste, état d'activation en cours) | State management |
| SC-PH5-FE-T04 | Implémenter retour temps réel après activation scène : chaque commande publiée met à jour `device_states` → WebSocket → store devices → widgets | Flux activation scène |

### Phase 6 — Widget system générique

| ID proposé | Texte | US |
|---|---|---|
| SC-PH6-WG-T01 | Créer composant `WidgetDisplay` (valeur lecture seule, unité) | SC-US-PH6-NEW-01 |
| SC-PH6-WG-T02 | Créer composant `WidgetToggle` (on/off binaire, onValue/offValue) | SC-US-PH6-NEW-01 |
| SC-PH6-WG-T03 | Créer composant `WidgetSlider` (valeur numérique, min/max, unité) | SC-US-PH6-NEW-01 |
| SC-PH6-WG-T04 | Créer composant `WidgetSelect` (liste enum, valeurs configurables) | SC-US-PH6-NEW-01 |
| SC-PH6-WG-T05 | Créer composant `WidgetThermostat` (composite : température courante display + consigne slider) | SC-US-PH6-NEW-01 |
| SC-PH6-WG-T06 | Créer composant `WidgetChart` (modal full-screen, appel service telemetry, timerange selector) | SC-US-PH6-NEW-02 |
| SC-PH6-WG-T07 | Créer composant `WidgetRaw` (fallback universel — affiche payload brut jsonb) | SC-US-PH6-NEW-01 |
| SC-PH6-WG-T08 | Créer composant `CardRenderer` : lit `card.selectedKeys` + `device.capabilities.controls` → rend le bon widget par key | SC-US-PH6-NEW-01 |
| SC-PH6-WG-T09 | Créer composant `CardCompact` (vue résumée dans la page) et `CardModal` (vue complète avec tous les widgets) | SC-US-PH6-NEW-01 |

### Phase 6 — Système profiles/pages/cards

| ID proposé | Texte | US |
|---|---|---|
| SC-PH6-PP-T01 | Créer page "Mes profils" : liste, création, suppression (sauf isDefault) | SC-US-PH6-NEW-03 |
| SC-PH6-PP-T02 | Implémenter détection automatique du profil par User-Agent → deviceType → matching → fallback isDefault | SC-US-PH6-NEW-03 |
| SC-PH6-PP-T03 | Créer composant navigation par pages dans un profil (tabs ou sidebar selon deviceType) | SC-US-PH6-NEW-03 |
| SC-PH6-PP-T04 | Créer interface création/édition de cards (choix device/groupe/scène + selectedKeys) | SC-US-PH6-NEW-01 |
| SC-PH6-PP-T05 | Implémenter drag & drop des cards dans une page (mise à jour `position`) | SC-US-PH6-NEW-01 |
| SC-PH6-PP-T06 | Implémenter déplacement d'une card vers une autre page (UPDATE `pageId`) | SC-US-PH6-NEW-01 |
| SC-PH6-PP-T07 | Implémenter duplication d'une card (INSERT avec nouveau id + nouveau pageId) | SC-US-PH6-NEW-01 |

### Phase 7 — Tests manquants

| ID proposé | Texte | US |
|---|---|---|
| SC-PH7-TEL-T01 | Créer tests unitaires `telemetry_policies` (enable/disable, minInterval, onChangeOnly) | SC-US-PH7-01 |
| SC-PH7-TEL-T02 | Créer tests flux complet MQTT → `device_states` → publish `s1c/internal/telemetry/` → INSERT `device_history` | SC-US-PH7-01 |
| SC-PH7-TEL-T03 | Créer tests Continuous Aggregates TimescaleDB (vérifier calcul `agg_15min` et `agg_1d`) | SC-US-PH7-03 |
| SC-PH7-TEL-T04 | Créer tests Retention Policy (vérifier suppression chunks après 30 jours) | SC-US-PH7-03 |
| SC-PH7-TEL-T05 | Créer tests widget system frontend (CardRenderer, WidgetChart, WidgetThermostat) | SC-US-PH7-02 |

### Phase 8 — Ajustements déploiement

| ID proposé | Texte | US |
|---|---|---|
| SC-PH8-TELEMETRY-T01 | Ajouter service `telemetry` dans `docker-compose.production.yml` avec ses variables d'env propres | SC-US-PH8-05 |
| SC-PH8-TELEMETRY-T02 | Ajouter service `timescaledb` dans `docker-compose.production.yml` (volume persistant) | SC-US-PH8-05 |
| SC-PH8-TELEMETRY-T03 | Smoke test : vérifier TimescaleDB opérationnel + hypertable créée + 1 agrégat calculé | SC-US-PH8-05 |
| SC-PH8-TELEMETRY-T04 | Configurer backup TimescaleDB en production (pg_dump ou timescaledb-backup) | SC-US-PH8-05 |
| SC-PH8-TELEMETRY-T05 | Ajouter ACL MQTT pour `s1c/internal/telemetry/#` : publish=backend, subscribe=telemetry uniquement | SC-US-PH8-02 |

---

## Récapitulatif des volumes

| Phase | US supprimées | US mises à jour | US ajoutées | Tâches supprimées | Tâches mises à jour | Tâches ajoutées |
|---|---|---|---|---|---|---|
| Phase 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| Phase 1 | TERMINÉE | TERMINÉE | TERMINÉE | TERMINÉE | TERMINÉE | ventilées en Phase 2 (3 tâches prérequis) |
| Phase 2 | 2 | 4 | 8 | 20 | 10 | 50 (dont 3 prérequis Ph1 + 12 telemetry + 11 frontend) |
| Phase 3 | 0 | 3 | 3 | 0 | 3 | 12 (dont 4 frontend) |
| Phase 4 | 0 | 1 | 0 | 0 | 2 | 3 (frontend) |
| Phase 5 | 0 | 3 | 0 | 0 | 3 | 4 (frontend) |
| Phase 6 | 3 | 12 | 3 | 3 | 8 | 16 |
| Phase 7 | 0 | 2 | 0 | 0 | 3 | 5 |
| Phase 8 | 0 | 2 | 0 | 0 | 3 | 5 |
| **TOTAL** | **5** | **27** | **14** | **23** | **32** | **95** |

---

## Notes finales sur les décisions de fond

### Telemetry : prérequis de dev, pas une feature tardive
`packages/telemetry` + TimescaleDB doivent être dans `docker-compose.yml` et fonctionnels **avant** que le backend publie sur `s1c/internal/telemetry/`. Sinon chaque message publié en dev est perdu sans trace. C'est le premier bloc de travail de Phase 2.

### Custom MQTT et LOGO : même flux que Zigbee
Il n'y a pas de traitement spécifique par protocole dans le core backend. Le dispatch est : topic reçu → identifier le device par son champ `subscribe` → UPSERT `device_states` → évaluer policies → publier telemetry → émettre WebSocket. Le protocole ne change que la façon dont on parse le payload entrant (LOGO nécessite un parser Q/I/M/V → clé/valeur) et dont on formate la commande sortante. Cette logique de normalisation est encapsulée dans des adaptateurs par protocole.

### Scènes Phase 5 : système unifié multi-protocole
Il n'y a qu'un seul type de scène applicative. Elle cible des devices (ou groupes) via `targetId` + `devices.publish` + payload adapté aux `capabilities.controls` du device. Un groupe Zigbee est un device comme les autres (`protocol=zigbee_group`) — son activation envoie une seule commande broadcast. Le chauffage (LOGO, TRV) est dans le scope des scènes via le même mécanisme.
