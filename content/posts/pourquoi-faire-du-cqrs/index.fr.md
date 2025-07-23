+++
date = '2021-05-04T23:26:40+02:00'
draft = false
title = 'Pourquoi Faire Du Cqrs'
categories = [ "Tech" ]
tags = ["NET", "Architecture", "CQRS"]
+++

Depuis une dizaine d’années, une alternative à la conception n-tiers a émergée: CQRS.

Pendant cette période de temps, j’ai justement eu l’opportunité de mettre en oeuvre plus d’une vingtaines d’applications basées sur CQRS.

## CQRS, c’est quoi ?

CQRS, c’est l'acronyme de **C**ommand **Q**uery **R**esponsibility **S**egregation. C’est un design pattern qui pousse un cran plus loin le concept **CQS** de [Bertrand Meyers](https://fr.wikipedia.org/wiki/Séparation_commande-requête).

**CQS**, (Command Query Separation) c’est un concept simple: ça stipule que chaque méthode peut être:

- Soit une Commande (**Command**) qui effectue une action
- Soit une Requête (**Query**) qui renvoie des données à l’appelant

Mais pas les 2 à la fois:

> Lire le journal ne devrait pas changer son contenu.

CQS permet d’isoler les méthodes à effet de bord des autres; une information précieuse pour identifier des bugs de données. Il permet aussi une [séparation des préoccupations](https://fr.wikipedia.org/wiki/Séparation_des_préoccupations) décente et encourage à créer un code davantage [SOLID](https://cleancoders.com/series/clean-code/solid-principles).

Enfin, CQS est très facile à mettre en oeuvre sur tous types d’architectures.

Le CQRS pousse le paradigme un cran plus loin en créant des objets autonomes dédiés aux Queries & Commands.

Le pattern repose sur 4 objets:

- Une Dto d’entrée (la Query ou Command)
- Une Dto de retour (pour les queries)
- Un objet de traitement (QueryHandler ou CommandHandler), où est situé le code qui fait quelque chose.
- Un objet sachant comment instancier et appeler un handler pour une query ou command donnée

L’utilisation du framework [MediatR](https://github.com/jbogard/MediatR), qui implémente la tuyauterie, fournit dores et déjà l’interface IMediator dont la fonction est d’assurer la redirection.

En prenant l’exemple d’une Query permettant la récupération d’un utilisateur par son identifiant, cela peut se présenter de la manière suivante:

### La consommation dans le controller REST

![Illustration de l'appel d'une query dans un controleur REST](pourquoi-faire-du-cqrs/image-41.png)

La Query (DTO d’entrée qui spécifie aussi le type de retour)

![Implémentation d'une query](pourquoi-faire-du-cqrs/image-42.png)

Le DTO de retour

![Implémentation d'un dto de retour](pourquoi-faire-du-cqrs/image-43.png)

Le QueryHandler

![Implémentation d'un query handler](pourquoi-faire-du-cqrs/image-44.png)

Ma compréhension actuelle du [Single Responsibility Principle](https://blog.cleancoder.com/uncle-bob/2014/05/08/SingleReponsibilityPrinciple.html) est de mettre ensemble les choses qui ont les mêmes raisons de changer. Aussi j’ai tendance à :

- Mettre **dans le même fichier** les classes qui ont des raisons de changer ensemble, à savoir: `query`, `result`, `handler` et quand il y en a `validateur`
- Encapsuler en tant que Classe Imbriquée la DTO de retour (cela me permet de ne pas perdre de temps sur les noms des objets ou a suffixer QueryXXXResult, l’imbrication de types m’assurant qu’il n’y aura pas 2 classes Dto identitiques)

![Les classes, ensemble](pourquoi-faire-du-cqrs/image-45.png)

## Les avantages par rapport au n-tiers

Les 2 principaux avantages sont la **maintenabilité** et l’**évolutivité**.

### La maintenabilité

Parce qu’on a différentes **unités de travail complètes** pour modifier le système (les Commands & leurs handlers) et que l’objectif est qu’une méthode de contrôleur re-route simplement l’instruction au handler (pas d’appels composés à différentes commandes).

En cas de 2 méthodes qui sont « *presque les mêmes sauf […]* » (ce qui correspond à 2 raisons de changer différentes d’un point de vue métier), ce sera donc 2 commandes qui seront créées; chacune pouvant évoluer indépendamment de l’autre et n’impactant pas l’autre par sa complexité / ses spécificités fonctionnelles.

D'un point de vue lisibilité, c'est simple : le handler a une méthode `Execute` ou `Handle` qui sert de point d'entrée. Tout le reste de la classe peut etre utilisé pour extraire des méthodes et donner de la lisibilité au code actuel.

### L’évolutivité

Car les besoins du système et les attentes des utilisateurs sont différents pour la lecture et l’écriture.

En écriture, l’utilisateur a conscience que les opérations peuvent **prendre du temps** mais ont des attentes fortes sur la **cohérence des données** (pas de commande payées mais perdue par exemple). Le système aura donc des besoins **transactionnels**.

En lecture, un utilisateur s’attend à une **réponse rapide** et donc, le système peut souvent être consistent à terme et n’a pas besoin de transaction : la donnée ne doit pas être manipulée.

CQRS permet de dissocier ces 2 modèles de fonctionnement et permet même dans des utilisations avancées d’avoir 2 modèles de stockage de données différents (Par exemple, un moteur d’[event sourcing](https://www.eventstore.com/) pour l’écriture et une base [documentaire no-sql](https://www.elastic.co/fr/elasticsearch/) pour la lecture.

## C'est compliqué ?

Non, l’illusion de la complexité vient surtout dans le fait de séparer le modèle de lecture du modèle d’écriture, dans des stores différents.

La raison de cette séparation n’est pas liée au CQRS; elle est liée au fait d’avoir une charge (réelle ou projetée) qui le justifie. Et dans ce cas, le CQRS est une solution viable et cadrée (probablement moins complexe qu’une solution maison empirique).

Si la charge ne le justifie pas, il est tout à fait possible (et viable) de n’avoir qu’un seul modèle de données exposé via une base de données et de le consommer par l’intermédiarie des différentes queries et commands.

Dans ce fonctionnement, CQRS apporte le gain en flexibilité car en cas de charge plus importante, il reste possible de mettre en oeuvre une [réplication transactionnelle](https://docs.microsoft.com/fr-fr/sql/relational-databases/replication/transactional/transactional-replication) et de consommer les répliques lors des opérations de lecture.

## Et vous, vous en pensez quoi ?

Pour travailler depuis plus de 7 ans avec ce modèle et avec plus d’une vingtaine d’applications, je cherche toujours un inconvénient à ce modèle de programmation. Quel est votre point de vue ?

- Avez vous déjà eu l’opportunité de faire du CQRS ?
- Quelle expérience en avez vous retiré ?
- Quels challenges avez vous du relever ?

(Et comme d’habitude, le code source est disponible sous [GitHub](https://github.com/trucs2dev/pourquoi-faire-du-cqrs) même si pour cet article, il ne sert qu’à présenter MediatR)
