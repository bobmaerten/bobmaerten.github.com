---
layout: post
title: "Tmux+Vim Mega Combo"
date: 2012-08-30 13:15
comments: true
categories: [Développement, Configuration, Ubuntu]
---
Je vous propose pour ce billet de rentrée, un petit retour sur la récente refonte de mon environnement de développement sous Ubuntu.

Pour situer, j'utilise quasi-exclusivement [vim](http://www.vim.org) car c'est un usage qui me convient cérébralement. Pour mémoire, vim est un éditeur de texte qui permet (entre autre) la distinction du mode insertion et du mode commande, et rien que ça, une fois mappé ça mentalement c'est une tuerie.

D'autre part, je me sers depuis peu et de plus en plus volontiers de [tmux](http://tmux.sourceforge.net/), un _terminal multiplexer_ qui offre la possibilité de travailler en sessions (détachables et réattachables à souhait), et d'organiser son terminal en fenêtres et panels. C'est une alternative à l'outil GNU standard _screen_ que j'utilise plus volontairement lorsque je dois exécuter des commandes longues sur mes serveurs, car il y est installé par défaut. Bien que les deux outils se valent, je préfère tout de même le _feeling_ de tmux.

L'objectif n'est pas de faire l'apologie des outils que j'utilise, mais de présenter l'usage que j'en ai et de partager les trucs, astuces et écueils techniques.
<!--more-->
## Objectif
Dans cette refonte entamée, je souhaitais pouvoir disposer d'un environnement « transportable » ou tout du moins réplicable facilement, voire automatiquement sur mes différents postes (boulot, maison, portable). L'idée de travailler au maximum dans un shell \*nix s'est rapidement imposée. La plupart des outils se configurant à l'aide de fichier de configuration texte, l'usage d'un dépot git ou d'un outil de synchronisation de fichiers comme [UbuntuOne](http://one.ubuntu.com) ou [DropBox](http://dropbox.com) permettrait d'obtenir le résultat escompté.

J'ai donc décidé de placer l'ensemble de mes fichiers de configuration (dotfiles) dans mon Dropbox, et d'avoir un petit script pour placer des liens symboliques en lieu et place des fichiers de base. Ainsi la synchronisation s'effectue (de manière transparente ou à la demande selon la configuration de Dropbox mise en place) sur mes différents postes.
```bash Création des liens symboliques à la racine du compte depuis le dossier Dropbox
#!/bin/bash
cd ~
ln -s Dropbox/dotfiles/tmux/tmux.conf .tmux.conf
ln -s Dropbox/dotfiles/zsh/zshrc .zshrc
ln -s Dropbox/dotfiles/git/gitconfig .gitconfig
ln -s Dropbox/dotfiles/vim .vim
ln -s Dropbox/dotfiles/vim/vimrc .vimrc
ln -s Dropbox/dotfiles/ruby/gemrc .gemrc
ln -s Dropbox/dotfiles/ruby/railsrc .railsrc
ln -s Dropbox/dotfiles/fonts/ .fonts
/usr/bin/fc-cache -vf
```
## Mon usage de tmux
Tmux me permet essentiellement de partager ma fenêtre de terminal en plusieurs onglet ou panels, et de gérér/surveiller/lancer plusieurs activités simultanées. Vous pouvez voir sur l'image ci-dessous une session tmux organisée en 3 panels : mon éditeur vim en haut en cours d'édition de ce billet, et deux autres en bas qui font tourner deux programmes, un serveur Webrick pour prévisualiser le billet, et un système de rechargement automatique de page web (guard-livereload).
![Session Tmux](/images/sessiontmux.png)
Tmux permet également de travailler avec des onglets (appelés fenêtres dans le contexte). Des commandes clavier permettent bien entendu de passer d'une fenêtre ou d'un panel à un autre, de changer la disposition, de réorganiser, et même de scripter tout cela (exemple plus loin dans le billet). Je ne vais pas détailler tout cela ici, il faudrait plusieurs articles voire un site complet, par contre si la configuration de tmux vous intéresse, je ne saurais trop vous conseiller [l'excellent bouquin sur tmux](http://pragprog.com/book/bhtmux/tmux) de [Brian P. Hogan](http:/twitter.com/bphogan) disponible chez [The Pragmatic Bookshelf](http://pragprog.com) (une excellente source sur le développement logiciel au passage) qui fait le tour de l'outil pour pas bien cher.

Vous pourrez bien entendu trouver pas mal de documentation sur le net, comme toujours pour ce genre d'outil bien répandu dans la communauté.
## Trop de mapping tue le mapping
Un des gros problèmes auquel j'ai été confronté fut de résoudre le croisement des différents raccourcis et de l'utilisation de certaines touches dans vim, à la fois sous tmux et en dehors tmux. En effet, tmux et vim ne s'entendent pas toujours sur certaines combinaisons. Des raccourcis qui fonctionnaient dans vim ne s'activent plus du tout une fois ouvert sous tmux. Ajoutez à cela les raccourcis du terminal lui même et c'est devenu un vrai casse-tête quand j'ai voulu configurer des raccourcis pour retailler les panels, tout en gérant les _splits_ de vim...

[Stackoverflow](http://stackoverflow.com/search?q=vim+tmux) a été d'une grande aide pour résoudre tout ces petits problèmes, bien sur jamais bloquants, mais quand même bien frustrants. Sachez si vous rencontrez le problème que l'option tmux "set-window-option -g xterm-keys on" est d'un grand secours ! ;)

Si jamais, comme moi, vous vous servez des flêches de direction dans vim, voici les quelques lignes que j'ai du ajouter dans mon .vimrc afin que je puisse m'en servir sous tmux. Alors oui, je sais, il parait que les flèches dans vim c'est le mal ! [Mais tout dépend de ce qu'on en fait](http://jeetworks.org/node/89) :P
```vim
" Fix usage of arrows in tmux
" http://superuser.com/questions/401926/how-to-get-shiftarrows-and-ctrlarrows-working-in-vim-in-tmux
if &term =~ '^screen'
    " tmux will send xterm-style keys when its xterm-keys option is on
    execute "set <xUp>=\e[1;*A"
    execute "set <xDown>=\e[1;*B"
    execute "set <xRight>=\e[1;*C"
    execute "set <xLeft>=\e[1;*D"
endif
```
## _One command to rule them all_
Dernièrement je me suis fait un petit script tmux qui me permet de réorganiser ma fenêtre courante afin d'avoir une petite zone de shell (agrandissable dans une autre fenetre, voir plus bas),
```bash Formattage de la fenetre courante pour de développement Rails
# select the first (0) pane
selectp -t 0

# split it into two halves and open vim in the second one
splitw -h -p 66 'vim ; zsh'
# reselect the first pane again and split it verticaly
selectp -t 1
splitw -v -p 80 'guard'
splitw -v -p 20 'rails server'

# reselect first pane
selectp -t 1
```
J'ai ensuite ajouté un nouveau bind dans mon .tmux.conf
```bash Extrait de .tmux.conf
# Rails dev-panes
bind-key R source-file ~/Dropbox/dotfiles/tmux/rails-dev-panes
```
Ainsi, à la racine d'un projet rails, il me suffit de presser <Prefix>R pour me obtenir ceci. Pratique, hein ?
![Session Rails](/images/sessionrails.png)
## _There and back again_
En bonus, voici deux directives de configuration tmux permettant d'agrandir un panel dans une fenêtre et inversement (avec <Prefix>+ et <Prefix>-).
Cette configuiration me permet de passer temporairement d'un panel réduit (telle une zone de logs) à une fenêtre de la taille de mon terminal.
```bash Zoom et DeZomm d'un panel tmux
unbind +
bind + \
  new-window -d -n tmux-zoom \;\
  swap-pane -s tmux-zoom \;\
  select-window -t tmux-zoom

unbind -
bind - \
  last-window \;\
  swap-pane -s tmux-zoom \;\
  kill-window -t tmux-zoom
```
## Pour finir
Voila, comme pour tous les billets ou je pars dans l'idée de faire court, simple et rapide, je me plante complètement alors je préfère arrêter là. Chacun son usage des outils avec lesquels il se sent à l'aise, vous avez désormais un aperçu de ma façon de travailler.
