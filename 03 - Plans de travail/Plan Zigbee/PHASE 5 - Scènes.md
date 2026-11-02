## PHASE 5 : Scènes (Semaines 16-17 | 18-24h)

---

> **Système de scènes unifié — concept refondu :**
>
> Il n'y a **qu'un seul type de scène applicative**. Une scène est une collection d'états cibles par device (`targetId` + `key` + `value` cible), couvrant tous les protocoles (Zigbee, MQTT custom, LOGO) via le même mécanisme.
>
> - Son activation publie les commandes MQTT sur `devices.publish` de chaque device ciblé
> - Pour un device Zigbee : la commande va sur `s1c/zigbee2mqtt/[friendlyName]/set`
> - Pour un device LOGO : la commande va sur son `devices.publish` avec le payload `{ "Q1": true }`
> - Pour un device custom MQTT : idem avec le payload adapté à ses `controls`
> - Pour un groupe Zigbee (`protocol=zigbee_group`) : une seule commande broadcast suffit via `s1c/zigbee2mqtt/[friendlyName]/set`
>
> Les zones de chauffage (LOGO, TRV) sont dans le scope des scènes via le même mécanisme — pas de traitement spécial.
>
> La source de vérité des états est toujours `device_states.state` — jamais `devices.current_state`.

---

### A - API REST Scènes

**User Stories :**

- **SC-US-PH5-01** : En tant qu'utilisateur, je veux créer une scène "Cinéma" (lumières tamisées, stores, chauffage) — une scène couvre tous les protocoles (Zigbee, MQTT custom, LOGO) via le même mécanisme de commande MQTT
- **SC-US-PH5-02** : En tant qu'utilisateur, je veux activer une scène d'un seul clic
- **SC-US-PH5-03** : En tant qu'utilisateur, je veux capturer l'état actuel pour créer une scène rapidement — le snapshot lit `device_states.state` par deviceId, couvre tous les devices actifs, tous protocoles confondus
- **SC-US-PH5-04** : En tant qu'utilisateur, je veux déclencher une scène via une automatisation

**Tâches :**

- **SC-PH5-T01** : Créer endpoint `GET /api/scenes` (liste scènes) (SC-US-PH5-01)
- **SC-PH5-T02** : Créer endpoint `POST /api/scenes` (créer manuelle ou capture) (SC-US-PH5-01, SC-US-PH5-03)
- **SC-PH5-T03** : Créer endpoint `PUT /api/scenes/:id` (modifier) (SC-US-PH5-01)
- **SC-PH5-T04** : Créer endpoint `DELETE /api/scenes/:id` (supprimer) (SC-US-PH5-01)
- **SC-PH5-T05** : Créer endpoint `POST /api/scenes/:id/activate` (activer scène) (SC-US-PH5-02)
- **SC-PH5-T06** : Créer endpoint `POST /api/scenes/capture` (snapshot états devices depuis `device_states.state`) (SC-US-PH5-03)
- **SC-PH5-T07** : Implémenter validation payloads scènes avec `fastify-type-provider-zod` (SC-US-PH5-01, SC-US-PH5-03)
- **SC-PH5-T08** : Créer tests unitaires pour controllers scènes (SC-US-PH5-01 à SC-US-PH5-04)

### B - Service d'activation scènes

**User Stories :**

- **SC-US-PH5-05** : En tant que système, je veux activer une scène en appliquant tous les états définis — l'activation publie les commandes sur le topic `devices.publish` de chaque device ciblé ; pour les groupes Zigbee, une seule commande broadcast couvre tous les membres
- **SC-US-PH5-06** : En tant que système, je veux gérer l'activation de scènes complexes (50+ devices)

**Tâches :**

- **SC-PH5-T09** : Créer service activation scène (SC-US-PH5-05)
- **SC-PH5-T10** : Implémenter récupération `device_states.state` par deviceId — tous protocoles confondus, pas `devices.current_state` (SC-US-PH5-05)
- **SC-PH5-T11** : Implémenter publication commandes MQTT pour chaque device ciblé sur `devices.publish` — pour un groupe Zigbee (`protocol=zigbee_group`), une seule commande broadcast via `s1c/zigbee2mqtt/[friendlyName]/set` (SC-US-PH5-05)
- **SC-PH5-T12** : Inclure états zones chauffage dans scènes — les zones chauffage sont des devices (`protocol=logo` ou TRV Zigbee), même mécanisme que les autres devices, pas de traitement spécial (SC-US-PH5-05)
- **SC-PH5-T13** : Gérer ordre exécution (séquentiel pour éviter surcharge) (SC-US-PH5-06)
- **SC-PH5-T14** : Implémenter timeout par device (3s max) (SC-US-PH5-05)
- **SC-PH5-T15** : Implémenter logging activation (succès/échec par device) (SC-US-PH5-05)
- **SC-PH5-T16** : Intégrer scènes comme actions dans automatisations (SC-US-PH5-04)
- **SC-PH5-T17** : Créer tests unitaires service activation scènes (SC-US-PH5-05, SC-US-PH5-06)
- **SC-PH5-T18** : Tester scènes complexes (50+ devices) (SC-US-PH5-06)

### C - Tâches frontend (accompagnent les features scènes)

**User Stories :**

- **SC-US-PH5-01** : En tant qu'utilisateur, je veux créer une scène "Cinéma" (lumières tamisées, stores, chauffage)
- **SC-US-PH5-02** : En tant qu'utilisateur, je veux activer une scène d'un seul clic
- **SC-US-PH5-03** : En tant qu'utilisateur, je veux capturer l'état actuel pour créer une scène rapidement

**Tâches :**

- **SC-PH5-FE-T01** : Définir types TypeScript `Scene`, `SceneTarget` (deviceId, key, value cible) (SC-US-PH5-01)
- **SC-PH5-FE-T02** : Créer service API `scenes` (CRUD, activate, capture) (SC-US-PH5-01/02/03)
- **SC-PH5-FE-T03** : Créer store `scenes` (liste, état d'activation en cours) (SC-US-PH5-02)
- **SC-PH5-FE-T04** : Implémenter retour temps réel après activation scène : chaque commande publiée met à jour `device_states` → WebSocket → store devices → widgets (SC-US-PH5-02)
