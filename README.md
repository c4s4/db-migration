
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

    - La base Oracle de test a crashé, ils ont perdu toutes les
      données de test. Tous leurs tests sont inutilisables maintenant.

Enetendu il y a 4 ans en arrivant sur un projet :

    - Comment on va faire la mise en production de la base de données ?
    - On fera un dump de la pré-production.

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
- **Toute modification du schéma** de la base.

C'est souvent le la situation dans laquelle se trouvent la pluspart des projets un tant soit peu sérieux.

---
# Organisation des scripts

Lorsqu'on applique ces principes de gestion des données, on en vient logiquement à organiser les scripts SQL de la manière suivante :

- Un répertoire d'initialisation permettant de créer la base.
- Un répertoire par version du logiciel comportant une modification du schéma ou des données.
- Des scripts par plateforme (test, préprod ou production) pour les données spécifiques.

Pour installer une nouvelle plateforme, on passera d'abord les scripts d'initialisation, puis les scripts des versions jusqu'à la version à installer.

Dans le répertoire d'une version, on passera les scripts communs à toutes les plateformes puis les scripts spécifiques.

---
# Automatisation des scripts

Passer ces scripts à la main s'avère vite pénible et source d'erreurs.

Se pause alors la question de l'automatisation du passage de ces scripts.

La solution d'un script *all.sql*, qui appelke tous les autres scripts, est la plus mais présente les inconvénients suivants :

- Il faut ajouter à la main tout nouveau script.
- On ne peut installer une version arbitraire de la base, mais toujours la dernière.
- On ne peut passer des scripts pour une plateforme spécifiquement.
- On doit écrire des scripts spécifiques pour toute migration.

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

Les exploitants des plateformes ont l'habitude de passer les scripts avec les outils fournis par le développeur le la base de données (*mysql* pour les bases MySQL et *sqlplus* pour Oracle).

Par conséquent, pour limiter les risques d'incompatibilité, il faudrait que cette application appelle ces outils en ligne de commande.

---
# DB Migration

Nous avons cherché un outil répondant à toutes ces contraintes mais n'en avons trouvé aucun. Ceux que nous avons trouvé utilisaient pour passer les scripts, des drivers spécifiques (comme Flyway par exemple, qui utilise le driver JDBC Java).

Par conséquent, en 2008 chez Orange, moi et d'autres collègues (en particulier ) avons développé un outil en interne appelé *db_migration*. Il a été développé en Python et supportait MySQL. En 2015 le support a été étendu à Oracle.

Cet outil est utilisé chez Orange depuis 2008 et SQLI depuis 2015.

---



















