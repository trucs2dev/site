+++
date = '2021-04-06T11:07:02+02:00'
draft = false
title = 'Exposition Rest en .NET'
categories = [ "NET", "Development" ]
tags = [ "NET", "API", "REST" ]
+++

Bonjour à tous! Cet article traite de l’exposition http et de la documentation d’une API REST en .NET tout en rappelant certains paradigmes propre à ce protocole.

## REST

Tout d’abord, pour être alignés sur le contenu, quand je parles de REST, il s’agit principalement du niveau 2 du [modèle de maturité de Richardson](https://martinfowler.com/articles/richardsonMaturityModel.html). A ce jour, .NET ne propose pas d'API natives pour passer au niveau 3.

## Mise en route

Pour démarrer, créons un projet .NET de type WebApplication / Api

![Creation d'un nouveau projet](exposition-rest-en-net/image-9.png)

La qualité des modèles de projet a énormément évoluée ces dernières années et ce modèle comprends une configuration d’API standard et une configuration Swagger fonctionnelles.

## Versionnage d’API

Le métier du développement logiciel est passionnant par le nombre d’analogies empruntées à différents métiers. Souvent, c’est le bâtiment qui est visé, quand on parle d’architecture, de fondations; cette fois, c’est de commerce:

> Casser un contrat, c’est donner l’opportunité à son client de changer de fournisseur / prestataire

Souvent, les services développés le sont pour une même organisation, mais quand bien même, c’est tellement plus simple d’un point de vue organisationnel de garantir la version 1 de l’application que les clients et services actuels peuvent consommer et de faire des évolutions sous la forme d’une version 2 pour un client précis. Le fait de déprécier et de ne plus exposer une version de l’API n’est alors plus une contrainte de déploiement ou d’exploitation mais un choix.

Pour pouvoir faire ce choix, cependant, il est nécessaire de prévoir le versionnage d’API au plus tôt. Heureusement, c’est quelque chose qui est très peu coûteux.

## Versionnage de contrôleurs

Pour activer le versionnage de l’API, il va tout d’abord falloir installer les paquets NuGet requis: [Microsoft.AspNetCore.Mvc.Versioning](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Versioning/) et [Microsoft.AspNetCore.Mvc.Versioning.ApiExplorer](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Versioning.ApiExplorer/).

Ensuite, il faut enregistrer les mécaniques de versionnage dans le conteneur IoC de l’application, dans la méthode ConfigureServices de la classe Startup.

![Enregistrement des services dédiés au versionning](exposition-rest-en-net/image-11.png)

Et pour finir, aller dans chaque contrôleur pour préciser que la route dépends de la version d’API des contrôleurs:

![Nom de route qui prend en compte la version](exposition-rest-en-net/image-12.png)

Et c’est tout ce dont .NET à besoin pour prendre en compte le versionnage de contrôleurs.

Les contrôleurs non marqués par l’attribut [ApiVersion] seront enregistrés avec la version définie par défaut (ici, 1.0).

## Prise en compte du versionnage sous Swagger

Pour faire reconnaître les différentes versions d’API à Swagger, il va falloir créer deux classes:

Un fichier ConfigureSwaggerOptions dont le rôle va être de créer une « SwaggerDoc » par version d’API enregistrée:

```csharp
public class ConfigureSwaggerOptions : IConfigureOptions<SwaggerGenOptions> 
{
    private readonly IApiVersionDescriptionProvider _apiVersionDescriptionProvider; 
    
    public ConfigureSwaggerOptions(IApiVersionDescriptionProvider apiVersionDescriptionProvider) 
    {
        _apiVersionDescriptionProvider = apiVersionDescriptionProvider; 
    } 
    
    public void Configure(SwaggerGenOptions options) 
    {
        foreach (var description in _apiVersionDescriptionProvider.ApiVersionDescriptions) 
        { 
            options.SwaggerDoc(description.GroupName, CreateInfoForApiVersion(description)); 
        } 
        
        options.OperationFilter<SwaggerDefaultValues>(); 
    } 
    
    private OpenApiInfo CreateInfoForApiVersion(ApiVersionDescription description) 
    {
        var info = new OpenApiInfo 
        {
            Title = "My Api", 
            Version = description.ApiVersion.ToString() 
        }; 
        
        if (description.IsDeprecated) 
        {
            info.Description += "[DEPRECATED]";
        }
        
        return info; 
    } 
}
```

Un fichier SwaggerDefaultValues qui aura la charge de décrire les opérations et leurs paramètres:

```csharp
public class SwaggerDefaultValues : IOperationFilter 
{
    public void Apply(OpenApiOperation operation, OperationFilterContext context) 
    { 
        var apiDescription = context.ApiDescription; 
        operation.Deprecated |= apiDescription.IsDeprecated(); 
        if (operation.Parameters != null) 
        { 
            foreach (var parameter in operation.Parameters) 
            { 
                var description = apiDescription.ParameterDescriptions.First(x => x.Name == parameter.Name); 
                if (string.IsNullOrEmpty(parameter.Description)) 
                { 
                    parameter.Description = description.ModelMetadata?.Description; 
                } 
                
                parameter.Required |= description.IsRequired; 
            } 
        } 
    } 
}
```

Enfin, il ne reste plus qu’à modifier les méthodes ConfigureServices

![Enregistrement des swagger options](exposition-rest-en-net/image-13.png)

Et Configure de la classe Startup

![Prise en compte des versions dans le client swagger](exposition-rest-en-net/image-14.png)

Et Swagger sait maintenant lister les différentes versions d’API utilisées et saisit automatiquement le paramètre spécifiant la version. Nous pouvons valider cela en créant une classe HelloController et en lui attribuant la version 1.5:

```csharp
using Microsoft.AspNetCore.Mvc;
namespace Api.Controllers 
{
     [Route("api/v{version:apiVersion}/[controller]")] 
     [ApiController] 
     [ApiVersion("1.5")] 
     public class HelloController : ControllerBase 
     { 
        [HttpGet] 
        public string Get() 
        { 
            return "World"; 
        } 
    } 
}
```

![Versions 1 et 1.5 dans l'interface swagger](exposition-rest-en-net/image-15.png)

Et elles sont appliquées automatiquement en tant que paramètre d’url:

![Versions présentes en tant que composant de la route](exposition-rest-en-net/image-16.png)

## Retour d’expérience

Dernière section de cette article qui présente le retour de bientôt une décennie à travailler avec des APIS REST tout d’abord en .NET Framework via la stack Owin/WebApi puis en .NET Core. Cette partie contient davantage un pense-bête que des réflexions révolutionnaires:

- REST signifie **RE**presentationnal **S**tate **T**ransfert. Cela signifie que ce transite via le protocole http peut être dé-corrélé de la structure de données utilisée pour la persistance.
- REST consiste à utiliser au maximum le protocole HTTP. Le verbe (GET, PUT, POST, DELETE) décrire l’opération que l’on souhaite réaliser sur une ressource, représentée par un path(chemin) d’url (Le chemin représente donc un état,un nom et non pas une action) et enfin, il est possible d’identifier un succès d’une erreur directement via le code de retour http.
- REST ne signifie pas CRUD: il est tout à fait possible d’exposer une ressource pour une seule opération (les ressources sont des représentation métier dé-corrélées de la persistance)
- Il est préférable de se donner comme règle : *un contrôleur par ressource*: cela permet de définir un cadre structurant et d’éviter les fichier controleurs de 1000 lignes et plus.
- la grande majorité des verbes doivent être par convention [idempotents](https://fr.wikipedia.org/wiki/Idempotence). Habituellement, il s’agit des méthodes GET, HEAD, PUT et DELETE.

## Conclusion

REST n’est qu’un paradigme comme un autre pour représenter les données. Il fait aujourd’hui partie du trio de tête avec graphql et grpc et il me semblait judicieux de poser par écrit cette synthèse et ce retour d’expérience.

Le code source ayant servi à cet article est disponible sous [Github](https://github.com/trucs2dev/exposition-rest-en-net).
