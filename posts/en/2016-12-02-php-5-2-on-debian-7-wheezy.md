---
title: PHP 5.2 on <br/> Debian 7 <em>Wheezy</em>
---

Sometimes you need to bring back to life old lines of code which will only works with old PHP versions. We will here see how to install PHP 5.2 on Debian 7 *Wheezy*[[because I couldn't get it work on Debian 8 *Jessie*...]].

PHP 5.2 end of life occured in [January 2011](http://php.net/eol.php); thus it is obviously a very bad idea to install it on a production environment, for obvious security reasons.

## Sources

We will use the Debian 6 *Lenny* sources. To do so, let's edit `/etc/apt/sources.list` in order to add the repo:

    deb http://archive.debian.org/debian lenny main contrib non-free

We will also edit `/etc/apt/preferences` in order to tell `apt` we want to use PHP older versions. We add:

    Package: *
    Pin: release n=lenny*
    Pin-Priority: 100

    Package: php5 php5-common php5-cli php5-curl php5-gd php5-mcrypt php5-mysql php5-mhash php5-xsl php5-xmlrpc php5-sqlite  libapache2-mod-php5 phpmyadmin
    Pin: release n=lenny*
    Pin-Priority: 999

## Installation

The installation isn't straightforward due to a conflict with the dependency `libkrb53`. We will first download the packages: 

    cd ~
    apt-get update
    apt-get download php5-common php5-cli php5-curl php5-gd \
     php5-mcrypt php5-mysql php5-mhash php5-xsl php5-xmlrpc \
     php5-sqlite libapache2-mod-php5 

Then, we install them, while specifying we want to ignore the missing `libkrb53`:

    dpkg --ignore-depends=libkrb53 -i *.deb

## After the installation

Now, PHP 5.2 should work, but now `apt` doesn't accept to install new packages anymore. It will ask to run `apt-get -f install`, wich will delete PHP:

    You might want to run 'apt-get -f install' to correct these.
    The following packages have unmet dependencies:
     libapache2-mod-php5 : Depends: libkrb53 (>= 1.6.dfsg.2) but it is not installed
     php5-cli : Depends: libkrb53 (>= 1.6.dfsg.2) but it is not installed
    E: Unmet dependencies. Try using -f.

So, each time you need to install or upgrade a package, you will have to run `apt-get -f install` before doing your stuff, then to install PHP 5.2 again with `dpkg --ignore-depends=libkrb53 -i *.deb`.
