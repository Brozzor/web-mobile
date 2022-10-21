<script setup>
 /* eslint-disable */
import {
  ref,
  inject
} from 'vue'
import marker from "@/assets/marker.png"

const center = ref([-1.553621, 47.218371])
const projection = ref('EPSG:4326')
const zoom = ref(7)
const rotation = ref(0)
const contextMenuItems = ref([])

const markers = ref(null);
let markersList = ref([]);
const view = ref(null);
const isAdmin = ref(true);

const Feature = inject('ol-feature')
const Geom = inject('ol-geom')

contextMenuItems.value = [{
  text: 'Centrer la map ici',
  classname: 'some-style-class',
  callback: (val) => {
    view.value.setCenter(val.coordinate)
  }
},
{
  text: 'Ajouter un repère',
  classname: 'some-style-class',

  icon: marker,
  callback: (val) => {
    if (!isAdmin.value) return alert('Vous devez etre administrateur pour effectuer cette action')
    console.log(val)
    let feature = new Feature({
      geometry: new Geom.Point(val.coordinate),
    });
    markers.value.source.addFeature(feature)
    markersList.value = markers.value.source.getFeatures()
    save()

    console.log(markers.value.source)
  }
},
  '-' // this is a separator
]

// eslint-disable-next-line
function removeMarker(feature){
  markers.value.source.removeFeature(feature)
  markersList.value = markers.value.source.getFeatures()
  save()
}

function save(){
  localStorage['markers'] = JSON.stringify(markersList.value)
}

function initDb(){
  if (localStorage['markers']){
    markersList.value = JSON.parse(localStorage['markers'])
  }
  for (const marker of markersList.value) {
    
   console.log(marker)
    let feature = new Feature({
      geometry: new Geom.Point(marker.geometryChangeKey_.target.flatCoordinates),
    });
    markers.value.source.addFeature(feature)
  }
  markersList.value = markers.value.source.getFeatures()
  save()

}
setTimeout(()=> {
  initDb()
},200)
</script>

<template>
  <div id="app">
    <div class="navbar">
      <button @click="isAdmin = true" v-if="!isAdmin">Devenir administrateur</button>
      <button @click="isAdmin = false" v-else>Devenir utilisateur</button>
      <div v-if="!isAdmin" style="align-self: center; margin-right: 40px;">Vous êtes un utilisateur </div>
      <div v-else style="align-self: center; margin-right: 40px;">Vous êtes Administrateur </div>
    </div>
    <div class="content">
      <div class="map">
        <ol-map :loadTilesWhileAnimating="true" :loadTilesWhileInteracting="true" style="height:800px">

          <ol-view ref="view" :center="center" :rotation="rotation" :zoom="zoom" :projection="projection" />

          <ol-tile-layer>
            <ol-source-osm />
          </ol-tile-layer>

          <ol-context-menu :items="contextMenuItems" />

          <ol-vector-layer>
            <ol-source-vector ref="markers">

            </ol-source-vector>
            <ol-style>
              <ol-style-icon :src="marker" :scale="0.05"></ol-style-icon>
            </ol-style>
          </ol-vector-layer>

        </ol-map>

      </div>
      <div class="list">
        <div class="title">Liste des points</div>
        <div style="margin-top: 30px">
          <div v-for="marker in markersList" :key="marker.ol_uid" class="element">
          Repère numéro : {{marker.ol_uid}} <button v-if="isAdmin" @click="removeMarker(marker)">Supprimer</button>
          </div>

          <div v-if="!markersList.length && isAdmin" >Vous pouvez ajouter des repères en faisant clique droit, puis ajouter un repère</div>
          <div v-if="!markersList.length && !isAdmin" >Vous devez etre administrateur pour ajouter des repères</div>
        </div>
      </div>
    </div>

  </div>
</template>

<style lang="scss">
.content {
  display: flex;
  gap: 10px;
  padding: 5px;
}
.map {
  width: 100%;
  height: 50%;
}

.navbar {
  width: 100%;
  height: 50px;
  background-color: #23272a;
  margin-bottom: 30px;
  display: flex;
  justify-content: space-between;

  * {
    color: white
  }

  button {
    background-color: blueviolet;
    border-radius: 5px;
    cursor: pointer;
  }

  padding: 10px;
}

body {
  margin: 0 !important;
}

.list {
  width: 300px;
}
</style>