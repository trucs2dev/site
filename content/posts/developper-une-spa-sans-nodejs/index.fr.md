+++
date = '2021-03-23T23:58:36+02:00'
draft = true
title = 'Developper Une Spa Sans Nodejs'
tags = ["Dev Xpérience", "Docker", "DevTools"]
categories = ["Development", "Tooling"]
+++

*Note: Méthode obsolète pour les utilisateurs de VS Code.*

Le [dernier article]({{< ref "installer-sonarqube-avec-docker" >}}) a abordé docker d’un point de vue de l’exploitation.

Docker est aussi un outil excellent pour les développeurs.

Une utilisation assez évidente est de provisionner très rapidement les dépendances applicatives (bases de données, RabbitMQ, etc.) sans avoir a les installer sur son poste, et de ce fait la facilité à onboarder d’autres développeurs. Mais une autre, plus méconnue et moins utilisée consiste à utiliser les images docker pour utiliser les logiciels installés dessus.

Aujourd’hui, cet article va traiter du développement d’une application basée sur [create-react-app](https://create-react-app.dev/) sans installer node.js.

L’idée est de créer un Dockerfile (ici, nommé Dockerfile.dev) basé sur nodejs et ciblant un dossier de travail:

```Dockerfile
FROM node:14.16.0WORKDIR /app
```

Ce fichier vas nous servir de base notamment pour initialiser l’application front.

## 1 : Initialisation de l’application

Pour cela, nous devons d’abord le transformer en image locale (ici, appelée sobrement **n**)

```script
docker build . -t n -f Dockerfile.dev
```

Une fois l’image générée, il ne reste alors plus qu’à l’utiliser en prenant bien soin de lier de dossier courant au dossier de travail de l’image et d’appeler la commande `yarn create react-app app`.

```script
docker run --rm -it -v %CD%:/app n yarn create react-app app
````

*On peut noter que je travaille sous windows, avec l'interpréteur CMD.*

## 2 : Gérer les dépendances

La gestion des dépendances est le second cas pour lequel il est nécessaire d’utiliser la commande docker.

Cette fois, c’est un peu plus manuel: au lieu de lancer la commande directement depuis la ligne de commande, nous allons prendre un invité de commande dans le container:

```script
docker run --rm -it -v %CD%/app:/app n /bin/bash
```

Notez comment cette fois, ce n’est pas **%CD%** mais **%CD%/app** qui est utilisé pour représenter le dossier de l’hôte car l’application react n’est pas à la racine mais dans un dossier **app**.

Cela permettra alors par la suite de lancer toutes les commandes nécessaires pour ensuite sortir proprement du container avec un exit.

Ce scénario est le moins optimal : le container étant recréé à chaque lancement, il ne persiste pas de cache npm et va donc devoir re-télécharger nombre d’éléments à chaque utilisation.

## 3 : Développement

Maintenant que l’application est initialisée, il va falloir pouvoir l’utiliser en développement.

Pour cela, l’utilisation d’un fichier docker-compose est tout de même bien plus concise qu’une ligne de commande longue comme le bras:

```yaml
version: "3.9" 
services: 
  dev: 
    container_name: react 
    build: 
      context: . 
      dockerfile: Dockerfile.dev 
    environment: CHOKIDAR_USEPOLLING: "true" 
    ports: 
      - 3000:3000 
    volumes: 
      - ./app:/app 
    command: yarn run start
```

Les éléments importants du fichier sont:

- Il ne récupère pas une image docker connue mais **contient les instructions pour la construire**
- Il expose le port ***3000**
- Il définit la variable d’environnement **CHOKIDAR_USEPOLLING** à la valeur **true** (utilisé pour le hot-reload sous windows)
- Il monte le sous dossier **app** de l'hôte à l’emplacement **/app** du container

Il suffit alors de compiler l’image correspondant au fichier docker-compose

```script
docker-compose build
```

Puis de la consommer

```script
docker-compose run --rm --service-ports dev
```

Pour éviter d’avoir à s’en rappeler, je vous conseille de l’enregistrer dans un script et de démarrer ce script.

Je vous recommande également de créer un script permettant l’accès à un shell pour installer ou modifier les modules npm:

```script
docker-compose run --rm --service-ports dev /bin/bash
```

A partir de là, vous avez un environnement de développement fonctionnel avec lequel je vous encourage à vous amuser et expérimenter en attendant le prochaine article qui décrira une manière de configurer dynamiquement cette image avec des variables d’environnement tel que décrit dans la méthodologie des [12 facteurs](https://12factor.net/fr/config).

Le code source de l’article se trouve dans un dépôt [GitHub](https://github.com/trucs2dev/developper-une-spa-sans-nodejs) dédié.
