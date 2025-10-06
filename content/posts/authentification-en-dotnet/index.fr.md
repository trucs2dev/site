+++
date = '2021-10-12T00:22:10+02:00'
draft = false
title = 'Authentification en .NET'
categories = ["Tech"]
Tags = ["Authentication", "NET", "API"]
+++

Comment depuis une API vérifier qu’un utilisateur est bien qui il prétend être ?

C’est le sujet de cet article qui repose sur les principes Oauth2  / OpenId Connect. Pour le rattrapage, je vous renvoie à [cet article]({{< ref "open-id-oauth2" >}}).

Concernant les choix, techniques, il s’agira d’un **jeton opaque** transmis via les entêtes HTTP.

La [faille CSRF](https://fr.wikipedia.org/wiki/Cross-site_request_forgery) repose sur une authentification *implicite* du navigateur (cookie, session, etc.).

Le fait de passer par les entêtes HTTP est une action explicite qui mitige donc cette faille puisque les entêtes HTTP doivent être positionnées par le code appelant.

## Fournisseur d’identité

Pour pouvoir valider un jeton, il faut être un client valide du fournisseur d’identité (ça tombe bien, le [dernier article]({{< ref "installer-un-serveur-didentite-en-local" >}} ) traite du sujet).

Créons donc le client « api-rest » sous keycloack.

Comme le client n’est pas utilisé à des fins d’authentification, nous pouvons décocher les options qui y sont lié:

![Configuration du client Keycloak](authentification-en-dotnet/image-90.png)

Petite différence avec le frontend, nous allons déclarer l’`access_type` en **confidential** de manière à obtenir un `client_secret` pour pouvoir appeller le fournisseur d’identité de manière un peu plus sécurisée par l’API.

![Access type confidential](authentification-en-dotnet/image-91.png)

Après avoir enregistré, l’onglet **credentials** est ajouté au client et permet de (ré)générer et afficher le  **client_secret**.

![Génération du client secret](authentification-en-dotnet/image-92.png)

## Du côté de l’API REST

Pour pouvoir identifier un utilisateur via un jeton opaque, la librairie [IdentityModel.AspNetCore.OAuth2Introspection](https://www.nuget.org/packages/IdentityModel.AspNetCore.OAuth2Introspection) est nécessaire.

Après avoir installé le paquet nuget sur le projet, il reste à le configurer dans la méthode `ConfigureServices` de la classe `Startup`:

![Enregistrement dans l'Ioc](authentification-en-dotnet/image-93.png)

Ensuite, comme l’appel à l’API est réalisée depuis un frontend hébergé sur une URL distincte, il faut configurer le [CORS](https://developer.mozilla.org/fr/docs/Web/HTTP/Guides/CORS) pour informer le client qu’il peut faire des appels.

Dans le cadre de l’article, on autorise tout maisen production, il faut évidemment être beaucoup plus vigilant.

![Configuration CORS](authentification-en-dotnet/image-95.png)

Ensuite, plus qu’à activer tout ça dans la méthode « Configure » de la classe « Startup »:

Pour imposer l'authentification, il suffit d'ajotuer l'attribut `[Authorize]` à un controlleur.

![Contrôleur doté d'un attribut Authorize](authentification-en-dotnet/image-97.png)

{{< admonition type=note title="💡" open=true >}}
L’attribut [Authorize] peut-être positionné sur la classe ou individuellement par méthode.
Sa contrepartie, l’attribut [AllowAnonymous] aussi.
{{< /admonition >}}

Ou mieux: utiliser une politique de sécurité par défaut, dans la classe Startup:

![Authorization par défaut sur tous les controleurs](authentification-en-dotnet/image-98.png)

Puis à ajouter un code d’appel depuis le frontend:

![Appel depuis le frontend](authentification-en-dotnet/image-99.png)

On peut alors constater que l’appel renvoie un contenu valide.

## Conclusion

Un article plus court qu’habituellement, il permet de répondre à la question « qui »  d’une application REST en .NET Core. Le prochain article traitera de la gestion des droits (le « quoi »).

Comme toujours, le code source est disponible dans un dépôt [GitHub](https://github.com/trucs2dev/authentification-en-dotnet).

Bon code, et n’hésitez pas à me faire part de vos retours en commentaire ! (comment faites vous habituellement ? Et surtout: pourquoi ?)
