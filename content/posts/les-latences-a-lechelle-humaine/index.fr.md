+++
date = '2023-03-14T00:22:10+02:00'
draft = false
title = 'Les latences à l’échelle humaine'
categories = ["Billet d'humeur", "Tech"]
Tags = ["Billet d'humeur", "Tech"]
+++

Aujourd’hui, j’ai eu affaire à un collègue qui m’a demandé comment optimiser une requête linq.

C’est quelqu’un que j’apprécie et qui est en plein processus de construction de sa vision du code.

Il me parle de contraintes métier, et du fait qu’il aurait souhaité optimiser ses temps de traitement.

## Anticipation du problème

Ok, j’ai une connaissance de la pile technique de l’application, je sais qu’elle est basée sur Entity Framework.

J’ai aussi entendu les mots clés « optimiser » et « requête Linq ».

J’ai un peu peur: le modèle de données est complexe et je n’en ai qu’une vision / connaissance très superficielle pour être pertinent.

Je concentre alors toute mon attention sur ses explications, comment il essaie de supprimer une boucle « foreach » de ses appels Linq.

Ca me parait faire sens : avec une boucle foreach, il va multiplier les appels à la base de données et tomber dans le trop classique cas vu avec les ORMs : le problème du « Select N+1 ».

## Prise de conscience

Au bout d’un moment, je regarde de plus près et je réalise alors la signature de la méthode qui contient ces fameuses requêtes à optimiser.

```csharp

public static List<Result> maFonction(List<EntiteA> input1, List<EntiteB> input2)

```

Méthode statique, pas d’injections depuis le container IoC et aucune mention d’Entity Framework.

De plus les listes sont les implémentations finales et à ce stade, les appels ont déjà été fait.

Vu le modèle de données de l’application, le fait de charger ces éléments en mémoire peut aussi être pertinent.

Me reviennent alors en tête les propos d’un collègue, la semaine passée :

{{< admonition type=quote open=true >}}

on ne peut optimiser que ce qu’on mesure

{{< /admonition >}}

Et également un [tweet de Udi Dahan](https://twitter.com/udidahan/status/1298648965441880065), sur lequel il met à l’échelle les temps de traitements informatiques à des dimensions sur lesquelles nous pouvons raisonner.

![cout de différents appels si un cycle CPU prenait 1s](les-latences-a-lechelle-humaine/cpu_time_scaled.png)

Nous avons pu alors en discuter et avons décidé qu’après déjà 1 an dépensé (l’appel en DB), ce n’étais pas quelques minutes de plus (le traitement de l’algorithme) qui allaient faire la différence.

## Conclusion

Après avoir écrit cet article, plusieurs enseignements me viennent à l’esprit:

- Le plus gros des optimisations d’une application de gestion se fait via l’optimisation des appels externes
- Un ORM facilite la consommation des données mais demande une bonne connaissance de son comportement (requêtage, projections)
- Utiliser un ORM ne dispense pas de comprendre le SQL et les bases de données: utiliser une requête Linq complexe pour limiter les appels va faire appel a des jointures, il est important de savoir ce qu’est un index et comment il permet d’optimiser une recherche.
