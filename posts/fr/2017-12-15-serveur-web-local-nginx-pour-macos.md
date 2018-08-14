---
title: Serveur web local<br/> <em>Nginx</em> pour macOS
original: 2016-01-05
redirect: /serveur-web-local-nginx-pour-os-x/
categories: Nginx
---

Cherchant une solution satisfaisante pour créer un serveur web local permettant de programmer sous macOS avec PHP et MySQL, j'ai été déçu de constater que les solutions "clés en main" étaient loin d'égaler les WAMP qui peuvent exister sous Windows.

C'était oublier qu'macOS est un système Unix, et que contrairement à Windows, il est parfaitement possible d'y créer un serveur local sur mesure avec quelques paquets.

Nous allons voir ici comment installer *Nginx*, PHP-FPM et *MariaDB* (MySQL) sous macOS *High Sierra* grâce au gestionnaire de paquets *Homebrew*.

## *Homebrew*

*HomeBrew* est un gestionnaire de paquets pour macOS, qui permet d'installer facilement différentes applications Unix.

Pour l'installer, il suffit d'exécuter la commande indiquée sur le [site officiel](http://brew.sh):

```none
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

Si vous ne les avez pas déjà, macOS vous proposera d'installer préalablement les outils en ligne de commande *Xcode*.


## *Nginx*

Bien qu'une version d'*Apache* soit nativement fournie avec macOS, nous nous proposons ici d'installer *Nginx*, particulièrement léger et facilement configurable.

### Installation

Pour installer et lancer *Nginx* au démarrage, on utilise:

```none
brew install nginx
sudo brew services start nginx
```

Bien qu'il ne faille pas utiliser `sudo` avc `brew install`, il est nécessaire de l'utiliser pour lancer *Nginx* afin de pouvoir l'utiliser sur le port 80. 

### Configuration 

Nous souhaitons pouvoir stocker notre site internet dans le dossier de notre choix, et y accéder à l'URL `http://localhost/`. Pour cela, on édite le fichier de configuration :

```none
nano /usr/local/etc/nginx/nginx.conf
```
To begin, we will have to give to *Nginx* the permission to access our files and avoid a nasty `403 Forbidden` error. To do so, we change the first line, where `<user>` is your username:

Pour commencer, il va nous falloir donner à *Nginx* la permission d'accéder aux fichiers, pour éviter une erreur `403 Forbidden`. Pour ce faire, on change la première ligne, où `<user>` est votre nom d'utilisateur :

```nginx
user <user> staff;
```

Ensuite, pour ajouter un nouveau site, on ajoute une nouvelle section dans la partie `http` du fichier :

```nginx
server {
  listen 80;
  server_name localhost;
  root /Users/<user>/Documents/chemin/vers/votre/site;
  index index.html index.htm;
}
```

Ensuite, nous relançons *Nginx* pour que les changements soient pris en compte :
```none
sudo brew services restart nginx
```


## PHP

Pour utiliser PHP avec *Nginx*, nous allons utiliser PHP-FPM. Nous allons ici utiliser PHP 7.2, mais il est facile d'obtenir d'autres versions :

```none
brew install php72
```

Ensuite, on édite à nouveau le fichier de configuration :

```none
nano /usr/local/etc/nginx/nginx.conf
```

On modifie la ligne commençant par `index` par :

```nginx
index index.php index.html;
```

Enfin, on ajoute dans la rubrique `server` les lignes suivantes pour faire fonctionner PHP pour tous les fichiers ayant l'extension `.php` :


```nginx
location ~ \.php {
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
    fastcgi_pass 127.0.0.1:9000;
    fastcgi_split_path_info ^(.+\.php)(/.+)$;
    fastcgi_buffers 16 16k;
    fastcgi_buffer_size 32k;
}
```

Pour éviter l'erreur `File not found.`, il faut également donner les droits d'accès à PHP-FPM. Pour cela, on édite le fichier suivant :

```none
nano /usr/local/etc/php/7.2/php-fpm.d/www.conf
```

On modifie les paramètres suivants :

```apache
user = <user>
group = staff
```

Enfin, on redémarre *Nginx* pour activer ces changements, et on lance bien PHP-FPM pour éviter un  `502 Bad Gateway`:

```none
sudo brew services restart nginx
sudo brew services start php72
```
## MySQL

On installe et lance désormais MariaDB pour obtenir une base MySQL :
```none
brew install mariadb
brew services start mariadb
```

Enfin, on finalise l'installation en choisissant un mot de passe `root` pour MySQL :

```none
mysql_secure_installation
```

Et voici une installation MAMP idéale !
