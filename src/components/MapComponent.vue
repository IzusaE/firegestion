<template>
  <template v-if="loader">
    <div id="centerLoader">
        <div class="spinner-grow text-success" role="status">
            <span class="sr-only"></span>
        </div>
    </div>
  </template>
  <template v-else>
    <div class="flexBox">
        <div class="blueBar">
            <div class="enteteBlueBar">
                <div class="whiteBand"></div>
                <div class="textEntete">En intervention</div>
            </div>
            <div class="boxPokemon" v-for="feu in feuxEnInter" :key="feu.id">
              <div class="imagePokemon">
                  <img :src="feu.sprite" />
              </div>
              <div class="caracteristiquesPokemon">
                  <p>
                      <b>Intensité : </b> {{feu.intensity}}<br>
                      <b>Fréquence : </b> {{feu.frequency}}
                  </p>
              </div>
              <div class="equipePokemon">
                  <div class="coordoneesPokemon">
                      <b>Lat : </b>{{feu.coords[0]}}<br>
                      <b>Lon : </b>{{feu.coords[1]}}
                  </div>
                  <template v-for="equipier in feu.equipe" :key="equipier.id">
                      <img :src="equipier.sprite" />
                  </template>
              </div>
            </div>
            <div class="enteteBlueBar">
                <div class="whiteBand"></div>
                <div class="textEntete">En attente</div>
            </div>
            <div class="boxPokemon" v-for="feu in feuxEnAttente" :key="feu.id">
              <div class="imagePokemon">
                  <img :src="feu.sprite" />
              </div>
              <div class="caracteristiquesPokemon">
                  <p>
                      <b>Intensité : </b> {{feu.intensity}}<br>
                      <b>Fréquence : </b> {{feu.frequency}}
                  </p>
              </div>
              <div class="equipePokemon">
                  <div class="coordoneesPokemon">
                      <b>Lat : </b>{{feu.coords[0]}}<br>
                      <b>Lon : </b>{{feu.coords[1]}}
                  </div>
              </div>
            </div>
        </div>
        <div class="whiteMap" id="map">
            <div style="height:100%; width:100%">
              <l-map :zoom="zoom" :center="center" :options="{zoomControl: false}" @update:center="centerUpdate" @update:zoom="zoomUpdate">
                <l-tile-layer
                  :url="url"
                  :attribution="attribution"
                ></l-tile-layer>
                <template v-for="(marker, index) in casernes" :key="index">
                  <l-marker :lat-lng="marker.coords" :icon="caserne">
                    <l-popup>
                      Caserne de {{marker.name}}
                    </l-popup>
                  </l-marker>
                </template>
                <template v-for="(marker, index) in feux" :key="index">
                  <l-marker :lat-lng="marker.coords" :icon="listeOfIcons[marker.pokemonID]">
                    <l-popup>
                      Intensité : {{marker.intensity}}<br>
                      Fréquence : {{marker.frequency}}<br>
                    </l-popup>
                  </l-marker>
                </template>
                <template v-for="(marker, index) in listeCamionsInterventin" :key="index">
                  <l-marker :lat-lng="marker.coords" :icon="camions">
                    <l-popup>
                      Camion en intervention
                    </l-popup>
                  </l-marker>
                </template>
              </l-map>
            </div>                      
        </div>
    </div>
  </template>
</template>

<script>
import L from 'leaflet';

import "leaflet/dist/leaflet.css";
import { LMap, LTileLayer, LMarker, LPopup } from "@vue-leaflet/vue-leaflet";

export default {
  components: {
    LMap,
    LTileLayer,
    LMarker,
    LPopup
  },
  data() {
    return {
      loader: false,
      url: 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',
      attribution:
        '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors',
      zoom: 13,
      center: [45.7578137, 4.8320114],
      currentZoom: 12,
      currentCenter: [45,7578137, 4.8320114],
      caserne: L.icon({
        iconUrl: 'https://answers.unity.com/storage/temp/127850-building-transparent.png',
        iconSize: [40, 40],
        popupAnchor: [1, -10]
      }),
      listeOfIcons: [],
      mapOptions: {
        zoomSnap: 0.5
      },
      pompiers: [],
      feux: [],
      camions: [],
      listeSprites: [],
      interventions: [],
      capacites: [],
      casernes: [],
      markers: [],
      markersCasernes: [],
      
    };
  },
  async beforeMount() {
    this.loader = true

    this.pompiers = await this.makeRequest("GET", "/api/fireman")
    this.feux = await this.makeRequest("GET", "/api/fire")
    this.camions = await this.makeRequest("GET", "/api/firetruck")
    this.interventions = await this.makeRequest("GET", "/api/intervention")
    this.casernes = await this.makeRequest("GET", "/api/firestation")
    this.listeSprites = await this.makeRequest("GET", "https://pokebuildapi.fr/api/v1/pokemon")

    this.feux.forEach(elem => {  
      elem.sprite = (this.listeSprites[Math.floor(Math.random() * 600)].sprite)
    })
    this.pompiers.forEach(elem => {  
      elem.sprite = (this.listeSprites[Math.floor(Math.random() * 600)].sprite)
    })

    this.feux.forEach(elem => {
      var myInters = []
      this.interventions.forEach(element => {
        if (elem.id == element.fire_id)
          myInters.push(element)
      })
      elem.intervention = myInters
    });
    this.feux.forEach(elem => {
      var equipes = []
      if (elem.intervention) {
        elem.intervention.forEach(element => {
          this.pompiers.forEach(pomp => {
            if (element.firetruck_id == pomp.firetruck_id)
              equipes.push(pomp)
          })
        })
      }
      elem.equipe = equipes
    })
    var pokemons = await this.makeRequest("GET", "https://pokebuildapi.fr/api/v1/pokemon")
    this.listeOfIcons.push("")
    pokemons.forEach(elem => {
      var iconTMP = L.icon({
        iconUrl: elem.sprite,
        iconSize: [60, 60]
      })
      this.listeOfIcons.push(iconTMP)
    })
    this.loader = false
  }, 
  methods: {
    zoomUpdate(zoom) {
      this.currentZoom = zoom;
    },
    centerUpdate(center) {
      this.currentCenter = center;
    },
    makeRequest(method, url, body) {
      return new Promise(function (resolve, reject) {
        let xhr = new XMLHttpRequest();
        xhr.open(method, url);
        xhr.onload = function () {
          if (this.status >= 200 && this.status < 300) {
            resolve(JSON.parse(xhr.response));
          } else {
            reject({
              status: this.status,
              statusText: xhr.statusText
            });
          }
        };
        xhr.onerror = function () {
          reject({
            status: this.status,
            statusText: xhr.statusText
          });
        };
          xhr.send(body);
      });
    }
  },
  mounted() {
  },
  computed: {
    listeCamionsInterventin() {
      return this.makeRequest("GET", "/api/firetruck");
    },
    feuxEnInter() {
      return this.feux.filter(i => i.intervention.length > 0 && !i.intervention.fin)
    },
    feuxEnAttente() {
      return this.feux.filter(i => i.intervention.length == 0)
    }
  }            
};
</script>

<style></style>