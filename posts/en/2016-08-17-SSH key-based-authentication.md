---
title: SSH key-based <br/> authentication
---

When you want to connect to an SSH server, default authentication is based on username and password. However, passwords are quite insecure, difficult to remember and hard to write; they are effective against computers if they are very restrictive, even unusable, for humans.

The key-based authentication can fill these two requirements: it provides a very high safety and makes connection fast and easy. This authentication is based on three components:

* a public key, which is previously provided to the server;
* a private key, used to prove its identity while connecting;
* a passphrase *(optional)*, which will be requested every time the private key is provided.


## Key generation

First, we will locally create an SSH private key, and the associated public key:

```none
cd ~/.ssh
```

The algorithm `ed25519` appears so far to be one of the most secure, while remaining very fast:

```none
ssh-keygen -t ed25519
```

However, it is still new and is not supported on all systems. In this case, it is possible to use RSA instead:

```none
ssh-keygen -t rsa -b 4096
```

When the path is asked, simply write the username `<user>`.

It is then proposed to enter a password, which will be requested every time the private key is provided. It is not necessary. Of course, in any case, the private key should never be transmitted.

Once this has been accomplished, we now have:

* `~/.ssh/<user>`, your private key;
* `~/.ssh/<user>.pub`, the associated public key.


### Transfer on the server

We need to transmit the public key we just generated to the server. Its content must be stored on the server in `~/.ssh/authorized_keys`.

Locally, this can be done in one command line:

```none
cat ~/.ssh/<user>.pub | ssh <user>@<hostname> -p <port> 'umask 0077; mkdir -p .ssh; cat >> ~/.ssh/authorized_keys'
```

## Configuration

In order to be able to quickly connect to the server, we create a local configuration file:

```none
nano ~/.ssh/config
```

It will contain the following data:

```
Host <shortcut>
  HostName <hostname>
  Port <port>
  User <user>
  IdentityFile ~/.ssh/<user>
```

In order to connect to the server, we will now just have to use:

```none
ssh <shortcut>
```

If you associated a password to the private key, it will then be asked. Otherwise, you will be connected directly.
