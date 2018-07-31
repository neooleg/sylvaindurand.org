---
title: Authentification <br/>par clef avec SSH
---

Lorsque l'on souhaite se connecter à un serveur *via* SSH, l'authentification repose par défaut sur un nom d'utilisateur (`user`) et un mot de passe (`passphrase`). Cependant, les mots de passe sont à la fois très peu sécurisés, difficiles à retenir et pénibles à écrire : ils ne sont efficaces face aux ordinateurs qu'à la condition d'être très contraignants, voire inutilisables, pour les humains.

L'authentification par clef permet de concilier ces deux exigences : elle permet une très forte sécurité et de se connecter  simplement. Cette authentification fonctionne grâce à trois composants :

* une clef publique que l'on fournit préalablement au serveur ;
* une clef privée, que l'on conserve pour soi et qui permet de prouver son identité au serveur lors de la connexion ;
* un mot de passe *(facultatif)*, qui sera demandé à chaque utilisation de la clef privée.


## Génération de la clef

Dans un premier temps, nous allons générer en local une clef SSH privée et la clef publique associée :

```
cd ~/.ssh
```

L'algorithme `ed25519` apparaît à ce jour être l'un des plus sécurisés, tout en restant très rapide :

```
ssh-keygen -t ed25519
```

Néanmoins, ce dernier est encore récent et n'est pas supporté sur tous les systèmes. Dans ce cas, il est possible d'utiliser le RSA :

```
ssh-keygen -t rsa -b 4096
```

Lorsqu'il nous est demandé où placer la clef, saisissons simplement le nom d'utilisateur `<user>`.

Il est ensuite proposé de saisir un mot de passe pour l'associer à la clef, pour empêcher son utilisation par un tiers si elle était diffusée. Ce n'est pas nécessaire, mais si vous le faites, il vous sera demandé à chaque utilisation. Dans tous les cas, la clef privée ne doit bien sûr jamais être transmise.

Une fois cette étape accomplie, nous avons désormais :

* `~/.ssh/<user>`, votre clef privée ;
* `~/.ssh/<user>.pub`, la clef publique associée.


## Transfert sur le serveur

Pour que le serveur nous reconnaisse, nous devons lui transmettre la clef publique que nous venons de générer (`<user>.pub`). Son contenu doit être stocké dans `~/.ssh/authorized_keys` sur le serveur.

En local, cela peut être fait en une ligne de commande :

```
cat ~/.ssh/<user>.pub | ssh <user>@<hostname> -p <port> 'umask 0077; mkdir -p .ssh; cat >> ~/.ssh/authorized_keys'
```

## Configuration

Pour nous connecter sur le serveur très rapidement, nous allons créer en local un fichier de configuration avec ses coordonnées :

```
nano ~/.ssh/config
```

On y indique les données suivantes :

```
Host <shortcut>
  HostName <hostname>
  Port <port>
  User <user>
  IdentityFile ~/.ssh/<user>
```

Pour se connecter, il suffit désormais de taper :

```
ssh <shortcut>
```

Si vous aviez associé un mot de passe à la clef privé, celui-ci vous sera alors demandé. Dans le cas contraire, vous serez connecté directement.
