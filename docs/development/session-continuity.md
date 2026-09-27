# Continuité entre sessions et passation

**Statut :** convention transverse

Une conversation ne doit jamais être la seule mémoire opérationnelle d'un projet.

## 1. Point de reprise unique

Chaque projet important maintient un point courant, typiquement :

~~~text
docs/continuity/CURRENT_HANDOFF.md
~~~

Une nouvelle session ne doit pas choisir entre plusieurs passations « probablement récentes ».

## 2. Contenu minimum

Le handoff contient :

- date ;
- dépôt ;
- branche active ;
- HEAD/commit de référence lorsque utile ;
- tranche active ;
- dernier jalon validé ;
- VALIDATED ;
- IMPLEMENTED / TECHNICALLY GREEN BUT REVIEW PENDING ;
- IMPLEMENTED BUT NOT YET VALIDATED ;
- DECIDED BUT NOT YET IMPLEMENTED ;
- statut de la revue code / architecture ;
- décisions ou corrections issues de cette revue ;
- invariants ;
- dette/temporaire volontaire ;
- fichiers à lire ;
- tests/commandes de validation ;
- problèmes connus ;
- prochaine action exacte.

## 3. Mise à jour continue

Mettre à jour le handoff lorsqu'une étape change réellement l'état : décision
acceptée, tranche implémentée, CI terminée, validation locale, revue
code/architecture terminée, correction structurelle importante ou changement de
branche.

Une tranche techniquement verte mais dont la revue partagée n'a pas encore eu
lieu doit être décrite explicitement comme telle. Elle ne devient pas
`VALIDATED` uniquement parce que les tests passent.

Le handoff décrit l'état. Git prouve le code.

## 4. Reprise

Une nouvelle session suit :

~~~text
README
→ handoff
→ tranche active
→ ADR nécessaires
→ branche + HEAD réel
→ fichiers concernés
~~~

Puis compare la documentation au code.

## 5. Décisions transverses

Lorsqu'une décision devient commune à plusieurs projets :

- la centraliser dans NexusPrincipia ;
- remplacer la duplication locale par un lien ;
- garder localement uniquement le delta propre au projet.

## 6. Historique

Les anciens handoffs peuvent être archivés à des milestones utiles, mais ne doivent pas concurrencer le point courant.
