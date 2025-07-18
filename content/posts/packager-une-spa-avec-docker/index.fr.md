+++
date = '2021-03-30T00:22:10+02:00'
draft = false
title = 'Packager Une SPA Avec Docker'
categories = ["Development", "Build"]
Tags = ["Docker", "Development", "Exploitation"]
+++

*Update du 03/02/2023 : Le code mis à disposition dans le dépôt github nécessite quelques ajustements pour fonctionner.*

Dans la continuité de la série docker, cet article présente une manière de d’exposer une application de type SPA dans une image docker et de pouvoir paramétrer certains aspects de cette image à travers des variables d’environnement.

Tout d’abord posons nous la question « Que signifie empaqueter une application ? ».

Nous l’avons vu dans le [1° article de la série]({{< ref "quest-ce-que-docker" >}}): une image docker contient une application et son environnement d’exécution.

Dans le cadre d’une application Web purement statique, cela consiste alors « simplement » a exposer l’application packagée.

Pour cela, nous pouvons réaliser une [image « multi-stage »](https://docs.docker.com/develop/develop-images/multistage-build/) afin de déléguer à docker la construction du build dans une étape et son exploitation dans l’étape finale.

Pour exposer l’application, une solution évidente serait l’utilisation de Nginx pour l’exposer.

Cependant, souvent les SPA requièrent une redirection vers le fichier index en cas de page non trouvée. Et quand bien même [la configuration n’est pas très complexe](https://stackoverflow.com/a/19489441), pour des besoins à venir plus tard dans l’article, nous allons nous appuyer sur un serveur [express js](https://expressjs.com/en/starter/static-files.html).

Pour cela, créons un dossier « server » à la base de notre application, et initialisons le avec la commande suivante:

```script
npm init -y
```

Nous installons ensuite les dépendances requises:

```script
npm i -S express compression
```

Enfin, il ne reste plus qu’à créer un fichier index.js qui va configurer le serveur:

```javascript
const express = require("express"); 
const compression = require("compression"); 
const path = require("path"); 
const fs = require("fs");

// Conserve le fichier index en mémoire pour limiter les I/O. 
const indexPath = path.resolve(__dirname, "static_files", "index.html"); 
const indexPromise = fs.promises.readFile(indexPath, "utf-8"); 
const app = express(); 

// supprimer un header qui donne trop d'informations 
app.disable("x-powered-by"); 
// active la compression 
app.use(compression()); 
// expose l'application web statique 
app.use(express.static("static_files")); 

// La route vers les 404, à déclarer en dernier, elle renvoie le contenu du fichier index.html 
/app.get("*", (req, res) => { indexPromise.then((file) => { res.send(file); }); }); 

// démarre l'écoute du serveur sur le port 4000 
const instance = app.listen(4000, () => { 
    console.log("application started"); 
    // gère l'arrêt propre de l'application 
    process.on("SIGINT", () => { 
        instance.close(() => console.log("application ended"));
        });
    });
```

Le fichier ci-dessus permet d’exposer les fichiers statiques du dossier « static_files » et dans le cas où l’un d’eux ne serait pas trouvé, de renvoyer le contenu du fichier « static_files/index.html »

Un test rapide avec le en créant ces dossiers / fichiers permettent de valider le bon fonctionnement de l’application:

![les fichiers en question](packager-une-spa-avec-docker/image.png)

Mission accomplie pour le serveur, maintenant, plus qu’à passer à l’image docker:

```Dockerfile
# Image de "compilation" de l'application SPA. 
# Tout ce qui se passe ici ne fera pas partie de l'image finale 
FROM node:14.16.0 as build 
WORKDIR /app COPY app /app 
RUN yarn install --frozen-lockfile && yarn run build 

# Les fichiers produits sont dans le dossier /app/build de l'étape "build" 
# Image d'exploitation de l'image SPA 
FROM node:14.16.0-alpine3.13 
EXPOSE 4000 
# Une bonne pratique des images docker consiste à lancer les images avec des utilisateurs aux droits limités 
USER node WORKDIR /home/node/app 
COPY server /home/node/app RUN npm ci 
COPY --from=build --chown=node:node /app/build /home/node/app/static_files 

ENTRYPOINT ["node", "index.js"]
```

Ici, on utilise d’abord une première étape nommée **build** dans laquelle on package l’application SPA.

Ensuite, la seconde étape, utilisant un autre utilisateur que root, copie ces fichiers et les consommes à travers son serveur.

**A noter :** seul le dernier layer est uploadé en tant qu’image docker. Les images intermédiaires (ici, celle de build) ne font pas partie de ce qui est uploadé sur le registre docker.

## Comment gérer ses environnements ?

Le [3° aspect de la méthodologie 12 facteurs](https://12factor.net/fr/config) recommande l’utilisation de variables d’environnement pour stocker la configuration de l’application.

Cette remarque est parfaitement cohérente dans le cadre d’applications exécutées au sein du container, cependant, une application de type SPA s’exécute dans le navigateur:

![Comment  faire ?](packager-une-spa-avec-docker/undraw_Questions_re_1fy7-1024x713.png)

La question qui se pose alors est « Comment faire en sorte que la SPA, **depuis le navigateur de l’utilisateur** ait accès à une configuration personnalisée depuis les **variables d’environnement du serveur** ?

C’est là que le choix de nodejs en tant que serveur web prend toute sa cohérence.

## Du point de vue de la SPA

Introduisons un fichier de configuration. Par exemple : configuration.js

Celui-ci sera présent dans le dossier **public** de l’application React.

![fichier configuration.js dans l'applicaiton react](packager-une-spa-avec-docker/image-2.png)

et enregistre au sein de l’objet global une variable nommée arbitrairement (__APPCONFIGURATION__) qui contient les clés de configuration de l’application.

```javascript
(function () {
    window.__APPCONFIGURATION__ = { message: "Configuration statique", }; 
})();

```

Modifions ensuite le fichier index.html du dossier public pour qu’il charge ce fichier en premier:

![fichier configuration.js est chargé en premier](packager-une-spa-avec-docker/image-3.png)

Ensuite, nous allons consommer ce paramètre dans le composant `App`.

![fichier configuration.js est chargé en premier](packager-une-spa-avec-docker/image-4.png)

Nous constatons que le message affiché.

![Valeurs de configuration en plane](packager-une-spa-avec-docker/image-5.png)

## Du point de vue du serveur express

Ici, nous allons simplement modifier le fichier index.js de manière à ce qu’il génère dynamiquement le fichier configuration.js à chaque démarrage de l’application.

Pour cela, nous allons nous appuyer sur le module npm read-env qui est capable de produire un objet à partir de variables d’environnement.

```script
npm i -S read-env
````

Décidons arbitrairement que toutes les variables d’environnement préfixées par APP_ seront des variables de configuration.

Nous allons décrire dans le fichier Dockerfile la variable APP_MESSAGE avec une valeur par défaut. Comme les intructions **EXPOSE**, les instructions **ENV** ne sont pas obligatoires mais elles donnent de la lisibilité au fichier et fournissent aussi des valeurs par défaut:

![Illustration de la configuraiton d'une variable d'environnement avec un message par défaut](packager-une-spa-avec-docker/image-6.png)

Ensuite, allons modifier le fichier index.js du serveur (après la route 404) de la manière suivante:

```javascript
// La route vers les 404, à déclarer en dernier, elle renvoie le contenu du fichier index.html 
app.get("*", (req, res) => { 
    indexPromise.then((file) => { 
        res.send(file); 
    }); 
}); 

start(app); 

async function start(expressApp) { 
    await createConfiguration(); 
    // démarre l'écoute du serveur sur le port 4000 
    const instance = app.listen(4000, () => { 
        console.log("application started"); 
        // handle properly CTRL+C 
        process.on("SIGINT", () => { 
            instance.close(() => console.log("application ended")); 
        });
    }); 
    
    async function createConfiguration() { 
        const options = readEnv("APP"); 
        const fileContent = `(function () { 
            window.__APPCONFIGURATION__ = ${JSON.stringify(options, null, 4)};
        })();`; 
        
        await fs.promises.writeFile("static_files/configuration.js", fileContent, err => { 
            if (err) { 
                console.error("Unable to overwrite configuration.js file"); 
                process.exit(-1); 
            ‡} 
        });
    }
}
```

De cette manière, la configuration est crée à chaque démarrage de l’application et copiée dans le dossier static_files pour écraser le fichier existante et laisser à express le loisir de gérer son cache de fichiers statiques.

A présent, recompilons l’application:

```script
docker build . -t test
```

```script
docker run -p 4000:4000 --rm -it test
```

Nous pouvons constater que le message par défaut de l’image docker **Message de l’image docker** est bien affiché à l’écran:

![Affichage du bon message à l'écran](packager-une-spa-avec-docker/image-7.png)

Second test, cette fois-ci, lançons la même image en personnalisant la variable d’environnement:

```script
docker run -p 4000:4000 --rm -it -e APP_MESSAGE="Message de la ligne de commande" test
```

Cette fois, nous constatons que le message à été remplacé par **celui indiqué dans la ligne de commande**:

![Affichage du nouveau message à l'écran](packager-une-spa-avec-docker/image-8.png)

## Conclusion

En utilisant l’écosystème applicatif nodejs, il est possible et même aisé de pouvoir définir la configuration d’une SPA packagée dans une image docker en fonction des variables d’environnement spécifiées au runtime.

Cette astuce est particulièrement utile pour définir des clés de configuration à consommer (url d’une api, clientid/clientsecret pour l’authentification, API key, etc.) selon l’environnement d’exploitation (dev/staging/prod).

Cela permet de conserver et de réaliser les tests sur la même image tout au long du processus de validation et garantit mécaniquement que l’image validée est celle qui va être poussée.

Le code source (non-fonctionnel) de cet article est disponible sur [GitHub](https://github.com/trucs2dev/packager-une-spa-avec-docker).
