---
title: Setting up a <em>Nginx</em> <br/>web server on macOS
original: 2016-01-05
redirect: ['/setting-up-an-nginx-web-server-on-os-x/', '/setting-up-an -nginx-web server-on-os-x/', '/setting-up-a-nginx-web-server-on-os-x/']
---

Seeking a satisfactory solution to create a local web server for programming in macOS with PHP and MySQL, I was disappointed that the turnkey solutions were far from equaling the WAMP that may exist on Windows.

But I forgot macOS is a Unix system, and unlike Windows, it's perfectly possible to create a customized local server with some packages.

We will see how to install *Nginx*, PHP-FPM and *MariaDB* (MySQL) on macOS *High Sierra* thanks to *Homebrew* package manager.

## *Homebrew*

*HomeBrew* is a package manager for macOS, that allows to easily install various Unix applications.

To install, simply execute the command shown on the [official website](http://brew.sh):

```none
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

If you do not already have them, macOS will prompt you to first install Xcode Command Line Tools.


## *Nginx*

Although *Apache* is natively included with macOS, we propose here to install *Nginx*, particularly lightweight and easily configurable.

### Installation

To install and launch *Nginx* on startup, we use:

```none
brew install nginx
sudo brew services start nginx
```

Although we musn't use `sudo` with `brew install`, it is necessary to use it to launch *Nginx* if we want to use the the default port 80.


### Configuration

We want to store our web site in the folder of our choice, and access to the URL `http://localhost/`. To do this, edit the configuration file:

```none
nano /usr/local/etc/nginx/nginx.conf
```

To begin, we will have to give to *Nginx* the permission to access our files and avoid a nasty `403 Forbidden` error. To do so, we change the first line, where `<user>` is your username:

```nginx
user <user> staff;
```

Then, to add a new website, we will add a new section inside the `http` directive:

```nginx
server {
  listen 80;
  server_name localhost;
  root /Users/<user>/Documents/path/to/your/website;
  index index.html index.htm;
}
```

We then restart *Nginx* in order to take this changes into account:
```none
sudo brew services restart nginx
```

## PHP

In order to use PHP with *Nginx* we will use PHP-FPM.
Here, we will use PHP 7.2, but you can easily choose any other version:

```none
brew install php72
```

Then, we re-edit the configuration file:

```none
nano /usr/local/etc/nginx/nginx.conf
```

We modify the line starting with `index` by:

```nginx
index index.php index.html;
```

Finally, add in the section `server` the following lines to run PHP for all files with the extension` .php`:

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

To avoid a `File not found.` error, we also need to give the right permissions to PHP. In the following file:

```none
nano /usr/local/etc/php/7.2/php-fpm.d/www.conf
```

Change the following parameter to:

```apache
user = <user>
group = staff
```

At last, we restart *Nginx* to activate the changes, and we don’t forget to launch PHP, to avoid a `502 Bad Gateway`:

```none
sudo brew services restart nginx
sudo brew services start php72
```

## MySQL

We will now install and launch MariaDB:

```none
brew install mariadb
brew services start mariadb
```

Finally, complete the installation by choosing a `root` password for MySQL:

```none
mysql_secure_installation
```

Here is the perfect MAMP installation !
