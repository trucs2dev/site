+++
date = '2021-10-26T00:22:10+02:00'
draft = false
title = 'Personnaliser le client Swagger d’une API.NET'
categories = ["Tech"]
Tags = ["NET", "API", "REST", "Architecture", "Swagger"]
+++

Bon, les articles sur l’[authentification]({{< ref "authentification-en-dotnet" >}}) et la [gestion des droits]({{< ref "gestion-des-droits-en-dotnet" >}}), c’est fait!

Mais un détail plutôt critique manquait : l’intégration au client swagger de manière à ce qu’on puisse tester sans dépendre du frontend.

Cet article pallie à ce manque et présente comment configurer swagger pour y intégrer l’authentification.

Le fournisseur d’identité utilisé est **keycloack**, qui a déjà été utilisé dans [cet article]({{< ref "installer-un-serveur-didentite-en-local" >}} ). Et l’implémentation de swagger est [Swashbuckle](https://github.com/domaindrivendev/Swashbuckle.AspNetCore).

## Création du client

Dans la console d’administration de keycloak, sélectionner **Clients** dans la liste de gauche, puis ajouter un client:

![Création de fichiers liés à l'authorisation](personnaliser-le-client-swagger-dune-api-net/image-114.png)

## Modification de l’API.NET

Ok, là d’un point de vue configuration, ca devient un peu du lourd, du coup, plutôt que tout faire dans la méthode `ConfigureServices` de la classe `Startup`, nous allons créer une classe `ConfigureSwaggerOptions` que nous allons enregistrer dans le conteneur IoC et qui sera automatiquement utilisée pour la configuration de swagger:

![Enregistrement de la classe ConfigureSwaggerOptions](personnaliser-le-client-swagger-dune-api-net/image-115.png)

Ok, c’est bien beau mais et ce fichier me direz vous? et bien, il y a plusieurs points à noter. Tout d’abord, il faut savoir qu’il consomme un client http, injecté via le constructeur.

Ensuite, le gros du travail se déroule dans la méthode Configure qui reprends de prime abord une configuration classique:

![Configure Swagger options méthode configure](personnaliser-le-client-swagger-dune-api-net/image-128.png)

A laquelle viennent se greffer les problématiques de sécurité:

On ajoute un OperationFilter, dont le rôle sera de définir quelles méthodes ont besoin d’authentification.

![AuthorizeOperationFilter](personnaliser-le-client-swagger-dune-api-net/image-117.png)

La SecurityDefinition fait appel au discovery document et indique quel flux d’authentification sont présents et sur quels endpoints:

![Contenu de la méthode CreateSecurityDefinition](personnaliser-le-client-swagger-dune-api-net/image-118.png)

Avec ci-dessous le détail de la méthode GetDiscoveryDocument

![Récupération du DiscoveryDocument](personnaliser-le-client-swagger-dune-api-net/image-119.png)

Enfin, le SecurityRequirement précise le scheme à utiliser lors des appels de routes par Swagger (ici, oauth2).

![SecurityRequirement](personnaliser-le-client-swagger-dune-api-net/image-120.png)

Dernière étape : la configuration de l’UI pour indiquer comment s’authentifier (dans la classe Startup):

![Branchement des endpoints pour Swagger](personnaliser-le-client-swagger-dune-api-net/image-121.png)

Et voilà une configuration qui permet de s’authentifier depuis l’API pour tester les appels:

![Authentification via Swagger UI](personnaliser-le-client-swagger-dune-api-net/image-124.png)

![Appel sur une route de test](personnaliser-le-client-swagger-dune-api-net/image-129.png)

### Bonus

Il m’est arrivé chez un client de devoir positionner un header http dans les appels car il était utilisé pour déterminer quelle base de données consommer pour la multitenancy. 

Pour permettre à Swagger de positionner une valeur, il faut décrire cela dans un OperationFilter:

![Custom tenant header operation filter](personnaliser-le-client-swagger-dune-api-net/image-125.png)

A ajouter dans la méthode Configure de la classe ConfigureSwaggerOptions:

![Ajout de l'opération filter](personnaliser-le-client-swagger-dune-api-net/image-126.png)

On constate ainsi en relançant Swagger que l’interface demande maintenant une valeur pour le paramètre x-tenant qui sera utilisé en tant que header:

![Modification de swagger ui pour personnaliser le header](personnaliser-le-client-swagger-dune-api-net/image-127.png)

## Conclusion

Un article plus court qu’à l’accoutumé, mais qui permet de continuer sur la lancée des traitements des problématiques d’une API REST en .NET.

Comme toujours, je suis friand de vos retours et le code source est disponible sur un [dépôt GitHub](https://github.com/trucs2dev/personnaliser-le-client-swagger-dune-api-net).

Bon code !
