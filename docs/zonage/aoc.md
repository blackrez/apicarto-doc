# Zonage - interrogation des Zones AOC/IGP

## Introduction

Le service d'interrogation du cadastre permet de faire le croisement entre un polygone et la couche des AOC/IGP.
Les données proviennent de France Agrimer et de l'INAO, pour l'instant seul la Gironde est disponible.
Ce service ne nécessite pas de clef d'API dans sa version beta.

## API (version beta)

### URL de dévéloppement


  http://apicarto-dev.sgmap.fr/aoc/api/beta/


### Croisement geojson couche du cadastre


**URL** : http://apicarto-dev.sgmap.fr/aoc/api/beta/in

**Methode** : POST

**Param** : geom

**Authenfication nécessaire** : non

**Réponse** :

  * **Code HTTP** :  200
  * **JSON** :

```
{"type":"FeatureCollection",
  "features":
      [{"type":"Feature",
      "geometry":
        {"type":"MultiPolygon","coordinates":[...]},
      "properties":{
        "area_inter":xxx,
        "appellation":"XXX","id_uni":"X-XX-XXXX"
      }
  ]}
}
```

  Le paramètre geom est un geojson valide qui permettra de récupérer un geojson avec les polygones des aires AOC/IGP et la surface de l'intersection entre la parcelle et le geojson. Le champs properties est complété par le nom de l'appelation et l'identifiant fourni par France Agrimer.

### Exemple avec Jquery

On considère qu'il y a une instance de Leaflet est active sur la page.

La variable data contient un objet geojson qui est transformer en Json par la fonction JSON.stringify.

Dans le callback, on affiche les parcelles concernées sur la carte leaflet an ajoutant un layers leaflet.

```javascript
onEachFeatureaoc = function(feature, layer) {
  $('#parcelle-info tbody:last').after('<tr>commune:' + feature.properties.nom_com + '</tr><tr><td>' + feature.properties.numero + '</td><td>' + feature.properties.feuille + '</td><td>' + feature.properties.section + '</td><td>' + feature.properties.surface_intersection + '</td></tr>');
  layer.bindPopup('surface parcelle : ' + feature.properties.area + '<br>');
};

$('#testaoc').click(function() {
  return $.post('http://apicarto.coremaps.com/aoc/api/beta/aoc/in', {
    contentType: "application/json",
    dataType: 'json',
    geom: JSON.stringify(window.data)
  }).done(function(data) {
    var layer;
    $('#parcelle-aoc').html = "";

  });
});
```
