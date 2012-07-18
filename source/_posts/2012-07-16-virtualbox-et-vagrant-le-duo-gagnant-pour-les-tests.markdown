---
layout: post
title: "Virtualbox + Vagrant = le duo gagnant pour les tests"
date: 2012-07-16 08:48
comments: true
categories: [Developpement, Administration Système, Tests]
---
Lorsqu'il s'agit de tester une application ou une configuration, vient toujours le moment ou on se demande si cela vraiment fonctionner « en vrai », dans la mesure ou bien souvent, on effectue qu'une partie des tests en question afin de ne pas casser complètement son environnement de tests et donc d'avoir à le reconstruire à chaque fois.

Et si on avait le luxe de pouvoir casser cet environnement de test aussi souvent qu'on le voulait, sans se soucier d'avoir à péniblement le reconstruire ? La donne serait-elle différente et nos tests plus fréquents ou plus complets ?

C'est que ce je vous propose de regarder avec Virtualbox+Vagrant.
<!--more-->
## Introduction
{% img left http://www.prodigyproductionsllc.com/wp-content/uploads/2011/05/virtualbox_logo.png %}
[Virtualbox](http://virtualbox.org) est une solution de virtualisation de machine. Disponible sur la plupart des plate-formes du marché, il permet d'installer et de faire tourner différents systèmes en émulant le fonctionnement des périphériques (CPU, Mémoire, Disques, USB, etc.). L'intérêt dans notre cas, est que le disque du système dit « invité » est géré physiquement par un fichier sur le système « hôte », ce qui permet de le manipuler à loisir pour copier une machine virtuellement, la sauvegarder, la distribuer, ou s'en servir de référence.


{% img right http://vagrantup.com/static/images/hippie.png %}
Toutes les manipulations que l'on peut faire via l'interface utilisateur de Virtualbox étant disponible sous forme « d'API », il devient possible de scripter un certain nombre d'actions. C'est là que [Vagrant](http://vagrantup.com) entre en scène. C'est un logiciel développé en ruby, distribué [sous forme de gem](http://rubygems.org) pour les systèmes \*nix, et [sous forme de package](http://vagrantup.com) sous Windows, qui permet la manipulation d'image systèmes crées pour Virtualbox : création à partir d'une référence, démarrage, configuration, provisionnement, arrêt et suppression.

{% img left http://www.noxlogic.nl/wp-content/uploads/2012/04/Puppet_Logo.png %} {% img right http://wiki.opscode.com/download/attachments/688131/OC_Chef_Logo_small.png %}
Arrêtons-nous un moment sur une des fonctions citées : *provisionnement*. C'est là la vraie raison de l'utilisation du couple Virtualbox+Vagrant. En effet, Vagrant permet de préciser un état de configuration à la machine lancée, grâce à des outils comme [Chef](ihttp://wiki.opscode.com/display/chef/Home) ou [Puppet](http://puppetlabs.com), de telle sorte qu'au lieu de préparer un environnement « à la main », il convient de prendre un peu plus de temps à le préparer de manière automatisée et de pouvoir réinitialiser la machine virtuelle pour retrouver un environnement nominal. Je ne vais pas trop insister sur les techniques liées aux deux outils de provisionnement proposé par Vagrant, mais si cette partie vous intéresse j'en reparlerai plus en détails dans un prochain billet.

## Installation des outils
L'installation de Virtualbox est triviale. Sur les systèmes Debian, elle consiste à installer le paquet virtualbox-ose et sous Windows/Apple à télécharger l'installeur pour lequel il conviendra de cliquer autant de fois que nécessaire sur le bouton "suivant". :-)
```bash installation de Virtualbox
sudo aptitude install virtualbox-ose
```

L'installation de Vagrant l'est tout autant pour peu que vous ayez un environnement ruby fonctionnel sur votre système \*nix. À défaut d'un billet de mise à jour sur mon installation ruby courante, reportez-vous à [ce précédent billet](/environnement-de-developpment-rails-sous-ubuntu-11-dot-10). Sous Windows, Vagrant est proposé sous forme d'installateur, si bien que l'environnement ruby est installé en même temps.
```bash installation de Vagrant
gem install vagrant --no-ri --no-rdoc
```
## Préparation/récupération d'une VM de base
Le site de Vagrant propose un certain de nombre de [machines virtuelles pré-configurées](https://github.com/mitchellh/vagrant/wiki/Available-Vagrant-Boxes) (dernières version de Ubuntu en 32 et 64 bits), mais également une liste de [machines mises à dispositions par la communauté](http://www.vagrantbox.es/).
Il est bien évidemment possible de créer ses propres « baseboxes » en suivant [les préconisations](http://vagrantup.com/v1/docs/base_boxes.html) du site ou en utilisant une gem bien utile appelée [veewee](http://rubygems.org/gems/veewee).

Afin d'éviter de télécharger à chaque nouvelle utilisation, Vagrant peut stocker dans votre homedir (~/vagrant.d) les _basebox_ que vous manipulerez fréquemment:
```bash Téléchargement et enregistrement local d'une nouvelle basebox
vagrant box add precise64 http://files.vagrantup.com/precise64.box
```

## Mise en oeuvre
Prenons l'exemple d'un test sur une application web. L'idée étant de ne pas avoir à installer de serveur web, ni toute la couche applicative et ses dépendances sur son environnement de travail habituel (pour ne pas le « pourrir ») d'une part, et éventuellement d'autre part de tester le fonctionnement de l'application dans un environnement le plus proche possible de celui dans lequel il sera ammené à fonctionner lors de sa mise en production.

Mettons que l'on souhaite tester [ownCloud](http://owncloud.org), la page [listant les pré-requis](http://owncloud.org/support/install/) va nous servir de base pour préparer notre configuration. Créons un dossier dans lequel nous déposerons l'archive du logiciel.
```bash
mkdir /tmp/owncloud_test && cd /tmp/owncloud_test
wget http://download.owncloud.org/releases/owncloud-4.0.4.tar.bz2
```

Pour pouvoir fonctionner, Vagrant s'attend à trouver un fichier de configuration dans lequel il va nous être possible de définir en plus les quelques directives permettant de paramétrer notre environnement cible.
```bash création d'un fichier de configuration
vagrant init precise64
```
Une fois le ficher Vagrantfile de base créé, renseignons le un peu plus afin de configurer notre VM comme on le souhaite
```ruby Vagranfile
Vagrant::Config.run do |config|

  # Basebox utilisée pour construire la VM
  config.vm.box = "precise64"

  # Forward du port 8080 du système hôte vers le port 80 de la VM
  # afin de permettre l'accès au serveur web de le VM depuis le système
  config.vm.forward_port 80, 8080

  # Utilisation de puppet (standalone)
  # normalement installé par défaut sur les basebox)
  config.vm.provision :puppet do |puppet|
    puppet.manifests_path = "."
    puppet.manifest_file  = "owncloud.pp"
  end

  # execution de commandes spécifiques qui ne peuvent être prises en charge par Puppet
  config.vm.provision :shell, :inline => 'sudo rm -rf /var/www/* &&
                                          cd /tmp &&
                                          tar -xjf /vagrant/owncloud-4.0.4.tar.bz2 &&
                                          sudo mv /tmp/owncloud/* /var/www/ &&
                                          chown -R www-data\: /var/www &&
                                          sudo apache2 restart'
end
```

```ruby owncloud.pp : Le fichier de provisionnement Puppet
$owncloud_reqs = [
  "libapache2-mod-php5",
  "php5-json",
  "php5-gd",
  "php5-sqlite",
  "curl",
  "libcurl3",
  "libcurl3-dev",
  "php5-curl",
]
package { $owncloud_reqs:
  ensure => installed,
}
```

```bash Démarrage de la VM
$ vagrant up
[default] Importing base box 'precise64'...
[default] Matching MAC address for NAT networking...
[default] Clearing any previously set forwarded ports...
[default] Forwarding ports...
[default] -- 22 => 2222 (adapter 1)
[default] -- 80 => 8080 (adapter 1)
[default] Creating shared folders metadata...
[default] Clearing any previously set network interfaces...
[default] Booting VM...
[default] Waiting for VM to boot. This can take a few minutes.
[default] VM booted and ready for use!
[default] Mounting shared folders...
[default] -- v-root: /vagrant
[default] -- manifests: /tmp/vagrant-puppet/manifests
[default] Running provisioner: Vagrant::Provisioners::Puppet...
[default] Running Puppet with /tmp/vagrant-puppet/manifests/owncloud.pp...
notice: /Stage[main]//Package[php5-sqlite]/ensure: ensure changed 'purged' to 'present'

notice: /Stage[main]//Package[php5-curl]/ensure: ensure changed 'purged' to 'present'
notice: /Stage[main]//Package[php5-json]/ensure: ensure changed 'purged' to 'present'
notice: /Stage[main]//Package[php5-gd]/ensure: ensure changed 'purged' to 'present'

notice: /Stage[main]//Package[libcurl3-dev]/ensure: ensure changed 'purged' to 'present'
[default] Running provisioner: Vagrant::Provisioners::Shell...
```

Une fois la main rendue (c'est potentiellement _très_ long, Puppet n'étant pas véloce en plus du volume dess paquets à télécharger), il suffit de se connecter à http://localhost:8080 sur votre propre machine et constater le résultat.
![Résultat d'installation](/images/vagrant/localhost_owncloud.png)

Et voilà, une installation « jetable » de owncloud me permettant de tester l'installation, d'avancer dans la configuration, et de revenir à zéro en réinialisant complètement la machine virtuelle.

## Autres commandes
La commande 'vagrant' permet également de stopper ou suspendre (au sens Virtualbox du terme), de redémarrer, et détruire la VM. Sous \*nix, la commande 'vagrant ssh' comme elle l'indique permet de se connecter via SSH sur la machine virtuelle, comme on se connecterait à un serveur distant. La connexion s'effectue sans mot de passe, grâce à une clé semi-privé dans le sens ou c'est une clé utilisée par défaut dans toutes les installations de vagrant (il est préconisé de l'utiliser à la création d'une nouvelle basebox si elle a vocation à être distribuée).

Windows étant dépourvu de client SSH natif, la commande renvoie une erreur en précisant qu'il est possible d'utiliser un client externe tel que [PuTTY](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html) en convertissant la clé partagée au bon format.

La commande 'vagrant provision' relance le processus de provisionnement.

## Conclusion
Vagrant est un outil très intéressant pour manipuler facilement différents environnements. Que ce soit pour effectuer des tests, valider un environnement, isoler des services/applications ou tester des directives Puppet/Chef en vue d'un déploiement réel.

L'inconvénient est évidemment que, comme tout ce qui est virtualisé, c'est fortement consommateur de ressources et en l'occurence, pas très véloce sauf à avoir une très bonne machine. Le provisionnement à répétition peut être aussi un gouffre à temps perdu lorsque des paquets doivent être installés (ils sont re-téléchargés à chaque fois). Il peut-être intéressant dans cette situation d'avoir recours à un proxy-cache. Cela fera peut-être l'objet d'un prochain billet...

### Lectures complémentaires
L'éventail des possibilités est bien évidemment plus large que le petit cas particulier étudié ici.

 * Je ne peux que vous conseiller la lecture de [la partie doc site de vagrant](http://vagrantup.com/v1/docs/index.html) afin d'en apprendre davantage. N'hésitez pas à partager vos éventuels usages de Vagrant, c'est toujours intéressant de connaitres différents usages.

 * Présentation de [@jmfontaine](http://twitter.com/jmfontaine) au RMLL 2012 sur la [gestion des environnements de développement avec vagrant](http://www.slideshare.net/JMF/grer-ses-environnements-de-dveloppement-avec-vagrant-rmll-2012)
