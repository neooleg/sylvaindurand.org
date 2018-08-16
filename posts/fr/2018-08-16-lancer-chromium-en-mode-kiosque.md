---
title:  Lancer <em>Chromium</em> <br/> en mode kiosque
categories: Raspberry Pi
---

Chromium permet facilement d'être démarré en mode "kiosque", c'est-à-dire de se lancer en plein écran, sans aucune bordure de fenêtre, barre d'outils ni notification[[de façon surprenante, cette fonctionnalité n'est pas offerte de base par Mozilla Firefox]].

Cela permet notamment d'afficher une présentation, ou une page, sur un afficheur autonome ou sur une borne... ou, dans mon cas, d'[afficher un diaporama sur ma télévision](https://github.com/sylvaindurand/reddit-slideshow) pendant que le Raspberry Pi diffuse de la musique.



Notre objectif sera d'obtenir :

* une installation *minimale* du serveur graphique, sans environnement de bureau ni gestionnaire de fenêtre ;
* un lancement automatique au démarrage, en plein écran ;
* aucun affichage de barre d'outils ou de notification.


## Serveur d'affichage

### Installation

Pour afficher le navigateur, nous allons devoir installer un [serveur X](https://fr.wikipedia.org/wiki/X_Window_System). Il n'y a pas besoin d'installer d'environnement de bureau ni gestionnaire de fenêtre, inutilement volumineux, puisque le navigateur sera directement lancé en plein écran.


```none
sudo apt-get install xserver-xorg-video-all xserver-xorg-input-all xserver-xorg-core xinit x11-xserver-utils
```

### Lancement au démarrage

Pour démarrer automatiquement le serveur au démarrage, on modifie le fichier `~/.bash_profile`, qui est exécuté lors de la connexion de l'utilisateur, pour y mettre le contenu suivant[[le serveur se démarre avec `startx`, mais il est également nécessaire de vérifier qu'un écran est bien disponible pour éviter une erreur, par exemple, *via* SSH]] :

```bash
if [[ -z $DISPLAY ]] && [[ $(tty) = /dev/tty1 ]]; then
    startx
fi
```

Votre système devra également être configuré pour que l'utilisateur soit automatiquement connecté au démarrage. La façon de procéder dépend de votre configuration[[voir notamment [ici](https://unix.stackexchange.com/questions/401759/automatically-login-on-debian-9-2-1-command-line) pour Debian, [ici](https://askubuntu.com/questions/175248/how-to-autologin-without-entering-username-and-passwordin-text-mode) pour Ubuntu ou encore [ici](https://unix.stackexchange.com/questions/42359/how-can-i-autologin-to-desktop-with-systemd) pour Arch]]. Sur *Raspbian*, on lance l'utilitaire `raspi-config`, puis on choisit `Boot Options` et enfin `Console Autologin`.


## Chromium

### Installation

Nous allons installer, bien sûr, Chromium, mais aussi `unclutter`, qui nous permettra de masquer le pointeur de la souris :

```none
sudo apt-get install chromium-browser
sudo apt-get install unclutter
```

### Lancement au démarrage

Pour les démarrer automatiquement au démarrage, nous créons un fichier `~/.xinitrc`[[ce fichier est exécuté au lancement du serveur X]] qui contient les commandes suivantes (en prenant soin de choisir votre URL et d'indiquer la résolution de l'écran) :

```bash
#!/bin/sh
xset -dpms
xset s off
xset s noblank

unclutter &
chromium-browser /path/to/your/file.html --window-size=1920,1080 --start-fullscreen --kiosk --incognito --noerrdialogs --disable-translate --no-first-run --fast --fast-start --disable-infobars --disk-cache-dir=/dev/null
```

Les commandes `xset` servent à éviter la mise en veille automatique du système, qui interrompra sinon l'affichage au bout d'une durée déterminée.

L'option `--window-size=` est indispensable, sans quoi Chromium ne s'affichera que sur une moitié de l'écran, malgré les instructions contraires, en l'absence de gestionnaire de fenêtres.

Seule limitation : Chromium peut toujours afficher une notification proposant de traduire la page si celle-ci est dans une autre langue[[l'option `--disable-translate`, qui existait auparavant pour l'éviter, a été [supprimée en 2017](https://groups.google.com/a/chromium.org/forum/#!topic/chromium-reviews/nOgks4a7_uI)]]. Pour l'éviter, il n'y a pas d'autre choix que d'éviter d'indiquer, dans le HTML dans la page, une langue différente du navigateur.
