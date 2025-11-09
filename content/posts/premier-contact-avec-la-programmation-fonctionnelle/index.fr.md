+++
date = '2023-03-07T00:22:10+02:00'
draft = false
title = 'Premier contact avec la programmation fonctionnelle'
categories = ["Billet d'humeur"]
Tags = ["Billet d'humeur"]
+++

Il y a quelques temps de ça, j’ai fait la connaissance de Guillaume.

Guillaume intervenait pour proposer une stack technique en .NET à une équipe de développeurs .NET fraîchement formés.

Du standard: [Clean Architecture](https://github.com/jasontaylordev/CleanArchitecture), [Vertical slices](https://jimmybogard.com/vertical-slice-architecture/), [CQRS](https://lostechies.com/jimmybogard/2015/05/05/cqrs-with-mediatr-and-automapper/), ORM, etc.

Il y avait une librairie que je connaissais pas : [LanguageExt](https://github.com/louthy/language-ext).

Il me l’a présentée comme *« une librairie qui introduit des concepts fonctionnels en C# »*.

Intéressant, j’avais récemment visionné une [vidéo sur la stack SAFE en F#](https://www.youtube.com/watch?v=Ih5r5qD7bTo) et je suis quelqu’un de curieux par nature.

Je vais donc sur le site de la librairie, commence à parcourir son « README » et tombe rapidement sur ça:

![LanguageExt, section transformers](premier-contact-avec-la-programmation-fonctionnelle/image-139.png)

Qu’est ce que c’est un « monadic type » ?

Au cours de mes recherches, je tombe sur la librairie [monio](https://github.com/getify/monio) qui a un [article qui parait assez complet sur le sujet](https://github.com/getify/monio/blob/master/MONADS.md#expansive-intro-to-monads) mais ca reste encore très abstrait.

## A la recherche d’un langage

Les découvertes que j’ai pu faire m’ont donné des fragments de connaissances et m’ont poussé à creuser davantage pour comprendre ses concepts. Quoi de mieux que mettre les mains dans le code et manipuler un langage pour avoir un feedback rapide ?

Je me suis donc lancé sur une recherche sur « le meilleur langage fonctionnel » et bien que ce sujet soit subjectif, Haskell est souvent revenu en tant que « langage le plus riche / profond / complet ».

C’est parti pour une série de tutoriaux / livres & vidéos en tous genres (je recommande particulièrement [Learn You Haskell](https://lyah.haskell.fr/) que je n’ai pas encore fini et qui présente une introduction très exhaustive).

Dans le process, j’ai pu découvrir quelques éléments qui offrent d’énormes avantages à la programmation fonctionnelle. Ses principaux coût étant la courbe d’apprentissage et une communauté moindre (bien que très aidante).

### Avantage #1 – Les types algébriques

Dans sa présentation "[Functional Design Patterns](https://www.youtube.com/watch?v=srQt1NAHYC0)" , Scott Wlaschin préfère les appeler « types composables ». Leur particularité est qu’il est possible de limiter les états de l’application à des états cohérents.

Un exemple d’utilisation que j’ai souvent vu sur des applications web serait un objet qui dit si:

- Les données sont en train de charger
- Une erreur s’il y en a eu une
- Les données en question

J’ai très souvent vu l’objet suivant:

*Contenu manquant*

en Haskell, ca donnerait quelque chose comme ca:

*Contenu manquant*

Ca se ressemble, mais il faut savoir qu’il s’agit ici de type « ou », c’est à dire qu’il peut être Loading OU List Item OU Error String.

Autre particularité, les compilateurs et transpileurs de langages fonctionnels sont capable d’analyser si tous les cas sont couverts et (il est vivement conseillé de le faire) d’échouer si leur couverture n’est pas exhaustive.

Par exemple, avec la structure de donnée Shape et un calcul de l’aire incomplet:

![type algébrique et pattern matching incomplet](premier-contact-avec-la-programmation-fonctionnelle/image-143.png)

La compilation échoue avec un message d’erreur

![erreur de compilation](premier-contact-avec-la-programmation-fonctionnelle/image-144.png)

### Avantage #2 : Des fonctions pures

Une fonction pure au sens mathématique, c’est à dire que la sortie d’une fonction ne dépend que des paramètres qui lui sont passées et ne génèrent pas d’effet de bord.

Pour parvenir à ça, les langages fonctionnels imposent souvent l’immutabilité des objets: tout objet créé ne peut être modifié, c’est un nouvel objet qui est renvoyé à chaque fois.

Autre avantage, cela rend les fonctions « honnêtes », en décrivant ce qu’elle font et également facile à tester.

### Avantage #3: Gestion des effets de bords

Une application sans effet de bord n’aurait pas d’autre utilité que de chauffer un peu plus la pièce. Sont désignés comme effet de bord tous ce qui n’est pas déterministe / prédictible / qui peut changer avec le temps. Parmi lesquels:

- Le temps actuel
- Un appel en base de donnée
- Un appel HTTP
- La lecture ou affichage sur la console
- La lecture ou écriture dans un fichier
- etc.

Pour cela, j’ai pu voir 3 grandes approches dans les langages fonctionnels:

- Non géré, comme c’est le cas en [Scala](https://stackoverflow.com/a/11620444) ou [F#](https://www.compositional-it.com/news-blog/impure-functions-in-fsharp/)
- Géré à travers une monade, comme en [Haskell](https://lyah.haskell.fr/entrees-et-sorties) ou en [PureScript](https://jordanmartinez.github.io/purescript-jordans-reference-site/content/21-Hello-World/02-Effect-and-Aff/src/01-Effect/01-Native-Side-Effects-via-Effect.html)
- Par le runtime, comme pour [ELM](https://elmprogramming.com/side-effects.html)

### Avantage #4: Meilleure scalabilité

Quand on ne dépends pas d’états propre à l’application, il devient possible de paralléliser ou scaler de manière horizontale sans impacts. 

Ainsi, là où ces dernières années, la [loi de Moore](https://fr.wikipedia.org/wiki/Loi_de_Moore) ne semble plus concerner la vitesse d’un processeur mais le nombre de processeurs dans une machine, le paradigme de programmation fonctionnel semble tout indiqué pour bénéficier de cette transformation au maximum.

## Conclusion

Au regard de toutes ses promesses, le paradigme de programmation fonctionnelle me parait très prometteur je vais continuer à l’explorer.

J’ai commencé à parcourir Haskell, PureScript et récemment ELM.

J’en profiterai pour publier mon expérience à travers des articles à venir.

Si vous avez des ressources intéressantes, je suis preneur!
