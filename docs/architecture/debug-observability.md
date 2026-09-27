# Référence transverse — Debug & Observabilité dans l’interface Admin

**Statut :** document de référence / backlog transverse  
**Destination cible :** dépôt de l’interface Admin commune  
**Portée :** GameSaveSync, Claviger et futures applications/services administrés  
**Origine :** décisions et échanges de conception du 27 septembre 2026

---

## 1. Objectif

L’interface Admin doit permettre de diagnostiquer rapidement un problème sans devoir :

- se connecter en SSH sur une machine ;
- ouvrir manuellement les fichiers de logs ;
- redéployer un conteneur juste pour changer le niveau de journalisation ;
- parcourir un « mur de logs » mélangeant tous les composants, utilisateurs et workflows ;
- exposer inutilement des secrets ou des données sensibles.

L’objectif n’est pas de « tout logger ».

L’objectif est de fournir une **observabilité structurée, filtrable, corrélable et pilotable depuis l’interface Admin**, avec la possibilité d’augmenter temporairement le niveau de détail lorsque cela est nécessaire.

---

## 2. Principe général

Le niveau de log doit pouvoir être modifié **à chaud**, sans redéploiement du conteneur, lorsque la stack technique le permet.

Exemple conceptuel :

```text
Mode normal
Information / Warning / Error
        ↓
Admin : « Activer Debug 30 min »
        ↓
Le serveur applique le niveau Debug à chaud
        ↓
Expiration automatique
        ↓
Retour au niveau normal
```

Le conteneur ne doit pas être recréé pour une simple modification du niveau de logs.

Un redéploiement/restart ne doit être nécessaire que pour les paramètres qui sont réellement **startup-only**.

---

## 3. Pourquoi éviter un redéploiement Docker pour le Debug

Il serait techniquement possible de faire :

```text
LOG_LEVEL=Debug
↓
docker compose up -d --force-recreate
```

depuis une interface Admin.

Cette solution n’est toutefois pas privilégiée pour un simple changement de niveau de logs.

Elle introduirait inutilement :

- une interruption ou un redémarrage du service ;
- une dépendance directe de l’interface Admin à Docker ;
- des droits d’orchestration plus importants ;
- potentiellement l’exposition du socket Docker ;
- un couplage fort entre l’interface Admin et l’infrastructure de déploiement.

La préférence est donc :

```text
Admin
↓
commande contrôlée vers l’application
↓
changement dynamique du niveau de journalisation
```

Le redéploiement reste une solution de secours pour les paramètres impossibles à modifier à chaud.

---

## 4. La notion de « Debug Session »

Le Debug ne doit pas être un simple bouton booléen ON/OFF.

Il doit être modélisé comme une **session de debug administrée**.

Exemple conceptuel :

```text
DebugSession
├── Level
├── Scope
├── StartedAt
├── ExpiresAt
├── RequestedBy
├── Reason
└── PersistentAcrossRestart
```

Les noms exacts des champs pourront évoluer, mais les concepts doivent rester.

---

## 5. Modes d’activation

### 5.1 Debug temporaire — mode par défaut

Le comportement par défaut doit être sûr.

Exemple :

```text
Activer Debug
Durée : 30 minutes
Scope : Server
```

Le serveur enregistre :

```text
StartedAt = maintenant
ExpiresAt = maintenant + 30 min
```

À expiration :

```text
Debug → niveau normal
```

Durées proposées :

- 15 minutes ;
- 30 minutes ;
- 1 heure ;
- 4 heures ;
- éventuellement une durée personnalisée bornée.

Le choix exact sera défini lors du développement de l’Admin.

### 5.2 Debug planifié

Cas d’usage :

> Un bug se produit uniquement la nuit lorsque personne ne surveille le système.

L’Admin doit permettre une fenêtre de Debug planifiée.

Exemple :

```text
Debug planifié
Component = SyncEngine
Machine = WALL-E
Profile = ProjectZomboid

Début : 00:00
Fin   : 07:00
```

À l’heure de début :

```text
niveau → Debug
```

À l’heure de fin :

```text
niveau → Information
```

Ce mode doit être préféré au Debug permanent lorsque le problème se produit sur une plage horaire connue.

### 5.3 Debug persistant jusqu’à désactivation manuelle

Un mode persistant doit également exister pour les investigations longues ou imprévisibles.

Il doit être **volontairement plus explicite** que le mode temporaire.

Exemple d’interface :

```text
⚠ Debug persistant

[ ] Je comprends que le volume de logs peut augmenter fortement.

[ Activer jusqu’à désactivation ]
```

Ce mode reste actif jusqu’à une désactivation explicite.

Il doit pouvoir survivre au redémarrage du service **si l’administrateur a explicitement demandé cette persistance**.

---

## 6. Fermer l’interface Admin ne doit rien casser

L’état du Debug ne doit jamais dépendre du navigateur.

Exemple :

```text
Admin active Debug 30 min
↓
l’utilisateur ferme son navigateur
↓
le serveur continue la session
↓
ExpiresAt est atteint
↓
le serveur repasse automatiquement au niveau normal
```

Le TTL et l’état de la session sont donc **possédés par le serveur**, pas par l’interface Web.

---

## 7. Persistance et redémarrage

Le système doit distinguer au minimum :

### Session temporaire classique

Un redémarrage peut éventuellement :

- conserver le TTL restant ;
- ou revenir au niveau normal selon la politique retenue.

Cette politique devra être décidée explicitement lors de l’implémentation.

### Session persistante explicitement demandée

Elle doit pouvoir rester active après un restart.

Exemple :

```text
PersistentAcrossRestart = true
```

Un simple redémarrage du conteneur ne doit pas annuler silencieusement une investigation longue.

---

## 8. Scope du Debug

Le Debug ne doit pas nécessairement s’appliquer à toute l’application.

L’interface doit tendre vers des scopes ciblés.

Exemples :

```text
Application = GameSaveSync
Component = SyncEngine
Machine = WALL-E
Profile = ProjectZomboid
```

ou :

```text
Application = Claviger
Component = QuestionnaireRuntime
Guild = ...
```

Les dimensions exactes varient selon les applications, mais l’Admin doit permettre de limiter le bruit au strict nécessaire.

---

## 9. Logs structurés

Le besoin central n’est pas d’avoir « beaucoup de logs », mais des logs **structurés**.

Un événement devrait pouvoir transporter selon le contexte :

```text
Timestamp
Severity
Application
Component
MachineId
ProfileId
User/Guild/ContextId
Event / Category
CorrelationId
Message
StructuredContext
```

Tous les champs ne sont pas obligatoires pour tous les événements.

Ils doivent être ajoutés lorsqu’ils ont un sens métier ou opérationnel.

---

## 10. Correlation ID

Une opération métier importante doit pouvoir être suivie de bout en bout.

Exemple GameSaveSync :

```text
CorrelationId = 8f42...

14:32:01 SyncRequested
14:32:02 CentralVersionResolved V41
14:32:03 UploadStarted
14:32:08 ChecksumMismatch
14:32:08 PublicationRejected
14:32:09 LocalStateMarkedRequiresValidation
```

L’objectif est de pouvoir comprendre **l’histoire d’une opération** sans devoir reconstruire manuellement la chronologie depuis plusieurs sources.

---

## 11. Filtres indispensables dans l’Admin

L’interface Admin doit permettre de filtrer au minimum par :

- application ;
- composant ;
- machine ;
- profil de jeu ;
- utilisateur / serveur / contexte lorsque pertinent ;
- sévérité ;
- événement / catégorie ;
- correlation ID ;
- plage temporelle.

Exemple :

```text
Machine = WALL-E
Profile = ProjectZomboid
CorrelationId = 8f42...
Severity >= Warning
Time = 14:32–14:35
```

Le but est d’éviter l’anti-pattern :

> « Voici tout ce qui s’est passé pour cet utilisateur / ce système. Débrouille-toi. »

---

## 12. Philosophie UX

L’interface doit présenter d’abord une vue utile et lisible :

```text
Information
Warning
Error
Critical
```

Puis permettre de descendre volontairement vers :

```text
Debug
Trace
```

Le Debug sert à approfondir un problème.

Il ne doit pas être la seule manière de comprendre ce qui se passe.

La règle UX générale est :

> montrer d’abord l’histoire utile, puis permettre de descendre dans les détails techniques.

---

## 13. Indication visuelle obligatoire

Lorsqu’un Debug renforcé est actif, l’Admin doit l’afficher très clairement.

Exemple :

```text
🟠 DEBUG ACTIF

Scope      : SyncEngine / WALL-E
Depuis     : 02:14
Expiration : 07:00

[ Arrêter maintenant ]
```

Pour un mode persistant :

```text
🔴 DEBUG PERSISTANT ACTIF

Aucune expiration automatique.

[ Désactiver ]
```

Le but est qu’un administrateur ne puisse pas oublier qu’un niveau très verbeux est actif.

---

## 14. Expiration automatique

Le mode temporaire doit revenir automatiquement au niveau normal.

Cette expiration doit être :

- gérée côté serveur ;
- fiable même si l’interface est fermée ;
- traçable dans les logs ;
- visible depuis l’Admin.

Exemple :

```text
DebugSessionExpired
PreviousLevel = Debug
NewLevel = Information
```

---

## 15. Audit des actions Admin

Les actions d’exploitation importantes doivent être auditées.

Exemples :

```text
DebugSessionStarted
DebugSessionExtended
DebugSessionStopped
DebugSessionExpired
DebugSessionScheduled
PersistentDebugEnabled
PersistentDebugDisabled
```

Avec si possible :

```text
RequestedBy
Reason
Timestamp
Scope
PreviousLevel
NewLevel
ExpiresAt
```

L’objectif est de pouvoir répondre plus tard à :

> « Pourquoi l’application était-elle en Debug à 3h du matin ? »

---

## 16. Champ « Reason »

Lors de l’activation du Debug, l’interface peut proposer un motif.

Exemple :

```text
Reason:
"Investigation corruption sauvegarde PZ au réveil du PC"
```

Ce champ peut être facultatif pour les sessions courtes et recommandé/obligatoire pour les modes persistants ou planifiés.

---

## 17. Sécurité : Debug ne veut jamais dire « tout afficher »

La redaction des secrets doit rester active **quel que soit le niveau de log**.

Même en Debug ou Trace, ne jamais exposer volontairement :

- tokens ;
- mots de passe ;
- secrets applicatifs ;
- credentials SMB/NAS ;
- clés API ;
- cookies / secrets d’authentification ;
- données sensibles non nécessaires.

Règle :

```text
Debug ≠ désactiver la sécurité
```

---

## 18. Volume et rétention

Le Debug augmente fortement le volume de données.

Le système devra donc prévoir :

- rotation des fichiers locaux ;
- limites de taille ;
- politique de rétention ;
- indication de l’espace utilisé ;
- éventuellement avertissement avant activation prolongée ;
- protection contre la saturation disque.

Le dimensionnement exact sera défini lors de l’implémentation.

---

## 19. Logs locaux comme filet de sécurité

Même avec une interface Admin et un flux live, les applications doivent conserver des logs locaux rotatifs.

Pourquoi :

- l’Admin peut être indisponible ;
- le réseau peut tomber ;
- l’application peut planter avant d’envoyer le dernier événement ;
- la connexion entre service et Admin peut être interrompue.

Le flux Admin ne doit donc pas être l’unique source de vérité opérationnelle.

---

## 20. Flux live dans l’interface Admin

La cible à terme est un flux live filtrable.

Exemple :

```text
[Information] GameSaveSync.Server started
[Information] Sync requested — WALL-E / ProjectZomboid
[Warning]     Central version unavailable
[Debug]       Retrying metadata read...
[Error]       Metadata store unreachable
```

L’utilisateur doit pouvoir modifier les filtres sans redémarrer le service.

---

## 21. Niveau normal recommandé

Le niveau normal doit rester exploitable.

Une orientation possible :

```text
Production :
Information → Critical
```

Le Debug/Trace est activé uniquement lorsque nécessaire.

Les bibliothèques trop bavardes pourront avoir des niveaux spécifiques plus élevés afin d’éviter le bruit.

---

## 22. Runtime Debug plutôt que configuration figée

Lorsque la stack le permet, utiliser un mécanisme de niveau dynamique :

```text
LoggingLevelSwitch
IOptionsMonitor
configuration dynamique
ou mécanisme équivalent
```

Le choix technologique exact appartient à chaque application.

Le contrat fonctionnel reste :

> le niveau doit pouvoir être modifié à chaud lorsque cela est raisonnablement possible.

---

## 23. Interface Admin ≠ orchestrateur Docker

L’interface Admin ne doit pas posséder directement toutes les capacités d’orchestration de l’hôte.

Pour un changement nécessitant réellement un restart :

```text
Admin
↓
commande autorisée
↓
service / agent d’orchestration contrôlé
↓
Docker / systemd / autre
```

et non idéalement :

```text
Web Admin
↓
socket Docker complet
```

La séparation doit limiter l’impact d’une compromission de l’Admin.

---

## 24. Inspirations et anti-patterns

### Inspiration recherchée

Une expérience proche d’un runtime exploitable :

- visibilité centralisée ;
- filtres ;
- niveaux ;
- composants ;
- sessions d’investigation temporaires ;
- contrôle opérationnel depuis une console Admin.

### Anti-pattern à éviter

Un journal de debug qui signifie :

> « Tout ce qui s’est passé pour cet utilisateur pendant cette période, toutes APIs, tous workflows, toutes automatisations, toutes requêtes, sans hiérarchie utile. »

L’utilisateur ne doit pas devenir archéologue de ses propres logs.

---

## 25. Exigence transverse pour les applications

Chaque application intégrée à l’Admin doit tendre vers :

```text
Application
├── Structured logging
├── Dynamic log level
├── Correlation IDs
├── Scope/context metadata
├── Secret redaction
├── Local rotating logs
└── Admin debug-control endpoint / use case
```

Le nom des endpoints, interfaces ou messages reste propre à chaque application.

Le comportement global doit être cohérent.

---

## 26. Responsabilités

### Application métier

Responsable de :

- produire des événements structurés ;
- connaître les scopes métier pertinents ;
- appliquer dynamiquement le niveau demandé ;
- protéger les secrets ;
- gérer ou consommer l’état de la Debug Session.

### Interface Admin

Responsable de :

- demander l’activation/désactivation ;
- afficher l’état ;
- fournir les filtres ;
- afficher le flux ;
- rendre visibles les expirations ;
- signaler les modes persistants ;
- simplifier l’investigation.

### Infrastructure

Responsable de :

- rotation/rétention locale ;
- transport éventuel des logs ;
- redémarrage/redeploy lorsqu’il est réellement nécessaire ;
- stockage durable éventuel.

---

## 27. Déploiement progressif

Ne pas attendre que toute l’interface Admin existe pour structurer correctement les logs.

Les applications doivent dès maintenant éviter les choix qui rendent l’Admin futur difficile.

Progression proposée :

```text
Étape 1
Logs structurés locaux

Étape 2
Correlation IDs + scopes

Étape 3
Changement de niveau à chaud

Étape 4
Debug Sessions + TTL

Étape 5
Flux live vers Admin

Étape 6
Filtres / recherche / audit

Étape 7
Planification / Debug persistant / opérations avancées
```

L’ordre exact peut varier selon le projet.

---

## 28. GameSaveSync — exemple cible

Exemple d’investigation :

```text
Application : GameSaveSync
Machine     : WALL-E
Profile     : ProjectZomboid
Component   : Transfer
Level       : Debug
Durée       : 1h
Reason      : checksum mismatch during upload
```

Le flux pourrait montrer :

```text
SyncRequested
CentralVersionResolved
ManifestPrepared
UploadStarted
ChunkWritten
ChecksumValidationStarted
ChecksumMismatch
PublicationRejected
LocalStateMarkedRequiresValidation
```

Le contenu détaillé appartient au Debug.

La vue normale doit rester synthétique.

---

## 29. Claviger — exemple cible

Exemple :

```text
Application : Claviger
Component   : QuestionnaireRuntime
Guild       : Succumbrae
Level       : Debug
Durée       : 30 min
Reason      : rôle non appliqué après questionnaire
```

L’investigation doit pouvoir suivre un workflow sans noyer l’Admin sous tous les autres événements Discord.

---

## 30. Relation avec les décisions actuelles

Ce document ne demande pas que toutes ces fonctions soient codées immédiatement.

Il définit une **direction commune**.

Exemple :

Le checksum a été identifié comme une bonne idée pour l’intégrité GameSaveSync, mais il appartient aux tranches de transfert fiables et non aux fondations initiales.

Même philosophie ici :

> prévoir l’architecture et les contrats suffisamment tôt, implémenter les fonctions avancées seulement lorsque leur tranche ou leur consommateur réel arrive.

---

## 31. Décisions déjà considérées comme fortes

Les points suivants constituent la direction souhaitée et ne doivent pas être réinventés sans raison :

1. **Le Debug doit pouvoir être activé à chaud lorsque possible.**
2. **Le mode temporaire avec expiration automatique est le comportement par défaut.**
3. **Fermer l’Admin ne doit pas arrêter ou casser la session.**
4. **Le serveur possède le TTL et l’état de la session.**
5. **Un mode planifié doit exister pour les bugs nocturnes ou intermittents.**
6. **Un mode persistant doit être possible, mais explicitement demandé.**
7. **Les secrets restent redacted même en Debug.**
8. **Les logs doivent être structurés, filtrables et corrélables.**
9. **Les logs locaux rotatifs restent nécessaires même avec un flux Admin.**
10. **L’Admin ne doit pas recevoir inutilement un accès complet à Docker.**
11. **Le redéploiement n’est pas la méthode normale pour changer le niveau de log.**
12. **Le système doit privilégier la compréhension rapide d’un scénario plutôt que le déversement massif de traces.**

---

## 32. À décider lors du développement de l’interface Admin

Les points suivants restent volontairement ouverts :

- technologie exacte de changement dynamique de niveau ;
- stockage de l’état des Debug Sessions ;
- comportement exact d’une session temporaire après restart ;
- durées prédéfinies ;
- limites maximales ;
- format du flux live ;
- protocole entre applications et Admin ;
- stockage centralisé éventuel ;
- politique de rétention ;
- granularité maximale des scopes ;
- authentification et autorisations nécessaires ;
- rôles Admin autorisés à lancer un Debug persistant ;
- notifications éventuelles lorsqu’un Debug reste actif longtemps ;
- format et rétention de l’audit des actions Admin.

Ces décisions devront être prises avec un consommateur réel et non par spéculation.

---

## 33. Résumé opérationnel

La cible est :

```text
Admin
│
├── Voir les logs structurés
├── Filtrer
├── Rechercher par correlation ID
├── Activer Debug
│   ├── temporaire
│   ├── planifié
│   └── persistant
├── Voir clairement qu’un Debug est actif
├── Arrêter une session
└── Auditer qui a fait quoi

Application
│
├── change le niveau à chaud
├── applique les scopes
├── protège les secrets
├── garde des logs locaux
└── revient automatiquement au niveau normal

Infrastructure
│
└── ne redémarre/redéploie que si réellement nécessaire
```

---

## 34. But final

L’interface Admin ne doit pas seulement « afficher des logs ».

Elle doit permettre de répondre rapidement à :

> Qu’est-ce qui s’est passé ?  
> Où ?  
> Sur quelle machine / quel profil / quel workflow ?  
> Dans quelle opération ?  
> Pourquoi ?  
> Est-ce toujours en cours ?  
> Ai-je besoin d’activer davantage de détails ?  
> Quand le Debug va-t-il s’arrêter ?  
> Qui l’a activé ?

Si ces réponses exigent encore de parcourir manuellement plusieurs milliers de lignes, l’objectif n’est pas atteint.