+++
date = '2023-02-21T00:22:10+02:00'
draft = false
title = 'Pourquoi une application ça coute aussi cher ?'
categories = ["Billet d'humeur"]
Tags = ["Billet d'humeur"]
+++

Il y quelques années de ca, alors que j’ai pu développer une application dans le domaine de l’extraction de minerai, mon entreprise à connu une friction avec son client.

Selon lui, le prix payé n’était pas en adéquation avec la prestation fournie.

Un de ses représentant m’a alors posé cette question (j’étais responsable technique):

{{< admonition type=quote open=true >}}

Mais pourquoi ça coute aussi cher ?

Il y a 10~20 ans, c’était bien moins cher et long, et le métier était aussi complexe qu'aujourd'hui.

{{< /admonition >}}

## Petit retour dans le temps

La plus ancienne application que j’ai vue a été lors de mon passage à IBM.

J’étais alors magasinier et on se servait d’une application en mode terminal.

Pas seulement une application console, non. Mais une application qui était ouverte dans un terminal et qui envoyait les commandes au serveur pour inventorier les produits sur une palette par exemple.

L’application était couplée à un lecteur de code barre et était d’une vitesse folle!

Coté interface, on avait quelque chose qui ressemblait un peu à Vim. En moins user-friendly:

![Une application en mode terminal](pourquoi-une-application-ca-coute-aussi-cher/image-138.png)

Tout se passait coté serveur, l’utilisateur ne pouvait que rentrer des informations de manière manuelle.

Les plus:

- Coté rapidité, je n’ai jamais rencontré mieux dans le scénario nominal (ajout de produits)
- En terme d’empreinte mémoire, je penses qu’on était au top aussi (je n’ai pas mesuré à l’époque)
- Etat géré côté serveur

Les moins:

- Forte courbe d’apprentissage : les commandes devaient être connues par cœur (par facilité, on en avait imprimé certaines sur code-barre)
- Besoin critique du réseau
- Je suis assez dubitatif quand à la sécurité pour le transit des messages
- Pas d’interopérabilité possible depuis un autre client sans devoir implémenter jusqu’au protocole

J’ignore s’il s’agissait d’une contrainte de réalisation, mais l’application était conçue pour fonctionner en transactions : 1 utilisateur entre une liste de produit pour modéliser un déplacement de stock et valide cette opération.

## Retour dans le présent

Nous revoilà maintenant entre 2014 et 2020, il est alors impensable de réaliser une telle application si une demande explicite n’a pas été réalisée.
D’autant plus qu’il s’agit d’une application web.

Qui plus est, les écrans et la manière dont ils doivent fonctionner sont opérés par le client, plus d’aspect « grosse transaction » possible, d’autant plus que les pratiques sont que si quoi que ce soit est en erreur, l’utilisateur doit le voir le plus vite possible.

On est face ici à des écrans type ERP, mais farcis de règles métier et d’interactions avec d’autres parties de l’application.

Ce qui fait qu’on prends plus de temps aujourd’hui ? (liste non exhaustive)

### Frontend

- S’assurer de la bonne prise en compte, de manière transverse, de l’authentification et de la gestion des droits
- Permettre à l’utilisateur d’enregistrer certains écrans en favori de leur navigateur (ajout produit / etc) et pouvoir envoyer les liens à d’autres utilisateurs (détail - d’un produit par ex)
- Permettre de faire des recherches / tris multicritères (à l’époque, les implémentations d’odata n’étaient pas suffisantes)
- Optimiser la vitesse de chargement des listes (pagination)
- Proposer certaines actions au niveau des écrans, toujours en fonction des droits de l’utilisateur (consulter / modifier / supprimer)
- Valider les entrées utilisateur et afficher de manière intelligible les erreurs métier renvoyées
- Il ne s'agit ici que des éléments les plus basiques qui me viennent à l’esprit.

### Backend:

- Forcer l’authentification par défaut lors des appels
- [Gérer le fait que les erreurs remontées n’affichent pas de détails techniques malvenus]({{< ref "le-pipeline-http-en-net" >}})
- Respecter la gestion des droits
- Garantir la cohérence des données (soit l’appel réussi soit aucune modification n’est enregistrée)
- Donner à l’API ait une surface cohérente
- Fournir la pagination par défaut
- Prendre en compte les filtres multicritères
- [Valider (encore) les entrées de l’utilisateur]({{< ref "validation-cqrs-avec-fluentvalidation" >}})
- Fournir des erreurs métier intelligibles

## Conclusion

Une application aujourd’hui coute bien plus cher qu’il y a 10 ou 20 ans parce que les attentes qu’on en a sont beaucoup plus importantes.

Chaque jour apporte son lot de découverte en UX et la mise au point de mécanismes permettant leur bonne implémentation est loin d’être triviale.

La leçon que j’ai tirée de cette rencontre est de toujours échanger au préalable pour être aligné vis-à-vis des attentes de son client être certain que ses attentes soient bien en adéquation avec son budget.

Si ce n’est pas le cas, autant cesser là la collaboration, cela évitera une situation ou nul n’est satisfait.

![Le tatouage ne convient pas à la demande](pourquoi-une-application-ca-coute-aussi-cher/findsomeonecheaper.jpg)
