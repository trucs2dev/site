+++
date = '2021-04-13T11:33:27+02:00'
draft = false
title = 'Le Pipeline Http en Net'
categories = ["Tech"]
tags = ["NET", "API", "Architecture"]
+++

Élément structurel de .NET depuis OWIN, le pipeline Http permet d’adresser des problématiques structurelles de manière mécanique.

## Le pipeline Http, c’est quoi au juste ?

Le pipeline Http c’est ce qui adresse toutes les requêtes Http au sein d’une application .NET et fournit une réponse appropriée.

Il faut voir cela comme une chaîne de décorateurs s’appelant successivement et chacun effectuant un traitement pour chaque étape. Le dernier maillon étant la définition des endpoints.

![Illustration du pipeline http](le-pipeline-http-en-net/image-17.png)

Le pipeline Http se définit dans la méthode Configure de la classe Startup.

![Configuration du pipeline http](le-pipeline-http-en-net/image-18.png)

On peut voir alors que le modèle est très simple : chaque appel enrichit l’ensemble du programme et s’applique à l’intégralité des requêtes.

## Gestion transverse des exceptions

A l’aide de ce mécanisme, il devient possible de gérer systématiquement les exceptions. Et d’interpréter les codes de retour et le contenu en rapport avec l’exception interceptée. Le Framework MVC propose [un mécanisme de ce genre](https://docs.microsoft.com/fr-fr/aspnet/core/web-api/handle-errors). Cependant, le fait d’utiliser le pipeline http a l’avantage de permettre plus de contrôle et d’intercepter TOUTES les erreurs qui peuvent se produire dans le pipeline.

Pour cela, commençons par créer ce middleware. Dans le cadre de cet article, je m’attache davantage à démontrer la mécanique qu’à fournir un exemple complet. Un exemple complet devrait être en mesure de retourner les erreurs sous la forme de la RFC « [Problem Details for HTTP APIs](https://tools.ietf.org/html/rfc7807) » et pourrait être configurable par la méthode *ConfigureServices*.

Ici, rien de tout ça:

```csharp
public class GlobalExceptionHandlerMiddleware 
{ 
    private readonly RequestDelegate _next; 
    public GlobalExceptionHandlerMiddleware(RequestDelegate next) 
    { 
        _next = next; 
    }

    public async Task Invoke(HttpContext context) 
    { 
        try 
        { 
            await _next(context); 
        } 
        catch (Exception e) 
        { 
            switch (e) 
            { 
                // NDR : Cela peut être interressant de suivre la norme "Problem Details for HTTP APIs" https://tools.ietf.org/html/rfc7807 
                case ValidationException validationException: // Bad request 
                    context.Response.StatusCode = (int) HttpStatusCode.BadRequest; 
                    context.Response.ContentType = "application/json"; await 
                    context.Response.WriteAsJsonAsync(new 
                        { 
                            Errors = validationException.Errors 
                        }); 
                    break; 
                // ... 
                default: 
                    // Tout le reste 
                    context.Response.StatusCode = (int) HttpStatusCode.InternalServerError; 
                    break; 
            } 
        } 
    } 
}
```

Tout ce que fait ce middleware c’est :

- Un try / catch pour effectuer l’appel suivant
- La comparaison du type de l’exception pour retourner une information différente

Essayons le avec un contrôleur dédié:

```csharp
[ApiController] 
[Route("[controller]")] 
public class ErrorController : ControllerBase 
{ 
    [HttpGet("id")] 
    public IActionResult Error(string id) 
    { 
        if (!int.TryParse(id, out _)) 
        { 
            throw new ValidationException(new[] 
                { 
                    new ValidationFailure("id", "id n'est pas un nombre valide") 
                }); 
        } throw new Exception("Server exception"); 
    }
}
```

Et nous pouvons constater que lorsqu’on passe une chaîne de caractères en paramètre, nous avons une réponse http avec un code 400 et le détail de l’erreur:

![Erreur identifiant invalide](le-pipeline-http-en-net/image-19.png)

Alors que lorsqu’on l’appelle avec un chiffre valide, nous avons une erreur 500 et aucune information supplémentaire:

![Erreur 500](le-pipeline-http-en-net/image-20.png)

## Gestion transverse de l’Unit Of Work

Le pattern [Unit Of Work](https://martinfowler.com/eaaCatalog/unitOfWork.html) va permettre de garantir que l’ensemble des opérations de persistance vont soit réussir soit échouer mais en aucun cas il ne peut y avoir d’état inconsistant.

L’exemple suivant utilise Entity Framework Core. Mais il pourrait fonctionner avec n’importe quel système transactionnel. Fut-ce un ORM, Une connexion au sens ADO.NET ou encore un moteur NoSQL. L’idée est de créer un middleware ASP.NET pour :

1. Démarrer une transaction pour chaque requête Http
2. S’il y a eu des modifications, à la fin de la requête Http, elles sont persistée et la transaction est validée.

Un petit point sur ce qu’est une Session NHibernate ou un DBContext EntityFramework:

Ce sont tous deux des [Repositories](https://martinfowler.com/eaaCatalog/repository.html), dans le sens oùils permettent de manipuler les entités exposées en faisant abstraction du language de requêtage. Mais ce sont également des Unit of work du fait de leur fonctionnement (la méthode SaveChanges d’EF Core ou le fait que les modifications ne soient poussées avec NHibernate qu’avec le Commit de la transaction ou le Flush de la session).

Le projet utilise un DBContext, un modèle de données de personnes et un contrôleur permettant de créer plusieurs personnes. (et qui va bien entendu échouer en cours de route)

![Insertions inconsistentes](le-pipeline-http-en-net/image-21.png)

Dans la méthode ci-dessus, sans UnitOfWork, Rick, Morty, Summer et Beth sont créés et ajoutés à la base de données et tant pis pour les incohérences de données.

Pour rendre les appels http transactionnel, nous allons créer une classe UnitOfWorkMiddleware qui valide et traite ensemble les changements en attente dans le DbContext

```csharp
public class UnitOfWorkMiddleware 
{ 
    private readonly RequestDelegate _next; 
    public UnitOfWorkMiddleware(RequestDelegate next) 
    { 
        _next = next; 
    } 
    
    public async Task Invoke(HttpContext context, ApplicationContext dbContext) 
    { 
        var cancellationToken = context.RequestAborted; 
        var transaction = await dbContext.Database.BeginTransactionAsync(cancellationToken); 
        try 
        { 
            await _next(context); 
            if (dbContext.ChangeTracker.HasChanges()) 
            { 
                await dbContext.SaveChangesAsync(cancellationToken); 
            } 
            
            await transaction.CommitAsync(cancellationToken); 
        } 
        catch 
        { 
            await transaction.RollbackAsync(cancellationToken); throw; 
        } 
    } 
}
```

Et nous l’utiliserons dans la classe Startup juste avant que nous en ayons besoin :

![Branchement de l'Unit of Work dans la classe Startup](le-pipeline-http-en-net/image-22.png)

Cela permet de garantir mécaniquement que toutes les opérations HTTP seront transactionnelles: soit tout est persisté soit rien mais il est dorénavant impossible d’avoir un état inconsistant.

## Conclusion

Le pipeline HTTP est l’outil à utiliser quand il s’agit d’adresser les problématiques structurelles de manière mécanique :

- Log des exceptions
- Gestion des exceptions
- Aspect transactionnel
- Mesure des temps d’exécutions des appels http
- Analyse de consommation de l’API

Et surtout, contrairement aux [filtres du framework MVC](https://docs.microsoft.com/fr-fr/aspnet/core/mvc/controllers/filters?view=aspnetcore-5.0), ils permettent de capturer l’ensemble des exceptions lancées est pas seulement celles du framework MVC.

J’espère que ceux qui ne sont pas encore familier avec vont s’y habituer car on le retrouve le concept dans nombre d’outils du monde .NET (MediatR, MassTransit, NServiceBus pour ne citer que ceux que j’ai eu le plus l’occasion d’utiliser).

Comme à l’accoutumée, un [dépôt GitHub](https://github.com/trucs2dev/le-pipeline-http-en-net) accompagne cet article.

Toute critique est bienvenue et si vous avez des idées de sujet à adresser, elles sont bienvenues.
