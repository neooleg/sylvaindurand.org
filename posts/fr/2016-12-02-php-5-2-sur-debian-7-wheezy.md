---
title: PHP 5.2 sous<br/> Debian 7 <em>Wheezy</em>
---

Il est parfois nécessaire de ressusciter de vieux codes qui ne fonctionnent qu'avec des versions antiques de PHP. Nous allons ici voir comment installer PHP 5.2 sur Debian 7 *Wheezy*[[car je ne suis pas parvenu à le faire fonctionner sous Debian 8 *Jessie*...]].

PHP 5.2 n'étant plus maintenu [depuis 2011](http://php.net/eol.php), il est évidemment très déconseillé de l'installer sur un serveur de production, en raison des nombreuses failles de sécurité qu'il comporte.



## Sources

Nous allons utiliser les sources de Debian 6 *Lenny*. Pour cela, on édite `/etc/apt/sources.list` pour y ajouter :

```none
deb http://archive.debian.org/debian lenny main contrib non-free
```

Nous allons également modifier le fichier `/etc/apt/preferences` afin de signifier à `apt` que nous voulons des versions plus anciennes de PHP:

```
Package: *
Pin: release n=lenny*
Pin-Priority: 100

Package: php5 php5-common php5-cli php5-curl php5-gd php5-mcrypt php5-mysql php5-mhash php5-xsl php5-xmlrpc php5-sqlite libapache2-mod-php5 phpmyadmin
Pin: release n=lenny*
Pin-Priority: 999
```

## Installation

L'installation n'est pas directe, car il y a un conflit de version avec la dépendance `libkrb53`. Commençons par télécharger les paquets : 

```none
cd ~
apt-get update
apt-get download php5-common php5-cli php5-curl php5-gd \
    php5-mcrypt php5-mysql php5-mhash php5-xsl php5-xmlrpc \
    php5-sqlite libapache2-mod-php5 
```

Nous pouvons alors installer les paquets en demandant d'ignorer `libkrb53` :

```none
dpkg --ignore-depends=libkrb53 -i *.deb
```

## Après l'installation

PHP 5.2 doit maintenant fonctionner. Cependant, `apt` refuse désormais d'installer de nouveaux paquets. Il vous demandera d'utiliser la commande `apt-get -f install`, qui supprimera PHP :

```none
Vous pouvez lancer « apt-get -f install » pour corriger ces problèmes.
Les paquets suivants contiennent des dépendances non satisfaites :
    libapache2-mod-php5 : Dépend: libkrb53 (>= 1.6.dfsg.2) mais il n'est pas installé
    php5-cli : Dépend: libkrb53 (>= 1.6.dfsg.2) mais il n'est pas installé
    php5-imap : Dépend: libc-client2007b mais il n'est pas installé
                Dépend: libkrb53 (>= 1.6.dfsg.2) mais il n'est pas installé
E: Dépendances manquantes. Essayez d'utiliser l'option -f.
```

Aussi, à chaque fois que vous souhaiterez installer un paquet, il faut exécuter `apt-get -f install`, installer son paquet, puis réinstaller PHP avec `dpkg --ignore-depends=libkrb53 -i *.deb`.