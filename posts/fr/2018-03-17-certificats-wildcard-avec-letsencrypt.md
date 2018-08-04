---
title: Certificats <em>wildcard</em><br/> avec Let’s Encrypt
---

Lancé fin 2015, l'organisme à but non lucratif *[Let’s Encrypt](https://letsencrypt.org)* a permis de démocratiser l'utilisation de HTTPS en fournissant gratuitement des certificats SSL grâce à un système de validation automatisé.

Après plusieurs mois de tests, *Let’s Encrypt* vient de lancer la deuxième version de son client ([ACME v2](https://community.letsencrypt.org/t/acme-v2-production-environment-wildcards/55578)) avec une fonctionnalité très demandée : pouvoir obtenir des certificats *wildcard*, valables pour tous les sous-domaines d'un même domaine.

Cette possibilité est particulièrement intéressante lorsque l'on utilise de nombreux sous-domaines : jusqu'ici, il était nécessaire de rééditer un nouveau certificat pour ajouter un nouveau sous-domaine, ou pour en supprimer un ancien. Désormais, un simple `*.domain.tld` suffit !


### Installation

Pour obtenir nos certificats, nous allons utiliser le client `certbot`. Le site propose plusieurs [méthodes d'installation](https://certbot.eff.org) selon votre plateforme, mais la version 0.22.0 peut ne pas être disponible. À défaut, `certbot` doit être installé manuellement :


```none
cd /opt/
sudo wget https://dl.eff.org/certbot-auto
sudo chmod a+x certbot-auto
```

### Demande d'un certificat

Pour l'instant, l'utilitaire `certbot-auto` permet de demander des certificats *wildcard* à la condition de bien spécifier que l'on utilise la dernière version du serveur d'authentification.

Attention, si vous souhaitez un certificat à la fois pour la racine du domaine (`domain.tld`) et l'ensemble de ses sous-domaines (`*.domain.tld`), les deux doivent être spécifiés. Avec l'option `-d`, il est possible de lister les domaines et sous-domaines souhaités :

```none
sudo /opt/certbot-auto certonly \
    --server https://acme-v02.api.letsencrypt.org/directory \
    --manual -d *.domain.tld -d domain.tdl
```

### Validation

*Let’s encrypt* va désormais devoir nous demander de prouver que l'on possède bien le contrôle sur les noms de domaine demandés.

Pour les domaines (`domain.tld`) ou sous-domaines (`a.domain.tld`) simples, *Let's encrypt* demande de créer un fichier contenant un texte spécifique accessible depuis une URL, ce qui est normalement facilement réalisable avec votre serveur :

```
-------------------------------------------------------------------------------
Create a file containing just this data:

Um8Qj2-lQUSgOXsGRccbzqYFz0rsLmBDf-HXL3p2p_k.vdPe0

And make it available on your web server at this URL:

http://domain.tld/.well-known/acme-challenge/Um8Qj2-lQqYFz0rsLmBDUS
-------------------------------------------------------------------------------
Press Enter to Continue
```

Pour les certificats *wildcard*, *Let's encrypt* doit vérifier que vous possédez l'intégralité du nom de domaine, et demande pour cela la création d'un enregistrement TXT spécifique dans la zone DNS du nom de domaine, ce qui est réalisable depuis votre registar :

```
-------------------------------------------------------------------------------
Please deploy a DNS TXT record under the name
_acme-challenge.domain.tld with the following value:

c81US66r6JVk1LwyFHbzINQvIU_m5gJWXgcUm8Qj2

Before continuing, verify the record is deployed.
-------------------------------------------------------------------------------
Press Enter to Continue
```

Une fois créé, le certificat se situe dans `/etc/letsencrypt/live/domain.tld`.

### Renouvellement automatique

Le certificat est valable trois mois, avant de devoir être renouvelé. Pour s'en assurer, on crée une tâche `cron` :

```none
sudo crontab -e
```

We do the renewal in the night from Sunday to Monday:

```none
30 5 * * 1 /opt/certbot/certbot-auto renew
```

It is also necessary to renew its server at the same time, so that it takes into account the new certificates. If you use *nginx*:

```none
35 5 * * 1 /etc/init.d/nginx reload
```
