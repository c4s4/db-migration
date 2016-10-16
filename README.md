
Migration de Base de Données
============================

Michel Casabianca

casa@sweetohm.net

---
Proposition
-----------

Migration de base de données

Les scripts de base de données sont trop souvents la cinquième roue du carrosse, relégués en vrac dans un coin du projet. Or, projet bien ordonné commence par la base de données !

Cette conférence présente db_migration (http://github.com/c4s4/db_migration) un outil de migration de base de données mis en œuvre avec succès chez Orange et SQLI.

---
# Entendu

Entendu il y a encore un an :

```
- La base Oracle de test a crashé, ils ont perdu toutes les
  données de test. Tous leurs tests sont inutilisables maintenant.
```

Enetendu il y a 4 ans en arrivant sur un projet :

```
- Comment on va faire la mise en production de la base de données ?
- On fera un dump de la pré-production.
```

---
# Le degré Zéro

Ce stade où toutes les données sont dans la base est le degré zéro de la gestion de la base de données.

On s'expose alors aux risques suivants :

- **Pertes de données** en cas de crash de la base.
- **Perméabilité des données** entre plateformes (données de test en production et inversement).
- **L'installation d'une nouvelle base** est un processus compliqué, long et risqué, ce qui limite la capacité à tester.

Malheureusement on rencontre encore cette situation très fréquemment.

---
# La première étape

Lorsqu'on veut gérer sérieusement les données d'une base, on comprend rapidement qu'il faut écrire dans des scripts SQL :

- **Le schéma** des bases de données.
- **Les données** pour toutes les plateformes.
- **Les sctipts de migration du schéma** de la base.

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
# Automatisation par programme

On en vient alors à la conclusion qu'il faudrait un programme :

- Qui puisse installer n'importe quelle version de la base.
- Installer des scripts spécifiques aux plateformes.
- Gérer une migratiin entre versions.

Donc lorsqu'on l'appelle, il faudrait lui passer :

- La version à installer.
- La plateforme cible.

Connaissant les versions installée et cible, le programme en déduit les répertoires à passer. Connaissant la plateforme cible, il est capable de sélectionner les scripts des répertoires à passer.

---
# Autres contraintes

Les exploitants des plateformes ont l'habitude de passer les scripts avec les outils fournis par le fournisseur le la base de données (*mysql* pour MySQL et *sqlplus* pour Oracle).

Par conséquent, pour limiter les risques d'incompatibilité, il faudrait que cette application appelle ces outils en ligne de commande.

Ilest alors possible de revenir à la solution manuelle sans modifier les scripts existants. Il est aussi possible d'automatiser la migration sur les plateformes de développement et de garder la solution manuelle en production.

---
# DB Migration

Nous avons cherché un outil répondant à toutes ces contraintes mais n'en avons trouvé aucun. Ceux que nous avons trouvé utilisaient pour passer les scripts, des drivers spécifiques (comme Flyway par exemple, qui utilise le driver JDBC Java).

Par conséquent, en 2008 chez Orange, moi et d'autres collègues (en particulier *Grégoire Deveaux* chez Orange) avons développé un outil en interne appelé *db_migration*. Il a été développé en Python et supportait MySQL. En 2015 le support a été étendu à Oracle.

Cet outil est utilisé chez Orange depuis 2008 et SQLI depuis 2015.

Il est en Open Source, disponible sur Github : <http://github.com/c4s4/db_migration>.

---
# Organisation des scripts

Comme indiqué ci-dessus, les scripts SQL sont placés dans des répertoires par version du logiciel :

    .
    ├── 1.0.0
    │    ├── all.sql
    │    └── itg.sql
    ├── 1.1.0
    │    └── all.sql
    ├── db_configuration.py
    └── init
          ├── all.sql
          ├── itg.sql
          └── prod.sql

Dans ce cas, nous avons deux répertoires de version : *1.0.0* et *1.1.0*. Le répertoire init contient les scripts d'initialisation de la base de données.

---
# Versions supportées

Les versions doivent être de la forme *x.y.z*, avec autant de parties que nécessaires entre les points. Donc *1*, *1.2* et *1.2.3.4.5.6* sont valides. Par contre *1.2-rc3* ne l'est pas.

Les chiffres peuvent commencer par des zéros, donc *01.02.03* est valide et strictement équivalent à *1.2.3*.

Les versions sont triées en enlevant les zéros et l'absence de chiffre passe avant tout chiffre. Donc *1*, *02.03.04*, *2.3* seront triés comme suit : *1*, *2.3*, *02.03.04*.

---
# Scripts de plateformes

Les scripts commençant par *all* seront exécutés sur toutes les plateformes, dans l'ordres *lexicographique*, donc *all-10.sql* sera exécutés avant *all-2.sql*. On doit donc numéroter avec des zéros si nécessaire.

Les autres scripts seront exécutés sur les plateformes du même nom *après* les scripts *all-xxx.sql*. Le nom des plateformes est libre et listé fans le fichier de configuration.

Par conséquent si un répertoire de version contient les scripts  *all.sql*, *foo.sql* et *bar.sql*, alors les scripts *all.sql*, puis *foo.sql* seront exécutés pour migrer la plateforme *foo*.

---











