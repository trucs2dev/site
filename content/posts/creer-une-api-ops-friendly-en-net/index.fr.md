+++
date = '2021-04-27T22:58:04+02:00'
draft = false
title = 'Creer Une Api Ops Friendly en Net'
categories = ["Tech"]
tags = ["NET", "API", "DevOps", "Docker"]
+++

Un article antérieur évoquait les migrations de base de données. Il y a encore long à dire d’un point de vue opérationnel mais nous allons aborder aujourd'hui le point de vue de l’exploitation.

Dans les entreprises, généralement, le service développement (chargé de la réalisation de l’application) et le service exploitation (chargé de son hébergement / de sa mise à disposition) sont deux services différents.

Ayant une sensibilité Devops assez poussée, j'ai souhaité pouvoir livrer aux personnes qui vont opérer l'applicaiton (les "Ops") non seulmement l'application, mais aussi le moyen d'exécuter les opérations d'administration (comme les migrations).

Je vous présente la librairie [Oakton](https://jasperfx.github.io/oakton/) de Jeremy Miller.

## Si le problème c’est les migrations, pourquoi ne pas les lancer automatiquement au démarrage de l’application ?

Pour le développement, c'est une très bonne solution, mais en production,il peut y avoir plusieurs instances et être rapidement capable de faire un rollback. Fournir un ensemble d’opérations d’administrations permet cela.

Ensuite, les exploitants peuvent jouer sur les [roles & permissions du SGBDR](https://docs.microsoft.com/fr-fr/sql/relational-databases/security/authentication-access/database-level-roles) pour séparer l'utilisateur capable d'exécuter les migrations et modifier les schéma de base de données (rôle db_ddladmin ou db_owner) et limiter l’application aux droits db_datareader et db_datawriter.

De plus, les migrations sont la fonctionnalité la plus évidente, vu qu'on en a parlé il y a une semaine, mais les fonctionnalité d’exploitation pourraient concerner tout aussi bien l’affectation / suppression de droits d’un utilisateur (définir un utilisateur en tant qu’administrateur par exemple) ou encore des rotations de certificats.

## Présentation d'Oakton

Oakton, c’est une librairie pour faciliter l’utilisation de ligne de commande pour les applications .NET. Cette librairie a été extraite en 2016 d’un autre [ensemble de librairies crées par Jeremy Miller depuis 2010](https://jasperfx.github.io/oakton/documentation/getting_started/).

L’objectif de la librairie est de rendre trivial la création d’interface en ligne de commande, de proposer une consistance sur le nommage et l’utilisation des arguments, de valider le arguments saisis par l’utilisateur et de séparer l’interprétation de la ligne de commande de son traitement effectif.

Depuis .NET core, l’outil apporte une grosse plus-value avec notamment la possibilité de s’interfacer avec le « HostBuilder« .

Cela permet de transformer l’API en interface de ligne de commande pour laquelle l’exécution par défaut serait l’exposition des services Http.

De ce fait, cela permet de créer d’autres commandes qui pourront venir se greffer à l’API. Commandes qui pourront être lancées par ceux qui ont accès à l’exécutable (l’équipe d’exploitation).

## Mise en oeuvre

Comme beaucoup de librairies, ca commence par l'installation du paquet NuGet [Oakton.AspNetCore](https://www.nuget.org/packages/Oakton.AspNetCore).

Ensuite, il faut modifier le fichier  `Program.cs` de la manière suivante:

![Branchement d'Oakton dans la classe Startup](creer-une-api-ops-friendly-en-net/image-31.png)

Il faut aussi flagger le namespace de la classe program avec l’attribut suivant:

![Marquage de l'assembly comme contenant des commandes Oakton](creer-une-api-ops-friendly-en-net/image-32.png)

Qui permet d’indiquer à Oakton que des commandes se trouvent dans cet assembly.

A partir de là, Oakton est pret à recevoir des commandes. On peut le constater en tapant depuis le dossier du fichier .csproj la commande suivante:

```script
dotnet run -- help
```

Qui va lister les commandes disponibles

![Liste de l'aide fournie par défaut par Oakton](creer-une-api-ops-friendly-en-net/image-36.png)

## Ajouter une commande

Une instruction de ligne de commande est séparée en 2 éléments:

- Les arguments
- Le traitement

On va donc se demander « de quoi a besoin la commande pour fonctionner » lors de sa création. Pour la commande de migration ascendante, il s’agira des tags (optionnels) et de la version cible (optionnelle elle aussi).

Arbitrairement, je décide que la version sera définie en tant que paramètre et les tags en tant que flags.

Faire hériter les arguments de la classe NetCoreInput permettra d’instancier un WebHost et donc de s’appuyer sur sa configuration. Cela permettra dans le cadre d’une image docker de s’appuyer sur les variables d’environnement prévues à cet effet pour configurer l’image.

![Inputs d'une commande de migration Oakton](creer-une-api-ops-friendly-en-net/image-34.png)

Ensuite, pour la commande à proprement parler, elle impose d’hériter de la classe OaktonAsyncCommand<MigrateUpInput>

![Commande de migration Oakton](creer-une-api-ops-friendly-en-net/image-35.png)

Oakton permet de préciser les cas d’utilisation dans le constructeur. Ils seront affichés avec la sous commande –help (ou en cas d’erreur de syntaxe lors de la saisie de la commande).

```script
dotnet run -- migrate-up --help
```

Enfin, pour la réalisation du code de migration, dans un premier temps, il y a la récupération de la chaîne de connexion à la base de données et la vérification de sa validité.

![Récupération de la connection string](creer-une-api-ops-friendly-en-net/image-37.png)

![Affichage d'une erreur si la connection string est manquante](creer-une-api-ops-friendly-en-net/image-38.png)

Puis la méthode `CreateServices` pour enregistrer le runner

![Enregistrement du Runner dans le container IOC .NET](creer-une-api-ops-friendly-en-net/image-39.png)

et `UpdateDatabase`

![Méthode Update database](creer-une-api-ops-friendly-en-net/image-40.png)

Toutes deux très largement inspirées de celles détaillées dans l’[article précédent]({{< ref "migration-de-bdd-avec-fluent-migrator" >}}) concernant les migrations de données.

## Conclusion

A partir de là, il reste à dériver les commandes migrate-down et migrate-rollback qui ne seront pas présentées dans cet article mais seront présentes dans le dépôt [GitHub](https://github.com/trucs2dev/creer-une-api-ops-friendly-en-net) correspondant.

De mon point de vue, cela reste une bonne pratique de lancer automatiquement les migrations quand on est en environnement de développement (ce n’est pas illustré dans cet article): cela permet, en tant que développeur, d’être sûr que sa base de données est à jour.

Cependant cette mécanique n’est pas adaptée pour la production et le fait de pouvoir déclencher ces opérations à la demande est une fonctionnalité très confortable.

Pour pour peu que l’application soit packagée sous la forme d’image Docker, l’application et l’ensemble de ses opérations d’administration sont livrées avec la même image et s’appuient sur les mêmes éléments de configuration.

Et vous ? Avez-vous rencontré une situation semblable ? Ça vous semble trop lourd ? N’hésitez pas à laisser un commentaire !
