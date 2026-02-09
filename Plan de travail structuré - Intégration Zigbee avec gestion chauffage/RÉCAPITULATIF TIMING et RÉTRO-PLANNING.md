## RÉCAPITULATIF TIMING

|Phase|Semaines|Charge (h)|Période (démarrage Mars 2025)|
|---|---|---|---|
|**Phase 0 : Audit**|1-2|15-20h|S1-S2 Mars|
|**Phase 1 : Infra Zigbee + Dev**|3-5|20-25h|S3-S5 Mars|
|**Phase 2 : Backend Devices**|6-8|25-30h|S6-S8 Mars-Avril|
|**Phase 3 : Chauffage**|9-12|35-40h|S9-S12 Avril-Mai|
|**Phase 4 : Automatisations**|13-15|25-30h|S13-S15 Mai|
|**Phase 5 : Scènes**|16-17|15-20h|S16-S17 Mai-Juin|
|**Phase 6 : Frontend**|18-21|40-50h|S18-S21 Juin-Juillet|
|**Phase 7 : Tests/Optim**|22-24|25-30h|S22-S24 Juillet-Août|
|**Phase 8 : Sécurité + Prod**|25-27|25-30h|S25-S27 Août-Septembre|
|**TOTAL**|**27 semaines**|**225-275h**|**Mars → Septembre**|

**Rythme hebdomadaire suggéré :**

- **8-10h/semaine** en moyenne (2-3 soirées 2-3h + 4-6h weekend)
- Phases légères (0, 1, 5) : 7-8h/semaine
- Phases moyennes (2, 4, 7, 8) : 8-10h/semaine
- Phases lourdes (3, 6) : 10-12h/semaine

---

## RÉTRO-PLANNING (Objectif : Production Septembre 2025)

### Démarrage : **1er Mars 2025**

|Mois|Semaines|Phases|Jalons clés|
|---|---|---|---|
|**Mars**|S1-S5|Phase 0, 1|✓ Audit terminé (S2)  <br>✓ Env dev configuré (S5)  <br>✓ Zigbee2MQTT opérationnel (S5)|
|**Avril**|S6-S12|Phase 2, 3|✓ API Devices complète (S8)  <br>✓ Backend chauffage fonctionnel (S12)  <br>✓ Schéma câblage Logo prêt (S12)|
|**Mai**|S13-S17|Phase 4, 5|✓ Moteur automatisations opérationnel (S15)  <br>✓ API Scènes prête (S17)|
|**Juin-Juillet**|S18-S24|Phase 6, 7|✓ Frontend complet responsive (S21)  <br>✓ Tests E2E passés (S24)  <br>✓ Documentation complète (S24)|
|**Août**|S22-S24|Phase 7 (fin)|✓ Optimisations finalisées (S24)  <br>✓ Système validé pré-prod (S24)|
|**Septembre**|S25-S27|Phase 8|✓ HTTPS/WSS configuré (S25)  <br>✓ Déploiement production (S26)  <br>✓ Installation physique Logo (S27)  <br>✓ **Système opérationnel** (fin S27)|

### Points de contrôle critiques :

- **15 Mars (S2)** : Audit complet terminé, décision go/no-go remise à niveau
- **15 Avril (S7)** : Première démo interne (devices + MQTT + interface basique)
- **15 Mai (S15)** : Automatisations démontrables (use cases réels testés)
- **1er Juillet (S18)** : Alpha release frontend (tests utilisateurs famille)
- **1er Août (S22)** : Beta feature-complete (phase stabilisation)
- **1er Septembre (S25)** : Sécurisation terminée, préparation installation physique
- **15 Septembre (S27)** : **Mise en production finale** ✅

### Buffer et risques :

- **Buffer intégré :** 1-2 semaines (si dépassement, réduire tests non-critiques Phase 7)
- **Risques identifiés :**
    - Incompatibilité devices Zigbee → Mitigation : tester tôt Phase 2
    - Complexité régulation chauffage → Mitigation : tests intensifs Phase 3
    - Retards fournisseur matériel → Mitigation : commander avant fin Mars
    - Installation physique Logo complexe → Mitigation : préparation détaillée Phase 3

### Commandes matériel (deadlines) :

- **Avant 31 Mars :** Vannes thermostatiques Zigbee (délai 2-3 semaines)
- **Avant 15 Avril :** Capteurs température Zigbee additionnels
- **Avant 30 Avril :** Matériel électrique (relais, câbles, boîtiers, disjoncteur)
- **Avant 1er Août :** Tout matériel doit être reçu et testé

---

## LIVRABLES PAR PHASE

- **Phase 0 :** 📄 Document audit + Plan remise à niveau + Rapport couverture tests baseline
- **Phase 1 :** 🔧 Env dev configuré + Z2M opérationnel + 1 device testé + Doc basculement prod
- **Phase 2 :** 🔌 API Devices complète (Swagger) + Services MQTT + 5 devices testés
- **Phase 3 :** 🌡️ API Chauffage + Moteur régulation + Schéma câblage Logo détaillé
- **Phase 4 :** ⚙️ Moteur automatisations + API + 10 scénarios validés
- **Phase 5 :** 🎬 API Scènes + Service activation + 5 scènes démo
- **Phase 6 :** 🖥️ Interface complète responsive testée (mobile/tablet/desktop)
- **Phase 7 :** ✅ Rapport tests (≥70% couverture) + Documentation complète (user + dev + API)
- **Phase 8 :** 🏠 Système production sécurisé (HTTPS/WSS) + Installation physique validée