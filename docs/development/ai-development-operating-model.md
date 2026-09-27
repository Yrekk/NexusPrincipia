# Mode opératoire — Développement assisté par IA

**Statut :** référence transverse active  
**Portée :** projets développés conjointement par Damien et une assistante IA  
**Origine :** méthode consolidée à partir de Claviger et de la session GameSaveSync du 27 septembre 2026

## 1. Objectif

Le but n'est pas de demander à l'IA de produire un projet complet en une seule passe.

Le but est de combiner :

- l'expérience et les décisions du développeur ;
- la capacité de l'IA à analyser, produire, tester, documenter et refactorer rapidement ;
- des étapes suffisamment petites pour rester compréhensibles et réversibles ;
- une validation humaine réelle avant de considérer une tranche comme acquise.

Le code peut être largement délégué. La direction du produit, les arbitrages, la compréhension de l'architecture et l'acceptation finale ne le sont pas.

## 2. Gouvernance

### Développeur

Le développeur :

- définit le besoin et les contraintes ;
- apporte l'expérience issue de problèmes déjà rencontrés ;
- arbitre les choix d'architecture ;
- challenge les propositions ;
- accepte ou refuse une tranche ;
- supervise les validations locales et smoke tests ;
- décide des promotions de branche et des déploiements ;
- reste propriétaire du produit et de ses invariants.

Une expérience passée doit pouvoir devenir une règle explicite :

~~~text
problème déjà vécu
→ risque reconnu
→ invariant
→ validation automatique
→ test
→ documentation
~~~

### Assistante IA

L'assistante :

- inspecte l'existant avant de proposer ;
- challenge une idée fragile ou disproportionnée ;
- propose architecture et compromis ;
- produit le code lorsque la tranche est validée ;
- écrit ou adapte les tests ;
- maintient la documentation ;
- explique responsabilités et flux ;
- surveille les incohérences entre code, tests et documentation ;
- évite les abstractions spéculatives sans consommateur réel.

## 3. Sources de vérité

En cas de contradiction :

1. code réel sur la branche de travail ;
2. dernière décision explicite du développeur ;
3. handoff courant ;
4. documentation projet actuelle ;
5. solution technique validée ;
6. prompt maître du projet ;
7. conventions partagées NexusPrincipia ;
8. historique ancien.

Une contradiction importante doit être signalée, pas corrigée silencieusement.

## 4. Démarrage d'une session

Avant un travail substantiel :

1. identifier dépôt et branche ;
2. vérifier le HEAD distant réel ;
3. lire le README principal ;
4. lire le handoff courant ;
5. lire le document de tranche actif ;
6. relire les ADR/décisions nécessaires ;
7. inspecter les fichiers réellement concernés ;
8. seulement ensuite proposer le prochain changement.

Si le développeur annonce un push, re-vérifier le HEAD avant toute nouvelle écriture.

## 5. Travail par tranches

Une tranche doit être limitée à un objectif principal, compréhensible, testable et documentable.

Une bonne idée hors scope va au backlog. Elle n'est pas codée uniquement parce qu'elle est intéressante.

~~~text
checksum utile pour la résilience
≠
checksum à coder pendant le bootstrap
~~~

Le bon moment compte autant que la bonne idée.

## 6. Protocole avant implémentation

Avant une tranche significative, présenter :

- **Objectif** — ce que la tranche résout ;
- **Ce qui sera construit** — fichiers, responsabilités et comportements ;
- **Architecture** — où vit chaque responsabilité et pourquoi ;
- **Points critiques** — invariants, sécurité, persistance, compatibilité, effets de bord ;
- **Tests** — ce qui prouvera la correction ;
- **Question conceptuelle** — une courte question pour conserver la carte mentale du système.

Une réponse « je ne sais pas » est acceptable. La question sert à apprendre et à détecter une incompréhension.

Pour une décision architecturale non triviale, attendre la validation du développeur avant l'implémentation.

## 7. Implémentation

Après validation :

1. re-vérifier branche et HEAD ;
2. modifier le minimum cohérent de fichiers ;
3. préserver les frontières architecturales ;
4. ajouter les tests dans la même tranche ;
5. mettre à jour la documentation ;
6. exécuter les validations disponibles ;
7. produire un commit cohérent lorsque la méthode du projet l'autorise.

### Écriture Git directe

Lorsque l'IA dispose d'un accès GitHub autorisé et que le développeur lui confie l'écriture directe :

- elle peut écrire et committer sur la branche de travail ;
- elle re-vérifie le HEAD avant écriture ;
- elle ne merge pas, ne promeut pas vers stable et ne déploie pas sans autorisation explicite.

## 8. Revue de code ensemble

Le développeur n'a pas besoin de relire mécaniquement chaque ligne.

Il doit néanmoins pouvoir répondre à :

- où vit la nouvelle responsabilité ?
- quel est le flux principal ?
- quelles dépendances ont été introduites ?
- quel invariant protège le test principal ?
- où chercher si ce comportement casse demain ?

L'assistante explique les fichiers importants et leurs relations.

## 9. Code lisible et commentaires

Le code sert de première documentation.

Commenter principalement le **pourquoi** :

- invariant ;
- contrat non évident ;
- raison de sécurité ;
- ordre d'opérations ;
- compromis volontaire ;
- effet de bord ;
- comportement surprenant d'une API externe.

Ne pas sur-commenter les modèles, DTOs ou fonctions dont noms et types sont déjà explicites.

## 10. Tests comme gate

Ordre recommandé :

~~~text
tests ciblés
→ lint / analyse statique / build
→ suite complète
→ CI
→ smoke test réel si pertinent
→ validation humaine
~~~

Règles :

- un test protège un comportement ou invariant réel ;
- ne pas créer de faux tests uniquement pour faire disparaître un warning ;
- expliquer ce que garantit un nouveau test ;
- analyser un échec CI même si le poste local est vert ;
- une CI verte seule ne vaut pas acceptation de tranche.

## 11. Validation d'une tranche

Une tranche est validée seulement si :

- son scope accepté est implémenté ;
- les tests pertinents sont verts ;
- build/lint/analyse statique sont verts selon le projet ;
- les smoke tests nécessaires sont verts ;
- documentation, tranche et handoff sont à jour ;
- le développeur l'accepte explicitement.

Toujours distinguer :

~~~text
VALIDATED
IMPLEMENTED BUT NOT YET VALIDATED
DECIDED BUT NOT YET IMPLEMENTED
~~~

## 12. ADR et décisions structurantes

Créer un ADR lorsqu'une décision est non évidente, coûteuse à inverser, structurante, liée à sécurité/récupération/persistance ou susceptible d'être rediscutée sans contexte.

Un ADR explique contexte, décision, contraintes, conséquences, périmètre et éléments différés.

## 13. Configuration

Une configuration importante doit échouer tôt et explicitement si elle n'est pas valide.

Préférer :

~~~text
config lue
→ validation complète
→ démarrage
~~~

à une erreur tardive au milieu d'un workflow.

## 14. Observabilité

Les applications doivent progressivement suivre la référence NexusPrincipia :

- événements structurés ;
- niveaux cohérents ;
- correlation IDs ;
- contexte métier utile ;
- redaction des secrets ;
- logs locaux rotatifs ;
- Debug activable temporairement lorsque possible.

Voir [Debug & observability](../architecture/debug-observability.md).

## 15. Documentation et passation

Après une étape significative, maintenir :

- état de la tranche ;
- décisions ;
- tests exécutés ;
- limites connues ;
- branche/commit de référence ;
- prochaine action exacte.

Voir [Session continuity](session-continuity.md).

## 16. Definition of Done transverse

Une fonctionnalité n'est pas finie simplement parce qu'elle compile.

~~~text
conception comprise
+ code
+ tests
+ build/lint
+ documentation
+ continuité
+ validation réelle
~~~

## 17. Boucle cible

~~~text
besoin / expérience du développeur
→ analyse de l'existant
→ proposition et challenge
→ validation architecturale
→ implémentation par petite tranche
→ tests + CI
→ revue et explication
→ validation locale / smoke
→ acceptation explicite
→ documentation + handoff à jour
→ tranche suivante
~~~

Ce rythme est plus prudent qu'un « génère tout le projet », mais conserve contrôle, compréhension et résilience tout en allant beaucoup plus vite qu'une production entièrement manuelle.
