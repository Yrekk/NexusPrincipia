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
- **Architecture** — frontières nécessaires pour démarrer sans enfermer le projet ;
- **Points critiques** — invariants, sécurité, persistance, compatibilité, effets de bord ;
- **Tests** — ce qui prouvera la correction ;
- **Décisions bloquantes** — uniquement celles qui doivent réellement être arbitrées avant de coder.

Le cadrage préalable doit permettre de construire dans la bonne direction, sans
chercher à résoudre théoriquement tous les choix possibles avant d'avoir du code
concret à examiner.

Pour une décision architecturale non triviale et irréversible ou coûteuse à
reprendre, attendre la validation du développeur avant l'implémentation.

Les questions pédagogiques de type comparaison / compromis sont de préférence
posées **pendant la revue après validation technique**, lorsque le développeur
peut raisonner sur une implémentation réelle plutôt que deviner le raisonnement
de l'assistante.

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

## 8. Revue de code et d'architecture ensemble

La revue partagée est une **étape obligatoire avant l'acceptation d'une tranche
de développement**.

Elle intervient après une première implémentation et sa validation technique :

~~~text
implémentation
→ tests / build / lint / smoke nécessaires
→ correction des défauts observés
→ revue ensemble
→ éventuelle correction structurelle
→ revalidation si nécessaire
→ acceptation explicite
~~~

Le développeur n'a pas besoin de relire mécaniquement chaque ligne. La revue
doit lui faire parcourir les zones importantes et reconstruire la carte mentale
du changement.

L'assistante présente au minimum :

- les fichiers réellement modifiés ;
- la responsabilité de chacun ;
- le flux principal avant / après ;
- les dépendances introduites ou déplacées ;
- les invariants protégés par les tests ;
- le point où chercher si le comportement casse demain.

### 8.1 Questions de choix plutôt que restitution

La revue ne doit pas principalement demander :

> « Pourquoi l'assistante a-t-elle choisi cette architecture ? »

Elle doit plutôt placer le développeur en situation d'arbitrage :

> « Ici, deux solutions sont crédibles : A et B. Laquelle choisirais-tu dans ce
> contexte, et qu'est-ce qu'on gagne ou perd avec chacune ? »

L'assistante :

1. présente les alternatives réellement plausibles ;
2. donne suffisamment de contexte pour raisonner ;
3. laisse le développeur formuler son choix ;
4. compare ensuite les conséquences à court et long terme ;
5. explique son propre choix si nécessaire ;
6. accepte qu'une meilleure décision émerge de la discussion.

Une réponse différente de celle de l'assistante n'est pas une erreur si elle
repose sur un compromis défendable.

### 8.2 Objectif architectural

Cette revue doit notamment détecter les choix qui fonctionnent aujourd'hui mais
créeraient une dette structurelle disproportionnée demain.

Exemple générique :

~~~text
besoin actuel
→ SQLite suffit

question de revue
→ le métier doit-il dépendre directement de SQLite
  ou d'une frontière de persistence plus abstraite ?

conséquence
→ conserver SQLite aujourd'hui
→ sans rendre une future migration PostgreSQL équivalente à une réécriture
~~~

Une abstraction n'est pas justifiée uniquement par un futur hypothétique. Elle
l'est lorsque son coût actuel reste raisonnable et qu'elle protège une frontière
déjà pertinente.

### 8.3 Ce qui se passe si la revue révèle un problème

Si la revue découvre :

- une mauvaise frontière ;
- une dépendance trop concrète ;
- un invariant mal placé ;
- une évolution proche qui rendrait le choix actuel coûteux ;

alors le problème structurel est corrigé **avant de fermer la tranche**, puis les
validations nécessaires sont rejouées.

Une amélioration utile mais non bloquante va au backlog.

L'objectif est d'éviter :

~~~text
tranche N : choix fragile
→ tranche N+1 construite dessus
→ tranche N+2 découvre la dette
→ refactor tardif
~~~

et de préférer :

~~~text
tranche N : implémentation
→ tests
→ revue
→ correction de la fondation
→ acceptation
→ tranche N+1 sur une base saine
~~~

### 8.4 Proportionnalité

Cette étape s'applique à chaque développement cohérent, mais sa profondeur reste
proportionnée au changement.

Une micro-correction peut recevoir une revue de quelques phrases. Une nouvelle
boundary de persistence, un mécanisme de recovery ou une architecture réseau
méritent une vraie discussion de compromis.

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
- la revue de code / architecture partagée a eu lieu ;
- les problèmes structurels bloquants révélés par cette revue ont été corrigés
  et revalidés ;
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

Une fonctionnalité n'est pas finie simplement parce qu'elle compile ou que ses
tests sont verts.

~~~text
cadrage suffisant
+ code
+ tests
+ build/lint
+ smoke si nécessaire
+ revue code / architecture ensemble
+ compromis compris
+ corrections structurelles éventuelles
+ documentation
+ continuité
+ acceptation explicite
~~~

## 17. Boucle cible

Le cycle standard de chaque développement est :

~~~text
besoin / expérience du développeur
→ analyse de l'existant
→ cadrage de la tranche
→ arbitrage des seules décisions bloquantes
→ implémentation par petite tranche
→ commit/push lorsque autorisé
→ tests + build/lint + smoke pertinents
→ corrections jusqu'à état techniquement vert
→ revue de code ensemble
→ discussion des choix / alternatives / compromis
→ correction structurelle éventuelle
→ revalidation si le code change
→ acceptation explicite du développeur
→ documentation + handoff à jour
→ tranche suivante
~~~

La revue intervient volontairement **avant** la fermeture de la tranche. Elle
n'est pas un compte rendu tardif : c'est une dernière barrière contre les
fondations fragiles avant que la tranche suivante ne construise dessus.

Ce rythme est plus prudent qu'un « génère tout le projet », mais conserve
contrôle, compréhension et résilience tout en allant beaucoup plus vite qu'une
production entièrement manuelle.
