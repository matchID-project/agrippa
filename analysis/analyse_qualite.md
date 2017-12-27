# *Analyse initiale des données AGRIPPA*

----

# Statistiques générales

### Décompte des entités

- **Problème données ?** Autorisations accordées sans dates de début ou fin : 332144
- Nombre d'autorisations actives : 56357
- Nombre d'armes ayant une autorisation active : 55885
- Durée moyenne cumulée (en mois) d'autorisation d'une arme : 31
- Nombre moyen d'autorisations par dossier : 1.18


### Décompte des armes par statut

- Détenteur non décédé, arme autorisée : 347898
- Détenteur décédé, arme plus autorisée : 3
- Détenteur décédé, arme toujours autorisée : 5460
- **Problème données ou de terrain ?** Détenteur décédé, arme réautorisée : 4


----

# Visualisations

Voir visualisations Tableau ci-jointes (au format PDF) indiquant le nombre d'armes enregistrées : 
- par catégorie d'arme, total
- par catégorie d'arme, par détenteur / état-civil
- par département, total
- par département, par détenteur / état-civil
- par âge, total
- par âge, par détenteur / état-civil

----

# Identification des détenteurs

### Choix de la méthode d'identification

Un point préalable à une analyse plus fiable de ce jeu de données concerne la façon d'identifier de façon unique les détenteurs d'armes. L'objectif est d'obtenir des comptes corrects en résolvant les deux types d'erreur suivants  :
* on appelle ici doublon un "match manqué", i.e. deux enregistrements A et B non rapprochés alors qu'ils correspondent à la même identité de personne physique
* on appelle ici collision un "faux match", i.e. deux enregistrements A et B rapprochés en raison de valeurs de nom, prénom, date et lieu de naissance, alors qu'ils correspondent à des personnes physiques différentes

En effet, les données ne semblent pas contenir d'identifiant unique fiable pour les détenteurs d'armes, notamment des doublons existent dans le champ `CDET_ID`. L'analyse présentée ici utilise donc 3 types d'identification des détenteurs d'armes :
1. en se basant sur le champ `CDET_ID`
2. en opérant un dédoublonnage initial sur l'état-civil (nom/prénom/date de naissance) avec l'élimination de la plupart des doublons sur le `CDET_ID` mais l'introduction de nouveaux doublons pour les homonymes
3. en opérant un dédoublonnage plus poussé a été fait en se basant sur les résultats de MatchID (matching flou sur les attributs d'état-civil notamment) et en réalisant un clustering des identités à partir du graphe constitué des enregistrements ainsi appariés

Ces différents types d'identification apparaissent dans les visualisations Tableau "... par détenteur / état-civil / MatchID".

### Impact de la méthode d´identification

 Le tableau suivant donne le décompte total de détenteurs dans Agrippa selon chacune des 3 méthodes :

|Type d'identification|Décompte|
|---|---|
|1. Total détenteurs par CDET_ID |2341434|
|2. Total détenteurs par état-civil |2149707|
|3. Total détenteurs par MatchID |2077913|

On voit donc que la simple identification par état-civil permet d'éliminer environ 9% de doublons, et celle par analyse MatchID d'en éliminer 11%. 

### Performance de la méthode d'identification

Reste à estimer la performance de l'identification ainsi opérée. 

#### Analyse qualitative

Qualitativement, la comparaison des clusters complets vs. incomplets, ainsi que le faible diamètre des clusters, indiquent que le dédoublonnage basé sur ces clusters est assez fiable :

_Clusters incomplets vs. complets_

|Détenteurs dans clusters complets (appelés cliques) | 975763|
|Détenteurs dans clusters incomplets | 103445|

_Taille des cliques_

|Taille de clique incomplète|Nombre de cliques|
|---|---|
|>20|<100|
|20|151|
|19|117|
|18|196|
|17|231|
|16|267|
|15|347|
|14|484|
|13|613|
|12|910|
|11|1280|
|10|1701|
|9|2787|
|8|4418|
|7|7200|
|6|12589|
|5|19466|
|4|20818|
|3|29127|

#### Analyse quantitative

Afin de compléter cette estimation qualitative de la méthode d'identification retenue, une estimation quantitative peut être réalisée via une analyse manuelle des "faux doublons" sur un échantillon.

_Estimation du nombre de doublons non résolus_

La principale cause d'identités doublonnées qui demeure (i.e. d'un taux de rappel suboptimal) correspond à des erreurs de saisie et/ou des problèmes de cohérence, par exemple : commune de naissance ayant été rebaptisée et saisie deux fois avec deux noms différents, combinée à une date de naissance saisie avec une valeur par défaut dans un des deux cas. Le nombre de tels cas est par principe impossible à estimer.

_Estimation du nombre de collisions non résolues_

La principale cause de collisions d'identité qui demeure (i.e. d'un taux de précision suboptimal) correspond à des clusters d'identité indûment fusionnés par notre méthode 3 ci-dessus. 

La situation la plus courante est celle d'enregistrements ayant exactement les mêmes nom de famille, prénom (éventuellement 2e et 3e prénom), et date de naissance, sans indication de lieu de naissance. Deux cas sont alors possibles : soit tous ces enregistrements ont le même lieu de naissance (on les nomme alors "jumeaux parfaits"), soit ils possèdent k lieux de naissance, k>=2 (on parle alors de "k-jumeaux imparfaits").

* Estimation 1 (empirique) :

Une première estimation du taux de jumeaux imparfaits est donnée par un décompte sur un jeu de données suffisamment fiable, par exemple sur le SNPC (nombre d'enregistrements total 49814656) :

|k|Nombre de k-jumeaux imparfaits|
|---|---|
|2|32419|
|3|172|
|4|6|

|k|Nombre de k-jumeaux parfaits|
|---|---|
|2|1481574|
|3|2758|
|4|2575|
|5|3|
|6|21|
|7|1|
|8|5|
|10|1|
|26|1|
|36|1|

De ces deux histogrammes, on conclut que les k-jumeaux pour k>2 sont négligeables par rapport aux 2-jumeaux, et que la proportion de jumeaux imparfaits vs. parfaits, une approximation du taux de collisions en cas d'ambiguïté sur le lieu de naissance est de 2,22% - et surtout que le nombre d'erreurs d'identification dues aux erreurs de saisie de lieu de naissance est au pire de 6 pour 10000.

* Estimation 2 (probabiliste) :

Afin de valider cette estimation empirique, on peut calculer un ordre de grandeur de façon probabiliste et non plus statistique. Ainsi on estime le nombre de collisions de ce type à partir de la probabilité d'une collision sur l'état-civil moins le lieu de naissance :

Comme on a 1079208 clusters (cliques ou clusters incomplets), la probabilité de zéro collision sur la date de naissance est : 
`exp(- 1079208 * 1079207 / (2 * 365 * 120))`
et le nombre moyen de collisions sur la date de naissance est de :
`n - d + d ((d-1)/d)^n = 1042708`

Noter également que l'algorithme MatchID ne ŕéconcilie ces enregistrements (ce qui conduit à fusionner les cliques correspondantes) que si les combinaisons prénom/nom sont suffisamment rares, donc la probabilité réelle de collisions est encore plus faible.

### Analyse des causes de doublons/collisions

À noter différentes causes de collisions ou de doublons sur les identités. Ces enregistrements de mauvaise qualité induisent en général peu de biais sur ĺes statistiques concernant l'ensemble des 2388663 détenteurs mais peuvent induire des biais non négligeables sur les statistiques par détenteur. (Ils sont de toute façon filtrés dans toutes les analyses décrites ici, même celles agrégées sur l'ensemble des données.) 

|Cause du biais|Volume|
|---|---|
|Détenteurs sans lieu de naissance|362825|
|Enregistrements avec un patronyme "INCONNU"|4269|
|Détenteurs sans date de naissance|2735|

----

# Points à clarifier 

Cette section contient surtout des questions à destination des experts métier afin de clarifier notre (côté MGMSIC) compréhension du modèle de données, et donc de déterminer si certains observations correspondent à des anomalies réelles ou pas.

- _Dates de début et fin d'autorisation_ : Un grand nombre d'autorisation n´ont pas de date de fin (champ `AUTO_D_FIN_date`).
- _Catégorie manquante_ : On observe 27% des armes sans catégorie (`CEUR_C_CLAS_EURO`), est-ce normal ?
- _Déclaration nécessaire à autorisation_ : Est-ce le cas pour toutes les catégories A-D, ou uniquement pour les armes nécessitant une autorisation (catégorie B et éventuellement A) ?
- _Durée de déclaration et d'autorisation_ : La durée standard est de 3 ans, passée à 5 ans, quelle est la date exacte du changement de réglementation ? certains détails réglementaires ou législatifs expliquent-ils les durées observées différentes de ces deux valeurs ?
- _Définition over quota_ : Est-ce toujours un maximum de 12 armes autorisées pour un même individu, ou faut-il prendre en compte les catégories spéciales impliquant un quota plus bas (10) ?
- _Statut des dossiers_ : Pour un dossier donné (`DOSS_ID`) on peut avoir plusieurs demandes de déclaration / autorisation / carte européenne, avec un statut donné par `ETAD_L_LIB`, faut-il prendre en compte les valeurs autres que "Valide" pour ce statut ? (Pour l'instant toutes les mesures agrégées sont filtrées sur la validité des déclarations ou autorisations correspondantes, ce qui résoud les problèmes d'armes "multi-actives".)

----

# Reproduction des statistiques publiques

### Total armes à autorisation active + Total armes à déclaration

	En 2009, la France compterait légalement 762 331 armes soumises à autorisation (actuelle catégorie B), et 2 039 726 armes soumises à déclaration

Avec notre analyse des données Agrippa, on obtient des volumes inférieurs à ces valeurs :

Cause du biais|Volume
---|---
Armes de catégorie B ayant une autorisation active en 2017|55875
Armes de catégorie B ayant une autorisation active au 01/01/2009|24633
Cumul historique en 2017|171795
Cumul historique au 01/01/2009|49846
Cumul historique (incluant les autorisations non datées !)|668634