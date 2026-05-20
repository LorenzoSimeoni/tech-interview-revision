# Cours de synthèse — bases de données, index et performances SQL

## 1) L’idée générale d’une base de données

Une base de données relationnelle sert à **stocker** et **retrouver** des données efficacement.

La bonne manière de la voir n’est pas seulement comme un “grand tableau” SQL, mais comme un **moteur de stockage et d’optimisation** qui essaye de répondre aux requêtes avec le **moins d’accès disque possible**.

Ce que le moteur essaie de minimiser :

- les lectures disque aléatoires,
- le nombre de pages lues,
- le coût CPU,
- le coût mémoire.

---

## 2) Table, index, heap : les trois notions à bien séparer

### La table

La table contient les **vraies lignes**.

Selon le moteur, ces lignes peuvent être :

- physiquement rangées dans un ordre particulier,
- ou stockées dans un espace plus libre appelé **heap**.

### L’index

Un index est une structure auxiliaire qui permet de retrouver plus vite des lignes.

Il ne copie généralement pas toute la table.
Il stocke surtout :

- la/les colonne(s) indexée(s),
- un pointeur vers la ligne réelle,
- parfois d’autres informations selon le moteur.

### Le heap

Dans certains moteurs, la table est stockée dans un **heap** : les lignes ne sont pas triées physiquement selon l’index.

C’est un point essentiel pour comprendre pourquoi un index ne garantit pas forcément un accès séquentiel aux vraies lignes.

---

## 3) La structure des index : B-tree et B+tree

Les index des bases relationnelles sont souvent basés sur des structures proches de :

- **B-tree**
- **B+tree**

### Idée générale

L’index est un arbre équilibré qui permet de descendre rapidement vers la bonne zone de données.

Au lieu de parcourir toute la table, la base suit des chemins dans l’arbre.

### B-tree vs B+tree

#### B-tree

- Les nœuds internes peuvent contenir des clés et parfois des données.
- Les données peuvent être réparties dans plusieurs niveaux.

#### B+tree

- Les nœuds internes servent surtout à naviguer.
- Les vraies données ou les pointeurs vers les lignes sont surtout dans les feuilles.
- Les feuilles sont souvent chaînées entre elles, ce qui aide énormément les scans par intervalle.

### Pourquoi les B+trees sont très utilisés

- arbre peu profond,
- recherche rapide,
- scans par plage très efficaces,
- très bon compromis pour disque + cache.

---

## 4) Les pages : la vraie unité de lecture

Une base de données ne lit pas une ligne isolée à la fois. Elle lit des **pages**.

Une page est un bloc mémoire/disque de taille fixe ou quasi fixe.

### Tailles courantes

- PostgreSQL : souvent **8 KB** par page.
- MySQL InnoDB : souvent **16 KB** par page.

Une page peut contenir :

- plusieurs lignes de table,
- ou plusieurs entrées d’index.

### Pourquoi les pages comptent autant

Le moteur de base de données essaie de lire le moins de pages possible.

Le coût réel d’une requête est souvent moins “combien de lignes ?” que :
**combien de pages différentes faut-il lire ?**

---

## 5) Index primaire et index secondaires

### Clé primaire

Presque toutes les tables ont une clé primaire.

Mais selon le moteur, la clé primaire n’a pas le même rôle physique.

### MySQL InnoDB

Dans InnoDB, la table est stockée selon la clé primaire :

- c’est un **clustered index**.
- les vraies lignes sont organisées physiquement autour de la primary key.

Conséquence :

- lire par primary key peut être très séquentiel,
- les insertions croissantes sur une clé auto-incrémentée sont souvent très efficaces.

### PostgreSQL

Dans PostgreSQL, la table est généralement un **heap** :

- les lignes ne sont pas physiquement rangées selon la primary key.
- l’index primaire pointe vers les lignes du heap.

Conséquence :

- un scan par index peut retrouver rapidement les lignes à identifier,
- mais la récupération des vraies lignes peut ensuite provoquer des accès disque dispersés.

### Index secondaire

Un index secondaire est un index sur une autre colonne que la primary key.

Exemple :

```sql
CREATE INDEX idx_created_at ON users(created_at);
```

Il sert à accélérer les requêtes sur `created_at`.

Il ne recopie pas toute la table. Il contient surtout :

- la valeur indexée,
- un pointeur vers la ligne réelle, ou vers la primary key selon le moteur.

---

## 6) Full scan vs index scan

### Full table scan

La base parcourt toutes les pages de données de la table.

Cela peut être très rapide si la lecture est **séquentielle**.

Le disque aime les lectures séquentielles.

### Index scan

La base suit l’index pour trouver les lignes intéressantes.

Cela peut être excellent si la requête est très sélective.

Mais si la requête touche une grande partie de la table, l’index peut devenir moins rentable qu’un full scan.

### Pourquoi un full scan peut être plus rapide

Parce que :

- lire des pages contiguës est très efficace,
- alors qu’un index scan peut provoquer beaucoup d’allers-retours sur des pages très éloignées.

Le vrai problème n’est pas seulement le nombre de lignes lues, mais le **type d’accès disque**.

---

## 7) La différence entre RAM et disque

### Ce qu’il faut retenir

- La RAM est rapide.
- Le disque est beaucoup plus lent.
- La base essaie donc d’utiliser au maximum la mémoire cache.

### Ce qui est en RAM

Souvent, la base garde en mémoire :

- les pages les plus consultées,
- des pages d’index,
- des pages de données récentes ou chaudes,
- des caches internes.

### Ce qui est sur disque

Tout n’est pas en RAM.
La base persiste ses données sur disque, et va charger des pages en mémoire lorsqu’elle en a besoin.

### Pourquoi on parle des index “en RAM”

Parce qu’ils sont souvent :

- plus petits que la table,
- très réutilisés,
- donc fréquemment présents dans le cache.

Mais ça ne veut pas dire que l’index est “stocké uniquement en RAM”.
Cela veut dire qu’il est souvent **plus facile à garder chaud en mémoire**.

---

## 8) Cardinalité et sélectivité

Ces deux notions sont très liées.

### Cardinalité

C’est le **nombre de valeurs distinctes**.

Exemple :

- `gender` → faible cardinalité,
- `user_id` → forte cardinalité.

### Sélectivité

C’est la capacité d’une colonne à **réduire fortement** le nombre de lignes retournées.

Exemple :

- `WHERE user_id = 42` → très sélectif,
- `WHERE gender = 'M'` → peu sélectif.

### Lien entre les deux

En pratique :

- forte cardinalité → souvent forte sélectivité,
- faible cardinalité → souvent faible sélectivité.

Ce ne sont pas deux idées opposées : c’est presque la même idée vue sous deux angles différents.

---

## 9) Les index composites

Un index composite porte sur plusieurs colonnes.

Exemple :

```sql
CREATE INDEX idx_user_created ON orders(user_id, created_at);
```

### Règle fondamentale

L’ordre des colonnes dans l’index est crucial.

Un index `(a, b, c)` est efficace pour :

- `a`
- `a, b`
- `a, b, c`

Mais pas forcément pour :

- `b` seul,
- `c` seul,
- `b, c` seuls.

C’est l’idée du **leftmost prefix**.

### Comment choisir l’ordre

On regarde surtout :

1. les requêtes réellement utilisées,
2. la sélectivité,
3. les filtres les plus fréquents,
4. les tris et jointures fréquents.

### Erreur fréquente

Penser que l’ordre des conditions dans le `WHERE` change quelque chose.

En réalité :

```sql
WHERE gender = 'M' AND created_at > ...
```

et

```sql
WHERE created_at > ... AND gender = 'M'
```

reviennent au même pour l’optimiseur.

Ce qui compte, c’est l’ordre dans l’index, pas l’ordre dans le `WHERE`.

---

## 10) Pourquoi trop d’index peut être mauvais

Un index accélère certaines lectures, mais il a un coût.

### Coûts des index

- stockage supplémentaire,
- mémoire/cache supplémentaire,
- temps de maintenance.

### Impact sur les écritures

Chaque `INSERT`, `UPDATE`, `DELETE` doit aussi mettre à jour :

- la table,
- et tous les index concernés.

Donc trop d’index peut ralentir fortement les écritures.

C’est un compromis permanent :

- meilleurs temps de lecture,
- contre écritures plus coûteuses.

---

## 11) Pourquoi un index n’est pas toujours meilleur qu’un scan complet

C’est une idée très importante.

Si une condition renvoie une grande partie de la table, utiliser l’index peut coûter plus cher que lire toute la table.

Pourquoi ?

Parce qu’un index scan peut impliquer :

- lecture de l’index,
- puis beaucoup de retours vers les pages de données,
- donc beaucoup d’accès aléatoires.

Exemple :

```sql
WHERE gender = 'M'
```

Si 50 % des lignes sont `M`, la base peut juger qu’un full scan séquentiel est moins coûteux qu’un index scan.

Donc :

- peu de sélectivité → souvent full scan,
- forte sélectivité → souvent index utile.

---

## 12) Le query planner : le cerveau de la base

Le query planner est la partie du moteur qui choisit **comment exécuter une requête**.

Il ne fait pas juste :

- “index ou pas index”

Il choisit parmi plusieurs stratégies.

Il estime notamment :

- le coût de lecture,
- le nombre de lignes,
- le coût mémoire,
- le coût CPU,
- la taille des tables,
- les statistiques disponibles.

---

## 13) Les grandes stratégies de jointure

### 1. Nested Loop Join

Principe :

- pour chaque ligne de la première table,
- chercher les lignes correspondantes dans la seconde.

Très bien si :

- la première table est petite,
- ou si la seconde table a un bon index sur la colonne de jointure.

Très mauvais si :

- les deux tables sont grosses,
- et qu’il n’y a pas d’index adapté.

---

### 2. Hash Join

Principe :

- construire une table de hachage en mémoire à partir d’une des tables,
- puis parcourir l’autre table et faire des recherches rapides dedans.

Très efficace pour de grosses jointures quand la mémoire le permet.

C’est une stratégie très courante.

---

### 3. Merge Join

Principe :

- si les deux ensembles sont déjà triés sur la clé de jointure,
- on les parcourt comme deux listes triées.

Très efficace lorsque les données sont déjà ordonnées ou qu’un index permet d’obtenir facilement cet ordre.

---

## 14) Le lien avec les relations Many-to-Many

Une relation Many-to-Many est un **modèle de données**.

Exemple :

- `users`
- `roles`
- `user_roles`

Le fait que ce soit du Many-to-Many ne force pas une stratégie d’exécution particulière.

Le planner peut utiliser :

- nested loop,
- hash join,
- merge join,

selon ce qu’il estime le moins coûteux.

---

## 15) EXPLAIN et EXPLAIN ANALYZE

Pour comprendre ce que la base fait réellement, on utilise souvent :

```sql
EXPLAIN
```

ou

```sql
EXPLAIN ANALYZE
```

Cela permet de voir :

- si la base utilise un index ou non,
- quel type de scan elle choisit,
- quel type de join elle utilise,
- combien de lignes elle estime,
- combien de lignes elle traite réellement,
- combien de temps prend chaque étape.

C’est un outil central pour diagnostiquer les lenteurs.

---

## 16) Pourquoi une requête peut être lente même avec un index

Même avec un index, la requête peut être lente si :

- la condition est peu sélective,
- les lignes sont très dispersées,
- la requête demande beaucoup de colonnes non couvertes par l’index,
- le planner estime que l’index n’est pas rentable,
- il y a beaucoup d’accès aléatoires disque.

---

## 17) Index scan, heap fetch, index-only scan

### Index scan

La base lit l’index pour trouver les lignes intéressantes, puis va chercher les vraies lignes dans la table.

### Heap fetch

Dans PostgreSQL, aller chercher la vraie ligne dans le heap peut coûter cher si les lignes sont dispersées.

### Index-only scan

Si toutes les colonnes nécessaires sont déjà dans l’index, la base peut répondre sans aller lire la table.

C’est souvent beaucoup plus rapide.

---

## 18) Ton cas de migration volumineuse

Quand tu as batché par primary key avec une requête du type :

```sql
WHERE id > last_id
ORDER BY id
LIMIT 1000
```

tu as profité de la **localité des accès**.

### Pourquoi ça marche bien dans MySQL InnoDB

Parce que la table est physiquement organisée par primary key.

Donc lire par primary key permet souvent une lecture beaucoup plus séquentielle.

### Pourquoi c’est moins direct dans PostgreSQL

Parce que la primary key n’ordonne pas physiquement la table.

Mais la stratégie peut quand même rester utile :

- elle évite les `OFFSET` coûteux,
- elle améliore le batching,
- elle limite les lectures inutiles,
- elle reste souvent meilleure qu’un parcours aléatoire.

---

## 19) Pourquoi les pages et les nœuds sont de taille fixe

La taille des pages est généralement fixe parce que cela simplifie :

- la lecture disque,
- le cache,
- le calcul des offsets,
- le préchargement,
- la gestion de l’arbre.

Comme les pages ont une taille standard, le moteur peut naviguer facilement dans les structures de stockage.

---

## 20) Le fill factor et les splits de page

Les pages ne sont pas toujours remplies à 100 %.

Il faut souvent laisser de la place pour :

- les nouveaux inserts,
- les mises à jour,
- éviter les divisions trop fréquentes.

### Split de page

Quand une page est pleine et qu’on doit insérer un élément, la base peut :

- diviser la page,
- créer une nouvelle page,
- réorganiser l’arbre.

Cela a un coût.

### Impact des clés aléatoires

Une clé très aléatoire, comme certains UUID, peut provoquer davantage de fragmentation et de splits.

Une clé croissante, comme un identifiant auto-incrémenté, favorise souvent des écritures plus séquentielles.

---

## 21) Résumé mental simple

Tu peux garder ce modèle en tête :

### Dans la table

- les vraies lignes sont stockées dans des pages.

### Dans l’index

- l’index est un arbre de pages aussi,
- il contient des clés et des pointeurs,
- il permet de retrouver rapidement des lignes.

### Le moteur choisit

- index scan,
- full scan,
- nested loop join,
- hash join,
- merge join,
- index-only scan,
  selon les coûts estimés.

### Le vrai enjeu

- réduire les lectures disque,
- éviter les accès aléatoires,
- tirer parti du cache,
- choisir des index cohérents avec les requêtes réelles.

---

## 22) Les phrases clés à retenir pour un entretien

- Une base relationnelle n’est pas seulement un stockage : c’est un moteur d’optimisation.
- Un index accélère certaines lectures, mais il a un coût en écriture et en stockage.
- Un index composite doit être pensé selon les requêtes réelles.
- La sélectivité et la cardinalité sont deux façons proches de parler de l’intérêt d’une colonne pour un index.
- Un full scan peut être plus rapide qu’un index scan si la requête touche une grande partie de la table.
- Le query planner choisit la stratégie la moins coûteuse estimée.
- Dans PostgreSQL, l’index et le heap sont séparés.
- Dans MySQL InnoDB, la primary key joue un rôle physique central.
- Les pages sont la vraie unité de lecture disque.

---

## 23) Mini check-list pratique

Avant de créer un index, se poser les questions suivantes :

- Quelles requêtes sont réellement fréquentes ?
- Quelle colonne est la plus filtrante ?
- Est-ce que l’index va aussi aider les `ORDER BY` ou les `JOIN` ?
- Est-ce que l’index couvrira la requête ?
- Est-ce que la table est assez grande pour que l’index soit utile ?
- Est-ce que l’impact sur les écritures est acceptable ?

---

## 24) Conclusion

La bonne façon de penser une base de données n’est pas :

- “j’écris du SQL”

mais plutôt :

- “j’essaie de faire en sorte que le moteur lise le moins de pages possible, dans le meilleur ordre possible, avec le meilleur plan d’exécution possible.”

C’est cette manière de raisonner qui permet de comprendre les index, les scans, les jointures, et les optimisations de performance.

---

## 25) Différences essentielles avec le NoSQL

En NoSQL, on change complètement de logique par rapport au relationnel. Au lieu de normaliser les données et de les recomposer avec des JOINs, on les **stocke souvent déjà “prêtes à lire” dans des documents ou des structures clé/valeur**, afin d’optimiser les accès directs. Cela permet des lectures très rapides et un excellent **scaling horizontal**, car les données sont facilement réparties sur plusieurs machines via des **shards**. En contrepartie, on perd souvent une partie des garanties fortes du relationnel (contraintes, cohérence immédiate, relations complexes), et on doit concevoir le modèle de données en fonction des **requêtes et de la distribution**, plutôt que des entités métier. Les **index existent toujours**, mais ils sont pensés différemment : souvent liés aux patterns de lecture et au partitionnement, et plus contraints par les coûts de distribution et de réseau.
