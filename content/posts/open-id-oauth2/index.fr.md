+++
date = '2021-06-01T00:22:10+02:00'
draft = false
title = 'Open ID / OAuth2'
categories = ["Development", "Build"]
Tags = ["Authentication", "Basis", "Theory", "API"]
+++

Suite aux [différents]({{<ref exposition-rest-en-net>}}) [articles]({{<ref le-pipeline-http-en-net>}}) [permettant]({{<ref decouverte-de-hotchocolate-partie-1>}}) d’[exposer une api]({{<ref decouverte-de-hotchocolate-partie-2>}}), j’ai besoin d’introduire un pré-requis pour la gestion des accès: la norme Open ID (elle même, très largement appuyée sur Oauth2).

Un article un peu académique pour présenter la norme OAuth2, son application dans le contexte d’un couple applicatif SPA / API et de ce que vient ajouter la couche OpenID Connect.

## OAuth2, c’est quoi ?

Oauth2, c’est un protocole de sécurité utilisé par la grande majorité des acteurs du Web, notamment les GAFAM. Il permet la fédération et la délégation d’identité.

### La fédération d’identité

L’objectif de la fédération d’identité est qu’un même utilisateur puisse naviguer entre différents services après s’être authentifié une seule fois auprès d’un tiers de confiance garant de son identité: Le **fournisseur d’identité** (**IDP**).

Dans le flux, l’utilisateur s’authentifie auprès du fournisseur d’identité et obtient un **jeton d’authentification** (**access_token**). Ensuite, il utilise ce jeton auprès des différentes APIS avec qui il communique. Le diagramme suivant illustre ce fonctionnement:

![Flux de fonctionnement Oauth](open-id-oauth2/image-130.png)

### La délégation d’identité

C’est le fait qu’un IdP puisse reposer sur un autre pour les problématiques d’authentification  et d’identification. C’est le fameux « se connecter avec XXX » que l’on peut retrouver sur différents services:

![Délégation d'identité](open-id-oauth2/image-131.png)

## Et OpenId Connect ?

OpenId Connect s’appuie sur Oauth2 et l’étend en lui fournissant un nouveau type de jeton : le **jeton d’identité** (**id_token**). Celui-ci a pour vocation d’être utilisé côté frontend et d’embarquer avec lui les informations personnelles de l’utilisateur

## Les concepts

### Les clients

Quand on parle de client en OAuth, il ne s’agit pas d’un utilisateur mais d’une application. Beaucoup de paramétrage est propre à l’application qui va interroger les jetons, le client est l’endroit où il sera stocké. Selon les IdPs il sera nommé « client » ou « application ».

### Les scopes

Sous OAuth 2.0, le scope est un méchanisme qui limite l’accès à une application à un utilisateur. Une application peu tdemander un ou plusieurs scopes. Cette information est présentée à l’utilisateur sur la page de consentement et l’access token émis à l’application sera limité aux scopes alloués. [Oauth2](https://www.oauth.com/oauth2-servers/scope/defining-scopes/) ne définit aucune valeur particulière pour les scopes (car intrinsèquement liés à l’architecture interne des différents services). Open Id quand à lui permet d’[utiliser les scopes pour obtenir des claims](https://openid.net/specs/openid-connect-core-1_0.html#ScopeClaims) et en fournit une certaine liste par défaut (que les différents IdPs peuvent enrichier).

### Les claims

Un claim est une paire clé/valeur qui contient une information relative à l’utilisateur. Ils sont stockés dans les jetons « self-encoded ». Plus d’informations sur les claims dans le cadre d’un jeton JWT [ici](https://datatracker.ietf.org/doc/html/rfc7523).

### Différents jetons

| TOKEN	| ROLE | TYPE |
| - | - | - |
| access_token | Authentifier l’utilisateur auprès d’une Application web | Self encoded / Opaque |
| id_token | Fournir des informations sur l’utilisateur connecté | JWT |
| refresh_token | Renouveller les accés de l’utilisateur | Self encoded / Opaque |

#### Types de jetons

##### Self encoded

Les jetons « [self encoded](https://www.oauth.com/oauth2-servers/access-tokens/self-encoded-access-tokens/) » portent les informations en leur sein. Ils sont autonomes et portent un certain nombre d’informations sous la forme paire clé-valeur. Bien que la norme ne le [spécifie pas](https://datatracker.ietf.org/doc/html/rfc6749#section-1.4), le format [JWT](https://datatracker.ietf.org/doc/html/rfc7519) est communément utilisé car il permet de garantir par un jeu de clé privé / publique que l’émetteur est bien celui qui l’a encodé.

##### Opaque

Un jeton opaque n’est pas déchiffrable côté client. Ils s’agit souvent d’un [Universally Unique Identifier](https://fr.wikipedia.org/wiki/Universally_unique_identifier) qui doit être transmis au serveur d’identité pour le lire.

##### JWT

Il s’agit d’un self encoded token utilisant la méthode JWT. Les spécifications OpenId 1.0 décrivent que les [informations relatives à l’authentification effectuée est stockée dans  un jeton JWT nommé id_token](https://openid.net/specs/openid-connect-core-1_0.html#Introduction).

### Les flux

Selon le cas d’utilisation, OAuth2 propose différent flux de fonctionnement. Nous allons voir les plus commun:

#### Authorization Code (Frontent)

A date de rédaction, le flux le plus communément utilisé pour authentifier un utilisateur est le flux « Authorization Code flow with Proof Key for Code Exchange (PKCE) » (qui se prononce « pixi »).

Ce flux est le suivant:

![Authorization code flow](open-id-oauth2/image-132.png)

#### Renouvellement de jeton (Frontend)

Un flux assez simple qui peut se déclencher soit périodiquement (lorsque l’obtention de l’access_token [indique aussi sa durée de vie(https://datatracker.ietf.org/doc/html/rfc6749#section-5.1)]) ou par un mécanisme d’interception des requêtes http combiné à la présence dans un storage (local, session, etc.) d’un refresh_token lors de la réception d’un code [HTTP 401](https://developer.mozilla.org/fr/docs/Web/HTTP/Status/401).

![Informations sur les tokens dans le local storage](open-id-oauth2/image-133.png)

#### Introspection (Backend)

Lors de l’utilisation d’access_token opaques, il est nécessaire pour le backend d’appeler une URL dite d’[introspection](https://datatracker.ietf.org/doc/html/rfc7662) pour valider les jetons d’accès fourni par le client. Cela se fait de la manière suivante:

![Séquence d'introspection](open-id-oauth2/image-134.png)

#### User infos (Frontend / Backend)

Ce endpoint est apparu avec OpenId Connect, il s’agit du endpoint [UserInfo](https://openid.net/specs/openid-connect-core-1_0.html#UserInfo). Son rôle est de renvoyer les informations utilisateur lorsqu’on l’appelle. Il peut être appelé par le client ou le serveur (par exemple pour ajouter les informations d’un utilisateur qui souhaite synchroniser son profil à l’application).

#### Révocation du jeton / Logout (Frontend)

La norme prévoit la possibilité de [révoquer un jeton](https://datatracker.ietf.org/doc/html/rfc7009) s’il ne doit plus être utilisé et prévoit l’appel à une URL de callback de manière à ce qu’un traitement soit fait dans l’application après la révocation:

![Séquence de révocation](open-id-oauth2/image-135.png)

## Conclusion

Un article bien plus théorique que d’habitude, mais nécessaire avant de voir l’implémentation avec différents providers et frameworks.

