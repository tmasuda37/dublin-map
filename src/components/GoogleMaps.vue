<template>
  <div>
    <h1>Dublin Bike Usage Info on Map</h1>
    <gmap-map style="width: 100%; height: 80%; position: absolute"
      :options="{styles: mapStyles}"
      :center="{lat: 53.3498, lng: -6.2603}"
      :zoom="15"
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
    </gmap-map>
  </div>
</template>

<script>
  import Vue from 'vue'
  import moment from 'moment'
  import * as VueGoogleMaps from 'vue2-google-maps'
  import VueAnalytics from 'vue-analytics'
  import axios from 'axios'

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
        mapStyles: [
          {
            'elementType': 'geometry',
            'stylers': [
              {
                'color': '#242f3e'
              }
            ]
          },
          {
            'elementType': 'labels',
            'stylers': [
              {
                'visibility': 'off'
              }
            ]
          },
          {
            'elementType': 'labels.text.fill',
            'stylers': [
              {
                'color': '#746855'
              }
            ]
          },
          {
            'elementType': 'labels.text.stroke',
            'stylers': [
              {
                'color': '#242f3e'
              }
            ]
          },
          {
            'featureType': 'administrative.land_parcel',
            'stylers': [
              {
                'visibility': 'off'
              }
            ]
          },
          {
            'featureType': 'administrative.locality',
            'elementType': 'labels.text.fill',
            'stylers': [
              {
                'color': '#d59563'
              }
            ]
          },
          {
            'featureType': 'administrative.neighborhood',
            'stylers': [
              {
                'visibility': 'off'
              }
            ]
          },
          {
            'featureType': 'poi',
            'elementType': 'labels.text.fill',
            'stylers': [
              {
                'color': '#d59563'
              }
            ]
          },
          {
            'featureType': 'poi.park',
            'elementType': 'geometry',
            'stylers': [
              {
                'color': '#263c3f'
              }
            ]
          },
          {
            'featureType': 'poi.park',
            'elementType': 'labels.text.fill',
            'stylers': [
              {
                'color': '#6b9a76'
              }
            ]
          },
          {
            'featureType': 'road',
            'elementType': 'geometry',
            'stylers': [
              {
                'color': '#38414e'
              }
            ]
          },
          {
            'featureType': 'road',
            'elementType': 'geometry.stroke',
            'stylers': [
              {
                'color': '#212a37'
              }
            ]
          },
          {
            'featureType': 'road',
            'elementType': 'labels.text.fill',
            'stylers': [
              {
                'color': '#9ca5b3'
              }
            ]
          },
          {
            'featureType': 'road.highway',
            'elementType': 'geometry',
            'stylers': [
              {
                'color': '#746855'
              }
            ]
          },
          {
            'featureType': 'road.highway',
            'elementType': 'geometry.stroke',
            'stylers': [
              {
                'color': '#1f2835'
              }
            ]
          },
          {
            'featureType': 'road.highway',
            'elementType': 'labels.text.fill',
            'stylers': [
              {
                'color': '#f3d19c'
              }
            ]
          },
          {
            'featureType': 'transit',
            'elementType': 'geometry',
            'stylers': [
              {
                'color': '#2f3948'
              }
            ]
          },
          {
            'featureType': 'transit.station',
            'elementType': 'labels.text.fill',
            'stylers': [
              {
                'color': '#d59563'
              }
            ]
          },
          {
            'featureType': 'water',
            'elementType': 'geometry',
            'stylers': [
              {
                'color': '#17263c'
              }
            ]
          },
          {
            'featureType': 'water',
            'elementType': 'labels.text.fill',
            'stylers': [
              {
                'color': '#515c6d'
              }
            ]
          },
          {
            'featureType': 'water',
            'elementType': 'labels.text.stroke',
            'stylers': [
              {
                'color': '#17263c'
              }
            ]
          }
        ],
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
      this.fetchData().then(items => {
        items.forEach(item => {
          item.label = `${item.available_bikes}/${item.bike_stands}`
          this.markers.push(item)
        })
      })
    }
  }
</script>