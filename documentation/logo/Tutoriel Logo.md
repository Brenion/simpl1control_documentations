Serveur web et Logo web editor

Basé sur les video youtube d'Hervé Discours

01 - cabler un logo

Mémoire : 
	I = input
	Q = output
	M = Memento

02 - le serveur web

A - cree un nouveau projet et une petite simulation (voir exemple tuto02)
B - sur le diagramme clique droit et choisir propiété
C - dans la nouvelle fenetre cliqué sur Parametre en ligne
D - cliquer sur se connecter
E - Dans la barre de navigation de la fenetre cliquer sur ```Parametre de controle d'acces```
F - Acces à LSC et LWE 
	activer la protection par mot de passe -> Miministry
quand tout ca est. fait

G - sortir des propriete et envoyer le programme au logo

se connecter au server web

H - dans un navigateur : 192.168.0.127
Entré son mot de pas et nous somme connecter

on vois le logo dnas l'accueil et il y a plusieurs onglet possible dasn le navigateur gauche.

I - on va sur les variables et on ajoute une variable I et une variable Q

Lorsqu'on appuie sur le bouton on va c'est variable passer en true.
Dans Q  on voit que l'on peut aussi modifier la valeur.
Etant donner que nous avons un programmer qui tourne sur le logo et qui impose une valeur a la variable Q meme si on venait a mettre 1 il ne se passerait rien

J - ajoutons une variable M1 et la nous écrivons 1 dans modification valeur et nous cliquons sur Modifier. 

nous constatons bien que la valeur M1 passe en true mais aussi Q1 du a notre programme

Visualisation

Dans l'onglet LOGO!BM nous avons la posibilité de visualisé le logo en temps reel.

dans notre programme nous allons ajouté un message lorsque le bouton est actif

K - pour se faire dans l'onglet autre de softconfort nous allons prendre texte de message

En double cliquant sur la boite nous pouvons ouvrir un fenetre de paramettrage de l'ecran. 

L - on active ```Ecran LOGO!``` et ```serv. web``` et dnas le visuel texte de message on y ajoute un message de notre cru
M - nous devons pour finir ajouter une borne ouverte (X) car sur logo on ne peut finir sans avoir une borne de sorti.

et on pousse la nouvelle programmation dans notre logo
quand on active notre bouton nous avons bien notre écran rouge et le texte qui s'écrit

Page web personnalisée 

Avec le logo il est possible d'avoir un serveur web perso

N - installer web editor
O - ouvrir un nouveau projet
P - placer un bouton switch digital dans la homePage et dans block type prendre M pour memento et Block number M1 
cela nous permettra de modifier via le site web la variable M1 qui interragi deja dans notre programme

Q - clique sur deploy to SD card quand fini remettre la carte SD dans le logo.
R - dans le navigateur aller sur le logo et se connecter mais cette foi avec la casse cochée ```ur le site personnalisé```

la page s'ouvre et il ne reste plus qu'a tester


03 - grafcet

Regle 

 ETAPE : 
	 -Doit être active ou inactive -> bit (memento)
	 - correspond a un état -> Fonction mémoire (bascule RS)
TRANSITION:
	-elle est franchissable si étape en amount est active et la réceptivité est vrai

franchissement de la transistion entraine :
	l'activation de létape en aval
	la désactivation de l'étape en amont









