# Zonage - interrogation du cadastre

## Introduction

Le service d'interrogation du cadastre permet de faire le croisement entre un polygone et la couche vectoriel du cadastre.
Ce service ne nécessite pas de clef d'API dans sa version beta.

## API (version beta)

### URL de dévéloppement


  http://apicarto-dev.sgmap.fr/cadastre


### Croisement geojson couche du cadastre


**URL** : http://apicarto-dev.sgmap.fr/cadastre

**Methode** : POST

**Param** : geom

**Authenfication nécessaire** : non

**Réponse** :

  * **Code HTTP** :  200
  * **JSON** :

```
  {"type":"FeatureCollection","features":
    [{"type":"Feature","geometry":
      {"type":"MultiPolygon",
      "coordinates":[...]},
      "properties":{"surface_intersection":"...",
                    "surface_parcelle":...,
                    "numero":"XXXX",
                    "feuille":X,
                    "section":"XX",
                    "code_dep":"XX",
                    "nom_com":"X",
                    "code_com":"XXX",
                    "code_arr":"XXX"}
      }]
  }
```

  Le paramètre geom est un geojson valide qui permettra de récupérer un geojson avec les polygones des parcelles cadastrales et calcule la surface de la parcelle, et la surface de l'intersection entre la parcelle et le geojson. Les autres informations conernent les données qui permettent l'identification de la parcelle.

### Exemple avec Jquery

On considère qu'il y a une instance de Leaflet est active sur la page.

La variable data contient un objet geojson qui est transformer en Json par la fonction JSON.stringify.

Dans le callback, on affiche les parcelles concernées sur la carte leaflet an ajoutant un layers leaflet.

```javascript
$('#testcadastre').click(function() {
  return $.post("http://apicarto.coremaps.com/cadastre", {
    contentType: "application/json",
    dataType: 'json',
    geom: JSON.stringify(data)
  }).done(function(data) {
    var layer;
    map.removeLayer(data);
    $('#parcelle-info').html = "";
    layer = L.geoJson(data, {
      onEachFeature: onEachFeature,
      color: "#ff0000"
    }).addTo(map);
  });
});
onEachFeature = function(feature, layer) {
  $('#parcelle-info tbody:last').after('<tr>commune:' + feature.properties.nom_com + '</tr><tr><td>'
   + feature.properties.numero + '</td><td>' + feature.properties.feuille + '</td><td>' + feature.properties.section + '</td><td>' + feature.properties.surface_intersection + '</td></tr>');
  layer.bindPopup('surface parcelle : ' + feature.properties.surface_parcelle + '<br>');
};
```
