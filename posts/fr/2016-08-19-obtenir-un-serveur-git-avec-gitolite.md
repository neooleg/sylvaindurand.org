---
title: Obtenir un serveur git <br/>avec <em>Gitolite</em>
---

Github a permis de populariser *git* et ses concepts au plus grand nombre. Néanmoins, il implique de rendre public tous les codes que vous souhaitez y héberger, sauf si vous achetez un abonnement assez coûteux, et pour un nombre limité de projets. De plus, Github n'est pas libre, et il est donc nécessaire d'y accorder une certaine confiance pour y stocker ses projets.

Plusieurs solutions existent pour héberger un serveur *git* sur son serveur personnel. Par exemple, [*Gitlab*](https://about.gitlab.com/) se pose en équivalent de Github, avec une interface graphique et de nombreux outils ; en contrepartie, il est très volumineux et relativement lourd.

Nous allons ici voir comment installer [*Gitolite*](http://gitolite.com/gitolite/index.html), qui permet d'obtenir un serveur *git* très léger et fonctionnel, permettant de synchroniser des répertoires entre différents postes de travail.


## Installation

### Création de l'utilisateur *git* sur le serveur

En premier lieu, nous allons créer sur le serveur un utilisateur `git` qui servira par la suite au fonctionnement de `gitolite`:

```
sudo adduser git
```

Une fois connecté à `git` sur le serveur, nous allons créer un répertoire `bin` qui contiendra les fichiers binaires, que nous ajoutons ensuite au `PATH` :

```
cd ~
mkdir bin
PATH=$PATH:~/bin
PATH=~/bin:$PATH
```

### Création d'une clef d'authentification

Nous nous connecterons à *Gitolite* avec une [authentification par clefs SSH](http://localhost:4000/authentification-par-clef-avec-ssh/), à la fois plus simple et plus sécurisée qu'une connexion HTTP. Si vous ne possédez pas déjà de clef, il est nécessaire d'en créer une. Pour cela, on exécute en local :

```
cd ~/.ssh
ssh-keygen -t ed25519
```

Lorsque le chemin de la clef est demandé, indiquez simplement votre nom d'utilisateur `<user>`. Le mot de passe est facultatif : si vous en choisissez un, il sera demandé à chaque commande *git* sollicitant le serveur.

Nous envoyons alors la clef publique sur le serveur :

```
cat ~/.ssh/<user>.pub | ssh git@<hostname> -p <port> 'umask 0077; mkdir -p .ssh; cat >> <user>.pub'
```

Enfin, pour nous connecter plus simplement à *git*, ajoutons les informations suivantes dans le fichier local `~/.ssh/config` :

```
Host git
  HostName <hostname>
  Port <port>
  User git
  IdentityFile ~/.ssh/<user>
```

### Installation de *Gitolite*

Connecté à l'utilisateur `git` sur le serveur, il est désormais possible installer *Gitolite* :

```
cd ~
git clone git://github.com/sitaramc/gitolite
gitolite/install -ln
gitolite setup -pk ~/<user>.pub
```

En local, vous pouvez désormais vérifier que tout fonctionne bien avec la commande `ssh git`. Celle-ci doit vous retourner quelque chose comme :

```
PTY allocation request failed on channel 0
hello <user>, this is git@<hostname> running
gitolite3 v3.6.5-9-g490b540 on git 2.1.4

 R W    gitolite-admin
 R W    testing
```

C'est tout ! Désormais, pour cloner un répertoire `repo`, il suffira d'exécuter la commande `git clone git:repo`. Les commandes `git push`, `pull`, `fetch`... fonctionneront sans mot de passe.

## Configuration

La principale originalité de *Gitolite* est que son système de configuration repose sur un répertoire *git*. Pour le configurer, il suffit de cloner le répertoire `gitolite-admin` :

```
git clone git:gitolite-admin
```

Il contient un fichier `conf/gitolite.conf`, qui permet de lister les répertoires et leurs autorisations, ainsi qu'un dossier `keydir` dans lequel nous plaçons les clefs publiques des différents utilisateurs (votre clef `<user>.pub`, fournie à l'installation, s'y trouve déjà).

Ainsi, par exemple, pour ajouter un répertoire `projet` avec tous les droits, il suffit d'éditer `conf/gitolite.conf` pour y ajouter :

```
repo projet
    RW+     =   <user>
```

Pour prendre en compte les modifications, on `commit` puis on `push` ces fichiers sur le serveur. N'hésitez pas à consulter la [documentation de *Gitolite*](http://gitolite.com/gitolite/basic-admin.html), particulièrement complète.
