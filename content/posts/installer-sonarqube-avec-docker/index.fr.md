+++
date = '2021-03-16T22:23:09+02:00'
draft = false
title = "Installer SonarQube avec Docker"
tags = ["Container", "Docker", "Devtool"]
categories = ["Tooling", "Quality"]
+++

Docker est un outil d’une versatilité exceptionnelle. Ce post illustre l’utilisation de Docker dans un contexte d’exploitation sans nécessairement passer par des orchestrateurs.  

Votre entreprise a décidé d’améliorer sa qualité et c’est bien connu, **on ne peut améliorer que ce qu’on mesure**, donc, il vous incombe maintenant de mettre en place SonarQube pour l’équipe de développement. Seul élément à votre disposition: un serveur.

SonarQube n’a pas besoin de beaucoup de [dépendances](https://docs.sonarqube.org/latest/requirements/requirements/): seulement une JVM et une base de données. Ceci dit on ne sait pas ce qui tourne déjà ou peut tourner à l’avenir sur ce serveur. Pour assurer un cloisonnement des applications, nous allons donc utiliser docker et plus spécialement docker-compose pour héberger SonarQube et sa base de données.

L’objectif est que la base de données en question ne puisse être accédée que par l’instance de SonarQube mais que les volumes liés aux données persistantes puissent être accessibles pour des backups quotidiens.

![SonarQube exécuté par docker](installer-sonarqube-avec-docker/Sonarqube_Docker.png)

## Prérequis

Tout d’abord avant de les consommer, nous allons créer les volumes. Les volumes ont besoin d’être accédés depuis l’extérieurs donc ne doivent pas être portés par le fichier docker-compose.yml mais référencés par celui ci.

Pour créer un volume sous docker, rien de plus simple, aussi créons les volumes requis :

```bash
docker volume create sonarqube_confdocker volume create sonarqube_datadocker volume create sonarqube_logsdocker volume create sonarqube_extensionsdocker volume create postgres_data
```

## Configuration de la machine hôte

Toute image utilisant ElasticSearch requiert une [configuration de la machine hôte](https://www.elastic.co/guide/en/elasticsearch/reference/current/vm-max-map-count.html).

Cela se fait depuis un shell linux avec la ligne de commande suivante:

```bash
sysctl -w vm.max_map_count=262144
```

Si vous utilisez Docker Desktop sous windows avec l’intégration WSL, la commande suivante permet de démarrer un shell sur la machine WSL exécutant docker à partir duquel vous pourrez appliquer la configuration:

```bash
wsl -d docker-desktop
```

## Docker–compose

Docker-compose est un format de fichier permettant de décrire plusieurs containers et les positionnant tous dans le même sous-réseau par défaut.

Voici donc ce à quoi ressemble le fichier:

```yaml
version: "3.8" 
services: 
  db: 
    image: postgres:9.6.21-alpine 
    env_file: db.env 
    volumes: 
      - dbdata:/var/lib/postgresql/data 
  sonarqube: 
    depends_on: 
      - db 
    image: sonarqube:8.7.1-community 
    env_file: sonarqube.env 
    ports: 
      - 9000:9000 
    volumes: 
      - conf:/opt/sonarqube/conf 
      - data:/opt/sonarqube/data 
      - extensions:/opt/sonarqube/extensions 
      - logs:/opt/sonarqube/logs 

volumes: 
  dbdata: 
    name: postgres_data 
    external: true 
  conf: 
    name: sonarqube_conf 
    external: true 
  data: 
    name: sonarqube_data 
    external: true 
  extensions: 
    name: sonarqube_extensions 
    external: true 
  logs: 
    name: sonarqube_logs 
    external: true
```

Cela fait beaucoup a appréhender d’un coup, alors, voici quelques clés de décodage:

- Le fichier est découpé en 2 sections: les **services** et les volumes**
- Les services décrivent avec quelle image ils travaillent et comment elle est configurée (variables d’environnement, volumes, ports)
- Les volumes décrivent l’ensemble des éléments de stockage utilisés dans le fichier docker-compose (ici, nous avons utilisés les volumes externes créés précédemment).
- Chaque service peut être résolu par les autres services décrits dans le fichier par son nom (par exemple, le container sonarqube peut accéder au container db en utilisant comme nom « db »).

Pour éviter de trop surcharger le fichier, les variables d’environnement ont été crées dans 2 fichiers séparés: `db.env` et `sonarqube.env`.

Lançons docker-compose et l’on peut voir que les containers tournent et que sonarqube est accessibles sur le port 9000 comme décrit dans le fichier.

## Script de sauvegarde

Si vous avez suivi jusque là, c’est là ou docker va réellement briller:

Pour sauvegarder, faire des manipulations de fichiers, et autres, il est possible de simplement utiliser docker. Ainsi, un script de backup aura la forme suivante:

`backup.ps1`

```powershell
# Arrête les containers 
docker-compose stop 

# Crée les dossiers où sauvegarder les backups 
md backups -Force 

# définit la date actuelle 
$now = date -Format "yyyy-MM-dd" 

# Utilise docker pour créer les backups 
docker run --rm -v postgres_data:/mnt/data -v $PWD/backups:/mnt/target busybox:1.33.0-uclibc tar -zcf /mnt/target/backup-db-$now.tar.gz /mnt/data/ docker run --rm -v sonarqube_data:/mnt/data -v $PWD/backups:/mnt/target busybox:1.33.0-uclibc tar -zcf /mnt/target/backup-sq-$now.tar.gz /mnt/data/ 

# Redémarre les containers 
docker-compose start

```

On utilise une image minimale (**busybox:1.33.0-uclibc**) à laquelle nous faisons exécuter une commande **tar** pour archiver le contenu du dossier. Pour sauvegarder les données sur le disque, nous **lions les dossier source et cible** utilisés par la commande sur le container à l’aide de l’instruction **-v**.

l’instruction **–rm** permet de rendre le container éphémère en le supprimant dès qu’il a fini de s’exécuter.

Plus qu’à placer ce script dans un job cron et nous avons un backup périodique fonctionnel.

Il est évidemment possible d’aller plus loin dans la logique de backup en s’assurant par exemple de ne conserver que les « N » derniers backups mais ce n’est pas le sujet de cet article.

J’espère que cet article vous aura fait découvrir une autre utilisation de docker. Ce cas d’utilisation est un moyen de mettre le pied à l’étrier et d’apprendre à moindre impact à utiliser et exploiter des containers et les utiliser pour la sauvegarde / restauration des données.

Le code source de l’article se trouve dans un dépôt [GitHub](https://github.com/trucs2dev/installer-sonarqube-avec-docker) dédié.
