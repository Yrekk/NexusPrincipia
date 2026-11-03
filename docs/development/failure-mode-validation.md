# Validation contrôlée des défaillances et de la reprise

**Statut :** convention transverse active  
**Portée :** projets de l'écosystème, en mode livraison comme en mode pédagogique.

## 1. Objectif

Les tests nominaux disent si le chemin attendu fonctionne. Les tests de défaillance vérifient **ce que le système garantit quand il échoue** : intégrité, autorisation, état observable, possibilité de reprise et absence d'effet secondaire interdit.

Ne jamais accélérer la livraison en supprimant un test de panne pertinent. À l'inverse, ne pas provoquer des incidents sans hypothèse ni observabilité : **chercher à faire casser un comportement en environnement contrôlé**, puis comprendre précisément le résultat.

Une CI verte, un service qui ne crashe pas ou un simple message d'erreur ne prouvent pas la résilience.

## 2. Préconditions de sécurité

Avant une injection de panne ou un test live :

- choisir un environnement et des données de test **isolés**, sans toucher aux fichiers ou sauvegardes irremplaçables ;
- disposer d'un état initial connu et d'une procédure de remise à zéro vérifiée ;
- identifier les dépendances partagées susceptibles d'être affectées (NAS, conteneurs, DB, réseau, API externes) ;
- obtenir l'accord explicite du développeur pour toute coupure ou action perturbatrice qui touche une ressource partagée, persistante ou assimilable à de la production ;
- ne jamais contourner les permissions, les confirmations administrateur, ni les contrats anti-destruction du produit au nom d'un test ;
- rendre l'observation possible : logs, état, événements, fichiers et métadonnées avant/après.

Si le test live comporte un risque non maîtrisé, utiliser d'abord une simulation, une dépendance substituée ou un environnement jetable ; documenter la limite de ce qui a réellement été validé.

## 3. Décrire chaque scénario

Une défaillance à tester possède au minimum :

| Champ | Question |
| --- | --- |
| **État initial** | Quel état, quelles données et quelles dépendances avant le test ? |
| **Déclencheur** | Timeout, perte réseau, indisponibilité du stockage, corruption artificielle, interruption, erreur externe, concurrence… à quel instant exact ? |
| **Invariant attendu** | Qu'est-ce qui doit absolument rester vrai ? |
| **Comportement autorisé** | Erreur explicite, mise en attente, mode minimal, retry sûr, refus, demande humaine… |
| **Observables** | Quels logs, codes d'état, findings, métadonnées ou fichiers doivent le démontrer ? |
| **Reprise** | Comment le service revient-il à l'état souhaité ? Que se passe-t-il au second essai ? |
| **Nettoyage** | Comment réinitialiser l'environnement et confirmer l'absence de séquelles ? |

La définition des résultats distingue **PASS**, **FAIL** et **NON VÉRIFIÉ** ; « impossible à reproduire » n'est pas un succès.

## 4. Trois couches de validation complémentaires

1. **Tests automatisés ciblés** : reproduire les erreurs prévisibles au plus près de la logique et figer les invariants en non-régression.
2. **Tests d'intégration contrôlés** : vérifier les frontières réelles entre services, stockage, base, API, fichiers et traitements asynchrones.
3. **Tests live encadrés** : lancer réellement le système sur un environnement autorisé et provoquer, quand cela est sûr, les scénarios reproductibles que le mock ne garantit pas.

Les tests live incluent les situations où l'ordre temporel compte : interruption entre deux écritures, retour tardif d'un callback, coupure de stockage, redémarrage et retry. Les actions réelles doivent rester réversibles et observables.

## 5. Familles de pannes à considérer selon le périmètre

- **Persistance / stockage** : chemin absent, NAS indisponible, fichier verrouillé, DB illisible, écriture partielle, snapshot inutilisable.
- **Asynchrone / concurrence** : callback hors ordre, duplicat, timeout, deux opérations incompatibles, crash avant acknowledgement.
- **API / réseau / dépendance externe** : latence, réponse invalide, quotas, indisponibilité, récupération après retour du service.
- **Autorité / sécurité** : acteur non autorisé, décision admin non donnée, état dégradé qui doit rester fail-closed.
- **Exploitation** : redémarrage, configuration invalide, observabilité incomplète, restauration échouée.

Ces exemples sont des **candidats**, pas des instructions de couper automatiquement une ressource réelle.

## 6. Exécution et trace

~~~text
hypothèse + invariant
→ état initial confirmé
→ injection contrôlée
→ observation des effets et états finaux
→ PASS / FAIL / NON VÉRIFIÉ
→ correction si besoin
→ nouveau test
→ non-régression automatisée si possible
→ procédure live reproductible sinon
→ documentation et handoff projet
~~~

Après une anomalie, chercher la cause et un garde-fou durable (contrat, invariant, test, alerte ou procédure). Un rollback qui « semble avoir marché » doit lui aussi être vérifié. Noter précisément ce qui a été simulé et ce qui a été réellement exécuté.

## 7. Répartition documentaire

**NexusPrincipia** conserve cette règle générale. **Chaque dépôt applicatif** garde ses scénarios spécifiques, commandes d'exécution, jeux de données, résultats de validations significatifs et statut dans la tranche/handoff existants. Ne pas créer un deuxième système de passation transverse.

La revue code/architecture doit demander, avant acceptation d'une tranche critique : *« Quel échec reproductible pourrait invalider notre hypothèse et avons-nous vérifié le comportement correspondant ? »* L'acceptation reste humaine et explicite, même si tous les tests sont verts.
