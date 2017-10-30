<template>
    <gmap-map style="width: 100%; height: 100%; position: absolute; left:0; top:0"
      :center="{lat: 53.3498, lng: -6.2603}"
      :zoom="15"
  >
  <gmap-info-window :options="infoOptions" :position="infoWindowPos" :opened="infoWinOpen" @closeclick="infoWinOpen=false">
    <h5>{{stationName}}</h5>
    <h4>{{availableBikes}}&nbsp;/&nbsp;{{bikeStands}}</h4>
    Last Update: {{updatedTime}}
  </gmap-info-window>

  <gmap-marker
        :key="index"
        v-for="(m, index) in markers"
        :position="m.position"
        :clickable="true"
        :label="m.stopid"
        @click="toggleInfoWindow(m,index)"
      >
      </gmap-marker>
  </gmap-map>
</template>

<script>
  import Vue from 'vue'
  import moment from 'moment'
  import * as VueGoogleMaps from 'vue2-google-maps'
  import axios from 'axios'

  Vue.use(VueGoogleMaps, {
    load: {}
  })

  export default {
    data () {
      return {
        markers: [],
        infoContent: '',
        stationName: '',
        availableBikes: '',
        bikeStands: '',
        updatedTime: '',
        infoWindowPos: {
          lat: 0,
          lng: 0
        },
        infoWinOpen: false,
        infoOptions: {
          pixelOffset: {
            width: 0,
            height: -35
          }
        }
      }
    },

    methods: {
      fetchData () {
        console.log('call fetchData()')
        return axios
          .get(
            'https://api.jcdecaux.com/vls/v1/stations?contract=dublin&apiKey=eff457322380d25ccf635405ab615dd7df934db2'
          )
          .then(function (res) {
            return res.data
          })
      },

      toggleInfoWindow: function (marker, idx) {
        console.log('call toggleInfoWindow()')
        this.infoWindowPos = marker.position

        this.stationName = marker.name
        this.availableBikes = marker.available_bikes
        this.bikeStands = marker.bike_stands
        this.updatedTime = moment(marker.last_update).format('YYYY/MM/DD HH:mm')

        // check if its the same marker that was selected if yes toggle
        if (this.currentMidx === idx) {
          this.infoWinOpen = !this.infoWinOpen
        } else {
          // if different marker set infowindow to open and reset current marker index
          this.infoWinOpen = true
          this.currentMidx = idx
        }
      }
    },

    mounted () {
      this.fetchData().then(items => {
        items.forEach(item => {
          this.markers.push(item)
        })
      })
    }
  }
</script>