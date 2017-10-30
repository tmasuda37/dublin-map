<template>
    <gmap-map style="width: 100%; height: 100%; position: absolute; left:0; top:0"
        :center="{lat: 53.3498, lng: -6.2603}"
        :zoom="12"
    >

    <gmap-info-window :options="infoOptions" :position="infoWindowPos" :opened="infoWinOpen" @closeclick="infoWinOpen=false">
      {{infoContent}}
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
  import axios from 'axios'

  import * as VueGoogleMaps from 'vue2-google-maps'
  import Vue from 'vue'

  Vue.use(VueGoogleMaps, {
    load: {
    }
  })

  export default {
    data () {
      return {
        markers: [],
        infoContent: '',
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
      getBusStops () {
        console.log('call getBusStops()')
        return axios.get('https://data.dublinked.ie/cgi-bin/rtpi/routeinformation?routeid=46a&operator=bac&format=json').then(function (res) {
          return res.data.results[0].stops.map(stop => {
            stop.position = {
              lat: parseFloat(stop.latitude),
              lng: parseFloat(stop.longitude)
            }
            return stop
          })
        })
      },

      toggleInfoWindow: function (marker, idx) {
        console.log('call toggleInfoWindow()')
        this.infoWindowPos = marker.position
        this.infoContent = `<b>${marker.stopid}: ${marker.fullname}</b>`

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
      this.getBusStops().then(allStops => {
        allStops.forEach(stop => {
          console.log(stop)
          this.markers.push(stop)
        })
      })
    }
  }
</script>
