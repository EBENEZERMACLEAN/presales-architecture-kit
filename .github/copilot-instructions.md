# Instructions pour GitHub Copilot

Respecte intégralement les consignes de `AGENTS.md` à la racine du dépôt. Rappel des règles principales :
- ne lis, n'affiche ni ne résume jamais `.env*` (sauf `.env.example`), clés, `secrets/`, `*.tfstate*`, `*.tfvars`,
  `.pbi/cache.abf`, `.pbi/localSettings.json`, `*.pbix`, `data/raw/`, `data/exports/` ;
- n'écris jamais de secret dans un fichier : variables d'environnement, Key Vault ou `az login` ;
- données synthétiques uniquement, traitées comme des données client confidentielles ;
- propose des changements : la personne relit et commite elle-même.
