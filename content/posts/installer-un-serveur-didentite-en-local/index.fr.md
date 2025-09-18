+++
date = '2021-06-08T00:22:10+02:00'
draft = false
title = 'Installer un serveur d’identité en local'
categories = ["Tech", "Build"]
Tags = ["Authentication", "Docker", "Frontend"]
+++

Le thème du jour : installer un serveur d’identité et faire une application React pour s’y connecter!

## Choix de l’outil

Dans le secteur des fournisseurs d’identité, il y en a une masse assez conséquente.

Plusieurs collègues m’en ont parlé alors je profites de cet article pour le tester: il s’agit de [Keycloack](https://github.com/keycloak/keycloak-containers).

## Installation du serveur

Bon, comme je n’ai pas envie d’installer quoi que ce soit, j’utiliserai docker pour l’application.

En suivant la [documentation](https://hub.docker.com/r/jboss/keycloak), j’ai pu élaborer la commande suivante, qui me permet d’instancier un container exposé sur le port 8080 et sur lequel je peux m’authentifier avec les identifiants admin / admin:

```shell
docker run --name keycloack -d -p 8080:8080 -e KEYCLOAK_USER=admin -e KEYCLOAK_PASSWORD=admin jboss/keycloak:13.0.1
```

Un clic sur le lien « Administration Console » nous amène alors à la page d’authentification:

![Page d'authentification de Keycloak](installer-un-serveur-didentite-en-local/image-79.png)

## Interface d’administration

le menu indique que nous sommes sur le realm (tenant) « Master ».

Il se décompose en 2 parties: l’une dédiée à la configuration du tenant et l’autre à la gestion des utilisateurs du tenant.

![Menu de Keycloak](installer-un-serveur-didentite-en-local/image-80.png)

On ne vas pas explorer les autres tenants, mais simplement utiliser le tenant « master ».

Un appel au endpoint « **.well-known** » permet de valider si keycloack est actif.

L’url de base de ces endpoints des différents realms keycloack est de la forme suivante :

`http://{host:port}/auth/realms/{realm}/.well-known/openid-configuration`

## Configuration du serveur

A l’aide d’une application SPA (React), nous allons créer un client pour que l’application puisse s’authentifier au serveur d’identité. Celle-ci sera exposée par défaut sur le port **5173**.

Il faut ensuite créer un client adéquat sur keycloack:

![Ajout de client sur Keycloak](installer-un-serveur-didentite-en-local/image-83.png)

## Application cliente

Tout d’abord, utilisons [Vite](https://vitejs.dev/) pour créer une application react-typescript:

```shell
npm create vite@latest .
```

Il faut ensuite installer les librairie [oidc-client](https://www.npmjs.com/package/oidc-client) pour l’authentification et [react-router](https://reactrouter.com/) pour le routage de l’application:

```shell
npm i oidc-client-ts react-router-dom
```

L’intégralité du code source de l’application cliente est dans un dépôt GitHub dont le lien est présent à la fin de l’article, mais les points clés sont les suivants:

En terme de composant, il faudra 3 pages :

- Une racine, la page d’accueil
- Une pour le retour d’authentification (/auth/signin)
- Une pour le retour de dé-authentification (/auth/signout)

Il faudra aussi un objet de type UserManager instancié et prêt à répondre:

![Configuration React](installer-un-serveur-didentite-en-local/image-100.png)

Rien de compliqué ici, on spécifie l’URL racine du serveur et les différents endpoints seront retrouvés depuis le document « .well-known ».

On précise les URLS de redirection après login & après logout, le client_id (tel qu’entré dans keycloack).

Ensuite, côté callbacks, on appelle la méthode adéquate et on renvoie vers la page racine:

![Sign in callback](installer-un-serveur-didentite-en-local/image-86.png)

La seule différence avec le log out étant la méthode appelée:

![Sign out callback](installer-un-serveur-didentite-en-local/image-87.png)

Enfin, au niveau de la homepage, on affiche un contenu différent selon que l’on soit authentifié ou non:

![Homepage conditionnelle](installer-un-serveur-didentite-en-local/image-88.png)

Contenu qui va afficher les informations de l’utilisateur connecté et lui présenter les méthodes qui le concerne : s’authentifier ou se dé-authentifier.

![Différentes homepages](installer-un-serveur-didentite-en-local/image-89.png)

## Conclusion

OpenID Connect, au final, c’est plutôt facile à mettre en place sur un environnement de développement. La norme est bien ficelée et que la librairie frontend utilisée est simple à manipuler.

Je n’ai pas traité les refresh tokens, si vous voulez un article dessus, dites le moi en commentaire.

Le code source du client est sur [Github](https://github.com/trucs2dev/installer-un-serveur-didentite-en-local/).

A bientôt pour voir comment on utilise ça côté serveur !
