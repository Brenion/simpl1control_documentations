### C - Mise en place CI GitHub Actions

**Contexte :** Le CD (déploiement vers RPi5) existe déjà. Cette section couvre la mise en place du pipeline CI et des règles de protection de branches sur GitHub.

**User Stories :**

- **SC-US-PH0-09** : En tant que développeur, je veux que chaque push/PR déclenche automatiquement les tests, le lint et le build pour garantir la qualité du code
- **SC-US-PH0-10** : En tant que développeur, je veux que la branche `main` soit protégée : merge uniquement si CI passe et qu'une review (humaine ou IA) est approuvée
- **SC-US-PH0-11** : En tant que développeur, je veux un rapport de couverture de tests visible sur chaque PR
- **SC-US-PH0-12** : En tant que développeur, je veux être notifié en cas d'échec du pipeline
- **SC-US-PH0-13** : En tant que développeur, je veux que le déploiement automatique vers le RPi5 se déclenche uniquement après merge sur `main` avec CI validée

**Tâches :**

#### Pipeline CI (GitHub Actions)

- **SC-PH0-T17** : Créer workflow `.github/workflows/ci.yml` déclenché sur push et PR (SC-US-PH0-09)
- **SC-PH0-T18** : Configurer job **Lint** : ESLint backend + frontend (SC-US-PH0-09)
- **SC-PH0-T19** : Configurer job **Tests backend** : Vitest + couverture (SC-US-PH0-09)
- **SC-PH0-T20** : Configurer job **Tests frontend** : Vitest + couverture (SC-US-PH0-09)
- **SC-PH0-T21** : Configurer job **Build** : vérifier que le build frontend et backend réussissent (SC-US-PH0-09)
- **SC-PH0-T22** : Configurer job **Audit sécurité** : `npm audit --audit-level=high` (SC-US-PH0-09)
- **SC-PH0-T23** : Configurer upload rapport couverture comme commentaire PR (SC-US-PH0-11)
- **SC-PH0-T24** : Configurer seuil minimum couverture (fail si <70% après Phase 7) (SC-US-PH0-11)
- **SC-PH0-T25** : Configurer cache `node_modules` pour accélérer les pipelines (SC-US-PH0-09)
- **SC-PH0-T26** : Configurer matrice Node.js (version production RPi5) (SC-US-PH0-09)

#### Protection de branches et review

- **SC-PH0-T27** : Configurer branch protection rules sur `main` : require status checks to pass (SC-US-PH0-10)
- **SC-PH0-T28** : Configurer branch protection rules : require 1 approval minimum avant merge (SC-US-PH0-10)
- **SC-PH0-T29** : Configurer branch protection rules : interdire push direct sur `main` (SC-US-PH0-10)
- **SC-PH0-T30** : Intégrer review IA automatique sur PR via GitHub Action (ex: `coderabbitai/ai-pr-reviewer` ou `CodiumAI-Agent/pr-agent`) (SC-US-PH0-10)
- **SC-PH0-T31** : Configurer la review IA comme required check (l'approbation IA compte comme review) (SC-US-PH0-10)
- **SC-PH0-T32** : Documenter convention de branches (`main`, `develop`, `feature/*`, `fix/*`) (SC-US-PH0-10)
- **SC-PH0-T33** : Créer template PR (`.github/pull_request_template.md`) avec checklist (SC-US-PH0-10)

#### Intégration CD existant

- **SC-PH0-T34** : Auditer le pipeline CD existant (scripts, méthode de déploiement SSH/rsync) (SC-US-PH0-13)
- **SC-PH0-T35** : Créer workflow `.github/workflows/deploy.yml` déclenché sur merge `main` (SC-US-PH0-13)
- **SC-PH0-T36** : Conditionner le déploiement au succès du workflow CI (needs: ci) (SC-US-PH0-13)
- **SC-PH0-T37** : Configurer secrets GitHub (SSH key RPi5, host, user) (SC-US-PH0-13)
- **SC-PH0-T38** : Implémenter étape déploiement : pull, install, build, PM2 reload sur RPi5 (SC-US-PH0-13)
- **SC-PH0-T39** : Implémenter smoke test post-déploiement (curl health endpoint) (SC-US-PH0-13)
- **SC-PH0-T40** : Implémenter rollback automatique si smoke test échoue (SC-US-PH0-13)
- **SC-PH0-T41** : Configurer notification GitHub (échec pipeline → notification email/GitHub) (SC-US-PH0-12)

#### Validation

- **SC-PH0-T42** : Tester pipeline CI complet sur une PR de test (SC-US-PH0-09)
- **SC-PH0-T43** : Vérifier que merge est bloqué si CI échoue (SC-US-PH0-10)
- **SC-PH0-T44** : Vérifier que merge est bloqué sans review approuvée (SC-US-PH0-10)
- **SC-PH0-T45** : Tester déploiement automatique après merge sur `main` (SC-US-PH0-13)
- **SC-PH0-T46** : Tester rollback en cas d'échec déploiement (SC-US-PH0-13)
- **SC-PH0-T47** : Documenter le pipeline CI/CD complet (schéma + README) (SC-US-PH0-09)

---

### Flux CI/CD complet

```
Feature branch → Push → CI (lint + tests + build + audit)
                   ↓
              Pull Request → Review IA automatique
                   ↓                    ↓
            CI ✅ + Review ✅ → Merge autorisé sur main
                                        ↓
                                CD (deploy.yml)
                                        ↓
                            SSH → RPi5 (pull, build, PM2 reload)
                                        ↓
                                Smoke test → ✅ OK / ❌ Rollback
```

### Estimation charge supplémentaire

|Sous-section|Charge estimée|
|---|---|
|Pipeline CI|4-5h|
|Protection branches + review IA|2-3h|
|Intégration CD existant|3-4h|
|Validation + documentation|2-3h|
|**Total**|**11-15h**|

> **Note :** La Phase 0 passe de 15-20h à **26-35h** avec cette section. Envisager d'étaler sur les semaines 1-3 au lieu de 1-2, ou paralléliser avec le début de Phase 1.

### Outils recommandés pour review IA

|Outil|Type|Avantage|
|---|---|---|
|**CodeRabbit**|GitHub App|Review IA complète, gratuit open source, commentaires inline|
|**CodiumAI PR-Agent**|GitHub Action|Self-hosted possible, configurable, résumé + review|
|**GitHub Copilot Code Review**|Natif GitHub|Intégré à GitHub, review comme reviewer assigné|

> Recommandation : **CodeRabbit** ou **GitHub Copilot Code Review** si vous avez Copilot — les deux peuvent servir de required reviewer sur la PR.