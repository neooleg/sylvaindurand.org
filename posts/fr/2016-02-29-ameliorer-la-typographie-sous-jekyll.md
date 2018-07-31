---
title: Améliorer la typographie <br/> sous <em>Jekyll</em>
original: 2014-10-30
redirect: /ameliorer-la-typographie-avec-jekyll/
---

Respecter les règles de typographie sur Internet n'est pas toujours évident : bien qu'Unicode réserve de nombreuses zones de caractères pour les symboles typographiques, signes de ponctuation et espaces de différentes longueurs, leur saisie difficile sur le clavier les rend pratiquement inutilisées.

Sous *Jekyll*, les articles sont rédigés très simplement au format Markdown, avant d'être généré en HTML par le moteur : c'est à ce moment qu'il nous est possible d'ajouter des règles automatiques pour améliorer la typographie sur notre site, sans nous en préoccuper lors de la rédaction des articles.

Depuis sa version 2.0, *Jekyll* utilise par défaut le moteur *Kramdown* et l'extension Typogruby, qui permet notamment :

* transforme les points de suspensions `...` en leur symbole ... ;
* de convertir les guillemets `""` en guillemets anglais “” ;
* de convertir les tirets semi-longs  `--` en -- ;
* de convertir les tirets longs  `---` en --- ;
* de détecter les acronymes et de leur appliquer une classe.

## Appliquer des filtres sur le contenu

### Modifier le texte, pas le code

Il faut prendre garde de n'appliquer les modifications que dans les paragraphes de texte, et non au sein des blocs de code qui pourraient devenir inutilisables pour les utilisateurs qui les copieront[[les codes qui ne compileront pas à cause d'espaces particulières seront difficiles à débugger]].

Pour ce faire, nous allons simplement remplacer {% raw %}`{{ content }}`{% endraw %}, présent dans `_layout/default.html`[[ou équivalent]], par :

{% raw %}
```html
{% capture hide %}
  {% assign content = content | split: '<pre' %}

  {% for parts in content %}

    {% assign part = parts | split: '</pre>' %}
    {% assign c = part.first %}
    {% assign t = part.last %}

    {% if part.size == 2 %}

      {% capture output %}{{ output }}<pre{{ c }}</pre>{% endcapture %}

    {% endif %}

    {% capture output %}{{ output }}{{ t }}{% endcapture %}

  {% endfor %}
{% endcapture %}{{ output }}
```
{% endraw %}

Il nous suffit alors d'agir sur la variable `t`, qui correspond aux textes, en appliquant différents filtres, juste après {% raw %}`{% assign t = part.last %}`{% endraw %}.

Par exemple, pour remplacer tous les *a* en *b* dans les textes, mais pas dans les blocs de code, il suffit d'utiliser :

{% raw %}
```html
{% assign t = t | replace: 'a' , 'b' %}
```
{% endraw %}

## Typographie française

Nous allons montrer ici comment faire en sorte que *Jekyll* génère automatiquement des guillemets français et des espaces insécables de la bonne longueur devant les divers signes de ponctuation.

### Utiliser les guillemets français

Pour obtenir des guillemets français `«` et `»`, remplaçons simplement les guillemets anglais en indiquant, sur la base de la section précédente, avec :[[nous ajoutons une espace insécable normale avant le guillemet fermant, et après le guillemet entrant, avec `&#160;`]]

{% raw %}
```html
{% assign t = t | replace: '“', '«&#160;'
                | replace: '”', '&#160;»' %}
```
{% endraw %}

### Utiliser les bonnes espaces devant la ponctuation

En français, les signes deux-points (`:`) et pourcent (`%`) doivent être précédés d'une espace insécable normale, qui s'obtient avec `&#160;`. Pour remplacer les espaces simples que vous allez taper devant[[bien que la plupart des navigateurs empêchent eux-mêmes un saut de ligne avant ces symboles]], on utilise comme précédemment :

{% raw %}
```html
{% assign t = t | replace: ' :', '&#160;:'
                | replace: ' %', '&#160;%' %}
```
{% endraw %}

En revanche, le point-virgule (`;`), le point d'exclamation (`!`) et le point  d'interrogation (`?`) sont précédés d'une espace insécable *fine*. Celle-ci s'obtient à l'aide de `&thinsp;`. Cependant, il est préférable d'utiliser `<span style="white-space:nowrap">&thinsp;</span>;` en raison de certains navigateurs qui ne traitent pas l'espace comme insécable.

On utilise alors le code suivant pour obtenir le résultat désiré :

{% raw %}
```html
{% assign t = t | replace: ' ;', '<span style="white-space:nowrap">&thinsp;</span>;'
                | replace: ' !', '<span style="white-space:nowrap">&thinsp;</span>!'
                | replace @: ' ?', '<span style="white-space:nowrap">&thinsp;</span>?' %}
```
{% endraw %}

### Résultat final

Finalement, le code suivant permet d'obtenir toutes les améliorations typographiques présentées ci-dessus :

{% raw %}
```html
{% capture hide %}
  {% assign content = content | split: '<pre' %}

  {% for parts in content %}

    {% assign part = parts | split: '</pre>' %}
    {% assign c = part.first %}
    {% assign t = part.last %}

    {% assign t = t | replace: '“', '«&#160;'
                    | replace: '”', '&#160;»'
                    | replace: ' :', '&#160;:'
                    | replace: ' %', '&#160;%'
                    | replace: ' ;', '<span style="white-space:nowrap">&thinsp;</span>;'
                    | replace: ' !', '<span style="white-space:nowrap">&thinsp;</span>!'
                    | replace @: ' ?', '<span style="white-space:nowrap">&thinsp;</span>?' %}

    {% if part.size == 2 %}

      {% capture output %}{{ output }}<pre{{ c }}</pre>{% endcapture %}

    {% endif %}

    {% capture output %}{{ output }}{{ t }}{% endcapture %}

  {% endfor %}
{% endcapture %}{{ output }}
```
{% endraw %}

Si votre site est multilingue[[par exemple sur la base de l'article [Rendre *Jekyll* multilingue](/rendre-jekyll-multilingue/)]], vous pouvez n'appliquer ces modifications que sur les pages françaises en plaçant, autour des parties concernées :

{% raw %}
```html
{% if page.lang == 'fr' %}
...
{% endif %}
```
{% endraw %}

