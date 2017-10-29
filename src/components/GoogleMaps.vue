<template>
    <gmap-map style="width: 100%; height: 100%; position: absolute; left:0; top:0"
        :center="{lat: 53.3498, lng: -6.2603}"
        :zoom="12"
    >
    <gmap-marker
          :key="index"
          v-for="(m, index) in markers"
          :position="m.position"
                :clickable="true"
                @click="center=m.position"
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
        markers: []
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
