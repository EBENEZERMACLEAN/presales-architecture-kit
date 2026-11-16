# Données du projet — fiche et classification

**Origine** : aucune donnée pour l'instant (documentation seulement). Dès le premier jeu de données (module M3) : 100 % synthétique, générée par `<script de génération>` (graine fixe : `<seed>`). Aucune donnée réelle.
**Client fictif** : `Client Démo (fictif)` — traité comme un vrai client (confidentialité, minimisation, accès).

| Jeu de données | Emplacement | Classification | Données personnelles (fictives) | Versionné ? |
|---|---|---|---|---|
| Échantillon de démonstration | `samples/` | Interne | Non (ou pseudonymisées) | Oui (petit volume) |
| Données brutes générées | `data/raw/` | Confidentiel | Oui | Non (régénérées) |

**Classification** : Public · Interne · Confidentiel · Restreint.
**Règles** : seules les données « Public » ou « Interne » sont versionnées ; les colonnes personnelles sont masquées ou
pseudonymisées hors de la couche brute ; durée de conservation : données régénérables, supprimées en fin d'exercice.
