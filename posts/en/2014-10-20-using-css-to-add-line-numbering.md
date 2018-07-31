---
title: Using <em>CSS</em> to add<br/> line numbering
---

When you want to display a code listing with *HTML*, you use a `<pre>` tag in order to indicate that the text is preformatted, then inside one or multiple `<code>` tags to specify it is a code.

Showing line numbers appears may be quite useful, and the methods to achieve this vary widely: many use *Javascript* or tables...

Yet it is possible to achieve this in a simple way, using only CSS and HTML. The line numbers won't be selected when the user wants to copy the code.

## HTML
The only thing to do in our HTML code is to use a `<code>` tag for each line of code. This will produce a perfectly valid HTML code:

```html
<pre>
<code>line 1</code>
<code>line 2</code>
<code>line 3</code>
<code>line 4</code>
<code>line 5</code>
</pre>
```

## CSS

The `:before` pseudo-element will allow us to show an element before each line, while the CSS counters will allow us to count the lines.

We will first define a counter that starts at one for each block, then which is incremented at each new line of code:

```css
pre{
    counter-reset: line;
}
code{
    counter-increment: line;
}
```

Then we display the number at the beginning of each line:

```css
code:before{
    content: counter(line);
}
```

With *Webkit* browsers, selecting the code to copy it gives the impression that the numbers are selected[[although fortunately they are not copied]]. To avoid this, simply use the property `user-select`:

```css
code:before{
    -webkit-user-select: none;
}
```

It only remains to improve their style as desired. That's it!
