# Mode opératoire — Développement assisté par IA

**Statut :** référence transverse active  
**Portée :** projets développés conjointement par Damien et une assistante IA  
**Origine :** méthode consolidée à partir de Claviger et de la session GameSaveSync du 27 septembre 2026

## 1. Objectif

Le but n'est pas de demander à l'IA de produire un projet complet en une seule passe.

Le but est de combiner :

- l'expérience et les décisions du développeur ;
- la capacité de l'IA à analyser, produire, tester, documenter et refactorer rapidement ;
- une tranche fonctionnelle suffisamment large pour faire avancer réellement le produit ;
- des checkpoints techniques internes suffisamment petits pour garder le code testable et réversible ;
- une validation humaine réelle avant de considérer la tranche comme acquise.

Le code peut être largement délégué. La direction du produit, les arbitrages structurants, la compréhension de l'architecture et l'acceptation finale ne le sont pas.

### Principe de fluidité

La méthode doit protéger le projet **sans devenir le projet**.

**Une tranche fonctionnelle correspond à une session de développement dédiée.** Exemple : H2 dans une session, H3 dans une nouvelle session.

À l'intérieur de cette session, l'assistante peut découper librement le travail en checkpoints techniques internes pour coder, tester et committer proprement. Ces checkpoints ne deviennent pas des mini-tranches et ne déclenchent pas de nouvelle session.

~~~text
session dédiée à H2
→ plusieurs checkpoints techniques internes
→ tests et corrections continus
→ pauses seulement sur vraies décisions
→ revue de code ciblée sur les points tricky
→ validation / acceptation de H2
→ nouvelle session dédiée à H3
~~~

La rigueur reste forte ; la cérémonie reste proportionnée.

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


## 5. Travail par tranche-session et checkpoints internes

Une tranche est une unité fonctionnelle visible et cohérente : serveur central, agent, pipeline de transfert, onboarding, module Admin, etc.

**Chaque tranche dispose de sa propre session de développement.**

Une tranche peut contenir plusieurs sous-problèmes techniques tant qu'ils concourent au même objectif fonctionnel et architectural.

L'assistante découpe la tranche en checkpoints techniques internes pour :

- limiter la taille des changements ;
- garder la CI utile ;
- corriger tôt ;
- éviter un commit monolithique ;
- préserver une reprise claire si la session devient longue.

Ces checkpoints ne nécessitent **pas par défaut** :

- une acceptation explicite du développeur ;
- une revue pédagogique complète ;
- une validation locale séparée ;
- une clôture documentaire autonome ;
- une nouvelle session.

L'assistante suspend l'implémentation pour arbitrage seulement lorsqu'il existe une vraie décision : alternative architecturale crédible, risque de perte/corruption, changement difficilement réversible, rupture de contrat partagé, extension réelle du scope ou information métier que seule la personne peut fournir.

Une bonne idée hors scope va au backlog. Le bon moment compte autant que la bonne idée.


## 6. Protocole avant implémentation

Au début de la session dédiée à une tranche significative, cadrer seulement ce qui est nécessaire pour partir dans la bonne direction :

- objectif de la tranche ;
- frontières architecturales ;
- principaux risques/invariants ;
- vraies décisions bloquantes ;
- critères de réussite.

Ne pas transformer ce cadrage en spécification exhaustive si le code et les tests permettront de préciser le reste plus vite.

Une fois la tranche lancée, ne pas recommencer ce protocole pour chaque checkpoint interne.

Pour une décision architecturale non triviale et irréversible ou coûteuse à reprendre, attendre la validation du développeur avant l'implémentation.

Les questions pédagogiques sont de préférence posées pendant la revue de code, sur une implémentation réelle.

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
fonctionnelle**. Elle n'est pas exigée après chaque checkpoint interne.

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

La revue privilégie le **code réellement intéressant à apprendre** plutôt qu'un compte rendu exhaustif.

L'assistante guide le développeur vers quelques fichiers ou méthodes importants et explique notamment :

- un mécanisme tricky ;
- une frontière architecturale importante ;
- un invariant de sécurité ;
- un ordre d'opérations non évident ;
- une API ou un compromis qui mérite d'être retenu ;
- le point où chercher si le comportement casse demain.

Les changements triviaux peuvent être résumés sans revue ligne par ligne.

### 8.1 Questions utiles, pas questionnaires

La revue ne doit pas devenir une série de questions de restitution. Quelques questions ciblées valent mieux qu'un questionnaire systématique.

Les questions servent à faire raisonner sur un vrai morceau de code, une frontière ou un compromis.

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

La revue complète s'applique à la tranche fonctionnelle. Les checkpoints internes n'exigent pas une mini-revue séparée.

Pendant la tranche, l'assistante peut néanmoins signaler immédiatement un point tricky ou structurant lorsqu'il est utile de le voir dans le code au moment où il apparaît.

Une micro-correction peut être résumée en une phrase. Une nouvelle boundary de persistence, un mécanisme de recovery ou une architecture réseau mérite une vraie discussion.

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

## 11. Validation d'une tranche fonctionnelle

Les checkpoints internes peuvent être techniquement verts sans nécessiter une acceptation formelle séparée.

Une tranche fonctionnelle est validée seulement si :

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

Maintenir la documentation lorsqu'une évolution change réellement l'architecture, le contrat, les invariants ou l'état de reprise.

Ne pas produire une clôture documentaire complète pour chaque checkpoint interne.

Au minimum, garder suffisamment d'information pour reprendre :

- état de la tranche fonctionnelle ;
- décisions structurantes ;
- validations significatives ;
- limites connues ;
- branche/commit de référence lorsque utile ;
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

Le cycle standard est :

~~~text
nouvelle session dédiée à la tranche
→ analyse de l'existant
→ cadrage léger
→ arbitrage des seules décisions réellement bloquantes
→ implémentation continue par checkpoints internes
→ tests / CI / corrections en continu
→ revue de code ciblée sur les points tricky et structurants
→ correction structurelle éventuelle
→ validation locale / smoke pertinents
→ acceptation explicite de la tranche fonctionnelle
→ documentation / handoff à jour
→ nouvelle session pour la tranche suivante
~~~

La revue intervient volontairement **avant** la fermeture de la tranche. Elle
n'est pas un compte rendu tardif : c'est une dernière barrière contre les
fondations fragiles avant que la tranche suivante ne construise dessus.

Ce rythme est plus prudent qu'un « génère tout le projet », mais conserve
contrôle, compréhension et résilience tout en allant beaucoup plus vite qu'une
production entièrement manuelle.
