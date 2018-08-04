---
title: Getting a git server <br/>with <em>Gitolite</em>
---

Github helped popularize *git* and its concepts to many. Nevertheless, it involves making public all the codes you want to host, unless you buy a fairly expensive subscription, and for a limited number of projects. Moreover, Github is not open, and it is therefore necessary to give it some trust to store its projects.

Several solutions exist to host a *git* server on his personal server. For example, [*Gitlab*](https://about.gitlab.com/) aims to become equal to Github, with a graphical interface and many tools; thereby, it is very large and relatively heavy for a small server.

Here we will see how to install [*Gitolite*](http://gitolite.com/gitolite/index.html), which provides a very light and functional *git* server, in order to synchronizing repositories between different working stations.

## Installation

### Creating *git* user on the server

First, we will create on the server a `git` user, which will be needed:

```none
sudo adduser git
```

Once connected to `git` on the server, we create a `bin` directory that will contain binaries, that we then add to the `path`:

```none
cd ~
mkdir bin
PATH=$PATH:~/bin
PATH=~/bin:$PATH
```

### Creating an authentication key

In order to connect to *Gitolite*, we will used SSH key-based authentication, both simpler and more secure than HTTP. If you do not already have a key, it is necessary to create one. For this, we will used `ssh-keygen` locally:

```none
cd ~/.ssh
ssh-keygen -t ed25519
```

When the path is requested, enter your username `<user>`. The password is optional: if you choose one, it will be asked each time a *git* command is requesting the server.

We then send the public key on the server:

```none
cat ~/.ssh/<user>.pub | ssh git@<hostname> -p <port> 'umask 0077; mkdir -p .ssh; cat >> <user>.pub'
```

Finally, we create a `~/.ssh/config` file that contains the following information:

```
Host git
  HostName <hostname>
  Port <port>
  User git
  IdentityFile ~/.ssh/<user>
```


### Installing *Gitolite*

Back to the user `git` on the server, we can now install *Gitolite*:

```none
cd ~
git clone git://github.com/sitaramc/gitolite
gitolite/install -ln
gitolite setup -pk ~/<user>.pub
```

Locally, you can now check that everything works well with `ssh git`. This should return something like:

```none
PTY allocation request failed on channel 0
hello <user>, this is git@<hostname> running
gitolite3 v3.6.5-9-g490b540 on git 2.1.4

 R W    gitolite-admin
 R W    testing
```

That's all ! Now, if you want to clone a `repo` directory, simply run the command `git clone git:repo`. The `git push`,` pull`, `fetch` ... will operate without password.


## Configuration

The main originality of *Gitolite* is that its configuration system uses a specific *git* repository. To configure *Gitolite*, simply clone the `gitolite-admin` repository:

```none
git clone git:gitolite-admin
```

This repository contains a `conf/gitolite.conf` file that lists the directories and permissions, and a `keydir` folder containing the users public keys (your `<user>.pub` key, provided during the installation, is already there).

For example, to add a `project` repository, just edit `conf/gitolite.conf` to add:

```
repo project
    RW+     =   <user>
```

To apply all changes, `commit` then `push` those files on the server. Don't hesitage to check the very complete [*Gitolite* documentation](http://gitolite.com/gitolite/basic-admin.html).
