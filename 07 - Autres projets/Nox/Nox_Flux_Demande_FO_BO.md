# Nox — Flux métier d'une demande (Front-Office / Back-Office)

> Schéma **à compléter**. Version de travail dérivée des US E04 (Ticketing) et E05 (Dispatch & Workflow interne).
> Légende : **vert** = couvert par une US validée · **orange** = à définir / à compléter · **bleu** = décision.

```mermaid
flowchart TD
  subgraph FO["FRONT-OFFICE · Client"]
    F1["1 · Nouvelle demande<br/>(US-030)"]
    F2["2 · Choix du type de demande<br/>(US-030 · liste des types à définir)"]
    F3["3 · Formulaire propre au type<br/>(branche à créer · questionnaire par type)"]
    F4["4 · Récapitulatif + validation<br/>(débit crédits · US-031 · 1 ou 2 ?)"]
    F5["5 · Client reçoit la réponse<br/>(statut Réalisé)"]
    FD{"La réponse clôt-elle<br/>la demande ?"}
    KA["Accepte la clôture<br/>(US-033)"]
    KC["Continue l'échange<br/>(précision · US-233)"]
    L["Clôturé<br/>(auto 2 sem US-034 · pas de réouverture US-230)"]
  end

  subgraph BO["BACK-OFFICE · Avocat / Nox"]
    B1["Réception · file des tickets non assignés<br/>(US-040)"]
    B2["Dispatch par matière (fiscal, société...)<br/>(US-041 manuel · suggestion auto US-240/241)"]
    B3["Avocat traite la demande assignée"]
    BD{"Avocat en relecture<br/>systématique ? (US-044)"}
    HD["Responsable : envoi direct<br/>(non supervisé)"]
    HV["Collaborateur : validation<br/>(valideur à formaliser)"]
    BM["Réponse envoyée au client<br/>(avec ou sans clôture)"]
    B5["Sinon : orienter le client<br/>(nouveau ticket US-037 · consultation/RDV US-231-232)"]
  end

  F1 --> F2 --> F3 --> F4
  F4 -->|demande soumise| B1
  B1 --> B2 --> B3 --> BD
  BD -->|non| HD
  BD -->|oui| HV
  HD --> BM
  HV --> BM
  BM -->|réponse| F5
  F5 --> FD
  FD -->|oui| KA --> L
  FD -->|non| KC
  KC -. relance / précision .-> B3
  B3 -. sinon .-> B5

  classDef ok stroke:#2f9e44,stroke-width:2px,fill:#f2fbf4,color:#14321f;
  classDef todo stroke:#e0863a,stroke-width:2px,fill:#fdf3ea,color:#3d2710;
  classDef dec stroke:#3b82f6,stroke-width:2px,fill:#eaf1fd,color:#10254a;
  class F1,F4,F5,KA,L,B1,B2,B3,HD,BM ok;
  class F2,F3,KC,HV,B5 todo;
  class FD,BD dec;
```

## Étapes → US → point à compléter

| # | Étape (métier) | Acteur | US | À compléter |
|---|---|---|---|---|
| FO-1 | Nouvelle demande | Client | US-030 | — |
| FO-2 | Choix du type de demande | Client | US-030 | **Liste des types** (Christophe) |
| FO-3 | Formulaire propre au type | Client | US-030 | **Questionnaire conditionnel par type** (branche à créer) |
| FO-4 | Récapitulatif + validation | Client / Système | US-031 | Débit à la création ? 1 ou 2 crédits ? |
| FO-5 | Réception de la réponse | Client | US-032 | — |
| FO-6a | Accepte la clôture | Client | US-033 / US-034 | — |
| FO-6b | Continue l'échange (précision) | Client | US-233 | Gel du SLA pendant l'attente ? |
| BO-1 | Réception file non assignée | Dispatcher | US-040 | — |
| BO-2 | Dispatch par matière | Dispatcher | US-041 / US-240 / US-241 | **Matrice matières x avocats** (Christophe) |
| BO-3 | Traitement par l'avocat | Avocat | — | — |
| BO-3a | Responsable : envoi direct | Avocat | US-044 | — |
| BO-3b | Collaborateur : validation | Avocat | US-044 | **Qui valide** (rôle responsable d'équipe non formalisé) |
| BO-4 | Orienter : nouveau ticket / consultation | Avocat | US-037 / US-231 / US-232 | Matrice + workflow refus client |

## Incohérence à arbitrer
Le schéma linéaire historique débite les crédits **à la clôture**, alors que **US-031 débite à la création** du ticket. À aligner avant de figer le flux.
