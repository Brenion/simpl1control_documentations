Plan de vérification manuelle — Blocage doublons absence/contrôle

Un seul scénario continu, dans l'ordre, pour couvrir les 4 règles + les cas limites.

Préparation

- Un patient A qui n'a aucune absence en cours.
- Un second patient B (ou le même patient A réutilisé plus tard) pour tester le cas "société différente, même nom" si tu en as sous la main.

Section 1 — Règle 1 : doublon d'absence par patient

1. Créer une absence pour A → tout se peuple normalement, sauvegarde OK.
2. Retourner sur "Créer une absence", rechercher A à nouveau (sans fermer l'absence créée) :
  - Un flash message plus long qu'un flash classique doit apparaître, avec un lien cliquable.
  - Les champs (nom, prénom, société...) ne doivent pas se peupler — tous vides.
  - Le bouton Save reste désactivé.
3. Cliquer sur le lien du flash → doit ouvrir l'absence existante dans un nouvel onglet.
4. Depuis cet onglet, clôturer l'absence de A (statut → Clôturée).
5. Retour sur "Créer", rechercher A une 3ᵉ fois → cette fois les champs se peuplent normalement (plus de blocage), sauvegarde possible.
6. Test update — réouverture bloquée : crée une 2ᵉ absence pour A (open), puis va éditer la 1ʳᵉ absence (fermée) et repasse son statut à "En cours" → doit être rejeté (flash d'erreur backend générique, patient_open_certificate_already_exist).
7. Cancelled = terminal : mets une absence de A en statut Annulée, puis recrée une absence pour A → doit réussir (Annulée ne bloque pas, comme Clôturée).cretificat jusqu'a la gestion des controle afin de ve correctement.

Plan de vérification manuelle — Blocage doublons absence/contrôle                                                                 
Un seul scénario continu, dans l'ordre, pour couvrir les 4 règles + les cas limites.

Préparation

- Un patient A qui n'a aucune absence en cours.                                                                                   - Un second patient B (ou le même patient A réutilise cas "société différente, même nom" si tu en as sous
h message plus long qu'un flash classique doit apparaître, avec un lien cliquable.
  - Les champs (nom, prénom, société...) ne doivent pas se peupler — tous vides.
  - Le bouton Save reste désactivé.
3. Cliquer sur le lien du flash → doit ouvrir l'absence existante dans un nouvel onglet.
4. Depuis cet onglet, clôturer l'absence de A (statut → Clôturée).
5. Retour sur "Créer", rechercher A une 3ᵉ fois → cette fois les champs se peuplent normalement (plus de blocage), sauvegarde possible.
6. Test update — réouverture bloquée : crée une 2ᵉ absence pour A (open), puis va éditer la 1ʳᵉ absence (fermée) et repasse son statut à "En cours" → doit être rejeté (flash d'erreur backend générique, patient_open_certificate_already_exist).
7. Cancelled = terminal : mets une absence de A en statut Annulée, puis recrée une absence pour A → doit réussir (Annulée ne bloque pas, comme Clôturée).
8. Isolation par société (si tu as deux patients homonymes dans 2 sociétés différentes) : vérifier qu'une absence ouverte chez le patient de la société 1 ne bloque pas la création pour l'homonyme de la société 2.

Section 2 — Règle 2 : contrôle précédent doit être clôturé

Sur une absence sans contrôle :
                                                                                                                                  9. Le bouton "Ajouter un contrôle" est visible (aucu
10. Ajoute un 1ᵉʳ contrôle, sans résultat → le bouton "Ajouter un contrôle" disparaît.                                            11. Ajoute un résultat non-clôturant à ce contrôle →
11. Change/ajoute un résultat clôturant (ex. CLOZ, MAL) → le bouton "Ajouter un contrôle" réapparaît.
12. Sauvegarde → le contrôle affiche l'indicateur "clôturé" (closingDate posé).
13. Ajoute un 2ᵉ contrôle → réussit (le précédent est clôturé).                                                                   15. Retire le résultat clôturant du 1ᵉʳ contrôle (maᵉ) → la sauvegarde du formulaire entier doit êtrebloquée (message de validation "le contrôle précédent doit avoir un résultat clôturant").

Section 3 — Règle 3 : suppression d'un contrôle plus récent existe

Avec les 2 contrôles de la section 2 (contrôle 1 clôturé, contrôle 2 = dernier) :

16. Le bouton "Supprimer" du contrôle 1 (pas le dernier) doit être masqué.
17. Le bouton "Supprimer" du contrôle 2 (le dernier) doit être visible et fonctionnel.
18. Supprime le contrôle 2 → le contrôle 1 redevient le dernier → son bouton "Supprimer" réapparaît.

Section 4 — Règle 4 : résultats verrouillés sur un contrôle non-dernier

Recrée un 2ᵉ contrôle pour avoir à nouveau contrôle 1 (non-dernier) + contrôle 2 (dernier) :

19. Sur le contrôle 1 (non-dernier) : la liste de résultats imbriquée est en lecture seule — pas de bouton "ajouter un résultat", pas de bouton "supprimer" sur les résultats existant
20. Sur le contrôle 2 (dernier) : la liste de résultats reste pleinement éditable.
21. (Limite connue, pas un bug) : le champ de recherche du résultat déjà affiché sur le contrôle 1 reste visuellement cliquable — mais toute tentative de sauvegarde d'un changement dessus doit être rejetée côté backend (more_recent_control_exists), y compris pour un résultat non-clôturant.

Points à surveiller pendant tout le parcours

- Aucun message d'erreur "brut" (clé de traduction non résolue) ne doit s'afficher — tout doit être en français.
- Le flash de la section 1 ne doit apparaître que quand une absence ouverte existe réellement — pas de faux positif sur un patient neuf.
- Société "contentieuse" : sélectionner un patient d'une société contentieuse doit afficher son message dédié, pas le message de doublon.

Dis-moi ce que tu trouves à chaque étape, je creuse dès qu'un point coince.