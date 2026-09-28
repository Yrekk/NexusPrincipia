# Démarrage d'un nouveau projet — Solution technique + prompt maître

**Statut :** convention transverse

Un projet non trivial commence idéalement ainsi :

~~~text
besoin
→ Solution Technique
→ prompt maître dédié
→ bootstrap du dépôt
→ développement par tranches
~~~

## 1. Solution Technique

La Solution Technique décrit la cible avant de figer l'implémentation.

Elle couvre selon le projet :

- objectif et utilisateurs ;
- périmètre V1 et hors périmètre ;
- architecture générale ;
- composants et responsabilités ;
- flux de données ;
- persistance ;
- transport/réseau ;
- sécurité ;
- résilience et récupération ;
- configuration ;
- stratégie de tests ;
- déploiement ;
- roadmap en tranches ;
- décisions encore ouvertes.

La ST peut évoluer, mais évite que chaque session réinvente la direction.

## 2. Prompt maître dédié

Chaque projet important possède un prompt maître adapté.

Il précise notamment :

- rôle attendu de l'IA ;
- objectif du projet ;
- architecture et contraintes déjà acceptées ;
- niveau d'explication attendu ;
- hiérarchie des sources de vérité ;
- workflow de développement ;
- règles de sécurité ;
- règles de tests et documentation ;
- comportement avant/après une tranche ;
- cycle obligatoire implémentation → validation technique → revue code/architecture
  → correction éventuelle → acceptation ;
- pédagogie orientée arbitrage entre alternatives plutôt que restitution du
  raisonnement de l'IA ;
- décisions nécessitant une validation explicite.

Le prompt maître complète NexusPrincipia. Il ne recopie pas inutilement toutes les conventions communes.

## 3. Bootstrap H0

Le premier jalon crée surtout des frontières :

- solution/projet/package ;
- structure source et tests ;
- CI ;
- configuration minimale ;
- documentation ;
- conventions Git ;
- host technique si la frontière réseau est déjà décidée;
- entrypoint minimal : composition/bootstrap uniquement, les capacités réutilisables Admin/CLI/IA vivent dans des services/use cases.

Éviter d'ajouter de la logique métier spéculative uniquement pour « remplir » le projet.

## 4. Documentation initiale

Structure indicative :

~~~text
README.md
docs/
├── architecture/
├── continuity/
├── decisions/
├── tranches/
└── development/
src/
tests/
~~~

Les conventions transverses restent dans NexusPrincipia. Le dossier development local contient surtout des liens et exceptions propres au projet.

## 5. CI tôt

La CI est installée au bootstrap.

Elle vérifie ce que le projet considère bloquant : restore/install, build ou syntaxe, lint/analyse statique et tests.

Elle devient un deuxième environnement de validation.

## 6. Continuité dès le départ

Créer un handoff courant dès le bootstrap pour qu'une nouvelle session puisse reprendre sans dépendre de la conversation d'origine.

Voir [Session continuity](session-continuity.md).

## 7. Revue avant fermeture d'une tranche

Dès le bootstrap, le projet doit hériter du cycle transverse :

~~~text
implémenter
→ tester / corriger
→ revoir le code ensemble
→ discuter les alternatives structurantes
→ corriger la fondation si nécessaire
→ accepter
→ passer à la tranche suivante
~~~

Cette revue doit être proportionnée, mais ne doit pas disparaître au motif que
les tests sont verts.

## 8. Entrypoints

Dès H0, appliquer la convention [Entrypoints et opérations administratives réutilisables](entrypoints-and-reusable-operations.md).

Un `Program.cs`, `main.py` ou équivalent ne doit pas devenir le propriétaire d'une capacité qui pourrait ensuite être appelée depuis l'Admin, une CLI ou un agent IA.

## 9. Règle finale

La ST et le prompt maître ne servent pas à créer de la bureaucratie.

Ils servent à rendre l'IA **rapide dans la bonne direction**.
