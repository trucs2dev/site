+++
date = '2021-05-11T22:40:42+02:00'
draft = false
title = 'Validation Cqrs Avec Fluentvalidation'
categories = ["Tech"]
tags = ["NET", "Valdation", "CQRS", "Architecture"]
+++

A travers les différents projets sur lesquels j’ai pu travailler, j’ai pu identifier 4 couches différentes de validation, chacune avec une utilité propre:

- La validation côté UI, qui permet d’informer l’utilisateur de l’invalidité des données saisies avant même d’effectuer un appel http. Cette validation a pour rôle principal d’**améliorer l’expérience utilisateur**.
- La validation des données entrantes côté serveur, souvent identique à la validation UI mais avec un rôle radicalement différent: celui de garantir aux couches de traitement métier certains postulats simples sur les données qui vont être manipulées (champs texte présent, de moins de n caractères, date de début inférieure à date de fin, numéro de téléphone valide, etc). Cela permet de **limiter les entrées malicieuses** et de s’épargner le traitement métier si l’on sait qu’il manque des informations. **Ces données n’ont besoin de rien d’autre qu’elles mêmes pour être validées**.
- La validation **métier**, qui va dépendre des règles métier de l’application (Ex: pas possible de créer 2 objets avec le même nom, pas possible d’accéder aux données d’un autre utilisateur, etc). Ces validations réalisées sur le serveur lors du traitement métier. Ells peuvent nécessiter la connaissance de l’état de l’application. Elles permettent d’arrêter le traitement en renvoyant une **erreur métier intelligible par l’utilisateur**.
- Les contraintes d'intégrité. Il est possible d’utiliser des contraintes d’unicité ou un trigger pour garantir d’un point de vue transactionnel que l’opération en BDD est bel et bien valide. C’est le dernier filet de sécurité mais c’est aussi celui qui renvoie un texte le plus incompréhensible pour l’utilisateur de l’application.

Cet article traite du second point : la validation des données entrantes côté serveur, mais dans un contexte CQRS, pour faire suite à l’article relatif au [CQRS]({{< ref "pourquoi-faire-du-cqrs" >}}).

## Pourquoi FluentValidation ?

.NET fournit déjà un [mécanisme de validation](https://docs.microsoft.com/fr-fr/aspnet/core/mvc/models/validation) alors pourquoi utiliser FluentValidattion ?

Je perçois au moins 2 raisons:

1. Le mécanisme de validation de .NET core est intrinsèquement lié aux mécanismes de controllers REST. Ici nous allons les intégrer dans le pipeline de MediatR, dans les appels aux Command Handlers / Query Handlers CQRS, cela permettra de les utiliser non seulement avec les controllers REST mais aussi dans un cadre d’utilisation gRPC ou GraphQL.

2. Vis à vis de ma sensibilité : je trouve que trop d’annotations polluent la lisibilité et sont plus complexes à mettre en oeuvre. Je trouve qu’une classe surchargée d’attribut est moins lisible, moins claire. Aussi, je privilégie souvent le fait de laisser les classes les plus « naturelles » possible et d’avoir des mécanismes transverses autour, que ce soit pour la validation avec FluentValidation ou pour EntityFramework avec l’utilisation de l’[API Fluent](https://docs.microsoft.com/fr-fr/ef/core/modeling/#grouping-configuration).

## Création et enregistrement des validateurs

Pour cet article, vous aurez besoin de 2 paquets NuGet:

- [FluentValidation.DependencyInjectionExtensions](https://www.nuget.org/packages/FluentValidation.DependencyInjectionExtensions) pour enregistrer les validateurs au sein du conteneur IoC ASP.NET
- [FluentValidation](https://www.nuget.org/packages/FluentValidation) pour les types de base permettant de créer les validateurs et leurs règles

*FluentValidation est installé par FluentValidation.DependencyInjectionExtensions. Si vous travaillez sur un mono-projet, pas besoin d’installer le second.*

Un validateur se crée en héritant de la classe AbstractValidator&lt;TypeAValider>

![Un validateur](validation-cqrs-avec-fluentvalidation/image-46.png)

L’objet de l’article n’est pas de rentrer dans le détail du Framework mais sa mise en oeuvre. La documentation de FluentValidation est suffisamment claire et exhaustive pour pouvoir s’y retrouver à mon sens. Si cependant vous souhaitez un article détaillant le Framework, n’hésitez pas à laisser un commentaire pour le demander 😊

Au niveau du branchement dans le conteneur IoC, une méthode d’extension permet d’enregistrer l’ensemble des validateurs d’une Assembly:

![Enregistrement des validateurs d'une assembly](validation-cqrs-avec-fluentvalidation/image-46.png)

## Mise en oeuvre dans le tunnel d’invocation CQRS

L’article sur le [pipeline HTTP]({{< ref "le-pipeline-http-en-net" >}}) a présenté la mise en oeuvre de middleware pour s’inscrire dans le traitement global de l’appel. Bonne nouvelle : MediatR fournit dispose d’un mécanisme similaire nommé [Behavior](https://github.com/jbogard/MediatR/wiki/Behaviors).

A l’instar du middleware dans le Pipeline HTTP, le Behavior s’inscrit dans le « tunnel » d’exécution qui va amener la Command / Query au CommandHandler / QueryHandler.

Le behavior à mettre en oeuvre est le suivant:

![Behavior de validation](validation-cqrs-avec-fluentvalidation/image-50.png)

et son ajout dans le conteneur IoC ASP.NET Core se fait de la manière suivante:

![Enregistrement du behavior de validation](validation-cqrs-avec-fluentvalidation/image-49.png)

(d’autres [exemples](https://github.com/jbogard/MediatR/tree/master/samples) sont fournis sur le GitHub de MediatR pour d’autres containers IoC)

Plus qu’à tester notre projet:

![Règle de validation](validation-cqrs-avec-fluentvalidation/image-54.png)

![Url qui doit échouer](validation-cqrs-avec-fluentvalidation/image-51.png)

La valeur est « invalide », cela est affiché dans le résultat HTTP:

![Appel swagger invalide](validation-cqrs-avec-fluentvalidation/image-52.png)

## Conclusion

Je suis conquis par l’outil pour des validations simples. Il est possible de faire beaucoup plus, de l’utiliser en asynchrone et d’y injecter des services pour faire des appels tiers, mais selon moi, c’est lui attribuer des responsabilités qu’il ne devrait pas avoir.

Comme d’habitude, le code source de l’article est sur son dépôt [GitHub](https://github.com/trucs2dev/validation-cqrs-avec-fluentvalidation) associé.

Et vous ? Avez-vous déjà utilisé cet outil ? Quel est votre retour d’expérience dessus ?
