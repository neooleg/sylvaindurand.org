---
title: <em>Wildcard</em> certificates<br/> with Let’s Encrypt
---

Launched in late 2015, *[Let’s Encrypt](https://letsencrypt.org)* is a public benefit organization which democratized the use of HTTPS by providing free SSL certificates with an automated validation system.


After several months of testing, Let's Encrypt has launched the second version of its client ([ACME v2](https://community.letsencrypt.org/t/acme-v2-production-environment-wildcards/55578)) with a highly anticipated feature : you can obtain wildcard certificates, valid for all subdomains of one domain.

This possibility is particularly interesting when using many subdomains: so far, it was necessary to issue a new certificate to add a new subdomain, or to delete an old one. Here, a simple `*.domain.tld` is enough!


### Installation

To get our certificates, we will use the `certbot` client. The site offers several [installation methods](https://certbot.eff.org) depending on your platform, but version 0.22.0 may not be available. By default, `certbot` can be installed manually:


```none
cd /opt/
sudo wget https://dl.eff.org/certbot-auto
sudo chmod a+x certbot-auto
```

### Issuing a certificate

For now, the `certbot-auto` utility allows you to request certificates wildcard provided you specify that you use the latest version of the authentication server.


For now, the `certbot-auto` utility allows you to request wilcard certificates, but you need to specify the last version of the server.

Be careful, if you want a certificate for both the domain root (`domain.tld`) and its subdomains (`*.domain.tld`), both must be specified. With the `-d` parameter, it is possible to list the desired domains and subdomains:

```none
sudo /opt/certbot-auto certonly \
    --server https://acme-v02.api.letsencrypt.org/directory \
    --manual -d *.domain.tld -d domain.tdl
```

### Validation

*Let's encrypt* will now have to ask us to prove that we have control over the domain names requested.

For domains (`domain.tld`) or specific subdomains (`a.domain.tld`), *Let's encrypt* asks to create a file containing specific text accessible from a given URL, which is normally easily achievable with your server:

```
-------------------------------------------------------------------------------
Create a file containing just this data:

Um8Qj2-lQUSgOXsGRccbzqYFz0rsLmBDf-HXL3p2p_k.vdPe0

And make it available on your web server at this URL:

http://domain.tld/.well-known/acme-challenge/Um8Qj2-lQqYFz0rsLmBDUS
-------------------------------------------------------------------------------
Press Enter to Continue
```

For wildcard certificates, *Let's encrypt* must verify that you have the full domain name, and request the creation of a specific TXT record in the DNS zone of the domain name, which can be done from your registar:

```
-------------------------------------------------------------------------------
Please deploy a DNS TXT record under the name
_acme-challenge.domain.tld with the following value:

c81US66r6JVk1LwyFHbzINQvIU_m5gJWXgcUm8Qj2

Before continuing, verify the record is deployed.
-------------------------------------------------------------------------------
Press Enter to Continue
```

Once created, the certificate is located in `/etc/letsencrypt/live/domain.tld`.

### Automatic renewal

The certificate is valid for three months, before having to be renewed. To do so, we create a cron task:

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
