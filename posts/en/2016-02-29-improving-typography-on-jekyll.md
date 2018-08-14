---
title: Improving typography <br/> on <em>Jekyll</em>
original: 2014-10-30
redirect: /improving-typography-with-jekyll/
categories: Jekyll
---

Observing typographical rules on the Internet is not always easy: although Unicode reserves many areas of characters for typographic symbols, many punctuation marks and spaces are most of the time unused.

With *Jekyll*, the articles are written very simply with Markdown before being generated in HTML by the engine: we can add automatic rules to improve typography on our site without carrying about it when writing articles.

Since version 2.0, *Jekyll* use by default *Kramdown* engine and Typogruby extension, which:

* transforms the ellipsis `...` in its symbol... ;
* convert quotation marks `""` in English quotation marks “” ;
* convert semi-long dashes  `--` in -- ;
* convert long dashes  `---` in --- ;
* detect acronyms and apply them a class.

## Apply filters to the content

### Modify paragraphs, leave the code

Care must be taken to apply the changes only in paragraphs, not in code blocks that could become unusable for users that will copy them.

To do so, we will replace {% raw %}`{{ content }}`{% endraw %}, in the `_layout/default.html` file[[or equivalent]], with :

{% raw %}
```liquid
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

We just have to act on the variable `t`, which corresponds to the texts, by applying different filters, just after {% raw %}`{% assign t = part.last %}`{% endraw %}.

For example, if you want to replace all *a*'s in *b*'s in the texts, but not in the code blocks, juste use:

{% raw %}
```liquid
{% assign t = t | replace: 'a' , 'b' %}
```
{% endraw %}

## French typography

We will show here how to make *Jekyll* automatically generates quotes French and non-breaking spaces to the right length before the various punctuation marks.

### Using French guillemets

For quotes French `«` and `»`, we simply replace English quotation marks, as seen in the previous section, with:[[we add a normal non-breaking space before the closing quotation mark, and after entering quote with `&#160;`]]

{% raw %}
```liquid
{% assign t = t | replace: '“', '«&#160;'
                | replace: '”', '&#160;»' %}
```
{% endraw %}

### Using the right spaces before punctuation

In French, the colons (`:`) and percent (`%`) must be preceded by a normal non-breaking space, which is obtained with `&#160;`. We use:

{% raw %}
```liquid
{% assign t = t | replace: ' :', '&#160;:'
                | replace: ' %', '&#160;%' %}
```
{% endraw %}

However, the semicolon (`;`), the exclamation point (`!`) and the question mark (`?`) are preceded by a *thin* non-breaking space. Thereof is obtained using `&thinsp;`. However, it is preferable to use `<span style="white-space:nowrap">&thinsp;</span>;` because some browsers that do not treat the space as indivisible.

We then use the following code to get the desired result:

{% raw %}
```liquid
{% assign t = t | replace: ' ;', '<span style="white-space:nowrap">&thinsp;</span>;'
                | replace: ' !', '<span style="white-space:nowrap">&thinsp;</span>!'
                | replace: ' ?', '<span style="white-space:nowrap">&thinsp;</span>?' %}
```
{% endraw %}

### Final result

Finally, the following code provides all the typographical improvements presented above:

{% raw %}
```liquid
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
                    | replace: ' ?', '<span style="white-space:nowrap">&thinsp;</span>?' %}

    {% if part.size == 2 %}

      {% capture output %}{{ output }}<pre{{ c }}</pre>{% endcapture %}

    {% endif %}

    {% capture output %}{{ output }}{{ t }}{% endcapture %}

  {% endfor %}
{% endcapture %}{{ output }}
```
{% endraw %}

If your website is multilingual[[as explained, for example, in the article [Making *Jekyll* multilingual](/making-jekyll-multilingual/)]], you can restrict the previous modifications to the French articles, by adding:

{% raw %}
```liquid
{% if page.lang == 'fr' %}
...
{% endif %}
```
{% endraw %}

