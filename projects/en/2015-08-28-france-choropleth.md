---
title: Choropleth map
language: Javascript, Python
desc: Dynamic map suitable for spatial data analysis.
---

### France Choropleth


Client side France map for spatial data analysis purposes, with municipalities search engine, legend generation and data display. Have a look on the [reference](https://github.com/sylvaindurand/france-choropleth) or check the [source code](https://github.com/sylvaindurand/france-choropleth).

<div class="iframe"><div style="padding-bottom: 640px"><iframe src="https://sylvaindurand.github.io/france-choropleth/" scrolling="no"></iframe></div></div>



### Usage

After having created a Leaflet `map` object, use `france()` to create a new choropleth. You can provide many layers:

```js
var choropleth = new france(map, layer1[, layer2, layer3])
```

A layer is defined as follows:

```js
{
  stat: 'filename', // column name in csv file
  file: '/path/to/file.csv', // url to csv file
 title: ['Layer title','Date'], // legend title (and date)
domain: [x_0, x_1, ..., x_n], // values stops and limits...
 range: ['color_0','color_1', ...,'color_n'], // ... and colors
  unit: 'unit name', // prefix after numbers (%, €)
  plus: '' // suffix before numbers (+)
}
```

### Requirement

The javascript code requires [Leaflet](https://github.com/Leaflet/Leaflet), [d3js](https://github.com/mbostock/d3), [Awesomplete](https://leaverou.github.io/awesomplete/) and [TopoJSON](https://github.com/mbostock/topojson).
The `makefile` and `_generate.py` need `python3`, `pandas` and `xlrd` libraries.


### Administrative levels

*‘Communes’* are the smallest administrative level in France. Although their high number (more than 36.000) is quite satisfying for spatial statistics purposes, users would have to download a 10MB file and to wait more than 20 seconds for the rendering. On the other side, there is only 2.000 ‘*cantons*’ (higher administrative level), which is not quite enough.

Keeping *communes* with more than 1.000 people, or with more than 75 people per square kilometre, and showing cantons for the smaller *communes*, gives us a 2.9MB file and a good looking map.


### License

Contents and codes are published under the MIT license; you are free to reuse them. Spatial data is provided by [OpenStreetMap](https://www.openstreetmap.org/) project (© OpenStreetMap contributor), and licensed under [ODbL](http://opendatacommons.org/licenses/odbl/). Tiles are kindly provided by [CartoCDN](https://cartodb.com/basemaps/) and released under [Creative Commons Attribution (CC BY 3.0)](https://creativecommons.org/licenses/by/3.0/).

Statistics data is provided by the French National Institute for Statistic [INSEE](http://www.insee.fr/en/default.asp) with the ‘code officiel geographique’ and ‘chiffres clefs’ databases ([conditions](http://www.insee.fr/en/service/default.asp?page=rediffusion/rediffusion.htm)).
