<template>

  <div style="height: 1000px; width: 100%">
    <l-map
      v-if="showMap"
      :zoom="zoom"
      :center="center"
      :options="mapOptions"
      style="height: 80%"
      @update:center="centerUpdate"
      @update:zoom="zoomUpdate"
    >
      <l-tile-layer
        :url="url"
        :attribution="attribution"
      />
       <l-marker  :lat-lng="marker" >
        <l-icon
                :icon-size=[20,40]
                :popupAnchor= [0,0] >
                  <v-icon medium color="success">
                    mdi-car
                  </v-icon>
            </l-icon>
      </l-marker>
      <l-marker v-for="location in locations" :lat-lng="[location.lat, location.long]" :key="location.id">
        <l-icon
                :style="`transform: rotatex(${45* slop})`"
                :icon-size=[20,40]
                :popupAnchor= [0,0]
                :iconUrl= "'assets/logo.png'"
                :shadowUrl= "'assets/logo.png'" >
                  <v-icon medium color="red" >
                    mdi-car
                  </v-icon>
            </l-icon>
      </l-marker>
    </l-map>
  </div>
</template>



<script>
import { latLng } from "leaflet";
import { LMap, LTileLayer, LMarker,  LIcon } from "vue2-leaflet";
const { io } = require("socket.io-client");
import axios from 'axios'
export default {
  name: "Example",
  components: {
    LMap,
    LIcon,
    LTileLayer,
    LMarker,
  
  },
  data() {
    return {
      locations: [],
      centrr: {
        lat:35,
        lng: 40
      },
      mrkr :{
        lat:35,
        lng: 40
      },
      zoom: 41,
      center: latLng(35,40),
      url: 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',
      attribution:
        '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors',
      currentZoom: 11.5,
      showParagraph: true,
      marker: latLng(35,40),
      mapOptions: {
        zoomSnap: 0.5
      },
      showMap: true
    };
  },
  computed: {
    slop(){
      let slop =  (this.centrr.lat - this.centrr.lng)/ (this.mrkr.lat - this.mrkr.lng)
      console.log(slop)
      return slop
    }
  },
  methods: {
    zoomUpdate(zoom) {
      this.currentZoom = zoom;
    },
    centerUpdate(center) {
      this.currentCenter = center;
    },
    innerClick() {
      alert("Click!");
    },
    getLocation(socket){
       navigator.geolocation.getCurrentPosition(
      position => {
          this.center = latLng(position.coords.latitude, position.coords.longitude)
          this.marker = latLng(position.coords.latitude, position.coords.longitude)
          this.mrkr.lat = position.coords.latitude, this.mrkr.lng =  position.coords.longitude
          socket.emit('update_location', {lat: position.coords.latitude, long: position.coords.longitude})
      },
      error => {
         console.log(error.message);
      },
   )
    }
  },
  
   async created(){
     const {data} = await  axios.get('/getLocations')
     this.locations = data
    const socket =  io('/')
    
    socket.on('update_location', data => {
      const location = this.locations.filter(location => location.id == data.id)
      location.lat = data.lat, location.long = data.long
    })
    socket.on('delete_location', id => {
      this.locations = this.locations.filter(location => location.id != id)
    })
    socket.on('send_location', data => {
      this.locations.push(data)
    })
    navigator.geolocation.getCurrentPosition(
      position => {
          this.center = latLng(position.coords.latitude, position.coords.longitude)
          this.marker = latLng(position.coords.latitude, position.coords.longitude)
          socket.emit('send_location', {lat: position.coords.latitude, long: position.coords.longitude})
      },
      error => {
         console.log(error.message);
      },
   )

    setInterval(() => {
      this.getLocation(socket)
    }, 500)
  },
  watch: {
    slop(){
      console.log(this.slop);
    }
  }
}
</script>