# Conventions Python

**Statut :** référence partagée initiale  
**Origine :** Claviger — Python 3.12+, discord.py, aiosqlite, pytest, Ruff

Le pyproject.toml du projet reste la source de vérité technique.

## Configuration du projet

Préférer pyproject.toml pour métadonnées, version Python, dépendances, pytest, Ruff et packaging.

Éviter des fichiers de configuration redondants sans raison.

## Layout

Pour une application packagée, préférer un layout source clair :

~~~text
src/<package>/
tests/
~~~

Faire refléter les sous-domaines dans tests lorsque cela simplifie la navigation.

## Async

Dans un runtime async :

- ne pas bloquer l'event loop avec de longues I/O synchrones ;
- utiliser les clients async adaptés ;
- garder clair ce qui est réellement async ;
- ne pas mettre async sur une fonction pure sans await utile.

aiosqlite est cohérent pour un bot Discord lorsque les accès DB ne doivent pas bloquer la boucle d'événements.

## Boundaries

Valeur par défaut :

~~~text
commands / handlers
→ entrée + adaptation

services
→ orchestration / métier

repositories
→ persistance

models
→ données / concepts
~~~

Une commande Discord ne doit pas devenir le backend métier. Un repository ne décide pas de la politique métier.

## Typage

Utiliser les type hints sur les contrats significatifs pour la compréhension, les outils et les refactors, sans transformer chaque fonction triviale en exercice de typage.

## Configuration

Valider la configuration au bootstrap : champs obligatoires, formats, IDs, chemins, combinaisons incompatibles et versions si nécessaire.

Un mauvais JSON/.env/config ne doit pas produire une panne tardive uniquement sur une machine.

## Ruff

Ruff est la référence actuelle pour Claviger. Les règles restent définies par le pyproject du projet.

Traiter les violations plutôt que désactiver globalement le lint par confort.

## Pytest

Les tests racontent un comportement. Utiliser pytest-asyncio ou l'outil choisi pour l'async.

Séparer unit tests et runtime/intégration lorsque coût et dépendances diffèrent.

## SQLite

- accès DB regroupé dans la couche dédiée ;
- transactions explicites pour les séquences critiques ;
- pas de SQL dispersé dans commands/services ;
- migrations/évolutions de schéma versionnées ;
- mode dégradé/récupération si la DB porte une autorité importante.

## Exceptions

Ne pas utiliser un except Exception silencieux.

À une boundary où une capture large est justifiée : logger le contexte, préserver la cause et distinguer échec métier et échec technique.

## Logging

Préférer logging ou l'abstraction structurée du projet à print pour le runtime.

Respecter la référence NexusPrincipia Debug & Observability.

## Fichiers

Préférer pathlib.Path lorsque cela améliore portabilité et lisibilité.

Pour les fichiers critiques : staging, validation puis remplacement atomique lorsque possible.

## Valeurs mutables

Éviter les valeurs mutables par défaut dans les signatures. Utiliser None + initialisation locale ou default_factory.

## Documentation

Documenter contrats, invariants, effets de bord et comportements async non évidents.

Voir [Documentation conventions](../documentation-conventions.md).
