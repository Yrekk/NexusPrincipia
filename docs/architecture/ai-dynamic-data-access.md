# NexusPrincipia — Accès dynamique IA

**2026-11-25 — DECIDED BUT NOT YET IMPLEMENTED.**

## Décision
Nexus fournit une mémoire externe structurée et fiable aux assistants IA, complémentaire de Git (autorité des projets logiciels). Reprendre progressivement les données contextuelles de Google Drive. Ce chantier précède l'interface Admin du bot.

- Un point d'entrée de lecture générique (GET conceptuel), indépendant du modèle IA ; aucun endpoint métier par question.
- Découverte du schéma autorisé et de sa sémantique : entités, champs, types, relations, contraintes. L'IA construit dynamiquement ses requêtes, filtres et jointures.
- Ajouter un domaine par évolution des données, métadonnées et permissions, sans nouveau GET métier ni code spécifique au moteur IA. Les applications déterministes peuvent conserver leurs endpoints spécialisés.
- Définir explicitement le périmètre de lecture autonome, celui nécessitant accord et celui interdit ; refus par défaut. Contrôles déterministes côté serveur, indépendants du modèle. Lecture seule, sans mutation ou privilège administratif implicite.
- Valider les requêtes et limiter coût, volume, délais ; auditer sans fuite de secrets. Le catalogue, les jointures et erreurs respectent les permissions. Distinguer résultat vide, refus et indisponibilité ; conserver provenance et fraîcheur. SQL optimisé par l'IA possible mais non garanti sans informations sur les index/statistiques.

## À concevoir ensemble avant implémentation
Cartographie/import Google, stockage, catalogue/versionnement, contrat de requête (SQL restreint ou autre), authentification, permissions fines, approbations/révocation, validation, audit, limites, accès réseau, adaptateur IA/API/MCP et compatibilité clients. GET ne préjuge pas du verbe HTTP. Aucun accès SQL arbitraire ni permission réelle accordée ici. Aucune donnée personnelle dans ce dépôt public.

## Tests attendus (non exécutés)
Lecture simple et relationnelle ; nouvelle entité interrogeable sans endpoint supplémentaire ; refus hors périmètre et tentative de mutation ; requête trop coûteuse maîtrisée ; lecture autonome vs accord requis ; distinction absence/refus/erreur ; contrôles indépendants du modèle sur données synthétiques.

**Reprise :** ouvrir une session de solution technique dédiée, challenger les alternatives et valider la conception avant toute implémentation. Voir [sélection du contexte](../development/context-selection.md) et [mode opératoire IA](../development/ai-development-operating-model.md).
