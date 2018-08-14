---
title: Rendre <em>Jekyll</em> <br/> multilingue
original: 2014-02-23
redirect: /site-multilingue-avec-jekyll/
categories: Jekyll
---

J<em>ekyll</em> laisse une grande liberté de choix en permettant de mettre simplement en place des fonctionnalités qui ne sont pas prévues par son moteur. C'est notamment le cas lorsque l'on souhaite proposer son site en plusieurs langues : alors que la plupart des CMS sont très rigides ou nécessitent des plugins, quelques filtres suffisent ici pour obtenir le résultat désiré.

Cet article a pour objectif de présenter une façon de créer un site multilingue avec *Jekyll*. Il suppose que celui-ci est bien installé[[l'article [Site statique avec *Jekyll*](/site-statique-avec-jekyll/) décrit comment installer et utiliser *Jekyll* pour obtenir un site simple]] et que vous savez l'utiliser pour générer un site simple.

Vous pourrez trouver un [exemple minimal de site multilingue sous *Jekyll*](https://github.com/sylvaindurand/multilingual-jekyll) hébergé sur [GitHub](https://github.com/sylvaindurand/multilingual-jekyll), basé sur cet article et prêt à l'emploi.

## Objectifs

La méthode permettra d'obtenir un site entièrement multilingue, tout en restant très flexible pour permettre à chacun d'en faire l'utilisation qu'il souhaite.

### Une grande flexibilité

Notre site pourra être traduit en autant de langues que souhaité. Chaque page pourra elle-même être traduite, ou non, dans une ou plusieurs langues[[quelle que soit la langue, il peut exister des pages qui ne sont pas forcément traduites dans toutes les autres]]. L'ensemble des pages et articles pourra être organisé sans contrainte.

### Une traduction complète

Quelle que soit la page affichée, l'intégralité du contenu doit être dans la même langue. Ce sera le cas des contenus, mais aussi du menu, de la date, des flux, ou encore de la typographie.

### Sélecteur de langue

Un sélecteur de langue tel que celui présent en haut à droite de ce site permettra pour chaque page d'indiquer la langue en cours et ses traductions disponibles[[les différents liens de ce sélecteur doivent renvoyer vers la traduction de la page en cours, et non vers la page d'accueil traduite]].

### Indexation par les moteurs de recherche

Les liens entre les différentes versions d'un même contenu doivent être connus des moteurs de recherche, qui pourront directement proposer aux utilisateurs un contenu dans leur langue.

### Aucun plugin

Tout cela fonctionnera sans plugin, afin d'assurer une meilleure compatibilité avec les futures versions de *Jekyll* et de pouvoir héberger le site sur [GitHub Pages](https://pages.github.com/).


## Principe

La façon de procéder est particulièrement simple : nous allons indiquer, pour chaque article, sa langue (`lang`) et un identifiant unique (`ref`) permettant de faire le lien entre les différentes traductions. *Jekyll* se chargera du reste !

Pour cela, on utilise des variables `lang` et `ref` dans l'entête de chaque article et de chaque page. Par exemple, pour l'anglais :

```markdown
---
title: Hello world!
lang: en
ref: hello
---
```

Puis en français :

```markdown
---
title: Bonjour le monde !
lang: fr
ref: hello
---
```

Et enfin, en chinois :

```markdown
---
title: 你好，世界！
lang: zh
ref: hello
---
```

## Liens entre les traductions

### Liste des articles par langue

Les pages affichant la liste des articles ne doivent afficher que ceux qui sont dans la bonne langue, ce qui peut être atteint facilement grâce à la métadonnée `lang`.

Le code suivant permet d'afficher les articles de la même langue que la page en cours :

{% raw %}
```html
{% assign posts=site.posts | where:"lang", page.lang %}
<ul>
{% for post in posts %}
    <li>
        <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
{% endfor %}
</ul>
```
{% endraw %}

Il est également possible d'avoir des comportements plus évolués, comme afficher les articles d'une autre langue (par défaut) lorsqu'ils n'existent pas dans la langue actuelle. Vous trouverez [ici](https://github.com/sylvaindurand/jekyll-multilingual/issues/4) un exemple.

Malheureusement, il n'existe pas de solution pour obtenir une pagination des articles selon la langue.

### Sélecteur de langue

Pour créer un sélecteur de langue, comme celui présent en haut à droite de cette page, la démarche est très similaire à celle présentée au paragraphe précédent. On affiche la langue de toutes les versions existantes, y compris l'article en cours, en triant par dossier pour toujours avoir le même ordre :

{% raw %}
```html
<ul>
{% assign posts=site.posts | where:"ref", page.ref | sort: 'lang' %}
{% for post in posts %}
  <li>
    <a href="{{ post.url }}" class="{{ post.lang }}">{{ post.lang }}</a>
  </li>
{% endfor %}

{% assign pages=site.pages | where:"ref", page.ref | sort: 'lang' %}
{% for page in pages %}
  <li>
    <a href="{{ page.url }}" class="{{ page.lang }}">{{ page.lang }}</a>
  </li>
{% endfor %}
</ul>
```
{% endraw %}

Comme vous pouvez le voir, nous devons répéter le code : une fois pour les pages (`site.page`) et une fois pour les articles (`site.articles`). Il sera possible de réduire le code lorsque *Jekyll* prendra en charge *Liquid* 4.

Pour emphaser la langue de la version affichée, il suffit d'utiliser CSS[[pour cela, il faut bien déclarer l'attribut `lang` de la balise `html` en indiquant `<html lang="{{ page.lang }}">` dans le *layout*]]. Par exemple, pour la mettre en gras :

```css
.en:lang(en), .fr:lang(fr), .zh:lang(zh){
    font-weight: bold;
}
```

### Liens vers les articles précédents et suivants

La même logique de boucle peut être utilisée pour afficher des liens vers l'article précédent et l'article suivant, depuis chaque article.

Pour l'article précédent :

{% raw %}
```html
{% for post in site.posts %}
  {% if post.lang == page.lang %}
    {% if prev %}
      <a href="{{ post.url }}">Précédent</a>
    {% endif %}
    {% assign prev = false %}
    {% if post.id == page.id %}
      {% assign prev = true %}
    {% endif %}
  {% endif %}
{% endfor %}
```
{% endraw %}

Et pour l'article suivant :


{% raw %}
```html
{% for post in site.posts reversed %}
  {% if post.lang == page.lang %}
    {% if next %}
      <a href="{{ post.url }}">Suivant</a>
      {% break %}
    {% endif %}
    {% assign next = false %}
    {% if post.id == page.id %}
      {% assign next = true %}
    {% endif %}
  {% endif %}
{% endfor %}
```
{% endraw %}


## Peaufinage

### Traduction des éléments du site
En dehors du contenu des articles, il est également nécessaire de traduire les différents éléments qui composent le site : textes des menus, du haut et de bas de page, certains titres...

Pour cela, on peut indiquer des traductions dans `_config.yml`[[depuis la deuxième version de *Jekyll*, il est également possible de placer ces informations dans le dossier [_data](http://jekyllrb.com/docs/datafiles/)]]. Ainsi, dans l'exemple suivant, {% raw %}`{{ site.t[page.lang].home }}`{% endraw %} génèrera `Home`, `Accueil` ou `首页` selon la langue de la page :

```yml
t:
  en:
    home:  "Home"
  fr:
    home:  "Accueil"
  zh:
    home:  "首页"
```

Il est possible d'utiliser cette technique pour générer les menus des différentes versions du site. Ainsi, dans le cas d'un menu à deux éléments comme sur ce site, on indique dans `_config.yml` :

```yml
t:
  en:
    home:
      name: "Home"
      url: "/"
    about:
      name: "About"
      url: "/about/"
  fr:
    home:
      name: "Accueil"
      url: "/accueil/"
    about:
      name: "À propos"
      url: "/a-propos/"
  zh:
    home:
      name: "首页"
      url: "/首页/"
    about:
      name: "关于"
      url: "/关于/"
```

Le menu peut alors être généré à l'aide d'une boucle :

{% raw %}
```html
<ul>
  {% for menu in site.t[page.lang] %}
    <li><a href="{{ menu[1].url }}">{{ menu[1].name }}</a></li>
  {% endfor %}
</ul>
```
{% endraw %}

### Traduction des dates
À ce stade, tout peut être traduit sur le site à l'exception des dates,  générées automatiquement par *Jekyll*. Les formats courts, composés uniquement de chiffres, peuvent être adaptés sans difficulté. Selon la langue de la page, nous cherchons à obtenir :

- en anglais : "2014/09/01" ;
- en français : "01/09/2014" ;
- en chinois : "2014年9月1号".

Pour cela, il suffit d'utiliser le code suivant, qu'il est possible ensuite de placer dans le dossier `_includes` afin de l'utiliser à plusieurs reprises :

{% raw %}
```liquid
{% if page.lang == 'en' %}
    {{ page.date | date: "%d/%m/%Y" }}
{% endif %}

{% if page.lang == 'fr' %}
    {{ page.date | date: "%Y-%m-%d" }}
{% endif %}

{% if page.lang == 'zh' %}
    {{ page.date | date: "%Y年%-m月%-d号" }}
{% endif %}
```
{% endraw %}

Pour les dates longues, il est possible d'utiliser astucieusement les filtres de date et les remplacements pour obtenir n'importe quel format. Par exemple, si nous cherchons à obtenir :

- en anglais : "1<sup>st</sup> March 2016" ;
- en français : "1<sup>er</sup> mars 2016".

Nous utilisons alors le code suivant, que l'on place dans un fichier `date.html` placé dans le dossier `_includes` :

{% raw %}
```liquid
{% assign day = include.date | date: "%-d" %}
{% if page.lang != 'fr' %}
    {% case day %}
        {% when '1' or '21' or '31' %} <sup>st</sup>
        {% when '2' or '22' %} <sup>nd</sup>
        {% when '3' or '23' %} <sup>rd</sup>
        {% else %} <sup>th</sup>
    {% endcase %}
{% else %}
    {% if day == "1" %}
        <sup>er</sup>
    {% endif %}
{% endif %}

{% if page.lang != 'fr' %}
    {{ include.date | date: "%B" }}
{% else %}
    {% assign m = include.date | date: "%-m" %}
    {% case m %}
            {% when  '1' %}janvier
            {% when  '2' %}février
            {% when  '3' %}mars
            {% when  '4' %}avril
            {% when  '5' %}mai
            {% when  '6' %}juin
            {% when  '7' %}juillet
            {% when  '8' %}août
            {% when  '9' %}septembre
            {% when '10' %}octobre
            {% when '11' %}novembre
            {% when '12' %}décembre
    {% endcase %}
{% endif %}

{{ include.date | date: "%Y" }}
```
{% endraw %}

Pour afficher la date, il n'y a alors plus qu'à appeler :
{% raw %}
```liquid
{% include date.html date=page.date %}
```
{% endraw %}


## Accès au site et référencement

Les pages étant entièrement statiques, il est difficile de deviner la langue de nos visiteurs pour envoyer la bonne version, que ce soit en détectant les entêtes envoyées par le navigateur ou en se basant sur sa localisation géographique. Néanmoins, il est possible d'améliorer le référencement en indiquant aux moteurs de recherche les pages qui constituent les traductions d'un seul et même contenu[[ainsi, les utilisateurs trouvant notre contenu par un moteur de recherche devraient se voir proposer automatiquement la bonne traduction]].

Pour ce faire, deux solutions sont possibles : [intégrer une balise `<link>`](https://support.google.com/webmasters/answer/189077?hl=fr) dans notre page, ou [l'indiquer dans un fichier `sitemaps.xml`](https://support.google.com/webmasters/answer/2620865?hl=fr).

### Avec la balise *link*

Il suffit d'indiquer dans la partie `<head>` chaque page, l'ensemble des traductions de la page en cours[[il convient cependant de faire attention d'utiliser les bons [identifiants de langue](https://support.google.com/webmasters/answer/189077?hl=fr) pour qu'ils soient reconnus]]. Nous pouvons pour cela d'utiliser le code suivant, semblable à ceux présentés précédemment :

{% raw %}
```html
{% assign posts=site.posts | where:"ref", page.ref | sort: 'lang' %}
{% for post in posts %}
  <link rel="alternate" hreflang="{{ post.lang }}" href="{{ post.url }}" />
{% endfor %}
{% assign pages=site.pages | where:"ref", page.ref | sort: 'lang' %}
{% for page in pages %}
  <link rel="alternate" hreflang="{{ page.lang }}" href="{{ page.url }}" />
{% endfor %}
```
{% endraw %}

### Avec un fichier *sitemaps*

Le fichier `sitemaps.xml`, qui permet aux moteurs de recherche de connaître les pages et la structure de votre site, [permet également d'indiquer aux moteurs de recherches quelles pages sont les différentes versions d'un même contenu](https://support.google.com/webmasters/answer/2620865?hl=fr).

Pour cela, il suffit d'indiquer l'intégralité des pages du site (quelle que soit leur langue) dans des éléments `<url>` et pour chacun d'entre eux l'ensemble des versions qui existent, y compris celle que l'on est en train de décrire.

Ce fichier peut être généré automatiquement par *Jekyll* en créant un fichier `sitemaps.xml` à la racine du site contenant :

{% raw %}
```xml
---
layout:
permalink: /sitemaps.xml
---
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9" xmlns:xhtml="http://www.w3.org/1999/xhtml">
  {% for post in site.posts %}
    {% if post.id contains "404" %}{% else %}
      <url>
        <loc>{{site.base}}{{ post.url }}</loc>
        {% assign versions=site.posts | where:"ref", post.ref %}
        {% for version in versions %}
          <xhtml:link rel="alternate" hreflang="{{ version.lang }}" href="{{site.base}}{{ version.url }}" />
        {% endfor %}
        <lastmod>{{ post.date | date_to_xmlschema }}</lastmod>
        <changefreq>weekly</changefreq>
      </url>
    {% endif %}
  {% endfor %}
  {% for page in site.pages %}
    {% if page.id contains "404" %}{% else %}
      <url>
        <loc>{{site.base}}{{ page.url }}</loc>
        {% assign versions=site.pages | where:"ref", page.ref %}
        {% for version in versions %}
          <xhtml:link rel="alternate" hreflang="{{ version.lang }}" href="{{site.base}}{{ version.url }}" />
        {% endfor %}
        <changefreq>weekly</changefreq>
      </url>
    {% endif %}
  {% endfor %}
</urlset>
```
{% endraw %}
