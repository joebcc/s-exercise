<template>
  <div id="app">
    <div v-if="error">{{error}}</div>
    <div class="input" v-if="!processed">
      <h2>EarthQuake Finder</h2>
      <div>
        Start Date:
        <br>
        <input type="date" class="startDate" v-model="inputData.startDate">
      </div>
      <div>
        End Date:
        <br>
        <input type="date" class="endDate" v-model="inputData.endDate">
      </div>
      <div>
        Minimum Magnitude:
        <br>
        <input type="number" minimum="0" maximum="10" class="minimumMagnitude" v-model="inputData.minimumMagnitude">
      </div>
      <div>
        Maximum Magnitude:
        <br>
        <input type="number" minimum="0" maximum="10" class="maximumMagnitude" v-model="inputData.maximumMagnitude">
      </div>
      <h4>Location:</h4>
      <div>
        Latitude:
        <br>
        <input type="number"  minimum="-90" maximum="90" class="latitude" v-model="inputData.location.latitude">
      </div>
      <div>
        Longitude
        <br>
        <input type="number"  minimum="-180" maximum="180" class="longitude" v-model="inputData.location.longitude">
      </div>
      <div>
        Radius from location in KM:
        <br>
        <input type="number" class="radius" maximum="20000" minimum="0" v-model="inputData.location.radius">
      </div>
      <button @click="processInput()">Find Earthquakes</button>
    </div>
    <div class="reponseData" v-if="processed && responseData.features[0]">
      <h2>Results:</h2>
      <h4>
        Median Magnitude of all earthquakes matching the chosen parameters: 
        {{ this.computedData.medianMagnitude }}
      </h4>
      <h4>
        Maximum Magnitude of all earthquakes matching the chosen parameters: 
        {{ this.computedData.maximumMagnitude }}
      </h4>
      <h4>
        Minimum Magnitude of all earthquakes matching the chosen parameters: 
        {{ this.computedData.minmumMagnitude }}
      </h4>
      <h4>
        Number of earthquakes matching the chosen parameters: 
        {{ this.computedData.minmumMagnitude }}
      </h4>
      <h4>
        Table of results:
      </h4>
      <div class="earthquakes">
        <div v-for="(eq, eqi) in responseData.features" :key="eqi" class="earthquake-entry">
          <h5>{{eq.properties.place}}</h5>
          <div>
            <span>Location:</span><br>
            <span>{{eq.geometry.coordinates[0]}}, {{eq.geometry.coordinates[1]}}, {{eq.geometry.coordinates[2]}}</span>
          </div>
          <div>
            <span>Magnitude:</span><br>
            <span>{{eq.properties.mag}}</span>
          </div>
          <div>
            <span>Time:</span><br>
            <span>{{ Date(eq.properties.time) }}</span>
          </div>
        </div>
      </div>

    </div>
    <div v-if="processed && !responseData.features[0]">
      No Results Found, reload and try again
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'App',
  data() { return {
    processed: false,
    inputData: {
      startDate: '',
      endDate: '',
      minimumMagnitude: 0,
      maximumMagnitude: 0,
      location: {
        latitude: [0,0],
        longitude: [0,0],
        radius: 0,
      }
    },
    responseData: {

    },
    error: '',
  } },
  computed: {
    computedData() {
      if (this.responseData.features) {

        const eqs = this.responseData.features;

        if (eqs && eqs.length) {
          const maxMag = eqs.reduce((a,b) => { if (a.properties.magnitude > b.properties.magnitude) { return a }});
          const minMag = eqs.reduce((a,b) => { if (a.properties.magnitude < b.properties.magnitude) { return a }});
          let combinedMag = 0;
          eqs.forEach(eq => {combinedMag += eq.magnitude})
          const medianMag = combinedMag / eqs.length;

          return {
            minimumMagnitude: minMag,
            maximumMagnitude: maxMag,
            medianMagnitude: medianMag,
            totalEQs: this.responseData.count,
          }
        }
      }
      return {}
    }
  },
  methods: {
    async processInput() {
      await this.getData()

      if (this.responseData.features && this.responseData.features.length > 100) {
        this.responseData = this.responseData.slice(0,100);
      }

    },
    async getData() {
      const url = `https://earthquake.usgs.gov/fdsnws/event/1/query?format=geojson&starttime=${this.inputData.startDate}&endtime=${this.inputData.endDate}&minmagnitude=${this.inputData.minimumMagnitude}&maxmagnitude=${this.inputData.maximumMagnitude}&latitude=${this.inputData.location.latitude}&longitude=${this.inputData.location.longitude}&maxradiuskm=${this.inputData.location.radius}`
      console.log(url)
      axios.get(url)
        .catch(err => {this.error = err; console.log(err)})
        .then(result => this.responseData = result.data)
        .then(() => this.processed = true)
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.input div,
.responseData > div,
.earthquake-entry {
  margin-bottom: 1rem;
}
</style>
