# Sécurité et protection des données

Projet de démonstration : **toutes les données sont synthétiques**. Elles sont néanmoins traitées comme des données
client confidentielles (classification dans `DATA.md`).

## Signaler un problème
Si vous trouvez un secret, une donnée qui semble réelle ou une faille dans ce dépôt, ouvrez un signalement privé :
onglet **Security → Report a vulnerability** (signalement privé de vulnérabilité). Ne créez pas d'issue publique.

## Mesures en place
- Aucun secret dans le dépôt : variables d'environnement, Azure Key Vault ou identité de l'utilisateur ; `.env.example` factice.
- Détection avant commit : hooks `pre-commit` (gitleaks, clés privées, fichiers interdits).
- Détection côté GitHub : analyse des secrets et protection des envois (push protection) activées.
- Branche `main` protégée (ruleset) : pas de force push, pas de suppression, changements par pull request.
- Agents IA : consignes dans `AGENTS.md`, blocages dans `.claude/settings.json` et `.cursorignore`, travail dans un
  dossier sans données (`git worktree add -b agent/<sujet> ../<depot>-agent main`).

## En cas de fuite (procédure)
1. **Contenir** : révoquer ou renouveler le secret immédiatement (avant tout nettoyage).
2. **Évaluer** : quels commits, quelles branches, depuis quand, dépôt public ou non, forks.
3. **Éradiquer** : purger l'historique (`git filter-repo`), forcer l'envoi, vérifier avec `gitleaks git`.
4. **Vérifier** : nouvelle analyse complète, contrôle des vues GitHub en cache et des forks.
5. **Documenter** : journal d'incident et post-mortem sans blâme ; registre des incidents de confidentialité (Loi 25).
