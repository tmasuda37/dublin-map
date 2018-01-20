<template>
  <div>
    <h1>Bikes in Dublin</h1>
    <gmap-map style="width: 100%; height: 80%; position: absolute"
      :options="{styles: mapStyles}"
      :center="currentLocation"
      :zoom="16"
    >
      <gmap-info-window :options="infoOptions" :position="infoWindowPos" :opened="infoWinOpen" @closeclick="infoWinOpen=false">
        <h1>{{stationName}}</h1>
        <h2>Available Bikes:&nbsp;{{availableBikes}}</h2>
        <h2>Available Docks:&nbsp;{{availableDocks}}</h2>
        Last Update: {{updatedTime}}
      </gmap-info-window>
      <gmap-marker
        :key="index"
        v-for="(m, index) in markers"
        :position="m.position"
        :clickable="true"
        :label="m.label"
        @click="toggleInfoWindow(m,index)"
      >
      </gmap-marker>
      <gmap-marker
        :key="'current'"
        :position="currentLocation"
        :clickable="false"
      >
      </gmap-marker>
    </gmap-map>
  </div>
</template>

<script>
  import Vue from 'vue'
  import moment from 'moment'
  import * as VueGoogleMaps from 'vue2-google-maps'
  import VueAnalytics from 'vue-analytics'
  import axios from 'axios'
  import gmapStyle from '../config/google-map-style'

  Vue.use(VueGoogleMaps, {
    load: {
      key: 'AIzaSyAV-l5jESvA5UUfg8qFvwxIuG86akYWhMY'
    }
  })

  Vue.use(VueAnalytics, {
    id: 'UA-37342953-10'
  })

  export default {
    data () {
      return {
        markers: [],
        mapStyles: gmapStyle,
        currentLocation: {lat: 53.3498, lng: -6.2603},
        infoContent: '',
        stationName: '',
        label: '',
        availableBikes: '',
        availableDocks: '',
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
        return axios
          .get(
            'https://qlthh0d5p5.execute-api.us-east-1.amazonaws.com/service'
          )
          .then(function (res) {
            return res.data
          })
      },

      geolocation: function () {
        navigator.geolocation.getCurrentPosition(position => {
          this.currentLocation = {
            lat: position.coords.latitude,
            lng: position.coords.longitude
          }
        })
      },

      toggleInfoWindow: function (marker, idx) {
        this.infoWindowPos = marker.position

        this.stationName = marker.name
        this.availableBikes = marker.available_bikes
        this.availableDocks = marker.bike_stands
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
      this.$ga.page('/')
      this.geolocation()
      this.fetchData().then(items => {
        items.forEach(item => {
          item.label = `${item.available_bikes}/${item.bike_stands}`
          this.markers.push(item)
        })
      })
    }
  }
</script>