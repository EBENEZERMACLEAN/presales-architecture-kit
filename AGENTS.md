# Consignes pour les agents IA (Claude Code, GitHub Copilot, Cursor, Codex…)

Ce dépôt applique le **Standard de développement responsable** du parcours : toutes les données sont fictives,
mais elles sont traitées **comme des données client confidentielles**.

## Règles impératives
1. **Ne jamais lire, afficher, copier ni résumer** : `.env*` (sauf `.env.example`), clés (`*.pem`, `*.key`, `*.pfx`),
   `secrets/`, états Terraform (`*.tfstate*`), variables Terraform (`*.tfvars`), caches Power BI (`.pbi/cache.abf`,
   `.pbi/localSettings.json`), fichiers `.pbix`, données brutes (`data/raw/`, `data/exports/`).
2. **Ne jamais écrire de secret** (clé, mot de passe, jeton, chaîne de connexion) dans un fichier : utiliser une variable
   d'environnement, Azure Key Vault ou l'identité de l'utilisateur (`az login`), et un exemple factice dans `.env.example`.
3. **Ne jamais committer ni pousser** : l'agent propose des changements, la personne relit puis commite elle-même.
4. **Données** : uniquement les données synthétiques générées par les scripts du dépôt. Aucune donnée réelle, aucun nom
   de client, de projet interne ou de personne réelle. Les colonnes de type donnée personnelle (nom, courriel,
   téléphone…) restent marquées « Confidentiel » dans `DATA.md`, même fictives.
5. **Power BI** : serveur et base de données passent par des paramètres (`expressions.tmdl`) avec des valeurs factices ;
   pas de table « Entrer des données » contenant des valeurs sensibles ; pas de membres RLS dans le TMDL.
6. **Dépendances** : versions épinglées ; aucune bibliothèque ajoutée sans la mentionner dans la description de la PR.
7. **Doute** : en cas de doute sur la sensibilité d'un fichier, ne pas l'ouvrir et demander.

## Où travailler
Ouvrir l'agent dans le **dossier de travail sans données** (worktree, voir `SECURITY.md`), jamais dans le dossier
ouvert par Power BI Desktop.
