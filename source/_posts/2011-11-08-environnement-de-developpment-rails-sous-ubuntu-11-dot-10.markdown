---
layout: post
title: "Environnement de développement rails sous Ubuntu 11.10"
date: 2011-11-08 22:09
updated : 2011-11-12
comments: true
categories: [Ubuntu, Developpement, Rails]
---
Alors que nombre de tutoriels, vidéos, [pod|screen]casts traitent de la meilleure façon de configurer son environnement *MacOSX* pour le développement ruby/rails, les articles concernant ce qu'il en est des autres systèmes sont plutôt rares (et d'autant plus en français), aussi je vous propose ici ma façon de procéder sur Ubuntu 11.10 que je viens tout juste de finir d'installer et de paramétrer sur [mon portable](http://www.dell.com/fr/entreprise/p/vostro-3450/pd).
<!--more-->
## Méthode générale
Comme il s'agit d'installer une configuration de développement, le principe réside en l'installation de ruby par l'intermédiaire de [RVM](http://rvm.beginrescueend.com). L'avantage est de pouvoir disposer des fonctionnalités offertes par RVM telles que le choix de la version de ruby en fonction du projet, et accessoirement la possibilité de gérer des *gemsets*.

## Pré-requis système
Les paquets suivants doivent être installés afin de pouvoir compléter l'installation.
```bash
sudo apt-get install build-essential openssl libreadline6 libreadline6-dev \
zlib1g zlib1g-dev libssl-dev libyaml-dev libsqlite3-0 libsqlite3-dev sqlite3 \
libxml2-dev libxslt-dev autoconf libc6-dev ncurses-dev automake libtool bison \
git curl libssl0.9.8 libpq-dev
```

## La commande magique ;-)
RVM s'installe à l'aide de cette commande
```bash
bash < <(curl -s https://raw.github.com/wayneeseguin/rvm/master/binscripts/rvm-installer )
```
Cette commande récupère le script d'installation directement sur le site de RVM et l'exécute en local. Ensuite, comme l'indiquent les consignes affichées à la fin de l'exécution du script, il convient d'ajouter la commande suivante dans votre fichier <code>.bashrc</code> (ou .zshrc).
```bash
[[ -s "$HOME/.rvm/scripts/rvm" ]] && source "$HOME/.rvm/scripts/rvm" # Load RVM into a shell
```
Il convient enfin de recharger votre environnement (en fermant votre terminal, ou en effectuant un <code>source .bashrc</code> (ou source .zshrc)

## Installation de Ruby
Maintenant nous disposons de la commande rvm, nous pouvons nous attaquer à Ruby sereinement, car RVM va nous faciliter grandement la tâche. Pour installer la dernière version en date, il suffit de taper :
```bash
rvm install 1.9.3
```
S'en suit une longue phase de récupération des sources, de paramétrage et de compilation, qui au final nous amène à terminer la configuration par :
```bash
rvm use 1.9.3 # ajouter --default pour en faire la version chargée par défaut au lancement du terminal
```

## Toute la puissance de ruby et de rubygems à votre portée
Et voila, c'est tout. Vous pouvez désormais utiliser la commande <code>gem install</code> pour installer les librairies nécessaires à vos projets.
```bash
gem install bundler rails
rails new my_new_app
cd my_new_app
rails s
```
**Note** : cette dernière commande va échouer sur la derniere version de rails (3.1.x) car il y a une dépendance sur un moteur d'exécution Javascript. Deux solutions :
```bash ajouter therubyracer au fichier Gemfile
echo "gem 'therubyracer'" >> Gemfile
bundle
rails s
```
ou alors, si jamais vous vous intéressez également un peu à Javascript coté serveur, vous pouvez installer le paquet <code>nodejs</code>.
```bash Installer Node.js sur le système, rails s'en servira au lieu de therubyracer
sudo apt-get install nodejs
rails s
```

## Conclusion
Comme vous pouvez le constater sur le nombre de commandes à exécuter, l'installation et la configuration d'un environnement de développement rails sous Ubuntu n'est pas très compliqué. Et même si vous développez sous *MacOSX*, il peut être intéressant de connaître la démarche pour peu que vous soyez amené à déployer votre code sur un système Linux, ou si vous souhaitez tester votre application sur un serveur virtualisé (avec la gem <code>vagrant</code> par exemple).

#### Ressources complémentaires
* [RailsReady](https://github.com/joshfng/railsready) : un script encore plus automatisé qui fonctionne sur MacOS, Ubuntu et CentOS permettant d'effectuer toutes les manipulations effectuées ci-dessus en une ligne.
* [rbenv](https://github.com/sstephenson/rbenv) est une nouvelle implémentation des fonctionnalités de RVM, avec un postulat légèrement différenti (notamment sur la façon de gérer l'environnement). Je l'évoque ici, mais ne l'ayant pas exploré vraiment, je ne me permettrais pas de porter un jugement.
* [La documentation Ubuntu officielle](http://doc.ubuntu-fr.org/rubyonrails) en français, mais elle préconise une installation de RVM au niveau du système, ce qui oblige à préfixer chaque action d'installation de gem par <code>sudo</code>.

# Mises à jour de cet article
2011-11-12 : Ajout d'un prérequis (libpq-dev) requis par l'hébergement heroku
2011-11-09 : Ajout d'un prérequis (libssl) pour permettre à certaines application rails 3.0.x de se lancer
