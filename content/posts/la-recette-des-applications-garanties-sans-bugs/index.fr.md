+++
date = '2023-04-11T00:22:10+02:00'
draft = false
title = 'La recette des applications garanties sans bugs'
categories = ["Billet d'humeur"]
Tags = ["Billet d'humeur", "Management", "Tech"]
+++

Au détour de la machine a café, j’ai été témoin d’une conversation entre deux développeurs.

L’un s’émerveillait d’une promesse de valeur d’une société spécialisée dans le software craftmanship.

{{< admonition quote "Quote" >}}

Tu te rend compte? Ils ont tellement mis l’accent sur la qualité qu’ils garantissent des applications sans bugs! Si le client en trouve, ils les corrigent gratuitement!

{{< /admonition >}}

Ca m’a donné l’idée de cet article.

Détaillons les mots-clés.

## Sans bugs

J’ai eu la chance, par le passé, de travailler en ESN avec un très bon commercial sur un projet au forfait.

Le projet s’est très mal passé.

De nombreux échanges ont eu lieu avec le client sur les anomalies rencontrées et la qualité des spécifications.

Il est rare que les développeurs aient connaissance des enjeux commerciaux, ça a été le cas ici aussi: je n’ai jamais su quel arrangement a été trouvé.

Par contre, il y a une phrase qui faisait mouche à chaque fois et que le commercial utilisait sans retenue:

{{< admonition quote "quote" >}}

Un bug, c’est un défaut de spécification. Si ce n’est pas spécifié, ce n’est pas un bug.

{{< /admonition >}}

Et ça rendait immédiatement le client plus conciliant.

Il faut dire que les spécifications étaient anecdotiques.

Je n’ai jamais retrouvé cette définition d’un bug, mais je la trouve redoutable: elle reporte sur le client la responsabilité de fournir des spécifications exhaustives.

Il ne manquerait plus qu’à la coupler à des clauses comme :

{{< admonition quote "quote" >}}

Tout remaniement des documents de spécifications fera l’objet d’un avenant au présent contrat.

{{< /admonition >}}

et par la magie de la contractualisation, le client à la responsabilité financière de la qualité de sa demande.

## une application garantie

C’est tellement commun !

C’est sonne tellement comme une promesse qu’on en néglige la définition.

Une garantie, sur un développement au forfait, c’est la norme, pas l’exception.

Quel client serait assez fou pour engager des centaines de milliers d’euros voire des millions dans un forfait de développement en faisant une confiance aveugle ?

Ici, le sentiment vient de l’expression **garanti sans bugs**.

On entend ca comme une promesse, une promesse d’application « réussie du premier coup ».

Revenons sur la définition d’une une garantie:

{{< admonition quote "quote" >}}

La garantie constitue une obligation contractuelle ou légale qui engage un fournisseur — le garant — envers un acquéreur lors de la vente d’un bien ou lors de la fourniture d’un service.

- [WIKIPEDIA](https://fr.wikipedia.org/wiki/Garantie)

{{< /admonition >}}

La majorité des produits qu’on achète sont garantis.

Si vous achetez un smartphone, la garantie est présente, la garantie fait partie des conditions générales de vente.

Elle se résume souvent à :

{{< admonition quote "quote" >}}

Les équipements sont couverts par une garantie pièces et main d’œuvre contre tout défaut ou vice de fabrication.

{{< /admonition >}}

Est-ce que ca veut dire que le smartphone ne tombera pas en panne ?

Non!

Par contre, s’il tombe en panne et qu’il est encore sous garantie, c’est à la charge de mon fournisseur de résoudre mon problème, en le réparant ou en le remplaçant.

Des clauses permettent de limiter la garantie.

D’ordinaire, il s’agit de clauses:

- Limitées dans le temps (1 ou 2 ans)
- Relatives à la bonne utilisation (ne pas utiliser son smartphone comme un marteau)

C’est la même chose pour les les bugs, le « garanti sans bugs », ne veut pas dire qu’il n’y en aura pas, mais que **le fournisseur a promis de les corriger pendant la période de garantie**.

Les clauses de limitations existent aussi, elles aussi, limitées dans le temps ou les conditions d’utilisation comme :

- Pas de modifications
- Doit tourner à minima sur un matériel spécifié
- Etc

Quant on sait à quelle vitesse évoluent les produits, il y a des chances que le client invalide lui-même sa garantie en modifiant son logiciel.

## Ils les corrigent gratuitement

![Meme when you pay to get something for free](la-recette-des-applications-garanties-sans-bugs/7h6dyc.jpg)

Pour savoir si un forfait est rentable, il y a 2 facteurs à prendre en compte:

- Combien est prêt à payer le client ?
- Combien la réalisation va couter ?

Si le cout de production (sur lequel nous avons le contrôle) est plus cher que le client peut payer, ce n’est pas rentable.

Le coût de production, s’appuie sur des [estimations]({{< ref "les-points-cles-des-estimations" >}}).

Ces estimations sont souvent faites d’abord assez finement en terme de couts de développement, en jour / homme.

On inclue ensuite les charges annexes en les estimant en appliquant un ration sur la production de code.

Les chiffres évoqués ici sont arbitraires, mais ca donnerait :

- Gestion de projet (15%)
- Comitologie (10%)
- Méthodologie (coucou les cérémonies SCRUM) (20%)
- Recette applicative (40%)
- **Période de garantie (10%)**

Ainsi, pour l’exemple un projet dont le développement est estimé à 100j/H se verrait avec un cout de :

100 +15 +10 +20 +40 +10 = **200** j/H

avec un jour/Homme à 500 € (arbitraire ici aussi), ca nous donnerait donc un montant de **100 000 €**

Le client n’a pas accès à ce détail.

Quand il signe le contrat, il s’engage à payer 100 000 € et le fournisseur s’engage à réaliser son application dans des délais impartis.

Il n’a aucune idée de ce que ca coute réellement au fournisseur.

Un peu comme quand on va au restaurant commander un steak, on ne sait pas comment s’appelait la vache sur laquelle il a été prélevé ni qui fait la préparation.

En tant que client, la seule chose qui nous préoccupe, c’est « c’est quand que ca arrive ? j’espère que ca va être bon ! »

Pour la garantie, c’est pareil: **elle fait partie du contrat**.

Quelle marge fait le fournisseur dessus ? est ce que les clients y recourent ?

Ce sont d’autres sujets.

Mais le simple fait qu’elle soit sur le contrat, c’est que le client la paye et qu’elle n’est pas gratuite.

Il est donc normal que la société corrigent les bugs gratuitement: **le client a payé pour ça**.

## Conclusion

Je suis admiratif de l’impact que peuvent avoir de telles phrases sur l’inconscient.

En particulier depuis que j’écrit ces articles et que je prends en compte l’importance de l’émotion que peuvent dégager les mots.

La phrase fait miroiter des promesses mais en creusant un peu, finalement, la promesse de valeur est ordinaire.

Pour une entreprise le fait d’axer sa communication autour de cette phrase est un très bon moyen de promouvoir ses valeurs, son « savoir-être ».

Mais ce n’est en aucun cas une offre disruptive.
