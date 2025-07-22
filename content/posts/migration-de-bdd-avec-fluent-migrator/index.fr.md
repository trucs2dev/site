+++
date = '2021-04-20T22:52:33+02:00'
draft = false
title = 'Migration De Bdd Avec Fluent Migrator'
categories = ["Tech"]
tags = ["NET", "API", "Database", "Versionning"]
+++

## Les migrations de BDD c’est quoi ?

Les migrations de BDD permettent d’avoir un outil pour aligner l’état dans le temps d’une application avec l’état dans le temps de la base de données qu’il consomme.

Bon, dis comme ça, c’est pas très parlant, mais prenons un exemple simple: une application de liste de tâches.

Le backlog produit est prêt et voici 2 sprints (pour faciliter, disons 2 versions de l’application)

Sprint 1:

- Retourner une liste de tâches
- Créer une nouvelle tâche
- Supprimer une tâche
- Editer une tâche

Sprint 2:

- Ajouter un état à la tâche (A faire / En cours/ Fait)
- Ajouter une notion de priorité sur la tâche (de 1 à 5)
- Rechercher une tâche par description

Le sprint 2 de l’application correspond à une nouvelle version qui prend en compte toutes ces nouvelles informations.

Mais la base de données doit aussi pouvoir les  stocker et donc, adpater sa "version".

Le versionning de base de données permet celà: à chaque modification du code, il est possible d'inclure des altérations de la base de données pour qu'elle corresponde au code qui la consomme.

Et les outils qui permettent cela sont les frameworks de migrations de données.

## Mais on peut le faire avec les ORMs ça non ?

En partie: les ORMs (NHibernate & EntityFramework) permettent de créer la structure de données qu’ils manipulent. Il existe les outils  `hbm2ddl` de NHibernate ou la méthode `EnsureCreated` du DbContext. Par contre ces outils là ne gèrent que la structure de données, pas l’ensemble des éléments structurels.

Dans les éléments structurels, il y a bien évidemment la **structure de données**, mais aussi tout ce qui est **index**, **foreign keys**, **unique constraints**, dans certains cas, des **triggers**, mais également **certaines données référentielles**: la liste des droits applicable à une application, et certaines autres listes de valeurs statiques.

Qui plus est, ce genre d’outil ne sait pas gérer les suppressions.

## Présentation de FluentMigrator

FluentMigrator permet de déclarer des Migrations sous la forme de classes .NET.

Une migration est une opération qui manipuler l’état de la base de données.

Une migration est identifiée par une numéro unique (sa version) et possède 2 méthodes: le « Up » et le « Down ».

*Le « Down » est très utile pour pouvoir par exemple faire un retour en arrière en production.*

Fluent Migrator comprends 2 modèles de migration supplémentaires:

- Les [AutoReversingMigrations](https://fluentmigrator.github.io/articles/migration/migration-auto-reversing.html). Qui permet de déduire le « Down » en fonction du « Up ». Pour cela, il y a, 2 contraintes: Utiliser l’API Fluent et n’avoir que des opérations de création ou de renommage dans le « Up ».
- Les [ForwardOnlyMigrations](https://fluentmigrator.github.io/api/v3.x/FluentMigrator.ForwardOnlyMigration.html+) qui sont utilisées pour les opérations destructrices, par exemple la suppression d’une colonne dans une table: une fois la colonne supprimée, il est possible de la recréer mais il n’est pas possible de recréer les données qu’elle contenait sur l’ensemble des enregistrements de la table.

Une fonctionnalité très utile de Fluent Migrator est le fait de marquer avec des **Tags** certaines migrations. Cela permet de définir un sous-ensemble de données qui ne remonte pas de manière systémique mais dans certains scénarios définis, par exemple pour alimenter les données de tests d'intégrations ou de démonstration client.

## Le cas du ForwardOnly

Les migrations de type **ForwardOnly** sont destructrices. Le fait de créer ces migrations le met en avant en empêchant l'outil de faire un rollback.
Cela fait émerger une bonne pratique vis à vis des migrations destructrices, comme par eemple, les réaliser en 2 temps:

1. Pendant le développement: ne plus utiliser les colonnes à supprimer mais leur affecter une valeur par défaut
2. Après une mise en production réussie: réaliser les migrations de suppression et les jouer

## Un peu de code

Fluen migrator propose un [Runner](https://fluentmigrator.github.io/articles/runners/runner-console.html), qui peut utiliser plus de 20 pilotes de connexion à différentes BDD et va lister les migrations concernées et les jouer **par numéro de version croissant**.

Si on lui précise des Tags, il sélectionnera les migrations qui n’ont pas de Tag et celles qui disposent des Tags spécifiés.

Si on devait reprendre l’exemple de l’application de tâches dont on parlait, on aurait une première migration qui ressemblerait à ça:

![Migration de données create table](migration-de-bdd-avec-fluent-migrator/image-23.png)

La nomenclature que j’ai retenu pour les numéros de versions est la suivante : **yyyyMMddHHmm**. Avec ce format, chaque incrément de quelque part de la date incrémente bien la version et si on préfixe le nom des fichiers avec la version dans l’IDE, il devient aisé de naviguer dedans. Concernant la granularité, il est très rare que l’on ait deux migrations réalisées la même minute.

L’ajout de l’état ressemblerait à ça:

![Migration de données alter table](migration-de-bdd-avec-fluent-migrator/image-26.png)

Et il en va de même pour l’ajout de la priorité:

![Migration de données alter table](migration-de-bdd-avec-fluent-migrator/image-25.png)

On pourrait même ajouter une migration qui va créer de la donnée pour que la base de développement ne soit pas trop vide:

![Migration de données ajout de données](migration-de-bdd-avec-fluent-migrator/image-27.png)

Ensuite, il suffit de déclencher le « Runner » soit [programmatiquement](https://fluentmigrator.github.io/articles/migration-runners.html#in-process) soit via l’extension de la CLI dotnet soit par l’exécutable console. Ici, nous allons le faire programmatiquement en suivant l’exemple fourni sur le site, mais en y ajoutant la sélection du tag « dev »:

![Code du runner de migrations](migration-de-bdd-avec-fluent-migrator/image-28.png)

Une fois l’application lancée, on constate que les migrations sont bien jouées:

![Status du runner](migration-de-bdd-avec-fluent-migrator/image-29.png)

Et on peut constater en ouvrant le fichier que la base de données contient Bien la table Tasksavec les colonnes spécifiées et les données de développement.

![Table de suivi des versions](migration-de-bdd-avec-fluent-migrator/image-30.png)

## Les migrations avec Entity Framework Core

C’est une question de sensibilité et de fonctionnalités.

Pour les fonctionnalités, à ce jour, les migrations d’Entity Framework n’ont pas la notion de Tag et c’est une fonctionnalité qui me manque, même si elle peut être remplacée au runtime avec la notion d’environnement.

Côté sensibilité, j'ai 3 gros points qui sont source de friction:

- Tout d’abord, j’apprécie beaucoup le feedback que fournit fluent migrator en affichant les migrations qu’il traite et les opérations qu’il exécute. Je trouve dommage qu’avec Entity Framework, on puisse au mieux lister l’ensemble des migrations déjà jouées sur la base de données depuis sa création.

- Ensuite, EntityFramework dispose d’une fonctionnalité piège: celle de déduire les migrations depuis l’état du DBContext.

A première vue cela semble une bonne idée pour faire gagner du temps : plus besoin de taper le code. Mais cela demande une réelle rigueur si on veut éviter la monstro-migrations qui créé 12 tables. Le fait d'enlever ça des mains du dévelopeur fait aussi perdre beaucoup de finesse sur les migrations.

- Le dernier mais pas le moindre, les migrations avec EntityFramework s'appuient sur un fichier partagé : le `snpashot`. Tout ceux qui ont travaillé avec les fichiers edmx en équipe se rappellent de la douleur et du reisque lors du merge de ces fichies... parfois en se réveillant en sueur... Ici, le snapshot présente le même inconvénient : du code généré, commun à toutes les migrations, ou plutôt nécessaire à toutes les migrations, et qui est une source de problèmes de merge.

## DACPAC pour initialiser une base de données

Un DACPAC (**D**ata-tier **A**pplication **C**omponent **Pac**kage) contient l’état souhaité de la base de données, sous la forme de ses différents composants (tables, vues, fonctions, procédures stockées, etc.).

Le paradigme est différent: les migrations sont immuables et permettent le rollback mais le DACPAC évolue avec le temps pour refléter l’état souhaité. Il ne dispose pas de mécanique de rollback.

D’un point de vue fonctionnalités, le DACPAC ne cible que SQL Server et n’a pas la flexibilité des Tags comme on la retrouve chez FluentMigrator. Ces fonctionnalités ne sont pas forcément requises selon les projets, mais elles amènent une flexibilité qui permet des ouvertures.

Enfin, d’un point de vue sensibilité, si l’éditeur graphique est plutôt intuitif (Editeur similaire à SQLServeur), la création de données structurelles ou de scripts manuels est très orientée T-SQL, impliquant des instructions de type **MERGE** qui m'étaient inconnues avant l'utilisation de cet unique script de post-update.

Autre point négatif pour moi: je trouve que le pipeline de build est plus rapide sous les agents Linux, mais les fichiers projets DACPAC (sqlproj) ne sont pas compatibles .NET Core et ne sont pas compatible avec cette plateforme.

## Conclusion

Savoir ce que l’on veut faire avant de le faire et de se poser des questions sur le « pourquoi » est quelque chose que je considère de primordial.

Le concept de migration m'a permis de ne plus jamais avoir une base de données incohérente dans un projet greenfield.

Pour avoir testé et travaillé avec les 3 outils de cet article, depuis 2014, je pars par défaut sur FluentMigrator.

Mais ce n'est là que le reflet de mon expérience.

Comme à l’accoutumée, le code source de l’article est disponible sur [GitHub](https://github.com/trucs2dev/migration-de-bdd-avec-fluent-migrator).

Vos retours sont bienvenus.
