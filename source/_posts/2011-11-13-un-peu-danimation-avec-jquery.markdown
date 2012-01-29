---
layout: post
title: "Un peu d'animation avec jQuery"
date: 2011-11-13 22:41
comments: true
categories: [jQuery, Developpement]
---
Dans le cadre d'un projet en cours, je souhaitais placer une image assez grande sur une page d'information, mais je trouvais que cela mettait en vrac la mise en page. Aussi, je me suis demandé ce que cela pourrait donner si cette image s'affichait de manière réduite mais agrémentée d'un défilement vertical.
<!--more-->
L'affichage se présente donc de la sorte : une image englobée dans un calque qui est plus petit, et donc tronquée. L'idée est de faire défiler verticalement l'image pour la dévoiler au fur et à mesure :

![Mon super Site](https://lh5.googleusercontent.com/-Z6Vkpu1xtdw/TsA9sQUDMxI/AAAAAAAAA-Y/ysQhdLEZfWA/s986/1321221551960.png)

Voici le code correspondant.
{% gist 1362818 page.html %}
Notez l'utilisation de <code>overflow:hidden</code> afin d'empêcher que le <code>div</code> ne prenne la taille de l'image.
{% gist 1362818 page.css %}
Et là, jQuery est ton ami comme dirait l'autre. L'astuce consite à jouer sur la propriété <code>top</code> de l'image et de demander à jQuery d'incrémenter grâcieusement à l'aide de la fonction [animate](http://api.jquery.com/animate/).
{% gist 1362818 page.js %}
L'avantage de cette fonction, c'est qu'elle accepte comme dernier paramètre le nom d'une fonction a exécuter une fois l'animation complétée. Ainsi, on peut effectuer une animation dans un sens (up), puis la chainer à une animation dans le sens inverse (down), et boucler. ;-)

Vous pouvez aller [tester/modifier ce code](http://jsfiddle.net/KgWK4/45/) sur jsFiddle, qui m'a bien aidé pour tripatouiller cette idée.
