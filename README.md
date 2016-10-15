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











