---
layout: post
category : web micro log
tags: [js, leaflet]
---

For April fools, Google released their [Pokemon Challenge](https://www.youtube.com/watch?v=4YMD6xELI_k).
As a challenge from a friend I decided to start a quick prototype of how you could clone this. The
library of choice I used was [leaflet](http://leafletjs.com/).

**Goals:**  

*  Create a map which spawns pokemon.
*  When you click on a pokemon to "catch" it, it turns into a pokemon.

More complex interactions like a [Pokedex](http://bulbapedia.bulbagarden.net/wiki/Pok%C3%A9dex) could then
be easily added. 

**Limitations:**

*  Pokemon can be randomly spawned quite easily, but you would need some API to determine whether it is a
   "valid" location (i.e. on a road, in a pond, in a park), since you wouldn't want to spawn pokemon
   in the middle of the Atlantic Ocean.

## Pseudocode

To get it working using leaflet the idea is this:

1.  Create a new leaflet layer.  
2.  Generate pokemon with the correct markers and add it to the leaflet layer.  
3.  Add interactivity, clicking on the pokemon will change them to a pokeball.
4.  Hide the leaflet layer if the player zooms out too far, show leaflet layer when the player zooms in

To achieve all of this, the code is as follows:

```javascript
/*1. Create a new leaflet layer*/
var pokemonLayer = new L.LayerGroup();
```

```javascript
/*2. Generate pokemon with correct markers*/
pika1 = L.marker([-33.934844, 151.231324], {icon: pikachu})
pika2 = L.marker([-33.954844, 151.234324], {icon: pikachu})
pika3 = L.marker([-33.934844, 151.2523241], {icon: pikachu})

/*add the pokemon to the layer above*/
pokemonLayer.addLayer(pika1);
pokemonLayer.addLayer(pika2);
pokemonLayer.addLayer(pika3);
```

We assume that we have some location for the icons and the coordinates are known (some how generated based
on API rules). Ideally, if this was done on a full scale, it would obviously be looped.


```javascript
/*3. add the interactivity*/
pika1.on('click', function(changeIcon) {
    changeIcon.target.setIcon(pokeball);
});
pika2.on('click', function(changeIcon) {
    changeIcon.target.setIcon(pokeball);
});
pika3.on('click', function(changeIcon) {
    changeIcon.target.setIcon(pokeball);
});
```

Again, this code would be looped if it was for many pokemon.

```javascript
/*4. add the zoom functionality*/
map.addLayer(pokemonLayer);
map.on('zoomend', displayPokemon);
map.on('zoomstart', displayPokemon);
function displayPokemon(){
    if(map.getZoom()>=13){
        map.addLayer(pokemonLayer);
    }
    else {
        map.removeLayer(pokemonLayer);
    };
};
```

Here based on the map, and when we zoom in and out, we would hide or show the pokemon layer as needed.

The result is below:

<div id="map" style="width: 600px; height: 400px"></div>
<script src="http://cdn.leafletjs.com/leaflet/v0.7.7/leaflet.js"></script>
<script>
    /*add css file...*/
    var link = document.createElement('link')
    link.setAttribute('rel', 'stylesheet')
    link.setAttribute('type', 'text/css')
    link.setAttribute('href', 'http://leafletjs.com/dist/leaflet.css')
    document.getElementsByTagName('head')[0].appendChild(link)

    var map = L.map('map').setView([-33.934844, 151.232324], 13);

    L.tileLayer('http://{s}.tile.osm.org/{z}/{x}/{y}.png', {
        attribution: '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors'
    }).addTo(map);

    var LeafIcon = L.Icon.extend({
        options: {
            iconSize:     [55, 55],
        }
    });

    var pokemonLayer = new L.LayerGroup();
    
    var pikachu = new LeafIcon({iconUrl: 'https://raw2.github.com/charliec443/charliec443.github.io/master/img/pokemon/pikachu.png'}),
        pokeball = new LeafIcon({iconUrl: 'https://raw2.github.com/charliec443/charliec443.github.io/master/img/pokemon/pokeball.gif'});
        //var redIcon = new LeafIcon({iconUrl: '../docs/images/leaf-red.png'}),
        //orangeIcon = new LeafIcon({iconUrl: '../docs/images/leaf-orange.png'});

        
    // should move all these to an array...but hard coding
    
    pika1 = L.marker([-33.934844, 151.231324], {icon: pikachu})
    pika2 = L.marker([-33.954844, 151.234324], {icon: pikachu})
    pika3 = L.marker([-33.934844, 151.2523241], {icon: pikachu})
    

    pokemonLayer.addLayer(pika1);
    pokemonLayer.addLayer(pika2);
    pokemonLayer.addLayer(pika3);

    pika1.on('click', function(changeIcon) {
        changeIcon.target.setIcon(pokeball);
    });
    pika2.on('click', function(changeIcon) {
        changeIcon.target.setIcon(pokeball);
    });
    pika3.on('click', function(changeIcon) {
        changeIcon.target.setIcon(pokeball);
    });
    
    var clickedMarker;

    map.addLayer(pokemonLayer);
    map.on('zoomend', displayPokemon);
    map.on('zoomstart', displayPokemon);
    function displayPokemon(){
        if(map.getZoom()>=13){
            map.addLayer(pokemonLayer);
        }
        else {
            map.removeLayer(pokemonLayer);
        };
    };
    
    // now you just have to have a database to record what you clicked and display a pokedex on the right...
</script>


