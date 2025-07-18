+++
date = '2021-03-03T22:21:40+02:00'
draft = false
title = "Qu'est ce que Docker ?"
tags = ["Container", "Docker"]
categories = ["Tooling"]
+++


Docker est une plateforme qui uniformise les processus de build, de distribution, d’exécution et de gestion des applications. Cela fonctionne grâce aux fonctionnalités de virtualisation du système d’exploitation de l’ordinateur.

Docker est développé en GO et sa première version a été publiée en 2013.

Docker permet d’empaqueter au sein d’une même image une application ainsi que tout son contexte d’exécution (OS, dépendances logicielles, etc.).

Cela permet d’exécuter ce paquet applicatif dans un environnement cloisonné afin d’éviter les conflits de configuration entre plusieurs systèmes. Par exemple, de pouvoir exécuter simultanément plusieurs versions d’un même Framework comme Nodejs 12 et Nodejs 14 ou le fait de pouvoir apporter des configurations très spécifiques par Framework (modifications de php.ini)

![Containerized applications schema](quest-ce-que-docker/docker_isolation-1024x887.png)

## Points d’extensions

« Ok, alors docker, c’est juste un système pour faire tourner une application de manière cloisonnée. Si on ne peut pas l’utiliser pour exposer des services l’intérêt est plutôt limité non? »

Et bien non car même si l’application est cloisonnée, docker permet de se brancher sur différents aspects périphériques.

## Exposer un port

Docker permet d’exposer des ports à l’extérieur. En ligne de commande cela passe par le paramètre **-p**.

La commande

```bash
docker run -p 8080:80 nginx:latest
```

## Monter un volume

Les habitués de l’univers Unix sont probablement familiers de ce processus. Pour les autres, pour faire simple, cela consiste à associer un élément de stockage à un endroit de l’arborescence fichier. Par exemple, il est possible avec docker de « monter » un dossier ou un fichier dans l’arborescence de fichier du container.

Cela se fait avec le paramètre **-v**.

La commande

```bash
docker run -p 8080:80 -v ./my-custom-index.html:/usr/share/nginx/html/index.html
```

remplace le fichier du conteneur à l’emplacement /usr/share/nginx/html/index.html par le fichier de la machine hôte à l’emplacement ./my-custom-index.html.

Le montage est plus souvent associé avec des dossiers entiers, mais il est possible de monter uniquement certains fichiers, un fichier de configuration par exemple.

## Variables d’environnement

Le 3° précepte de la méthodologie [12 facteurs](https://12factor.net/fr/) (la configuration) indique de stocker la configuration dans l’environnement. Cela permet de pouvoir consommer la même image mais dans des contextes sensiblement différents.

Chacun des 3 éléments qui viennent d’être abordés peuvent être configurés pour chaque container

![docker customization schema](quest-ce-que-docker/docker_customization.png)

## Terminologie

- **Dockerfile**: Script décrivant la création et le contenu d’une image.
- **image**: Produite depuis un fichier Dockerfile, elle contient l’état du système à lancer ainsi qu’une application et son environnement. Elle n’a pas d’état et est immutable.
- **conteneur**: instance d’une image. Un conteneur est composé d’une image et d’un environnement d’exécution (volumes, partages de ports, variables d’environnements, etc).

## Cheat sheet

### Gestion des images locales

- Construit à partir d’un fichier *Dockerfile* situé dans le dossier courant et l’enregistre avec **un nom et un tag** dans le store local: docker `build . -t my-image:1.0`
- Liste les images du store local: `docker images` ou `docker image ls`
- Supprime une image précise du store local: `docker image rm my-image:1.0`

### Gestion des images distantes

Une image est stockée sur un registre distant (par défaut, [dockerhub](https://hub.docker.com/)). S’il s’agit d’un autre registre, l’hote et le chemin sont indiqués dans le nom de l’image. Par exemple, les dernieres versions d’elastic search ne sont plus sur dockerhub mais sur un registre privé: docker.elastic.co/elasticsearch/elasticsearch:7.11.1.

- Télécharge une image du registre: `docker pull my-image:1.0`
- Renomme une image avec un nouveau nom et tag localement: `docker tag my-image:1.0 my-registry/my-image:2.0`
- Télécharge une image vers un registre distant: `docker push my-registry/my-image:2.0`

### Gestion de l’exécution

- Démarre un conteneur basé sur une image **nginx** en version **1.18.0-alpine** en tant que **service** en exposant sur le port **5000** de la machine hôte le port **80** du container et en lui donnant le nom **web**: `docker run -d –name=web -p 5000:80 nginx:1.18.0-alpine``
- Arrête un conteneur en envoyant le signal **SIGTERM**: `docker stop web`
- Arrête un conteneur en envoyant le signal **SIGKILL**: `docker kill web`
- Liste les containers en cours d’exécution: `docker ps`
- Liste tous les containers: `docker ps -a`
- Supprime un container: `docker rm web`
- Lance une commande sur un container en cours d’exécution: `docker exec web touch /tmp/a_random_file`
- Ouvre un shell sur un container en cours d’exécution et le lie au terminal ouvert: `docker exec -it web /bin/bash`