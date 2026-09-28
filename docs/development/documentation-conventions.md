# Conventions de documentation

**Statut :** convention transverse

La documentation réduit le coût de compréhension et de reprise. Elle ne doit pas devenir du bruit.

## 1. Niveaux

~~~text
code + types
→ fonctionnement immédiat

commentaires / docstrings
→ pourquoi une contrainte existe

tests
→ comportements démontrés

README local
→ responsabilité d'un dossier

README racine
→ but du projet et navigation

architecture
→ frontières et flux

ADR
→ décision structurante

tranche
→ travail en cours

handoff
→ état exact de reprise
~~~

## 2. Commentaires

Commenter lorsqu'il faut préserver une information que le code n'exprime pas bien :

- invariant ;
- raison de sécurité ;
- ordre obligatoire ;
- compromis ;
- effet de bord ;
- workaround ;
- règle non évidente.

Ne pas commenter ce que noms, types ou instructions disent déjà.

## 3. README de dossiers significatifs

Un dossier représentant une vraie frontière architecturale reçoit un README court expliquant :

1. pourquoi il existe ;
2. ce qui doit y vivre ;
3. ce qui ne doit pas y vivre ;
4. ses collaborations ;
5. le flux principal si utile.

Ne pas créer de README artificiel pour chaque dossier trivial.

## 4. Documentation des flux

Quand un système devient complexe, documenter aussi le trajet d'une opération :

~~~text
entrée
→ validation
→ use case/service
→ repository/storage
→ effet externe
→ résultat/reporting
~~~

## 5. ADR

Créer un ADR pour les décisions coûteuses ou structurantes.

Si une décision est remplacée, documenter explicitement la supersession au lieu de réécrire silencieusement l'historique.

## 6. Documentation de tranche

La documentation suit la tranche fonctionnelle, pas chacun de ses checkpoints internes.

Elle maintient objectif, scope, décisions structurantes, état global, validations significatives, limites et prochaine action.

Éviter de transformer chaque sous-étape technique en longue sous-section lorsque le code, les commentaires et les tests suffisent déjà à l'expliquer.

## 7. Documentation transverse vs locale

Une règle commune à plusieurs applications vit dans NexusPrincipia.

Les repos applicatifs gardent un pointeur et leurs différences locales.

## 8. Miroir source / tests

Lorsque cela facilite la navigation, faire refléter l'organisation source dans les tests. Les tests d'intégration ou end-to-end peuvent rester transverses.

## 9. Documentation périmée

Une documentation décrivant une architecture qui n'existe plus est un bug. La mettre à jour dans la même tranche que le changement.

La conversation de développement n'a pas vocation à reproduire toute la documentation : les détails durables doivent vivre dans le code/commentaires/tests/README/ADR appropriés.
