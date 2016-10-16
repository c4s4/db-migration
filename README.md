Migration de Base de Données
============================

Michel Casabianca

casa@sweetohm.net

---
Proposition
-----------

Migration de base de données

Les scripts de base de données sont trop souvent la cinquième roue du carrosse, relégués en vrac dans un coin du projet. Or, projet bien ordonné commence par la base de données !

Cette conférence présente *db_migration* (http://github.com/c4s4/db_migration) un outil de migration de base de données mis en œuvre avec succès chez Orange et SQLI.

---
# Entendu

Entendu il y a encore un an :

```python
- La base Oracle de test a crashé, ils ont perdu toutes les
  données de test. Tous leurs tests sont inutilisables maintenant.
```

Entendu il y a 4 ans en arrivant sur un projet :

```python
- Comment on va faire la mise en production de la base de données ?
- On fera un dump de la pré-production.
```

---
# Le degré Zéro

Ce stade, où toutes les données sont dans la base, est le degré zéro de la gestion de données.

On s'expose alors aux risques suivants :

- **Pertes de données** en cas de crash de la base.
- **Perméabilité des données** entre plateformes (données de test en production et inversement).
- **L'installation d'une nouvelle base** est un processus compliqué, long et risqué, ce qui limite la capacité à tester.

Malheureusement on rencontre encore cette situation trop fréquemment.

---
# La première étape

Lorsqu'on veut gérer sérieusement les données d'une base, on comprend rapidement qu'il faut écrire dans des scripts SQL :

- **Le schéma** des bases de données.
- **Les données** pour toutes les plateformes.
- **Les scripts de migration du schéma** de la base.

D'autre part, pour pouvoir faire évoluer le schéma en production sans perdre de données, il ne faut écrire que des scripts de migration du schéma. On ne doit pas redéfinir une table existante, on doit la faire évoluer.

C'est souvent la situation dans laquelle se trouvent la plupart des projets.

---
# Organisation des scripts

Lorsqu'on applique ces principes de gestion des données, on en vient logiquement à organiser les scripts SQL de la manière suivante :

- Un **répertoire d'initialisation** permettant de créer la base.
- Un **répertoire par version** du logiciel comportant une modification du schéma ou des données.
- Des **scripts par plateforme** (test, préprod ou production) pour les données spécifiques.

Pour installer une nouvelle plateforme, on passera d'abord les scripts d'initialisation, puis les scripts des versions jusqu'à la version à installer.

Pour mettre à jour la base de données, on passera les scripts des répertoires de la version actuelle (non comprise) à la version cible.

Dans le répertoire d'une version, on passera les scripts communs puis les scripts spécifiques à la plateforme cible.

---
# Automatisation des scripts

Passer ces scripts à la main s'avère vite pénible et source d'erreurs.

Se pose alors la question de l'automatisation du passage de ces scripts.

La solution du script *main.sql*, qui appelle tous les autres scripts, est la plus simple mais présente les inconvénients suivants :

- Il faut y ajouter à la main tout nouveau script.
- On ne peut installer une version arbitraire de la base, mais toujours la dernière.
- On ne peut passer des scripts pour une plateforme spécifique.
- On doit écrire un script *main.sql* spécifique pour toute migration.

---
# Contraintes

On en vient alors à la conclusion qu'il faudrait un programme :

- Qui puisse installer **n'importe quelle version** de la base.
- Installer des **scripts spécifiques aux plateformes**.
- Gérer une **migration entre versions**.

Donc lorsqu'on l'appelle, il faudrait lui passer :

- La plateforme cible.
- La version à installer.

Connaissant les versions installée et cible, le programme en déduit les répertoires à passer. Connaissant la plateforme cible, il est capable de sélectionner les scripts à passer dans les répertoires.

---
# Autres contraintes

Les exploitants des plateformes ont l'habitude de passer les scripts avec les outils du fournisseur le la base de données (*mysql* pour MySQL et *sqlplus* pour Oracle).

Par conséquent, pour limiter les risques d'incompatibilité, il faudrait que cette application appelle ces outils en ligne de commande.

Il est alors possible de revenir à la solution manuelle sans modifier les scripts existants. Il est aussi possible d'automatiser la migration sur les plateformes de développement et de garder la solution manuelle en production.

---
# DB Migration

Nous avons cherché un outil répondant à toutes ces contraintes mais n'en avons trouvé aucun. Ceux que nous avons trouvé utilisaient, pour passer les scripts, des drivers spécifiques (comme Flyway par exemple, qui utilise le driver JDBC Java).

Par conséquent, en 2008 chez Orange, moi et d'autres collègues (en particulier *Grégoire Deveaux*) avons développé un outil en interne appelé *db_migration*. Il a été développé en Python et supportait MySQL. En 2015 le support a été étendu à Oracle.

Cet outil est utilisé chez Orange depuis 2008 et SQLI depuis 2015.

Il est en Open Source (sous [licence Apache](http://www.apache.org/licenses/LICENSE-2.0)), disponible sur Github : <http://github.com/c4s4/db_migration>.

---
# Organisation des scripts

Comme indiqué ci-dessus, les scripts SQL sont placés dans des répertoires par version du logiciel :

```python
.
├── 0.1
│   ├── all.sql
│   └── itg.sql
├── 1.0
│   └── all.sql
└── init
    ├── all.sql
    ├── itg.sql
    └── prod.sql
```

Dans ce cas, nous avons deux répertoires de version : *0.1* et *1.0*. Le répertoire init contient les scripts d'initialisation de la base de données.

Chaque répertoire contient les scripts à passer sur toutes les plateformes (les schémas en général), commençant par *all*, et les autres, spécifiques à chaque plateforme (contenant en général des données).

---
# Versions supportées

Les versions doivent être de la forme *1.2.3*, avec autant de groupes que nécessaires. Donc *1*, *1.2* et *1.2.3.4.5.6* sont valides. Par contre *1.2-rc3* ne l'est pas.

Les chiffres peuvent commencer par des zéros, donc *01.02.03* est valide et strictement équivalent à *1.2.3*.

Les versions sont triées par ordre *numérique*, donc *1.10* vient après *1.2*. L'absence de chiffre passant avant tout chiffre, donc *1* passe avant *1.0*.

Ainsi, *1*, *02.03.04* et *2.3* seront triés comme suit : *1*, *2.3* puis *02.03.04*.

---
# Scripts de plateformes

Les scripts commençant par *all* seront exécutés sur toutes les plateformes, dans l'ordre *lexicographique*, donc *all-10-foo.sql* sera exécutés avant *all-2-bar.sql*. On doit donc numéroter avec des zéros si nécessaire.

Les autres scripts seront exécutés sur les plateformes du même nom *après* les scripts *all-xxx.sql*. Le nom des plateformes est libre et listé dans le fichier de configuration.

Par conséquent si un répertoire de version contient les scripts  *all.sql*, *foo.sql* et *bar.sql*, alors les scripts *all.sql*, puis *foo.sql* seront exécutés pour migrer la plateforme *foo*.

---
# Exemples

Pour initialiser la base de données en intégration et l'amener à la version *0.1*, on tapera la ligne de commande :

```bash
$ db_migration -i itg 0.1
on '0.1' on platform 'localhost'
Using base 'test' as user 'test'
Creating meta tables... OK
Listing passed scripts... OK
Running 4 migration scripts... OK
```

Pour effectuer la migration de la plateforme *itg* vers la version *1.0* :

```bash
$ db_migration itg 1.0
on '1.0' on platform 'localhost'
Using base 'test' as user 'test'
Creating meta tables... OK
Listing passed scripts... OK
Running 1 migration scripts... OK
```

---
# Dry run

On peut voir les scripts qui seront passés, sans réellement les exécuter, avec l'option *dry_run* (le `-d` sur la ligne de commande) :

```bash
$ db_migration -d -i itg 1.0
Version '1.0' on platform 'localhost'
Using base 'test' as user 'test'
Creating meta tables... OK
Listing passed scripts... OK
5 scripts to run:
- init/all.sql
- init/itg.sql
- 0.1/all.sql
- 0.1/itg.sql
- 1.0/all.sql
```

---
# Méta données

*db_migration* connait la version installée grâce à une table *_install* :

```python
+----+---------+------------------+------------------+---------+
| id | version | start_date       | end_date         | success |
+----+---------+------------------+------------------+---------+
|  1 | 0.1     | 2012-04-18 14... | 2012-04-18 14... |       1 |
|  2 | 1.0     | 2012-04-18 14... | 2012-04-18 14... |       1 |
+----+---------+------------------+------------------+---------+
```

*db_migration* connait les scripts déjà passés grâce à la table *_scripts* :

```python
+--------------+---------------------+---------+------------+---------------+
| filename     | install_date        | success | install_id | error_message |
+--------------+---------------------+---------+------------+---------------+
| init/all.sql | 2012-04-18 14:17:20 |       1 |          1 | NULL          |
| init/itg.sql | 2012-04-18 14:17:20 |       1 |          1 | NULL          |
| 0.1/all.sql  | 2012-04-18 14:17:20 |       1 |          1 | NULL          |
+--------------+---------------------+---------+------------+---------------+
```

---
# Développement

Lors du développement, on mettra ses scripts dans un répertoire appelé *next*. Les scripts de ce répertoire sont passés avec l'option `-a` (pour *all*, qui remplace la version), qui passe tous les scripts de migration.

Lorsqu'on effectue une release, on mergera les scripts présents dans le répertoire *next* et on le renommera avec le numéro de la release.

Ainsi, on évite les problèmes de merges sur une ancienne version.

---
# Scripts de migration

Parfois, il est utile de générer des scripts de migration d'une version vers une autre. On peut alors utiliser l'option `-m version`. Par exemple, pour générer le script de migration de la version *0.1* vers *1.0*, on tapera la commande :

```bash
$ db_migration -m 0.1 itg 1.0
-- Migration base 'test' on platform 'itg'
-- From version '0.1' to '1.0'

USE `test`;

-- Script '1.0/all.sql'
INSERT INTO pet
  (name, age, species)
VALUES
  ('Nico', 7, 'beaver');

COMMIT;
```

On pourra utiliser ce script pour migrer la base de données *à la main*.

---
# Précautions

L'utilisation de *db_migration* nécessite un certain nombre de précautions :

- *db_migration* ne gère pas les retours arrières.
- Il faut donc effectuer un **dump** de la base avant d'effectuer la migration.
- Il ne faut **jamais** modifier un script d'une version antérieure.
- Il faut prendre garde à ne pas laisser de commentaire ouvert en fin de script (`/*`).

---
# Fonctionnement interne

En interne, *db_migration* fonctionne de la manière suivante :

- Il interroge la table *_install* pour connaître la version installée.
- Il sélectionne les répertoires de version à passer.
- Il sélectionne les scripts à passer dans chaque répertoire en fonction de la plateforme à installer.
- Il génère le script de migration à passer sur la base par concaténation.

En plus des scripts de migration, *db_migration* ajoute du code SQL pour :

- Commmiter après le passage d'un script.
- Peupler les tables de méta-données.

---
# Conclusion

En utilisant *db_migration* :

```python
On peut migrer automatiquement la base de données lors de l'installation
d'un logiciel.
```

En effet, lors de l'installation, on connaît la plateforme et la version, ce qui est suffisant à *db_migration* pour effectuer la migration. Plus besoin alors d'intervention manuelle post-installation pour migrer la base de données.

```python
On peut automatiser le passage des scripts Oracle, sans avoir à vérifier
ligne à ligne les logs à la recherche d'éventuelles erreurs.
```

En effet, Si les scripts de migration contiennent des erreurs, la valeur de retour de *sqlplus* n'est pas nécessairement différente de *0*. Pour être certain qu'il ne s'est produit aucune erreur, il faut donc parcourir tous les logs, *à la main*. *db_migration* le fait pour nous et nous affiche clairement l'erreur.

---
# Merci de votre attention

**casa@sweetohm.net**















