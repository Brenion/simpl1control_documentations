
01 - lors de la mise en place du server backend. la l'étape 10. le cron job n'était pas bien implémenté du au fait que le TSconfig.ts était configuré sur commonJs. Il a fallu prendre un peu de temps pour modifier le projet pour qu'il soit compatible.


02 - les fichier .env sont mal lu et il manque des variables

03 - requete postman 
password authentication failed for user "johndoe"
2025-02-09 22:39:00 2025-02-09 21:39:00.819 UTC [72] DETAIL:  Role "johndoe" does not exist.

Besoin d'accompagnement : 

remonte d'octobre avec deadline
mvp
jrpc

ce qui reste a faire

- etape 9 dans un nouveau container nous allons devoir ajouter un mosquitto de test. il sera ouvert et ne sera installer que si nous desirons l'avoir pas de mot passe pas de user


---


 1. Système d’authentification avancé

- Configuration des JWT avec `@fastify/jwt`
    
- Gestion des rôles et permissions avec `casl`
