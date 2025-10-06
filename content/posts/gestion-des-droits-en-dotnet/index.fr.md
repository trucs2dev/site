+++
date = '2021-10-19T00:22:10+02:00'
draft = false
title = 'Gestion des droits en .NET'
categories = ["Tech"]
Tags = ["Authentication", "NET", "API", "REST"]
+++

Comment depuis une API vérifier qu’un utilisateur est bien qui il prétend être ?

Cet article fait suite à [celui concernant l’identification]({{< ref "authentification-en-dotnet" >}}). Il présente les mécanismes de gestion des droits présents dans ASP.NET et comment étendre ce mécanisme pour répondre à votre modèle d’authentification.

## Gestion des droits selon Microsoft

La [documentation Microsoft relative à l’autorisation](https://docs.microsoft.com/fr-fr/aspnet/core/security/authorization/introduction) présente différentes manières de valider les droits d’un utilisateur.

Quelle que soit l’option, du point de vue des contrôleurs REST, les attributs `AllowAnonymousAttribute` et `AuthorizeAttribute` permettent de déclarer pour une classe ou une méthode la nécessité d’être identifié ou si un accès anonyme est permis (la configuration par défaut étant l’accès anonyme pour tous).

### Par rôles

L’un passe par la notion de rôles, la documentation présente à ce sujet un exemple s’appuyant sur [ASP.NET Identity](https://docs.microsoft.com/fr-fr/aspnet/core/security/authorization/roles) mais il est également possible d’obtenir les [rôles depuis un Claim dédié](https://benfoster.io/blog/asp-net-identity-role-claims/).

Il y a une chose qui me dérange profondément dans cette approche, c’est le fait qu’en environnement distribué où plusieurs services dépendent du même fournisseur d’identité, cela signifie que c’est au fournisseur d’identité de « connaitre » les rôles disponibles dans les différent services (et espérer qu’ils n’ont pas les mêmes noms dans les différents services).

### Par Claim

Autre option présentée: [la gestion des accès par claim](https://docs.microsoft.com/fr-fr/aspnet/core/security/authorization/claims).

Ici, l’idée est de définir des politiques (policies) de droits basées sur les informations de l’utilisateur via ses claims (Ex: définir une policy **IsAdult** basée sur le fait que l’utilisateur dispose d’un Claim **Age** qui ait une valeur supérieure ou égale à 18.

Je n’ai jamais rencontré de scénarios ou cette option me paraissait la meilleure donc, je la présente pour la culture générale, mais cet article ne la détaillera pas davantage a défaut de cas pratiques. Si vous voyez des cas d’utilisation, je vous serais reconnaissant de les indiquer en commentaire.

### Par gestionnaire d’autorisation personnalisée

[Cette option](https://docs.microsoft.com/fr-fr/aspnet/core/security/authorization/policies#use-a-handler-for-multiple-requirements) est celle que j’utilise par défaut dans mes projets, c’est celle qui laisse la main à l’application sur la définition de sa politique d’accès.

De plus, dans le cas d’un bearer token, cela permet de considérablement réduire sa taille car il a moins d’information a porter.

ASP.NET permet l’utilisation de politiques d’authentification au niveau du pipeline HTTP. C’est ce cas d’utilisation qui va est présenté ici.

## Mise en œuvre

Tout d’abord, on va s’intéresser aux mécaniques. Si l’on décortique [la documentation](https://docs.microsoft.com/fr-fr/aspnet/core/security/authorization/secure-data?view=aspnetcore-5.0#create-owner-manager-and-administrator-authorization-handlers), on voit qu’il y a 2 objets qui fonctionnent de concert:

- Le requirement, dont la principale fonction est d’indiquer une vérification a appliquer et de pouvoir être validé
- Le handler, qui a pour fonction de valider le requirement selon les règles qu’il contient. C’est lui qui contient le code de vérification.

### Etape 1: brancher la mécanique

Vu qu’il s’agit de problématiques liées au transport, je positionne ces 2 classes dans un dossier `Infrastructure/Authorization` au sein de mon Api:

![Création de fichiers liés à l'authorisation](gestion-des-droits-en-dotnet/image-102.png)

Du point de vue du code, pour l’instant, les 2 classes sont vide.

![Classe CheckAuthorizationRequirement](gestion-des-droits-en-dotnet/image-103.png)
![Classe CheckAuthorizationRequirementHandler](gestion-des-droits-en-dotnet/image-105.png)

Ensuite, vient le branchement de ces classe. Ici, tout se passe dans la méthode `ConfigureServices` de la classe `Startup`:
![AuthorizationPolicyBuilder dans la méthode ConfigureServices](gestion-des-droits-en-dotnet/image-106.png)

D’abord on définit la politique de validation des accès.

Ensuite, on la définit en tant que politique de validation par défaut.

Et enfin, on enregistre l’AuthorizationHandler pour qu’il soit découvrable par son interface.

A noter que la mécanique de validation des droits pour MVC est toujours basé sur l’attribut `Authorize` au niveau de tous les controllers, cela va conditionner le fonctionnement du handler.

![Application de Authorize sur tous les controleurs](gestion-des-droits-en-dotnet/image-107.png)

Ici, on peut déjà mettre un point d’arrêt dans le le gestionnaire et constater que l’on passe bien dessus lors d’un appel de méthode.

### Etape 2 : vision de la logique de validation

Pour la logique de validation, nous allons partir sur quelque chose de primaire: je souhaiterais qu’au sein de mon API, par défaut, j’ai besoin d’être authentifié.

Ensuite, je souhaiterais pouvoir surcharger au niveau du Controller soit la classe soit soit la méthode (si les 2 sont surchargés, c’est la méthode qui fait foi) et indiquer via un attribut le droit nécessaire.

J’aimerai bien quelque chose de ce genre :

![Vision d'utilisation d'un attribut RequireRight](gestion-des-droits-en-dotnet/image-108.png)

### Etape 3 : implémentation

D’abord, on va créer l’attribut:

![Attribut RequireRight](gestion-des-droits-en-dotnet/image-109.png)

Ensuite, au niveau de du handler, tout d’abord, on vérifie qu’on est dans le bon contexte (ActionFilter).

![Récupération du contexte](gestion-des-droits-en-dotnet/image-110.png)

{{< admonition type=note title="💡" open=true >}}
La facon de récupérer les droits dépend beaucoup de comment est configurer l’api.

Plus d’information ici: https://stackoverflow.com/questions/59197631/context-resource-as-authorizationfiltercontext-returning-null-in-asp-net-core
{{< /admonition >}}

Si c’est le cas, on essaie de récupérer l’attribut (si on n’y parvient pas, il n’y a pas de validation complémentaire à appliquer et on considère que le requirement est valide).

![Récupération de l'attribut RequireRight apposé](gestion-des-droits-en-dotnet/image-111.png)

Puis, on récupère l’utilisateur connecté et on essaie de le valider.

![Vérification des droits de l'utilisateur connecté](gestion-des-droits-en-dotnet/image-112.png)

Le détail de la méthode `CanUserAccessAsync` est laissé libre à l’utilisateur. En voici tout de même un exemple:

![Exemple d'implémentation de CanUserAccessAsync](gestion-des-droits-en-dotnet/image-113.png)

**A noter:** la comparaison du nom d’utilisateur est un mauvais exemple, et comme le `RequirementHandler` est configuré **Scoped** dans le conteneur IoC, il est possible d’y injecter un `DBContext` par exemple pour faire des vérifications en BDD.

## Conclusion

Je voulais écrire un article court, c’est raté.

Cependant, j’ai pu partager avec vous une manière de gérer les droits applicatifs dans une API.NET.

Comme toujours, l’exemple est présent sur un [dépôt GitHub](https://github.com/trucs2dev/gestion-des-droits-en-dotnet).
