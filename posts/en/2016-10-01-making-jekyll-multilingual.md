---
title: Making <em>Jekyll</em> <br/> multilingual
original: 2014-02-23
redirect: /multilingual-website-with-jekyll/
categories: Jekyll
---

J<em>ekyll</em> has a very flexible design that allows a great freedom of choice, allowing the user to simply introduce features that are not integrated into its engine. This is particularly the case when one wants to create a multilingual website: while CMS remain very rigid and often require plugins, few filters are sufficient to achieve it with _Jekyll_.

This article aims to present a way to create a multilingual site with _Jekyll_. It has to be installed[[the article [Static website with *Jekyll*](/static-website-with-jekyll/) explains how to install and use *Jekyll* in order to get a simple website]] on your computer and you should be able to know how to generate a simple website.

You'll find a [minimal example of multilingual site in Jekyll](https://github.com/sylvaindurand/multilingual-jekyll) hosted on [GitHub](https://github.com/sylvaindurand/multilingual-jekyll), based on this article and ready to use.

## Goals

The method will provide a fully multilingual website, but will remain very flexible to allow everyone to do what he wants.

### Flexibility

Our website can be translated in as many languages as wanted. Each page may be translated or not in the various languages[[regardless of the version, they may be pages which are not translated in each language]]. All pages and posts can be placed as desired.

### Full translation

On each page, all content must be in the same language : title, article, but also menus, dates, feeds or typography.

### Language selector

A language selector, as the one on the top right of this website, will show the actual language and the different translations available[[the selector musn't lead to the translated homepage, but to the translation of the current page]].

### Search engine optimization

The links between the different versions of the same content must be known search engines, which can directly provide users with content in their language.

### No plugin

Everything will work without any plugin, in order to get a good compatibility with the future versions of *Jekyll* and to be able to host it on [GitHub Pages](https://pages.github.com/).

## Principle

The method is very simple: we will indicate, for each post and page, its language (`lang`) and an unique identifier (`ref`) to link the different translations. *Jekyll* will do the rest!

To do so, we will use `lang` et `ref` in the frontmatter of each post and page. For instance, in English:

```markdown
---
title: Hello world!
lang: en
ref: hello
---
```

Then, in French:

```markdown
---
title: Bonjour le monde !
lang: fr
ref: hello
---
```

Or, in Chinese:

```markdown
---
title: 你好，世界！
lang: zh
ref: hello
---
```

## Links between the two translations

### List of articles of the same language

The page listing the articles has to display only the one with the good translation. That is easy to do, thanks to the `lang` metadata.

The following code provides every article with the same language than the actual page:

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

It is also possible to achieve more complex behaviours, like showing the article in another (default) language if it's missing in one another. [Here is a way](https://github.com/sylvaindurand/jekyll-multilingual/issues/4) to do so.

Unfortunately, there is currently no way to paginate the articles, without any plugin or Javascript.

### Language selector

To create a language selector, like the one at the top right of this page, the process is very similar. We show the language of each translation available, including the actual page, sorting by path in order to always get the same order:

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

As you can see, we need to repeat the code for the pages (`site.page`) and the posts (`site.articles`). It will be possible to delete this redundancy when *Jekyll* will use *Liquid* 4.

Then, in order to emphase the actual version, just use CSS[[you need to declare the `lang` attribute on the `html`, with `<html lang="{{ page.lang }}">` in the *layout*]]. For instance, if you want to bold it:

```css
.en:lang(en), .fr:lang(fr), .zh:lang(zh){
    font-weight: bold;
}
```

### Links to previous and next posts

Using loops can also allow us to show the previous and the next posts:
{% raw %}
```html
{% for post in site.posts %}
  {% if post.lang == page.lang %}
    {% if prev %}
      <a href="{{ post.url }}">Previous</a>
    {% endif %}
    {% assign prev = false %}
    {% if post.id == page.id %}
      {% assign prev = true %}
    {% endif %}
  {% endif %}
{% endfor %}
```
{% endraw %}

And:

{% raw %}
```html
{% for post in site.posts reversed %}
  {% if post.lang == page.lang %}
    {% if next %}
      <a href="{{ post.url }}">Next</a>
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
## Tweaking

### Translation of website elements
Around the articles, it is also necessary to translate the various elements like menus, header, footer, some titles...

To do so, we can provide translations into `_config.yml`[[since *Jekyll* 2.0, it is also possible to put the translations in the [_data](http://jekyllrb.com/docs/datafiles/) folder]]. Then, in the following example, {% raw %}`{{ site.t[page.lang].home }}`{% endraw %} will generate `Home`, `Accueil` or `首页` depending of the page language:

```yml
t:
  en:
    home:  "Home"
  fr:
    home:  "Accueil"
  zh:
    home:  "首页"
```

It is possible to do the same in order to generate the menu of each version. For example, if you want to provide a two elements menu, you just have to provide in `_config.yml` :

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

Then, you can generate the menu with a simple loop:

{% raw %}
```html
<ul>
  {% for menu in site.t[page.lang] %}
    <li><a href="{{ menu[1].url }}">{{ menu[1].name }}</a></li>
  {% endfor %}
</ul>
```
{% endraw %}

### Translation of dates
At this point, everything can be translated on the site except the dates automatically generated by *Jekyll*. Short formats, consisting only of numbers, can be adapted without difficulty. Depending of the language, we want to get:

- in English : "2014/09/01" ;
- in French : "01/09/2014" ;
- in Chinese : "2014年9月1号".

To do so, we just have to use the following code, which we may then put in the `_includes` folder in order to use it when needed:

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

For the long format dates, it is possible to use date filters and replacements for any format. For example, we want to get:

- in English : "1<sup>st</sup> March 2016" ;
- in French : "1<sup>er</sup> mars 2016".

To do so, we just have to put the following code in a file named `date.html` stored in the `_includes`:

{% raw %}
```liquid
{{ include.date | date: "%-d" }}

{%- assign day = include.date | date: "%-d" -%}
{%- if page.lang != 'fr' -%}
    {%- case day -%}
        {%- when '1' or '21' or '31' -%} <sup>st</sup>
        {%- when '2' or '22' -%} <sup>nd</sup>
        {%- when '3' or '23' -%} <sup>rd</sup>
        {%- else -%} <sup>th</sup>
    {%- endcase -%}
{%- else -%}
    {%- if day == "1" -%}
        <sup>er</sup>
    {%- endif -%}
{%- endif %}

{% if page.lang != 'fr' -%}
    {{ include.date | date: "%B" }}
{%- else -%}
    {%- assign m = include.date | date: "%-m" -%}
    {%- case m -%}
            {%- when  '1' %}janvier
            {%- when  '2' %}février
            {%- when  '3' %}mars
            {%- when  '4' %}avril
            {%- when  '5' %}mai
            {%- when  '6' %}juin
            {%- when  '7' %}juillet
            {%- when  '8' %}août
            {%- when  '9' %}septembre
            {%- when '10' %}octobre
            {%- when '11' %}novembre
            {%- when '12' %}décembre
    {%- endcase -%}
{%- endif %}

{{ include.date | date: "%Y" }}
```
{% endraw %}

Then, we just have to call:

{% raw %}
```liquid
{% include date.html date=page.date %}
```
{% endraw %}


## Website access and search engine

The website is completely static, so it is difficult to know the language of our visitors, either by detecting the headers sent by the browser or on the basis of their geographical location. Nevertheless, it is possible to indicating the search engines which pages are translations of the same content[[thus, users finding our website through a search engine should be offered the good translation]].

To do so, two ways are possible: [use `<link>`](https://support.google.com/webmasters/answer/189077?hl=en) or [create a `sitemaps.xml` file](https://support.google.com/webmasters/answer/2620865?hl=en).

### With a *link* tag

You only have to provide in the `<head>` part of the page, every translation available of the actual version[[you need to be careful to use the good [country codes](https://support.google.com/webmasters/answer/189077?hl=en)]]. To do so, we can use the following code, similar to those used previously:

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

### With a *sitemaps* file

The `sitemaps.xml` file,  which allows search engines to know the pages and the structure of your website, [also helps tell the search engines which pages are different translations of the same content](https://support.google.com/webmasters/answer/189077?hl=en).

For this, just indicate all pages of the site (regardless of language) in `<url>` elements, and for each of them all the versions that exist, including the one we are now describing.

This file can be generated automatically by *Jekyll* with the following code, which will create a `sitemaps.xml` file on the root of the website:

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
