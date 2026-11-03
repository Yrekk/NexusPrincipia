# Sélection du contexte de travail — Git, Drive et sessions IA

**Statut :** cadre transverse de sélection ; cartographie documentaire détaillée à réaliser ultérieurement.  
**Portée :** développement assisté, discussion d'architecture, pédagogie et analyse d'expérience professionnelle.

## 1. But et frontières

Ne pas redécouvrir à chaque conversation les compétences déjà établies, les décisions du projet et l'état exact d'une tranche. **Charger seulement le contexte utile** au travail demandé, sans transformer chaque démarrage en relecture intégrale de Drive et Git.

Sources aux responsabilités différentes :

- **NexusPrincipia :** conventions transverses de conception, développement, tests et exploitation. Une seule définition de chaque règle commune.
- **Dépôt applicatif :** code et branche/HEAD réels, architecture, ADR, tranche en cours et point de reprise courant. Il reste l'autorité pour sa continuité.
- **Google Drive personnel/professionnel :** dossier de compétences et récits d'expérience, par exemple retours techniques permettant de calibrer le niveau d'explication et les analogies. Il ne remplace ni le code ni un ADR.
- **Conversation :** objectif et décisions de la session ; les décisions durables doivent rejoindre le dépôt/document approprié.

La **sélection des documents** est distincte de leur **autorité**. En cas de contradiction, respecter la hiérarchie déjà définie dans [le mode opératoire IA](ai-development-operating-model.md#3-sources-de-vérité) et signaler l'écart.

## 2. Quatre niveaux de chargement

| Niveau | Usage | À charger |
| --- | --- | --- |
| **0 — Références transverses** | Règles communes et expérience préexistante pertinente | Pages Nexus nécessaires ; résumés ciblés des compétences/retours Drive quand ils changent réellement la discussion |
| **1 — Contexte projet** | Architecture et contrats particuliers | README du dépôt, ST/prompt maître, ADR utiles, branche et HEAD vérifiés |
| **2 — Contexte de tranche/session** | Savoir où reprendre exactement | CURRENT_HANDOFF, tranche active, décisions récentes, fichiers concernés, état des validations |
| **3 — Contexte spécialisé à la demande** | Risques, incidents, audits, tests de panne ou technologie nouvelle | Références détaillées choisies pour la question précise ; historiques anciens seulement si nécessaire |

Ces niveaux sont **des filtres de lecture, pas une nouvelle arborescence obligatoire**, ni quatre copies de la documentation. Pour les projets, la continuité reste gérée par [Session continuity](session-continuity.md) et leur propre Git.

## 3. Séquence d'ouverture

1. Identifier la demande, le projet, le mode de collaboration et les sources accessibles.
2. Charger les références transverses strictement pertinentes ; exploiter Drive pour connaître les acquis professionnels lorsque cela peut changer la pédagogie ou le raisonnement.
3. Pour une tâche de développement, consulter le dépôt et vérifier la branche/HEAD réel, puis le handoff et la tranche. Les documents Drive ne suffisent jamais à déterminer l'état du code.
4. Charger le niveau spécialisé correspondant aux risques du travail (persistance, asynchrone, sécurité, tests de panne, etc.).
5. Signaler les sources manquantes, obsolètes ou contradictoires ; ne pas prétendre avoir lu un document non accessible.

Exemple d'intention de session :

~~~text
Contexte pertinent à charger :
- Nexus : méthode de développement + tests de défaillance
- Drive : expérience technique et connaissances démontrées utiles
- Projet : GameSaveSync, branche/HEAD, CURRENT_HANDOFF, tranche active
- Mode : production, revue des invariants et tests live contrôlés
~~~

Cette phrase exprime un **choix de sources**. Elle ne déclenche pas, à elle seule, un chargement automatisé ni l'accès à un document sans connexion et permissions appropriées.

## 4. Deux modalités de collaboration

- **Livraison prioritaire :** cadrage proportionné, confrontation des choix, implémentation par tranches, tests et revue ciblée ; éviter les rappels pédagogiques inutiles. Le développeur garde la validation des invariants et des comportements critiques.
- **Mode pédagogique :** laisser davantage le développeur proposer architecture, implémentation et scénarios ; l'assistante évalue sans complaisance, compare les compromis et aide à tester. Une explication nouvelle s'appuie sur ses acquis, sans présumer qu'il maîtrise toute technologie voisine.

Le mode pédagogique ne réduit pas l'exigence de tests. Le mode livraison n'interdit pas une explication lorsque son utilité est réelle.

## 5. Cartographie Drive à construire plus tard

Prévoir **un index privé**, plutôt que lister ici les documents personnels :

- catégorie, rôle, sujet, résumé court, mots-clés de déclenchement ;
- lien ou identifiant de source, propriétaire et accessibilité ;
- date de contrôle, degré de fiabilité, éventuel remplacement ;
- quand charger, quand ne pas charger, relation avec un projet.

À ce stade, **cet inventaire n'existe pas** dans NexusPrincipia : ne pas le présumer ni ajouter de faux liens.

**Confidentialité :** ce dépôt est public. Ne pas y recopier des récits professionnels internes, données sensibles, identifiants Drive privés, liens confidentiels ou informations personnelles. Conserver les contenus détaillés dans leur espace autorisé ; n'y référencer ici que des catégories génériques et des règles de consultation.

## 6. Maintien à jour

Une nouvelle expérience vérifiée peut enrichir le dossier de compétences Drive ; une règle générique issue d'une erreur répétée peut devenir une convention Nexus ; un invariant propre à une application reste dans son Git. Réviser les associations de contexte lorsqu'elles deviennent obsolètes, sans créer de copie concurrente du handoff.
