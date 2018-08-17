---
title: Carte choroplèthe
language: Javascript, Python
desc: Carte dynamique pour la visualisation de données.
---

### Carte choroplèthe de France

Carte de France générée côté client, avec recherche de communes, génération de la légende et affichage des valeurs au survol. Vous pouvez consulter le [répertoire](https://github.com/sylvaindurand/france-choropleth) ou le [code source](https://github.com/sylvaindurand/france-choropleth).

<div class="iframe"><div style="padding-bottom: 640px"><iframe src="https://sylvaindurand.github.io/france-choropleth/" scrolling="no"></iframe></div></div>

### Utilisation

Après avoir créé un objet Leaflet `map`, utilisez `france()` pour créer la carte. Vous pouvez définir plusieurs couches :

```js
var choropleth = new france(map, layer1[, layer2, layer3])
```

Une couche est définie comme suit :

```js
{
  stat: 'filename', // nom de la colonne dans le fichier csv
  file: '/path/to/file.csv', // url vers le fichier csv
 title: ['Layer title','Date'], // titre de la légende (et date)
domain: [x_0, x_1, ..., x_n], // bornes des valeurs...
 range: ['color_0','color_1', ...,'color_n'], // ... et couleurs
  unit: 'unit name', // unité après les nombres (%, €)
  plus: '' // et signe avant (+)
}
```

### Dépendances

The code javascript nécessite [Leaflet](https://github.com/Leaflet/Leaflet), [d3js](https://github.com/mbostock/d3), [Awesomplete](https://leaverou.github.io/awesomplete/) et [TopoJSON](https://github.com/mbostock/topojson).
Les fichiers `makefile` et `_generate.py` nécessitent `python3` et les librairies `pandas` et `xlrd`.


### Licence

Le contenu et les codes sont publiées sous la licence MIT. Les données spatiales proviennent du projet [OpenStreetMap](https://www.openstreetmap.org/)  (© contributeurs d'OpenStreetMap ), et sous licence [ODbL](http://opendatacommons.org/licenses/odbl/). Les tuiles sont gentiment fournies par [CartoCDN](https://cartodb.com/basemaps/) et proposées sous [licence CC BY 3.0](https://creativecommons.org/licenses/by/3.0/).

Les données statistiques proviennent de l'[INSEE](http://www.insee.fr/en/default.asp) ([conditions](http://www.insee.fr/en/service/default.asp?page=rediffusion/rediffusion.htm)).
