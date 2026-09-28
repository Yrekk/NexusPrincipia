# Entrypoints et opérations administratives réutilisables

**Statut :** convention transverse active

Cette règle s'applique aux points d'entrée tels que :

- `Program.cs` / top-level statements .NET ;
- `Main.py`, `main.py` ou fonction `main()` Python ;
- bootstrap de service ;
- commande CLI racine ;
- startup d'un worker, bot ou serveur.

## 1. Test architectural de base

Pour toute logique candidate à être placée dans un entrypoint, poser d'abord la question :

> **Cette action pourrait-elle être invoquée par l'interface Admin, une future IA/agent, une commande CLI, un autre adapter ou un outil d'exploitation ?**

Si la réponse est **oui pour au moins un de ces consommateurs**, alors la logique ne doit pas vivre dans l'entrypoint.

Elle doit devenir une opération réutilisable :

~~~text
Program.cs / main.py / CLI / Admin / Agent IA
                  ↓
            adapter / tool
                  ↓
         application use case
            ou service
                  ↓
        infrastructure / domaine
~~~

L'entrypoint peut déclencher l'opération. Il ne doit pas en posséder l'implémentation.

## 2. Responsabilité d'un entrypoint

Un entrypoint doit principalement :

- lire/binder la configuration nécessaire au bootstrap ;
- construire le conteneur de dépendances ;
- enregistrer les adapters ;
- initialiser le runtime ;
- démarrer la boucle/host ;
- parser une commande racine puis déléguer ;
- exposer une boundary de transport.

Il ne doit pas accumuler :

- migrations de base de données ;
- snapshot/backup/restore ;
- recovery ;
- réconciliation ;
- maintenance ;
- nettoyage métier ;
- import/export ;
- rotation opérationnelle ;
- orchestration complexe ;
- logique métier ;
- opérations que l'Admin ou une IA pourraient vouloir lancer plus tard.

## 3. Règle de réutilisation

Une capacité opérationnelle doit avoir **une implémentation d'autorité**.

Exemple :

~~~text
Interface Admin ─┐
CLI maintenance ─┤
Agent IA / tool ─┤
Startup adapter ─┘
        ↓
ApplyDatabaseMigration
        ↓
snapshot
→ validation
→ migration
→ validation finale
~~~

Les adapters ne réimplémentent pas les étapes.

Cela évite :

~~~text
Program.cs migration logic
!=
Admin migration logic
!=
AI tool migration logic
!=
CLI migration logic
~~~

## 4. Startup n'est pas une autorité administrative

Le fait qu'une action soit techniquement possible au démarrage ne signifie pas que le startup doit la décider.

Le startup peut :

- inspecter ;
- signaler ;
- refuser un mode opérationnel incompatible ;
- déléguer une action explicitement demandée par une politique validée.

Le startup ne doit pas transformer une opération administrative en effet de bord invisible.

### Plusieurs réponses légitimes = décision hors entrypoint

Lorsqu'un état peut conduire à plusieurs actions valides selon le contexte opérationnel, l'entrypoint ne doit pas choisir seul.

Exemple :

~~~text
DB absente / incompatible / en cours d'opération
→ initialiser ?
→ migrer ?
→ restaurer ?
→ rester en mode minimal ?
→ attendre parce qu'une maintenance externe est en cours ?
~~~

Coder une réponse automatique dans `Program.cs` ou `main.py` reviendrait à figer une hypothèse de contexte que l'entrypoint ne possède pas.

La règle est donc :

> **un entrypoint peut observer l'état ; il ne tranche pas silencieusement entre plusieurs actions administratives légitimes.**

Cette décision appartient à un use case/service réutilisable, invoqué par un acteur ou une politique explicitement autorisée : Admin, CLI, opérateur humain ou futur agent IA.

Pour les opérations destructives ou sensibles, préférer :

~~~text
détection automatique
≠
décision automatique
~~~

Exemple GameSaveSync :

~~~text
startup
→ détecte une migration pending
→ signale "migration required"
→ ne modifie pas la DB

Admin / CLI / future IA
→ demande explicitement l'opération
→ service/use case contrôlé
~~~

## 5. Future IA / tools

Une IA opératrice ne doit pas recevoir une voie spéciale cachée dans le code.

Elle consomme les mêmes capacités applicatives que les autres adapters, avec :

- autorisation explicite ;
- contrats d'entrée/sortie ;
- garde-fous ;
- audit ;
- mode dry-run/inspection lorsque pertinent ;
- rollback/recovery lorsque l'opération l'exige.

La future exposition sous forme de `tool` ne change pas la responsabilité métier du service.

## 6. Exceptions

Tout code de startup n'a pas besoin de devenir un service.

Une logique peut rester dans l'entrypoint lorsqu'elle est réellement limitée au bootstrap et n'a aucun sens comme opération réutilisable.

Exemples :

- création du host ;
- branchement middleware ;
- lecture du nom d'environnement ;
- enregistrement DI ;
- démarrage du client Discord ;
- lancement de la boucle principale.

La question n'est donc pas :

> « Peut-on extraire ce code ? »

mais :

> « Cette action représente-t-elle une capacité du système qu'un autre adapter pourrait raisonnablement vouloir invoquer ? »

Si oui, l'extraire tôt évite un refactor une tranche trop tard.

## 7. Revue de code

Pendant la revue partagée, contrôler explicitement les entrypoints.

Question standard :

> « Est-ce qu'une action ici devrait demain être utilisable depuis NexusPrincipia, une CLI ou un agent IA ? »

Si oui et que l'entrypoint contient encore l'implémentation, la frontière doit être corrigée avant fermeture de la tranche lorsqu'elle est déjà pertinente.

## 8. Conséquence pour les nouveaux projets

La Solution Technique et le prompt maître d'un nouveau projet doivent intégrer cette convention.

Les noms exacts varient selon la stack :

~~~text
C#/.NET
Program.cs
→ composition root / adapter
→ Application use cases

Python
main.py
→ bootstrap / adapter
→ services/use cases

Autre stack
entrypoint
→ adapter
→ capacité réutilisable
~~~

La règle porte sur la responsabilité, pas sur un framework particulier.

## 9. Inspection / classification / autorité

Pour toute capacité qui découvre des faits puis doit les interpréter avant d'autoriser des actions, appliquer la référence [Inspection, classification and authorized choice](../architecture/inspection-classification-authority.md).

En particulier :

~~~text
entrypoint / inspector
→ observe
→ propose des classifications compatibles
→ explique
→ ne confirme pas à la place de l'Admin
~~~

Une suggestion de startup ne devient donc jamais, par elle-même, une décision administrative.

Les adapters Admin, CLI et IA doivent recevoir les mêmes faits, candidats et garde-fous.

## 10. Database lifecycle / readiness

For database-backed applications, apply the shared [Database lifecycle, readiness and explicit administrative choice](../architecture/database-lifecycle-readiness.md) reference.

In particular, keep observed database state, runtime readiness and chosen administrative action separate.
