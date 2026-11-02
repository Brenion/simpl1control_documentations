# Le CI (Continuous Integration) — expliqué simplement

## C'est quoi le CI ?

Imagine que tu travailles sur ton projet de domotique. Chaque fois que tu modifies du code et que tu fais un `git push`, tu aimerais que **quelqu'un vérifie automatiquement** que tu n'as rien cassé. C'est exactement ça le CI : un robot qui exécute des vérifications à ta place, à chaque modification.

## L'analogie

Pense à un professeur qui corrige tes copies instantanément. Tu rends ta copie (push), il la corrige (tests, vérifications), et te dit en quelques minutes si tout est bon ✅ ou s'il y a des erreurs ❌. Sauf qu'ici le professeur ne dort jamais et corrige en 2 minutes.

## Comment ça marche concrètement ?

Le cœur du système, c'est un fichier `.github/workflows/ci.yml` que tu places dans ton dépôt Git. Ce fichier décrit **quand** et **quoi** exécuter.

**Quand ?** C'est le déclencheur. Dans ton cas : à chaque `push` et à chaque `Pull Request (PR)`. Une PR, c'est quand tu proposes des modifications avant de les fusionner dans la branche principale.

**Quoi ?** Ce sont les étapes. Par exemple : installer les dépendances, lancer les tests, vérifier le linting (qualité du code), tenter un build.

**Où ?** GitHub met à disposition des serveurs (appelés "runners") qui exécutent tout ça dans un environnement propre, comme si on installait ton projet de zéro à chaque fois. C'est important : ça garantit que le projet fonctionne ailleurs que sur ta machine.

## Les points importants à retenir

**1 — Le fichier YAML est le chef d'orchestre.** Toute la logique est dans ce fichier. Si tu le supprimes, plus de CI. Si tu le modifies mal, le CI échoue. C'est un fichier de configuration avec une syntaxe stricte (l'indentation compte, comme en Python).

**2 — Environnement propre à chaque exécution.** Chaque fois que le CI se lance, c'est une machine vierge. Rien n'est conservé d'une exécution à l'autre (sauf si tu configures un cache, on verra ça plus tard). Ça veut dire que si ton projet marche chez toi mais pas en CI, c'est probablement que tu as oublié de déclarer une dépendance ou une variable d'environnement.

**3 — Rapide = utile, lent = ignoré.** Un CI qui prend 20 minutes, tu vas finir par ne plus regarder les résultats. L'objectif c'est quelques minutes max. On optimise avec du cache (garder les `node_modules` entre les exécutions par exemple).

**4 — Le CI est un filet de sécurité, pas un remplacement.** Tu continues à tester en local avant de push. Le CI est là pour attraper ce que tu aurais pu oublier, et surtout pour vérifier que tout fonctionne dans un environnement neutre.

**5 — Le feedback est visible directement sur GitHub.** Tu verras un petit check vert ✅ ou une croix rouge ❌ à côté de chaque commit et PR. Tu peux cliquer dessus pour voir les logs détaillés.

## La structure type d'un workflow

Voici la logique (pas encore le code, juste le raisonnement) :

- **Nom** : comment tu appelles ce workflow (ex: "CI")
- **Déclencheurs** : push sur certaines branches, PR sur certaines branches
- **Jobs** : un ou plusieurs blocs de travail (ex: un job pour le backend, un pour le frontend)
- **Steps** dans chaque job : les étapes une par une (checkout du code, installer Node, installer les dépendances, lancer les tests…)

## Ce qu'on va faire pour ton projet

Pour ta tâche SC-PH0-T17, on va créer un workflow basique qui :

1. Se déclenche sur push et PR (branches `main` et `develop`)
2. Installe Node.js
3. Installe les dépendances
4. Lance le linting (vérification qualité code)
5. Lance les tests
6. Tente un build

## Le flux correct

```
1. Tu codes ta tâche
2. Tu testes en local
3. Tu push sur ta branche (ex: feature/ma-tache)
   └─→ Le CI se déclenche automatiquement sur le push
       └─→ Il fait le linting + tests
           ├─→ ❌ Erreur ? Tu corriges, tu re-push, le CI relance
           └─→ ✅ Tout vert ? Tu continues

4. Tu crées ta Pull Request (PR) vers develop/main
   └─→ Le CI se redéclenche sur la PR
       └─→ Même vérifications (linting + tests)
           └─→ ✅ Tout vert = tu peux merger en confiance
```

## La nuance importante

Le CI tourne **aux deux moments** : au push ET à la PR. Pas uniquement à la PR. Pourquoi ?

**Au push** : ça te donne un retour rapide. Tu sais tout de suite si ton code est propre, sans attendre de créer la PR. C'est ton filet de sécurité personnel.

**À la PR** : c'est la validation "officielle" avant de fusionner. Si entre-temps quelqu'un d'autre a modifié la branche cible, le CI vérifie que tout fonctionne encore ensemble.

## Pour le linting spécifiquement

Tu as raison de vouloir cibler ces erreurs en priorité :

- **`console.log`** restants → ça ne doit pas partir en production
- **`debugger`** oubliés → même raison
- **Erreurs TypeScript** → le code doit compiler proprement

Tout ça, c'est configurable dans ESLint. Le CI va simplement exécuter la même commande que tu pourrais lancer en local (genre `npm run lint`), mais de manière automatique et obligatoire.

## En résumé

Tu avais le bon raisonnement, juste le CI ne "attend" pas la PR pour se lancer. Il tourne à chaque push. La PR est une deuxième couche de vérification.