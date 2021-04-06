<template>
  <q-page class="flex column" :class="bgClass">
    <div class="col q-pt-lg q-px-md">
      <q-input @keyup.enter="getWeatherByCity" v-model="searchInput" placeholder="Search" dark borderless>
        <template v-slot:before>
          <q-icon name="my_location" @click="getUserLocation" />
        </template>

        <template v-slot:append>
          <q-btn round dense flat icon="search" @click="getWeatherByCity" />
        </template>
      </q-input>
    </div>

    <template v-if="weatherData">
      <div class="col text-white text-center">
        <div class="text-h4 text-weight-light">
          {{ weatherData[0].city_name }}
        </div>
        <div class="text-h6 text-weight-light">
          {{ weatherData[0].weather.description }}
        </div>
        <div class="text-h1 text-weight-thin q-my-lg relative-position">
          <span>{{ Math.round(weatherData[0].temp) }}</span>
          <span class="text-h4 relative-position degree">&deg;C</span>
        </div>
      </div>

      <div class="text-center col">
        <img :src="`https://www.weatherbit.io/static/img/icons/${weatherData[0].weather.icon}.png`">
      </div>
    </template>

    <template v-else>
      <div class="col column text-center text-white">
        <div class="col text-h2 text-weight-thin">
          Weather forecast
        </div>
        <div class="col text-h6 text-weight-thin">
          by
        </div>
        <div class="col text-h5">
          <span class="text-weight-thin">JOSIP</span>
          <span class="text-weight-bold">LEKO</span>
        </div>

        <q-btn class="col" flat @click="getUserLocation">
          <q-icon left size="3em" name="my_location" />
          <div>Find my location</div>
        </q-btn>
      </div>
    </template>

    <div class="col silhouette"></div>
  </q-page>
</template>

<script>
export default {
  name: 'PageIndex',
  data(){
    return {
      searchInput: undefined,
      weatherData: undefined,
      // api user: WeatherBit
      apiUrl: "https://api.weatherbit.io/v2.0/current",
      apiKey: "46a1983af7c1417991b9689a33758d67",
    }
  },

  computed: {
    bgClass(){
      if(this.weatherData){
        if(this.weatherData[0].weather.icon.endsWith("n")){
          return "bg-dark"
        } else {
          return "bg-light"
        }
      }
    }
  },

  methods: {
    getUserLocation(){
      this.$q.loading.show();

      // get users location
      navigator.geolocation.getCurrentPosition(position => {
        const lat = position.coords.latitude,
              lon = position.coords.longitude;
        this.getWeatherByCoords(lat, lon);
      });
    },

    getWeatherByCity(){
      if(!this.searchInput) return;
      
      this.$q.loading.show();

      const urlConfig = `${this.apiUrl}?city=${this.searchInput.trim()}&key=${this.apiKey}&units=M`;

      this.$axios(urlConfig)
      .then(response => {
        // retrieved weather data for city name
        this.weatherData = response.data.data;

        this.$q.loading.hide();
      })
      .catch(error => console.log("Error: " + error));

      this.searchInput = "";
    },

    getWeatherByCoords(lat, lon){
      this.$axios(`${this.apiUrl}?lat=${lat}&lon=${lon}&key=${this.apiKey}&units=M`)
      .then(response => {
        // retrieved weather data for coords
        this.weatherData = response.data.data;
      })
      .catch(error => console.log(error));

      this.$q.loading.hide();
    }
  }
}
</script>

<style lang="sass">
.q-page
  background: #23074d
  background: -webkit-linear-gradient(to top, #cc5333, #23074d)
  background: linear-gradient(to top, #cc5333, #23074d)

  &.bg-dark
    background: #232526
    background: -webkit-linear-gradient(to bottom, #414345, #232526)
    background: linear-gradient(to bottom, #414345, #232526)

  &.bg-light
    background: #2193b0
    background: -webkit-linear-gradient(to bottom, #6dd5ed, #2193b0)
    background: linear-gradient(to bottom, #6dd5ed, #2193b0)

.degree
  top: -43px

.silhouette
  flex: 0 0 100px
  background: url(../assets/silhouette.png)
  background-size: contain
  background-position: center bottom
</style>