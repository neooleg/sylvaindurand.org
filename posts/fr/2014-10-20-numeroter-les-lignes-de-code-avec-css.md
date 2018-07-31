---
title: Numéroter les lignes <br/> de code avec <em>CSS</em>
---

Lorsque l'on souhaite afficher des lignes de code en *HTML*, on utilise `<pre>` pour indiquer que le texte est préformaté dans laquelle on place une ou plusieurs balises `<code>` pour indiquer qu'il s'agit de code.

Numéroter ces lignes peut être utile pour permettre d'y faire référence, mais les techniques pour ce faire sont très variables : beaucoup utilisent *Javascript* ou des tableaux...

Pourtant, il est possible d'obtenir ce résultat de façon très simple, uniquement à l'aide de CSS et HTML, sans gêner l'utilisateur et en permettant de copier le code sans que ne se copient les nombres.

## HTML
La seule condition qui va peser sur notre HTML est d'utiliser une balise `<code>` pour chaque ligne. Cela produit un code HTML parfaitement valide et respectueux des standards :

```html
<pre>
<code>ligne 1</code>
<code>ligne 2</code>
<code>ligne 3</code>
<code>ligne 4</code>
<code>ligne 5</code>
</pre>
```

## CSS

Le pseudo-élément `:before` va nous permettre d'afficher un élément avant chaque ligne, tandis que les compteurs CSS vont nous permettre de compter les lignes.

On commence par définir un compteur qui part à zéro pour chaque bloc, et qui s'incrémente à chaque ligne de code :

```css
pre{
    counter-reset: line;
}
code{
    counter-increment: line;
}
```

Ensuite, nous affichons ce nombre au début de chaque ligne :

```css
code:before{
    content: counter(line);
}
```

Sous les navigateurs *Webkit*, sélectionner le code pour le copier donne l'impression que les numéros sont sélectionnés[[même si, heureusement, ils ne seront pas copiés]]. Pour l'éviter, il suffit d'utiliser la propriété `user-select` :

```css
code:before{
    -webkit-user-select: none;
}
```

Il ne vous reste qu'à améliorer leur style à votre gré. C'est tout !
