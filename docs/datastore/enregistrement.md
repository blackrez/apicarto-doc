# Datastore - enregistrement d'un dessin

## Introduction

Dans le but de faciliter l'intégration de l'enregistrement de dessin fourni par un usager, un web service est mis en place afin de pouvoir enregistrer des géométries au format GeoJson.
Ce service nécessite une clef d'API à demander au SGMAP.

Ce composant est sous licence AGPL v3 et disponible : https://github.com/blackrez/apicarto-datastore

## API (version beta)

### URL de dévéloppement


  http://apicarto-dev.sgmap.fr/store/api/v2/datastore


### Enregistrement d'une géométrie.


**URL** : http://apicarto-dev.sgmap.fr/store/api/v2/datastore/draw

**Methode** : POST

**Param** : geojson

**Authenfication nécessaire** : oui

**Réponse** :

  * **Code HTTP** :  200
  * **JSON** : `{reference : ID}`

Le paramètre geojson doit contenir un fichier GeoJson valide, il peut être de n'importe quel type qui respecte la spécification. Dans les cas de géométries multiples, il est recommandé d'envoyer une collection de géométrie au lieu de faire fusionner les géométries.
Le champ `properties` du GeoJson permet l'utilisation de n'importe quel champ.

L'ID servira à récupérer la ou les géométries.

Tout autre réponse doit être considéré comme une erreur et la géométrie n'est pas enregistrée.

____
Exemple:


```
$ curl -X POST -H "Authorization: Token 77def68e703a6bc65c0b9e4fa426306bc1f384d9" -H "Cache-Control: no-cache" -H -H "Content-Type: application/x-www-form-urlencoded" -d 'geojson=%7B%22type%22%3A%22Feature%22%2C%22properties%22%3A%7B%7D%2C%22geometry%22%3A%7B%22type%22%3A%22Polygon%22%2C%22coordinates%22%3A%5B%5B%5B4.839992523193359%2C43.95488856092349%5D%2C%5B4.8415374755859375%2C43.950316046754146%5D%2C%5B4.821968078613281%2C43.947597087764585%5D%2C%5B4.821624755859375%2C43.95167547960693%5D%2C%5B4.839992523193359%2C43.95488856092349%5D%5D%5D%7D%7D' 'http://apicarto-dev.sgmap.fr/store/store/api/v2/datastore/draw'

{"reference": "OSHPPY2G"}
```



## Intégration avec leaflet et JQuery

*NB il est possible d'utiliser d'autres librairies comme OpenLayers, mais seul Leaflet est supporté officiellement*

### Ajout des librairies nécessaires

Dans cet exemple, nous allons utiliser les plugins Leaflet.Draw (pour le dessin) et leaflet-geodesy (pour le calcul à la volée de surface).

```html
<link rel="stylesheet" type="text/css" href="http://leaflet.github.io/Leaflet.draw/leaflet.draw.css">
<link rel="stylesheet" type="text/css" href="http://cdn.leafletjs.com/leaflet-0.7.3/leaflet.css">
[...]
<script src='https://api.tiles.mapbox.com/mapbox.js/plugins/leaflet-geodesy/v0.1.0/leaflet-geodesy.js'></script>
<script src="https://code.jquery.com/jquery-2.1.3.js"></script>
<script src="http://cdn.leafletjs.com/leaflet-0.7.3/leaflet.js"></script>
<script src="http://leaflet.github.io/Leaflet.draw/leaflet.draw.js"></script>
```
### Configuration de Leaflet et de Leaflet.draw

Après une configuration classique de Leaflet.Draw (un exemple complet est visible ici : https://github.com/sgmap/apicarto-client/blob/master/examples/draw/demo.js )

L'ensemble de ce premier block permet dde franciser Leaflet.Draw qui n'a pas de support multi-langues par défaut.

```javascript
L.drawLocal.draw.toolbar.buttons.polygon = 'Dessiner un polygone';
L.drawLocal.draw.toolbar.actions.title = "Annule le dessin en cours";
L.drawLocal.draw.toolbar.actions.text = "Annuler";
L.drawLocal.draw.toolbar.undo.text = "Supprimer le dernier point";
L.drawLocal.draw.toolbar.undo.title = "Supprime le dernier point dessiné";
L.drawLocal.draw.handlers.polygon.tooltip.start = "Cliquer pour commencer le dessin";
L.drawLocal.draw.handlers.polygon.tooltip.cont = "Cliquer pour continuer le dessin";
L.drawLocal.draw.handlers.polygon.tooltip.end = "Cliquer sur le premier point pour finaliser votre dessin";
L.drawLocal.edit.toolbar.actions.save.title = "Valide les modifications";
L.drawLocal.edit.toolbar.actions.save.text = "Valider les modifications";
L.drawLocal.edit.toolbar.actions.cancel.title = "Annule les modifications";
L.drawLocal.edit.toolbar.actions.cancel.text = "Annuler les modifications";
L.drawLocal.edit.handlers.edit.tooltip.text = "Déplacer les points pour éditer le dessin";
L.drawLocal.edit.handlers.edit.tooltip.subtext = "Cliquer sur 'annuler' pour annuler les changements";
L.drawLocal.edit.toolbar.buttons.edit = "Édition du dessin";
L.drawLocal.edit.toolbar.buttons.editDisabled = "Aucun dessin à éditer";
L.drawLocal.edit.toolbar.buttons.removeDisabled = "Aucun dessin à supprimer";
L.drawLocal.edit.toolbar.buttons.remove = "Supprimer le dessin";
L.drawLocal.edit.handlers.remove.tooltip.text = "Cliquer sur le dessin pour le supprimer";
L.drawLocal.edit.handlers.remove.tooltip.subtext = "Cliquer sur 'annuler' pour annuler la suppression";
```

Ensuite on initialise Leaflet en ajoutant les couches nécessaires.

```javascript
mapId = "map";
layers = new Array;
window.mapcontrol = false;
OSM = L.tileLayer("http://{s}.tile.osm.org/{z}/{x}/{y}.png", {
  attribution: "&copy; <a href=\"http://osm.org/copyright\">OpenStreetMap</a>"
});
scanWmtsUrl = 'http://apicarto-dev.sgmap.fr/maps' + '/wmts?SERVICE=WMTS&REQUEST=GetTile&VERSION=1.0.0&LAYER=ORTHOIMAGERY.ORTHOPHOTOS&STYLE=normal&TILEMATRIXSET=PM&TILEMATRIX={z}&TILEROW={y}&TILECOL={x}&FORMAT=image%2Fjpeg';
ortho = L.tileLayer(scanWmtsUrl, {
  attribution: "&copy; <a href=\"http://www.ign.fr/\">IGN</a>"
});
cadWmtsUrl = 'http://apicarto-dev.sgmap.fr/maps' + '/wmts?SERVICE=WMTS&REQUEST=GetTile&VERSION=1.0.0&LAYER=CADASTRALPARCELS.PARCELS&STYLE=bdparcellaire_b&TILEMATRIXSET=PM&TILEMATRIX={z}&TILEROW={y}&TILECOL={x}&FORMAT=image%2Fpng&TRANSPARENT=TRUE';
cad = L.tileLayer(cadWmtsUrl, {
  attribution: "&copy; <a href=\"http://www.ign.fr/\">IGN</a>",
  transparent: true,
  format: 'image/png'
});

map = L.map(mapId, {
  center: new L.LatLng(44.727006, -0.475243),
  zoom: 13,
  layers: [OSM]
});
window.map = map;
baseMap = {
  "IGN Orthophto": ortho,
  "OpenStreetMap": OSM
};
overlayMaps = {
  "Ign Cadastre": cad
};
L.control.layers(baseMap, overlayMaps).addTo(map);
```

On initialise Leaflet.draw, c'est ici qu'on configure le module de dessin.

Pour plus d'informations sur les callbacks et les options, se référer à la documentation officielle du plugins https://github.com/Leaflet/Leaflet.draw.

```javascript
drawnItems = new L.FeatureGroup();
map.addLayer(drawnItems);
drawControl = new L.Control.Draw({
  position: "topright",
  draw: {
    polygon: {
      shapeOptions: {
        color: "purple"
      },
      allowIntersection: false,
      drawError: {
        color: "orange",
        timeout: 1000
      },
      showArea: false,
    },
    marker: false,
    polyline: false,
    rectangle: false,
    circle: false
  },
  edit: {
    featureGroup: drawnItems
  }
});
map.addControl(drawControl);
L.control.scale({
  imperial: false
}).addTo(map);
```

On ajoute plusieurs callbacks. Le but de chaque callback est de sauvegarder l'état du dessin dans une variable dans le navigateur de calculer la surface du dessin.

```javascript
map.on("draw:created", function(e) {
  var layer;
  drawnItems.clearLayers();
  layer = e.layer;
  drawnItems.addLayer(layer);
  window.data = layer.toGeoJSON();
  $('#parcelle-area').html('<span id=parcelle-area>' + (LGeo.area(e.layer) / 10000).toFixed(4) + ' ha</span>');
});
map.on("draw:edited", function(e) {
  drawnItems.clearLayers();
  e.layers.eachLayer(function(layer) {
    drawnItems.addLayer(layer);
    window.data = layer.toGeoJSON();
    $('#parcelle-area').html('<span id=parcelle-area>' + (LGeo.area(layer) / 10000).toFixed(4) + ' ha</span>');
  });

});
```

Les 2 callbacks suivants permettent de gérer la suppression d'un dessin.

```javascript
map.on("draw:deletestop", function(e) {
  window.data = {};
  $('#parcelle-area').html('<span id=parcelle-area></span>');
});
map.on("draw.started", function(e) {
  map.removeLayer(drawnItems);
});
```

Avec le plugin Leaflet.Draw, nous créons un dessin qui est ajouté à une collection de polygones. Le dessin est désormais comme une collections de polygones.


```javascript
window.featureCollection = new Object()
window.featureCollection.type = 'FeatureCollection';
window.featureCollection.features = new Array();

map.on("draw:created", function(e) {
  var layer;
  layer = e.layer;
  drawnItems.addLayer(layer);
  window.featureCollection.features.push(layer.toGeoJSON());
)}
```

Enfin la fonction store permet d'enregistrer le dessin en envoyant le geojson au datastore.

```javascript
function store() {
  return $.ajax("http://apicarto.coremaps.com/store/api/v2/datastore/draw", {
    method: 'POST',
    crossDomain: true,
    contentType: 'application/x-www-form-urlencoded',
    headers: {
      'AUTHORIZATION': 'VOTRE CLEF D'API'
    },
    data: {
      geojson: JSON.stringify(window.featureCollection)
    }
  }).done(function(data) {
    console.log('Références pour récupérer le fichier : ' + data.reference);
  });
};
```
