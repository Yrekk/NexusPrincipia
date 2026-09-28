# Conventions C# / .NET

**Statut :** référence partagée initiale  
**Origine :** pratiques validées pendant GameSaveSync (.NET 10)

## Compilation stricte

Pour les projets modernes, préférer lorsque pertinent :

~~~xml
<Nullable>enable</Nullable>
<TreatWarningsAsErrors>true</TreatWarningsAsErrors>
<Deterministic>true</Deterministic>
~~~

Traiter les warnings plutôt que désactiver globalement les garde-fous.

## Types et nullabilité

- exprimer explicitement l'absence ;
- valider les constructeurs/entrées importantes ;
- utiliser des value objects pour les concepts dont les invariants comptent ;
- séparer validation d'une identité et décision de générer une nouvelle identité.

## Immutabilité

Préférer l'immutabilité pour identifiants, états publiés, contrats, résultats et configuration validée lorsque le modèle s'y prête.

## Frontières

Pour une application suffisamment grande :

~~~text
Core / Domain
→ règles pures

Application
→ use cases + ports

Infrastructure / Persistence / Storage
→ effets externes

Host / Server
→ composition root + transport
~~~

La logique métier ne dépend pas inutilement d'EF Core, ASP.NET, filesystem ou providers externes.

## Composition root et DI

Assembler les dépendances concrètes au bord de l'application.

Éviter service locator, accès global statique aux services et logique métier dans `Program.cs`.

Si une action pourrait devenir une commande Admin, CLI ou un tool d'agent IA, `Program.cs` ne doit contenir que l'adapter/l'appel vers le use case ou service réutilisable.

Voir [Entrypoints and reusable administrative operations](../entrypoints-and-reusable-operations.md).

## EF Core

Lorsque utilisé :

- l'isoler dans la persistance ;
- versionner les migrations ;
- ne pas exposer les entities EF comme contrats HTTP ;
- éviter le lazy loading par défaut ;
- rendre les transactions critiques explicites ;
- garder le domaine indépendant du provider ;
- isoler tout SQL brut justifié.

## Configuration

Valider la configuration au démarrage avec des options typées ou une validation explicite.

Ne pas attendre un workflow tardif pour découvrir une clé absente ou mal formée.

## Async

Pour les I/O importantes :

- utiliser les API async lorsqu'elles ont un bénéfice réel ;
- propager CancellationToken sur les opérations longues/annulables ;
- éviter Result/Wait sur du code async ;
- ne pas rendre async une méthode pure sans raison.

## Logging

Utiliser ILogger<T> ou l'abstraction projet et des champs structurés plutôt qu'une concaténation opaque.

Respecter la référence NexusPrincipia sur redaction, scopes, correlation et Debug runtime.

## Exceptions

- exceptions précises pour violations de contrat ;
- résultats métier explicites lorsque l'exception n'est pas appropriée ;
- ne pas avaler Exception ;
- préserver la cause en cas de wrapping.

## Tests

xUnit est la référence actuelle de GameSaveSync.

Les tests doivent être déterministes, lisibles et séparés entre unit/integration lorsque les dépendances réelles diffèrent.

Validation typique : build Release puis tests Release sans rebuild.

## Documentation

Documenter les types publics importants et invariants non évidents. Éviter les XML docs qui répètent uniquement le nom d'une propriété.

Voir [Documentation conventions](../documentation-conventions.md).
