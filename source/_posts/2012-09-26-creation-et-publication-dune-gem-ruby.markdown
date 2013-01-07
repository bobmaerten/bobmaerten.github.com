---
layout: post
title: "Création et publication d'une gem ruby"
date: 2012-09-26 18:03
comments: true
categories: [Ruby, Gems, Développement]
---
Cela faisait quelques temps que je voulais comprendre le fonctionnement d'une gem[1] ruby. Et quoi de mieux pour apprendre que d'essayer d'en contruire une !

J'avais dans les cartons un script écrit en ruby un peu crade (tout dans un même fichier .rb) sur lequel je souhaitais faire un peu de [refactoring](http://fr.wikipedia.org/wiki/Refactorisation).

En commençant à réorganiser mon code en classes et remplacer les valeurs brutes par des constantes, je me suis dit que cela ferai un excellent thème pour un prochain billet, mais il m'est revenu à l'esprit un article de chez mes amis rubyistes de chez [Synbioz](http://www.synbioz.com/) appelé « [creer son gem[1] et le publier](http://www.synbioz.com/blog/2012/04/26/creer_son_gem_et_le_publier) ».

[1] personnellement je prefère gem au féminin, mais c'est un autre débat.

Je ne pourrais clairement pas décrire mieux la facilité avec laquelle se déroule le processus de création et publication de gems qu'ils ne l'ont fait, alors franchement, allez lire leur billet. Mais pour les pressés :

```bash TL;DR
bundle gem <gem_name>
cd <gem_name>
vim <gem_name>.gemspec
# Copier les fichiers de la gem dans ./lib
#  et les éventuels executables dans ./bin
rake install
rake release
```

Comme vous le constatez, ce n'est pas si compliqué, essentiellement du fait du fameux adage « Conventions over Configuration », mais également parce que les outils qui vont bien sont déjà sous la main.

Pas étonnant que le site [Rubygems](http://www.rubygems.org/) soit si fourni.
