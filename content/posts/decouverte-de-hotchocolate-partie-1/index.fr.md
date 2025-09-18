+++
date = '2021-05-18T23:01:51+02:00'
draft = false
title = 'Decouverte De Hotchocolate Partie 1'
categories = ["Tech"]
tags = ["API", "GraphQL", "NET"]
+++

Il y a peu, j’ai découvert [HotChocolate](https://chillicream.com/docs/hotchocolate/). C’est une librairie qui permet de faire du GraphQL en .NET.

Il y a d’autres [implémentations](https://graphql.org/code/#c-net) en .NET, mais je dois avouer que c’est celle qui m’a semblée la plus complète et flexible.

## REST vs GraphQL

### Similitudes

REST et GraphQL traitent tous 2 des requêtes HTTP pour fournir des réponses à ces requêtes

### Différences

#### Protocole

- REST appliuqe une **action** (verbe http) sur une **ressource** (URL) représentant l’élément à manipuler (*GET /api/user/33*)
- GraphQL consomme une ressource unique (souvent /graphlq).

Les informations sont transmises par une enveloppe (définie par la norme) qui précise:

- L'action à  appeler (Query ou Mutation)
- Les informations de retour
- les paramètres d’entrée s’il y en a

**Exemple:**

`POST`/graphql

```json
{ 
    "query": "query getBook($id: ID!) { 
        getBook(id: $id) { 
            id 
            title 
            price 
            author: { 
                firstname 
                lastname 
            } 
        } 
    }", 
    "variables": { 
        "$id": "4f47701e-0c27-4bb1-8843-03d2db304c09" 
    } 
}
```

#### Multiplexing

Contrairement à REST, GraphQL permet d'appeler plusieurs actions dans le même appel HTTP.

### Réflexions

La principale différence saute aux yeux: en GraphQL, on spécifie les données que l’on veut alors qu’en REST, on récupère tous les attributs.

GraphQL permet donc une granularité beaucoup plus fine et permet de demander explicitement les champs à utiliser. Pour peu qu’on puisse greffer du comportement à la résolution de champs, cela ouvre un éventail de possibilités qui n’est pas accessible sous REST comme par exemple:

- Conditionner la lecture d’un champs à un droit
- Mesurer la fréquence d’utilisation des champs (ou l'absence d'utilisation)

## Mise en oeuvre

Cet article contient une synthèse sur ce qui constitue GraphQL et une manière très sommaire d’exposer des points d’entrée pour faire des requêtes à l’API.

On commence donc à dérouler un projet de type Web API et on lui ajoute le paquet NuGet HotChocolate.AspNetCore.

On branche les méthodes ConfigureServices et Configure

![Enregistrement des éléments de HotChocolate dans le conteneur Ioc](decouverte-de-hotchocolate-partie-1/image-55.png)

![Branchement du endpoint graphql](decouverte-de-hotchocolate-partie-1/image-56.png)

On fait démarrer le navigateur sur la page « graphql », on lance et on arrive sur le client web GraphQL de Chilli Cream: [Banana Cake Pop](https://chillicream.com/docs/nitro).

Il est possible de désactiver l’outil « Banana Cake Pop » en l’ajoutant aux options lors de l’appel à la méthode .MapGraphQL.

![Désactivation du client graphique graphql](decouverte-de-hotchocolate-partie-1/image-56.png)

Pour l’instant, il n’y a pas encore de schéma découvrable. C’est l’opportunité de développer un peu les concepts de GraphQL.

Le **schema** est une description de tous les types exposées par une API GraphQL.

Il est composé de types de plusieurs natures.

### Le type objet

C’est le type le plus commun et basique de GraphQL. Il représente une donnée interrogée. Il est composé de champs.

*Par exemple un type Person a les champs id, firstname et lastname.*

Les champs sont les seuls éléments d’un objet que l’on peut demander via la requête GraphQL.

D’un point de vue C#, le type objet correspond à une classe ou une interface et chaque élément public correspond à un champs GraphQL; qu’il s’agisse d’un champs, d’une propriété, mais aussi d’une méthode.

### Le type scalaire

Un objet Graphql a un nom et des champs mais tôt ou tard, ces champs vont devoir contenir de la donnée concrète. C’est le rôle des types scalaires: ils représentent la feuille d’une requête.

la norme GraphQL définit par défaut 5 types scalaires:

- **Int**: un entier signé.
- **Float**: Un type à virgule ayant la précision d’un double.
- **String**: une chaîne de caractères encodée en UTF-8.
- **Boolean**: true ou false.
- **ID**: Le type ID représente un identifiant unique, souvent utilisé pour re-récupérer un objet ou en tant que clé pour un cache. Le type ID est sérialisé comme une chaîne de caractères, cependant définir un champs en tant qu’ID signifie qu’il n’a pas vocation à être lu par un humain.

Il est aussi possible de définir des types scalaires personnalisés. Par exemple, le type date:

`
scalar Date
`

### Le type Énumération

Il s’agit d’un type spécial de scalaire qui ne peut prendre qu’une des valeurs définies dans le type. Il permet de:

- Valider qu’un argument de ce type est une des valeurs autorisées ex: le genre ne peut être que **MASCULIN**, **FEMININ** ou **TRANSGENRE**.
- Communiquer à travers le système qu’un champs ne peut avoir qu’une des valeurs prédéfinies dans l’énumération.

### Les types Liste et Non-Null

GraphQL ne permet de définir que les types Objet, scalaires et les énumérations. Mais quand on utilise ces types à d’autres endroits, il est possible de leur apporter des modificateurs qui vont changer la valeur de ces types.

**Exemple:**

```graphql
type Product { 
    id: ID! 
    name: String! 
    reviews: [Review] 
}
```

### Le type Input

Il s’agit des types complexes au même titre que le type Objet.
La différence est qu’un type Objet a pour vocation d’être retourné par l’appel alors que le type Input est envoyé en tant qu’argument.

### Les type Query et Mutation

Ce sont deux types qui sont spéciaux au sein d’un schéma :

```graphql
schema { 
    query: Query 
    mutation: Mutation 
}
```

Chaque schema GraphQL a un type *Query* et peut avoir ou non un type *Mutation*.

Ces types sont similaires en tous points aux types objets mais ils sont spéciaux car ils définissent les point d’entrées de chaque requête GraphQL.

Donc si vous voyez une requête de la forme:

```graphql
query { 
    product(id: "33") { 
        id 
        name 
    } 
    order(id:"806") { 
        id 
        deliveryDate 
    } 
}
```

Cela signifie que le service GraphQL nécessite d’avoir un type Query exposant les champs product et order.

Les mutations fonctionnent de la même manière: les types définis sur le type Mutation sont ceux disponibles en tant qu’élément racines des champs de mutation qu’il est possible d’appeler dans la requête GraphQL.

Il est important de noter que excepté le fait que les types Query et Mutation soient les points d’entrées du schéma, ils fonctionnent de la même manière que n’importe quel type Objet.

## Création du type Query

Après cette longue (mais nécessaire) introduction, créons notre premier type Query :

![HelloQuery en C#](decouverte-de-hotchocolate-partie-1/image-59.png)

et branchons le.
Après avoir lancé l’application, on peut voir que le schéma est bien récupéré et documenté.
Et que les appels fonctionnent correctement.

![Appel à HelloQuery depuis l'utilitaire intégré](decouverte-de-hotchocolate-partie-1/image-62.png)

Nous pouvons aussi voir dans l’analyseur de requête intégré en bas de la fenêtre, le contenu de l’appel HTTP:

![Contenu de l'appel http](decouverte-de-hotchocolate-partie-1/image-64.png)

## Conclusion

On vient juste d’effleurer le sujet et c’est déjà la fin de l’article. En même temps, pas étonnant au vu du nombre de concepts à aborder.

Je découvre et essaie de présenter au mieux la technologie. Cependant je n’ai que peu de recul en terme d’exploitation en production. Plus que jamais vos retours sont précieux et nécessaire pour:

- Corriger cet article
- Ajouter des informations qui vous paraissent importantes
- Orienter la suite de la présentation de l’outil

Ah et sinon, le code source est disponible sur [GitHub](https://github.com/trucs2dev/decouverte-de-hotchocolate-partie-1)

