+++
date = '2021-05-25T23:01:51+02:00'
draft = false
title = 'Decouverte De Hotchocolate Partie 2'
categories = ["Tech"]
tags = ["API", "GraphQL", "NET"]
+++

La semaine dernière, on a pu creuser les concepts GraphQL et mettre en place une « Query » qui renvoyait « Hello World » ou « Hello {name} ». On a aussi vu que GraphQL ne supportait qu’une Queryet une Mutation dans son schéma mais que celle-ci pouvait être composée d’un certain nombre d’ObjectTypes.

Aujourd’hui, on va continuer à creuser de ce côté et voir dans quelle mesure personnaliser ces objets types pour leur ajouter du comportement (description, authentification, etc) et coup de chance: il y en a pour tous les goûts!

## Personnaliser les ObjectTypes

### Option 1 : les attributs

Une méthode que j’ai découverte en 2011 avec les attributs de validation en ASP.NET MVC3.

Les attributs sont particulièrement utiles pour ajouter des métadonnées aux objets ou à leurs contenu. Par exemple, est-ce que le retour de la méthode doit être cachée, est ce que ce champs est marqué obligatoire, etc.

Un détail important est que ce n’est pas directement l’attribut qui modifie le comportement mais un outil tiers qui sait l’interpréter. Prenons l’exemple de la librairie [Newtonsoft.Json](https://www.nuget.org/packages/Newtonsoft.Json/): il est possible de [définir pour une propriété quel convertisseur utiliser](https://stackoverflow.com/a/44893493).

![JsonConverter personnalisé](decouverte-de-hotchocolate-partie-2/image-65.png)

Avec GraphQL, les quelques attributs existant par défaut son liés à la description des métadonnées et à la [gestion des droits](https://chillicream.com/docs/hotchocolate/v10/). Une query sera donc modélisée de la manière suivante:

![Objet Query en C#](decouverte-de-hotchocolate-partie-2/image-66.png)

Vu qu’on n’a pas encore traité de l’authentification, on va conserver uniquement l’attribut GraphQLDescription.

On constate alors sa prise en compte au sein du schéma via l’outil de développement:

**Les pros:**

- Mécanisme standard
- L’information est portée par le type, sur ses méthodes & propriétés
- Pas de nouveaux types à créer

**Les cons:**

- Cela peut cela impliquer de référencer une librairie liée au transport (GraphQL) dans une couche de traitement métier
- Dans le cas de comportements personnalisés définis dans la couche d’exposition, cela peut même engendrer une référence circulaire

### Option 2 : Un type de description et une API Fluent

L’autre option est également très populaire bien que plus récente : une API Fluent.

Nous l’avons vue dans [l’article sur la validation]({{< ref "validation-cqrs-avec-fluentvalidation" >}}) ses avantages sont qu’elle requiert un objet supplémentaire qui aura le rôle de l’ensemble des métadonnées. Cela permet de passer outre les [limitations des attributs](https://docs.microsoft.com/fr-fr/dotnet/csharp/language-reference/language-specification/attributes#attribute-parameter-types) et permet de ne pas être contraint à ajouter des dépendances dans les couches basses de l’application.

Avec HotChocolate, cela se modéliserait de la manière suivante

![API Fluent Hotchocolate](decouverte-de-hotchocolate-partie-2/image-68.png)

Petite spécificité pour la méthode **ConfigureServices** de la classe **Startup**:

> Il faut ajouter l’ObjectType dans la configuration du serveur GraphQL en revanche, S’il s’agit d’un type qui doit être injecté, il faut aussi qu’il soit enregistré en tant que tel dans le conteneur IoC.

**Les pros:**

- Permet de conserver toutes les personnalisations dans la couche d’exposition indépendamment des types sous-jacents
- Permet de centraliser toutes les personnalisations de type

**Les cons:**

- Requiert de créer un objet de description pour chaque ObjectType à personnaliser
- Requiert d’ajouter chaque ObjectType lors de la configuration GraphQL (mais il est possible de tous les ajouter en 1 ligne)

### Étendre les ObjectTypes

Enfin, dernière fonctionnalité abordée dans cet article, parce qu’elle présente une manière d’organiser les fichiers  qui peut être utilisée de manière structurante, la possibilité d’[étendre un ObjectType](https://chillicream.com/docs/hotchocolate/defining-a-schema/extending-types/).

(J’ai du mal a avoir un avis tranché sur son utilisation et ait aussi du mal à en voir l’intérêt pour les autres types que Query et Mutation).

Dans cette hypothèse, la Query n’est qu’un élément racine permettant de « lister » les fonctionnalités métiers qui elles seront représentées par des objets dédiés.

C’est le cas du schéma actuel: ,la Query expose un ObjectType « Me » qui contient les méthodes relatives à l’utilisateur connecté et l’ObjectType « Hello » qui contient les méthodes de l’exemple:

L’application de cette fonctionnalité permet de créer une Query vide et de l’étendre avec des objets dédiés:

![Object Type partial - HelloQuery](decouverte-de-hotchocolate-partie-2/image-74.png)

![Object Type partial - MeQuery](decouverte-de-hotchocolate-partie-2/image-75.png)

avec la modification dans la méthode ConfigureServices pour enregistrer les extensions:

![Object Type partial - MeQuery](decouverte-de-hotchocolate-partie-2/image-78.png)

Cela permet de segmenter le métier jusqu’à la couche d’exposition.

## Conclusion

Cet article a permis de creuser les leviers de personnalisation des ObjectTypes à travers l’enrichissement de leur comportement (principalement des métadata) ou à la possibilité de les étendre.

Personnellement, j’ai du mal a avoir un avis tranché sur la pertinence d’utiliser ou non les extensions des ObjectType. N’hésitez pas à laisser un commentaire pour me dire ce que vous en pensez.

Ah, et sinon, le code source de l’article est disponible sur [GitHub](https://github.com/trucs2dev/decouverte-de-hotchocolate-partie-2).

A bientôt.
