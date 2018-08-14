---
title: Compressing <em>Liquid</em> <br/> generated code
original: 2014-10-19
redirect: /compressing-jekyll-generated-html/
categories: Jekyll
---

The *Liquid* syntax, which is used by *Jekyll*, has an unpleasant default: it generates significant spaces and unneccessary line breaks in the generated pages source code.

This is especially true when you want to indent your *Liquid* code, or want to use loops.

For instance, the following example[[which show each multiplication which gives "12"]] will generate almost 500 lignes of codes, almost empty:

{% raw %}
```liquid
<ul>
{% for i in (1..12) %}
  {% for j in (1..12) %}
  {% assign result = i | times: j %}
  {% if result == 12 %}
    <li> {{ i }} ⨉ {{ j }} = 12 </li>
  {% endif %}
  {% endfor %}
{% endfor %}
</ul>
```
{% endraw %}

What happens? *Liquid* reads all the characters in the loop, including spaces and line breaks, and restores them without asking any questions. Thus, this double loop will generate hundred of useless breaklines and spaces.

However, in order to get a readable HTML code, we would like to get:

```html
<ul>
    <li> 1 ⨉ 12 = 12 </li>
    <li> 2 ⨉ 6 = 12 </li>
    <li> 3 ⨉ 4 = 12 </li>
    <li> 4 ⨉ 3 = 12 </li>
    <li> 6 ⨉ 2 = 12 </li>
    <li> 12 ⨉ 1 = 12 </li>
</ul>
```

## First try : unindent your code

A first answer, which is also the less satisfying, would be to delete all spaces and linebreak which shoudn't be shown. In the previous example, it would give:

{% raw %}
```liquid
<ul>{% for i in (1..12) %}{% for j in (1..12) %}{% assign result = i | times: j %}{% if result == 12 %}
    <li> {{ i }} ⨉ {{ j }} = 12 </li>{% endif %}{% endfor %}{% endfor %}
</ul>
```
{% endraw %}

The HTML output is perfect, but our *Liquid* code is unreadable...

## First try : using `capture`

The `capture` tag will store, as a variable, every interpreted code which is inside.

By using a `capture` tag around a *Liquid* code, you will hide its output, and therefore the unwanted spaces and linebreaks. Then, you just have to use a second `capture` tag around the wanted output, then to print it.

The previous example would become:

{% raw %}
```liquid
<ul>{% capture hide %}
{% for i in (1..12) %}
  {% for j in (1..12) %}
  {% assign result = i | times: j %}
  {% if result == 12 %}
{% capture show %}{{ show }}
    <li> {{ i }} ⨉ {{ j }} = 12 </li>{% endcapture %}
  {% endif %}
  {% endfor %}
{% endfor %}
{% endcapture %}{{ show }}
</ul>
```
{% endraw %}

The code remains still a little verbous, and isn't that clean yet.

## Using {% raw %} `%}{%`{% endraw %} , the final answer ?

Since *Jekyll* 3.0.0, it is possible to put linebreaks *inside* *Liquid* tags, without any influence on the output code. So, it is possible to indent your code like:

{% raw %}
```liquid
<ul>{%
for i in (1..12) %}{%
  for j in (1..12) %}{%
    assign result = i | times: j %}{%
    if result == 12 %}
    <li> {{ i }} ⨉ {{ j }} = 12 </li>{%
    endif %}{%
  endfor %}{%
endfor %}
</ul>
```
{% endraw %}

Despite this unusual way to write  *Liquid* code, it remains quite readable and gives us the wanted result.

However, I couldn't find if this behaviour is wanted by *Liquid* developers, and thus if it will be maintain in the future.


## Compress *Jekyll* HTML output

Have a look to the clever [penibelst](http://jch.penibelst.de/) method, which provides many options.
