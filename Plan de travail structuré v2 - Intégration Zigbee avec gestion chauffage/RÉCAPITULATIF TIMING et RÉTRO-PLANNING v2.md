## RÉCAPITULATIF TIMING v2

> **Version :** 2.0 — Post-analyse DB Phase 2 (22 mars 2026)
>
> Les charges ont été révisées à la hausse pour refléter les nouvelles tâches intégrées : bootstrap `packages/telemetry`, nouvelles tables (`device_states`, `profiles/pages/cards`, `zigbee_groups`, `zigbee_scenes`, `telemetry_policies`), tâches frontend ventilées par phase (Phases 2 à 5), widget system Phase 6, tests TimescaleDB Phase 7, déploiement telemetry Phase 8.

| Phase | Semaines | Charge v1 (h) | Charge v2 (h) | Delta | Période |
|---|---|---|---|---|---|
| **Phase 0 : Audit** | 1-2 | 15-20h | 15-20h | = | S1-S2 Mars |
| **Phase 1 : Infra Zigbee + Dev** | 3-5 | 20-25h | 20-25h | = | S3-S5 Mars |
| **Phase 2 : Backend Devices + Telemetry** | 6-9 | 25-30h | **40-50h** | +15-20h | S6-S9 Mars-Avril |
| **Phase 3 : Chauffage** | 10-13 | 35-40h | **40-48h** | +5-8h | S10-S13 Avril-Mai |
| **Phase 4 : Automatisations** | 14-16 | 25-30h | **28-35h** | +3-5h | S14-S16 Mai |
| **Phase 5 : Scènes** | 17-18 | 15-20h | **18-24h** | +3-4h | S17-S18 Mai-Juin |
| **Phase 6 : Frontend** | 19-22 | 40-50h | **50-60h** | +10h | S19-S22 Juin-Juillet |
| **Phase 7 : Tests/Optim** | 23-25 | 25-30h | **30-38h** | +5-8h | S23-S25 Juillet-Août |
| **Phase 8 : Sécurité + Prod** | 26-28 | 25-30h | **28-35h** | +3-5h | S26-S28 Août-Septembre |
| **TOTAL** | **28 semaines** | **225-275h** | **269-335h** | **+44-60h** | **Mars → Septembre** |

**Rythme hebdomadaire suggéré :**

- **9-12h/semaine** en moyenne (2-3 soirées 2-3h + 5-7h weekend)
- Phase 2 (lourde — telemetry + migrations + frontend) : 11-13h/semaine
- Phases 3, 6 (lourdes) : 10-12h/semaine
- Phases 4, 5, 7, 8 (moyennes) : 8-10h/semaine

---

## RÉTRO-PLANNING v2 (Objectif : Production Septembre 2026)

### Démarrage : **1er Mars 2026** (phases 0 et 1 terminées)

| Mois | Semaines | Phases | Jalons clés |
|---|---|---|---|
| **Mars** | S1-S5 | Phase 0, 1 | ✓ Audit terminé (S2) — **TERMINÉ** <br>✓ Env dev configuré (S5) — **TERMINÉ** <br>✓ Zigbee2MQTT opérationnel (S5) — **TERMINÉ** |
| **Avril** | S6-S13 | Phase 2, 3 | ✓ `packages/telemetry` bootstrappé + TimescaleDB opérationnel (S7) <br>✓ API Devices complète + `device_states` + `profiles/pages/cards` (S9) <br>✓ Backend chauffage fonctionnel + driver LOGO unifié (S13) <br>✓ Schéma câblage Logo prêt (S13) |
| **Mai-Juin** | S14-S18 | Phase 4, 5 | ✓ Moteur automatisations opérationnel (S16) <br>✓ API Scènes unifiées (tous protocoles) prête (S18) |
| **Juin-Juillet** | S19-S22 | Phase 6 | ✓ Widget system complet (S20) <br>✓ Dashboard profiles/pages/cards fonctionnel (S21) <br>✓ Frontend complet responsive (S22) |
| **Juillet-Août** | S23-S25 | Phase 7 | ✓ Tests E2E passés (S24) <br>✓ Tests TimescaleDB + telemetry validés (S24) <br>✓ Documentation complète (S25) <br>✓ Système validé pré-prod (S25) |
| **Août-Septembre** | S26-S28 | Phase 8 | ✓ HTTPS/WSS configuré (S26) <br>✓ Déploiement production avec telemetry + TimescaleDB (S27) <br>✓ Installation physique Logo (S28) <br>✓ **Système opérationnel** (fin S28) |

### Points de contrôle critiques v2 :

- **Fin S7 (début Avril)** : `packages/telemetry` bootstrappé, TimescaleDB opérationnel en dev — **bloquant pour la suite**
- **Fin S9 (mi-Avril)** : Première démo interne (devices + MQTT + `device_states` + interface basique)
- **Fin S13 (mi-Mai)** : Chauffage démontrables (LOGO driver unifié + zones + régulation)
- **Fin S16 (début Juin)** : Automatisations démontrables avec scènes comme actions (use cases réels testés)
- **Fin S22 (mi-Juillet)** : Alpha release frontend (tests utilisateurs famille — widget system + profiles/pages/cards)
- **Fin S25 (mi-Août)** : Beta feature-complete (phase stabilisation + tests telemetry)
- **Fin S26 (début Septembre)** : Sécurisation terminée, préparation installation physique
- **Fin S28 (mi-Septembre)** : **Mise en production finale** ✅

---

## LIVRABLES PAR PHASE v2

- **Phase 0 :** 📄 Document audit + Plan remise à niveau + Rapport couverture tests baseline — **LIVRÉ**
- **Phase 1 :** 🔧 Env dev configuré + Z2M opérationnel + 1 device testé + Doc basculement prod — **LIVRÉ**
- **Phase 2 :** 🔌 `packages/telemetry` opérationnel (TimescaleDB + aggregates) + API Devices complète (Swagger) + Services MQTT flux complet (`s1c/`) + `device_states` + `profiles/pages/cards` + 5 devices testés + types/services/stores frontend
- **Phase 3 :** 🌡️ API Chauffage + Moteur régulation + Driver LOGO unifié + Schéma câblage Logo détaillé + types/services/stores frontend chauffage
- **Phase 4 :** ⚙️ Moteur automatisations + API + 10 scénarios validés + types/services/stores frontend automatisations
- **Phase 5 :** 🎬 API Scènes unifiées (tous protocoles) + Service activation + 5 scènes démo + types/services/stores frontend scènes
- **Phase 6 :** 🖥️ Widget system complet + Dashboard profiles/pages/cards + Interface complète responsive testée (mobile/tablet/desktop)
- **Phase 7 :** ✅ Rapport tests (≥70% couverture incluant `packages/telemetry`) + Documentation complète (user + dev + API + MQTT flows)
- **Phase 8 :** 🏠 Système production sécurisé (HTTPS/WSS + ACL MQTT `s1c/internal/telemetry/#`) + `packages/telemetry` en production + TimescaleDB avec backup + Installation physique validée

---

## DELTA DE CHARGE — EXPLICATION

| Ajout | Phase | Delta estimé |
|---|---|---|
| Bootstrap `packages/telemetry` + TimescaleDB + aggregates + retention | Phase 2 | +8-10h |
| Nouvelles tables : `device_states`, `telemetry_policies`, `profiles/pages/cards`, `zigbee_groups/scenes` | Phase 2 | +5-7h |
| Tâches frontend ventilées Phase 2 (types + services + stores) | Phase 2 | +2-3h |
| Architecture chauffage (SC-PH3-T00) + driver LOGO unifié | Phase 3 | +3-4h |
| Tâches frontend ventilées Phase 3 | Phase 3 | +2-4h |
| Tâches frontend ventilées Phase 4 | Phase 4 | +2-3h |
| Tâches frontend ventilées Phase 5 | Phase 5 | +2-3h |
| Widget system générique (9 composants) | Phase 6 | +5-6h |
| Système profiles/pages/cards UI | Phase 6 | +4-5h |
| Tests `packages/telemetry` + TimescaleDB | Phase 7 | +3-5h |
| Déploiement telemetry + TimescaleDB production | Phase 8 | +2-3h |
| ACL MQTT `s1c/internal/telemetry/#` | Phase 8 | +1-2h |
| **TOTAL DELTA** | | **+44-60h** |

---

## Buffer et risques v2

- **Buffer intégré :** 2-3 semaines (si dépassement, réduire tests non-critiques Phase 7 ou reporter optimisations avancées)
- **Risques identifiés :**
  - Complexité bootstrap TimescaleDB en dev (volumes Docker, Continuous Aggregates) → Mitigation : SC-PH2-TEL-T10 (validation early)
  - Driver LOGO : normalisation payload Q/I/M/V → Mitigation : SC-PH3-LOGO-T01 (validation flux générique avant implémentation)
  - Incompatibilité devices Zigbee → Mitigation : tester tôt Phase 2
  - Complexité régulation chauffage → Mitigation : tests intensifs Phase 3
  - Widget system : diversité des `capabilities.controls` non couverte → Mitigation : `WidgetRaw` comme fallback universel
  - Retards fournisseur matériel → Mitigation : commander avant fin Mars 2026
  - Installation physique Logo complexe → Mitigation : préparation détaillée Phase 3

### Commandes matériel (deadlines) :

- **Avant 31 Mars 2026 :** Vannes thermostatiques Zigbee (délai 2-3 semaines)
- **Avant 15 Avril 2026 :** Capteurs température Zigbee additionnels
- **Avant 30 Avril 2026 :** Matériel électrique (relais, câbles, boîtiers, disjoncteur)
- **Avant 1er Août 2026 :** Tout matériel doit être reçu et testé
