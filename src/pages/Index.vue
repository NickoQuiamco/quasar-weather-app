<template>
  <q-page class="flex column" :class="bgClass">
    <div class="col q-pt-lg q-px-md">
      <q-input
        v-model="search_loc"
        @keyup.enter="getWeatherbySearch"
        label="Search Location"
        dark
        dense>
        <template v-slot:before>
          <q-icon name="my_location" @click="getLocation" />
        </template>

        <template v-slot:append>
          <q-icon v-if="search_loc !== ''" name="close" @click="search_loc = ''" class="cursor-pointer" />
          <q-icon name="search" @click="getWeatherbySearch" />
        </template>
      </q-input>
    </div>
    <template v-if="weather_data" >
      <div class="col text-white text-center">  
        <div class="text-h4 text-weight-light">
          {{ weather_data.data.name }}
        </div>

        <div class="text-h6 text-weight-light">
          {{ weather_data.data.weather[0].main }}
        </div>
        
        <div class="text-h1 text-weight-thin relative-position ">
          <span> {{ Math.round(weather_data.data.main.temp) }} </span>
          <span class="text-h3 relative-position degree" >&deg;C</span>
        </div>

      </div>
      <div class="col text-center">
        <img :src="`https://openweathermap.org/img/wn/${ weather_data.data.weather[0].icon }@2x.png`" alt="weather_code">
      </div>
    </template>

    <template v-else>
      <div class="col column text-center text-white">
        <div class="col text-h2 text-weight-thin">
          Weather Application
        </div>
        <q-btn class="col" @click="getLocation" flat>
          <q-icon left size="2em" name="my_location" />
          <div>Find my location</div>
        </q-btn>
      </div>
    </template>
    
    <div class="col town"></div>
  </q-page>
</template>

<script>
import { defineComponent, ref, computed } from 'vue';
import axios from 'axios';
import { useQuasar } from 'quasar'

export default defineComponent({
  name: 'PageIndex',
  setup(){
    const $q = useQuasar();
    const search_loc = ref("");
    const weather_data = ref(false);
    const lat = ref(null);
    const long = ref(null);
    const api_url = ref('https://api.openweathermap.org/data/2.5/weather');
    const api_key = ref('e3ae2ef2d9fa4c9e38a129246509d6bf');
    //computed
    const bgClass = computed(function(){
      if(weather_data.value) {
        if(weather_data.value.data.weather[0].icon.endsWith('n')){
          return 'bg-night'
        }else{
          return 'bg-day '
        }

      }
    })
    //methods
    function getLocation(){
      $q.loading.show();
      if($q.platform.is.electron || $q.platform.is.cordova){
        axios.get('https://freegeoip.app/json/').then(res=>{
          console.log(res);
          lat.value = res.data.latitude;
          long.value = res.data.longitude;
          getWeatherByCoords();
          $q.loading.hide();
        })
      }else{
        navigator.geolocation.getCurrentPosition(position=>{
          lat.value = position.coords.latitude;
          long.value = position.coords.longitude;
          getWeatherByCoords()
          $q.loading.hide();
        })
      }
    }
    function getWeatherByCoords(){
      axios(`${ api_url.value }?lat=${ lat.value }&lon=${ long.value }&appid=${ api_key.value }&units=metric`).then(res=>{
        console.log(res);
        weather_data.value = res;
      })
    }
    function getWeatherbySearch(){
      $q.loading.show();
      axios(`${ api_url.value }?q=${ search_loc.value }&appid=${ api_key.value }&units=metric`).then(res=>{
        console.log(res);
        weather_data.value = res;
        $q.loading.hide();
      })
    }

    return{
      search_loc, weather_data, getLocation, getWeatherbySearch, bgClass
    } 
  }
})
</script>

<style lang="sass" scoped>
  .degree
    top:-28px
  .q-page
    background: linear-gradient(to top, #43c6ac, #191654)
    &.bg-day
      background: linear-gradient(to top, #22c1c3, #fdbb2d)
    &.bg-night
      background: linear-gradient(to bottom, #373b44, #4286f4)
  .town
    margin-top: 25px
    flex: 0 0 100px
    background: url(../assets/town.png)
    background-size: contain
    background-position: center bottom
</style>