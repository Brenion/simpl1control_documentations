## PHASE 6 : Frontend complet (Semaines 18-21 | 40-50h)

### A - Architecture et setup frontend

**User Stories :**

- **SC-US-PH6-01** : En tant que développeur, je veux une architecture frontend moderne et performante
- **SC-US-PH6-02** : En tant qu'utilisateur, je veux une interface responsive sur tous les appareils

**Tâches :**

- **SC-PH6-T01** : Configurer framework frontend (React/Vue/Svelte) avec Vite (SC-US-PH6-01)
- **SC-PH6-T02** : Configurer state management léger (Zustand/Pinia) (SC-US-PH6-01)
- **SC-PH6-T03** : Configurer client WebSocket (WSS pour temps réel) (SC-US-PH6-01)
- **SC-PH6-T04** : Configurer router SPA (SC-US-PH6-01)
- **SC-PH6-T05** : Configurer build optimisé (code splitting, lazy loading) (SC-US-PH6-01)
- **SC-PH6-T06** : Définir breakpoints responsive (320px, 768px, 1024px, 1440px) (SC-US-PH6-02)
- **SC-PH6-T07** : Créer design system (palette couleurs, variables CSS) (SC-US-PH6-02)
- **SC-PH6-T08** : Créer bibliothèque icônes (Material/Lucide/Phosphor) (SC-US-PH6-02)

### B - Composants réutilisables

**User Stories :**

- **SC-US-PH6-03** : En tant que développeur, je veux des composants UI cohérents et réutilisables

**Tâches :**

- **SC-PH6-T09** : Créer composant `Button` (variants primary/secondary/danger) (SC-US-PH6-03)
- **SC-PH6-T10** : Créer composant `Card` (SC-US-PH6-03)
- **SC-PH6-T11** : Créer composant `Input` (text, number, slider) (SC-US-PH6-03)
- **SC-PH6-T12** : Créer composant `Switch/Toggle` (SC-US-PH6-03)
- **SC-PH6-T13** : Créer composant `Modal/Dialog` (SC-US-PH6-03)
- **SC-PH6-T14** : Créer composant `Toast` (notifications) (SC-US-PH6-03)
- **SC-PH6-T15** : Créer composant `Dropdown/Select` (SC-US-PH6-03)
- **SC-PH6-T16** : Créer composant `Badge` (SC-US-PH6-03)
- **SC-PH6-T17** : Créer composants états (Loading skeleton, Empty state, Error state) (SC-US-PH6-03)
- **SC-PH6-T18** : Créer tests unitaires pour composants réutilisables (SC-US-PH6-03)

### C - Dashboard principal

**User Stories :**

- **SC-US-PH6-04** : En tant qu'utilisateur, je veux un dashboard avec tous mes appareils visibles
- **SC-US-PH6-05** : En tant qu'utilisateur, je veux voir l'état temps réel de mes appareils

**Tâches :**

- **SC-PH6-T19** : Créer layout responsive dashboard (CSS Grid/Flexbox) (SC-US-PH6-04)
- **SC-PH6-T20** : Créer section vue rapide (météo, résumé chauffage, alertes) (SC-US-PH6-04)
- **SC-PH6-T21** : Créer section pièces (cartes cliquables) (SC-US-PH6-04)
- **SC-PH6-T22** : Créer section raccourcis scènes (SC-US-PH6-04)
- **SC-PH6-T23** : Créer composant `DeviceCard` (nom, icône, état, contrôle) (SC-US-PH6-04)
- **SC-PH6-T24** : Implémenter badge update disponible sur cards (SC-US-PH6-04)
- **SC-PH6-T25** : Implémenter indicateur online/offline (SC-US-PH6-04)
- **SC-PH6-T26** : Créer filtres (par pièce, type, favoris) (SC-US-PH6-04)
- **SC-PH6-T27** : Implémenter mise à jour temps réel via WebSocket (SC-US-PH6-05)
- **SC-PH6-T28** : Créer switch mode liste/grille (SC-US-PH6-04)
- **SC-PH6-T29** : Tester dashboard sur mobile/tablette/desktop (SC-US-PH6-02)

### D - Gestion appareils

**User Stories :**

- **SC-US-PH6-06** : En tant qu'utilisateur, je veux gérer facilement mes appareils
- **SC-US-PH6-07** : En tant qu'utilisateur, je veux ajouter un appareil Zigbee en 3 clics max

**Tâches :**

- **SC-PH6-T30** : Créer page liste appareils avec recherche et filtres (SC-US-PH6-06)
- **SC-PH6-T31** : Créer page détails appareil (infos, contrôles, historique, logs) (SC-US-PH6-06)
- **SC-PH6-T32** : Implémenter graphique historique états (24h) (SC-US-PH6-06)
- **SC-PH6-T33** : Créer modal "Ajouter appareil" avec timer visuel (SC-US-PH6-07)
- **SC-PH6-T34** : Implémenter détection temps réel nouveaux appareils (SC-US-PH6-07)
- **SC-PH6-T35** : Implémenter édition inline pour renommage (SC-US-PH6-06)
- **SC-PH6-T36** : Implémenter drag & drop pour assigner pièce (SC-US-PH6-06)
- **SC-PH6-T37** : Créer modal suppression avec confirmation (SC-US-PH6-06)
- **SC-PH6-T38** : Implémenter mise à jour firmware avec progression (SC-US-PH6-06)

### E - Gestion chauffage

**User Stories :**

- **SC-US-PH6-08** : En tant qu'utilisateur, je veux gérer facilement mes zones de chauffage
- **SC-US-PH6-09** : En tant qu'utilisateur, je veux créer et modifier des plannings visuellement

**Tâches :**

- **SC-PH6-T39** : Créer page dédiée "Chauffage" (SC-US-PH6-08)
- **SC-PH6-T40** : Créer composant carte zone (température, consigne, mode, demande) (SC-US-PH6-08)
- **SC-PH6-T41** : Implémenter slider/boutons ajustement température (SC-US-PH6-08)
- **SC-PH6-T42** : Implémenter toggle mode (auto/manuel/off) (SC-US-PH6-08)
- **SC-PH6-T43** : Créer indicateur visuel demande chauffage (flamme) (SC-US-PH6-08)
- **SC-PH6-T44** : Créer vue globale (état Logo, graphique multi-zones) (SC-US-PH6-08)
- **SC-PH6-T45** : Créer graphique températures toutes zones (24h) (SC-US-PH6-08)
- **SC-PH6-T46** : Créer builder visuel planning hebdomadaire (SC-US-PH6-09)
- **SC-PH6-T47** : Implémenter timeline avec blocs glissables (SC-US-PH6-09)
- **SC-PH6-T48** : Créer templates prédéfinis plannings (SC-US-PH6-09)
- **SC-PH6-T49** : Implémenter duplication planning entre zones (SC-US-PH6-09)
- **SC-PH6-T50** : Créer modal override température temporaire (SC-US-PH6-08)

### F - Automatisations (Interface)

**User Stories :**

- **SC-US-PH6-10** : En tant qu'utilisateur, je veux créer des automatisations de manière visuelle et intuitive

**Tâches :**

- **SC-PH6-T51** : Créer page liste automatisations (SC-US-PH6-10)
- **SC-PH6-T52** : Créer builder visuel type "IF-THEN" (SC-US-PH6-10)
- **SC-PH6-T53** : Créer section déclencheur (dropdown catégories + événement) (SC-US-PH6-10)
- **SC-PH6-T54** : Créer section conditions (empilage avec AND/OR) (SC-US-PH6-10)
- **SC-PH6-T55** : Créer section actions (liste drag & drop) (SC-US-PH6-10)
- **SC-PH6-T56** : Implémenter configuration visuelle par action (SC-US-PH6-10)
- **SC-PH6-T57** : Créer prévisualisation logique en français (SC-US-PH6-10)
- **SC-PH6-T58** : Implémenter bouton "Tester" (simulation) (SC-US-PH6-10)
- **SC-PH6-T59** : Créer toggle enable/disable visible (SC-US-PH6-10)
- **SC-PH6-T60** : Créer page historique exécution avec filtres (SC-US-PH6-10)
- **SC-PH6-T61** : Créer templates automatisations suggérés (SC-US-PH6-10)

### G - Scènes (Interface)

**User Stories :**

- **SC-US-PH6-11** : En tant qu'utilisateur, je veux gérer mes scènes facilement

**Tâches :**

- **SC-PH6-T62** : Créer page liste scènes avec cartes (SC-US-PH6-11)
- **SC-PH6-T63** : Créer aperçu devices concernés (miniatures) (SC-US-PH6-11)
- **SC-PH6-T64** : Créer bouton activation gros et tactile (60x60px min) (SC-US-PH6-11)
- **SC-PH6-T65** : Créer modal création mode "Capturer" (SC-US-PH6-11)
- **SC-PH6-T66** : Créer modal création mode "Configurer manuellement" (SC-US-PH6-11)
- **SC-PH6-T67** : Implémenter prévisualisation avant sauvegarde (SC-US-PH6-11)
- **SC-PH6-T68** : Implémenter édition états devices individuels (SC-US-PH6-11)
- **SC-PH6-T69** : Implémenter système favoris (épinglage dashboard) (SC-US-PH6-11)

### H - Optimisations et responsive final

**User Stories :**

- **SC-US-PH6-12** : En tant qu'utilisateur mobile, je veux une interface fluide et tactile
- **SC-US-PH6-13** : En tant qu'utilisateur, je veux des animations subtiles et un design moderne

**Tâches :**

- **SC-PH6-T70** : Implémenter menu hamburger sur mobile (SC-US-PH6-12)
- **SC-PH6-T71** : Optimiser contrôles tactiles (min 44x44px) (SC-US-PH6-12)
- **SC-PH6-T72** : Tester sur iPhone SE (petit écran) (SC-US-PH6-12)
- **SC-PH6-T73** : Tester sur Android mid-range (SC-US-PH6-12)
- **SC-PH6-T74** : Tester sur iPad (SC-US-PH6-12)
- **SC-PH6-T75** : Tester sur Desktop 1080p et 1440p (SC-US-PH6-12)
- **SC-PH6-T76** : Optimiser images (WebP, lazy loading) (SC-US-PH6-12)
- **SC-PH6-T77** : Implémenter animations CSS subtiles (SC-US-PH6-13)
- **SC-PH6-T78** : Implémenter debounce sur inputs (search, sliders) (SC-US-PH6-12)
- **SC-PH6-T79** : Créer système notifications toast (SC-US-PH6-13)
- **SC-PH6-T80** : Implémenter dark mode (optionnel) (SC-US-PH6-13)