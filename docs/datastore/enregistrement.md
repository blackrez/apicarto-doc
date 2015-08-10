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

Après une configuration classique de Leaflet.Draw (un exemple complet est visible ici : https://github.com/sgmap/apicarto-client/blob/master/examples/draw/demo.js )


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

Avec le plugin Leaflet.Draw, nous créons un dessin qui est ajouté à une collection de polygones.


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
