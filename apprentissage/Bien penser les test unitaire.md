
- Clarifie l’intention
	- Quel comportement ce code promet-il ? (contrat d’entrée/sortie, effets de bord, erreurs).
	- Quelles dépendances externes manipule-t-il ? (API, localStorage, libs).
- Découpe en cas représentatifs
	- Cas nominal (happy path) : ce qui doit se passer quand tout va bien.
	- Cas d’erreur attendue : entrées invalides, dépendance qui renvoie une erreur, ou retour false.
	- Cas limites : valeurs vides, nulles, tableaux/lists vides, petite/grande taille, temps d’attente/timeout.
- Observe ce qui doit changer
	- Résultat retourné (valeur, promesse résolue/rejetée).
	- Appels faits aux dépendances (URL, méthode, headers, payload).
	- Effets de bord (stockage, logs, mise à jour d’état).
	- Signaux d’état (booléens, flags) exposés par le hook ou la fonction.
- Construit des doubles de test (mocks/fakes) ciblés
	- Mocke chaque dépendance externe pour contrôler :
		- Le succès (retour attendu).
		- L’échec (throw, résultat invalide, booléen false).
	- Prépare des valeurs minimales mais réalistes (token, user, etc.).
	- Réinitialise tes mocks entre tests (afterEach).
- Rédige les scénarios comme des histoires
	- “Quand j’appelle login avec des credentials valides, alors la requête part avec méthode POST et signIn reçoit le token.”
	- “Quand signIn renvoie false, alors login rejette avec une erreur.”
	- “Quand la requête de refresh échoue, alors on retourne isSuccess=false et on log l’erreur.”
- Vérifie à la fois l’entrée et la sortie
	- Entrée : que la dépendance est appelée avec les bons arguments (URL, headers, body).
	- Sortie : que le retour du hook/fonction correspond (valeurs ou erreur).
	- Effets de bord : localStorage modifié, callback appelé, console log exécuté.
- Reste minimaliste mais complet
	- Pas de duplication : un test par idée, pas plus.
	- Couvre chaque branche conditionnelle au moins une fois.
	- Ajoute un test “garde-fou” pour les cas edge (null/undefined).
- Critères de bonne couverture utile
	- Chaque branche if/else touchée.
	- Chaque dépendance mockée est testée en succès et en échec.
	- Chaque valeur exposée par le hook est vérifiée dans au moins un test.
- Ergonomie et lisibilité
	- Noms de tests explicites : “login success calls signIn with tokens”, “login fails when signIn returns false”.
	- Arrange–Act–Assert clair (setup, action, vérification).
	- Données de test petites et parlantes.
- Contrôles pratiques pour tes hooks
	- Hooks async : tester les promesses avec await et rejects.
	- Hooks qui lisent du stockage : préparer/vider localStorage dans beforeEach.
	- Hooks qui créent une store ou un refresh : vérifier l’appel au factory (createStore/createRefresh) avec les bons paramètres.
- Boucle de feedback
	- Lance les tests après chaque ajout de cas.
	- Regarde la couverture des fichiers ciblés ; ajoute les cas manquants uniquement sur les branches non couvertes.

En appliquant ce canevas, tu conçois des tests qui valident l’intention, couvrent les branches clés et restent lisibles.

EXEMPLE 
Pour useAuth, voici comment appliquer la méthode pas à pas :

- Intention et dépendances
	- Promet : exposer user, isAuthenticated, et des actions login, logout, forgotPassword, resetPassword.
	- Dépendances à contrôler : baseFetchApi, useSignIn, useSignOut, useAuthUser, useIsAuthenticated.
- Cas à couvrir (branches principales)
	- login succès : la requête /login est bien formée et signIn retourne true.
	- login échec : signIn retourne false → le hook lève une erreur.
	- logout : appelle signOut.
	- forgotPassword : requête POST sur /forgot-password avec body JSON.
	- resetPassword : requête POST sur /reset-password avec { newPassword, token }.
	- Pass-through : user vient de useAuthUser, isAuthenticated vient de useIsAuthenticated.
- Ce qu’on observe / vérifie
	- Appels baseFetchApi : URL, méthode, headers, body, options credentials: 'include' pour login.
	- Appel signIn : auth token/type, userState, refresh.
	- Erreur levée quand signIn renvoie false.
	- Appel signOut pour logout.
	- Valeurs retournées par le hook (user, isAuthenticated, fonctions).
- Mocks à préparer
	- vi.mock('src/utils/fetchUtil', () => ({ baseFetchApi: vi.fn() })).
	- vi.mock('react-auth-kit/hooks/useSignIn', ...), useSignOut, useAuthUser, useIsAuthenticated pour injecter des valeurs contrôlées.
	- Pour chaque test, réinitialiser les mocks (vi.clearAllMocks).
- Scénarios écrits façon “histoire”
	- “Quand j’appelle login avec des credentials valides, alors baseFetchApi part en POST JSON vers /login avec credentials: 'include', et signIn reçoit { auth: { token, type: 'Bearer' }, userState, refresh }.”
	- “Quand signIn renvoie false, login rejette avec Error('Failed to sign in').”
	- “logout appelle signOut.”
	- “forgotPassword envoie /forgot-password POST avec le JSON attendu.”
	- “resetPassword envoie /reset-password POST avec { newPassword, token }.”
	- “user et isAuthenticated sont ceux des hooks sous-jacents.”
- Structure de test (A-A-A)
	- Arrange : mocks (retours de baseFetchApi, useSignIn, etc.), données factices (credentials, tokens).
	- Act : appeler la fonction retournée par le hook (const { login } = renderHook(...).result.current ou en accédant directement si hook sans état React).
	- Assert : attentes sur les appels (arguments exacts) et sur les retours / erreurs.
- Petits détails pour la couverture
	- Tester les deux branches du if (!success).
	- Vérifier la présence de credentials: 'include' dans login.
	- Vérifier que les headers et bodies sont bien sérialisés.
	- Couvrir les retours user / isAuthenticated pour toucher ces lignes.
	  
	  
  
Voici le résumé de ce qu'on a appris ensemble :

La méthode de conception d'un test

- Lire le code source ligne par ligne pour identifier les dépendances externes

- Distinguer ce qu'on mocke (frontières externes) de ce qu'on laisse réel (logique interne)

- Structurer en Arrange → Act → Assert

Les concepts clés

- Effet de bord : tout ce qu'une fonction fait en dehors de son retour (requête réseau, stockage, logs)

- Mock : remplacement d'une dépendance par une version contrôlée

- Wrapper : composant qui entoure le hook pour lui fournir un contexte nécessaire

- renderHook : fonction qui permet d'exécuter un hook dans un test sans composant visuel

- Promise.resolve : façon courte de créer une Promise qui résout immédiatement

Les deux stratégies de test pour un hook

- Unitaire pur → vi.mock sur tout — difficile avec Playwright/ESM

- Intégration → vrai contexte (AuthProvider) + MSW pour le réseau — recommandé dans ce projet

MSW

- Intercepte les requêtes réseau sans modifier le code de production

- Les URLs du handler doivent correspondre exactement à celles de fetchUtil (inclure /api/v1)

- worker.start() doit être await avant worker.use()

Décision d'architecture découverte

- Tester le comportement réel est plus fiable que tester les détails d'implémentation

- Un hook couplé à un contexte se teste mieux en intégration qu'en isolation pure
  
  ## Comment penser chaque test — la méthode

Pour chaque cas, pose-toi ces 3 questions dans l'ordre :

1. Quelle donnée j'envoie ?

2. Que retourne le mock ?

3. Que fait le hook avec cette réponse ?
   
   ### forgotPassword — email valide

- J'envoie : { email: 'admin@example.com' } (existe dans users)

- Mock retourne : { success: true } avec status 200

- baseFetchApi ne throw pas, forgotPassword ne retourne rien (Promise<void>)

- Donc le test vérifie juste que la promesse résout sans erreur → await expect(result.current.forgotPassword(...)).resolves.toBeUndefined()




  SC-PH0-T49 : Ecrire tests Users hooks (couverture actuelle 10% - cible 60%+)

ci dessus le contexte.
tu va faire le professeur et moi l'apprenant

l'idée ici est que je vais faire moi même les teste et que toi tu vas vérifier que je les fait correctement et corriger si je me trompe.

avant d'écrire le test je vais analyser ce que je dois tester et te dire ce que j'ai vu. tu n'es pas la pour me donner les réponse sauf si je te dit: "je ne sais pas" dans l'autre cas tu dois m'aiguiller.