---
layout: post
title: "Création et publication d'une gem ruby, pas étonnant qu'il y en ait autant..."
date: 2012-09-26 18:03
comments: true
categories: [Ruby, Gems, Développement]
---
Suite si l'on peut dire du billet précédent, cela faisait quelques temps que je voulais comprendre le fonctionnement d'une gem ruby. Oui, on dit une gem en français comme le suggère [une étude récente](http://fr.slideshare.net/camilleroux/dissection-dun-dveloppeur-ruby) sur la communauté des développeurs ruby. Et quoi de mieux pour apprendre que d'essayer d'en contruire une.

Or vous allez le constater, ce n'est pas si compliqué qu'on le croit, d'une part du fait du fameux adage « Conventions over Configuration », et d'autre part que les outils qui vont bien sont vraiment sous la main.
<!--more-->
## TL;DR
```bash Version courte et non détaillée des étapes de la construction d'une gem ruby
bundle gem <gem_name>
cd <gem_name>
vim <gem_name>.gemspec
# Copier les fichiers de la gem dans ./lib
#  et les éventuels executables dans ./bin
rake install
rake release
```
## Avant-propos
J'avais dans les cartons un script écrit en ruby
