<template>
  <div class="meteo-container" :class="classDescription">
    <div class="modale visible">
      <div class="form">
        <label for="position">Entrez le nom d'une ville</label>
        <br />
        <input v-model="requete" v-on:keypress="goMeteo" type="text" id="position" />
      </div>
    </div>

    <div class="current-weather" v-for="(weather, index) in weatherCurrent" :key="index">
      <div class="sub-infos-wrapper">
        <div class="min-temp">
          <div>Min</div>
          <div>{{ Math.round(weather.main.temp_min) }}°C</div>
        </div>
        <div class="max-temp">
          <div>Max</div>
          <div>{{ Math.round(weather.main.temp_max) }}°C</div>
        </div>
        <div>
          <div>Ressenti</div>
          <div>{{ Math.round(weather.main.feels_like) }}°C</div>
        </div>
        <div>
          <div>Humidité</div>
          <div>{{ weather.main.humidity }}%</div>
        </div>
        <div>
          <div>Vent</div>
          <div>{{ weather.wind.speed }} km/h</div>
        </div>
      </div>

      <div
        class="description"
        v-for="(info, index) in weather.weather"
        :key="index"
      >{{ capitalize(info.description)}}</div>

      <div class="cercle-temp" @click="changeInfoCircle">
        <div class="temperature click-visible visible">{{ Math.round(weather.main.temp) }}</div>
        <div
          class="temperature click-visible not-visible"
        >{{ Math.round(weather.main.feels_like) }} ressenti</div>
      </div>

      <div class="city-wrapper">
        <div class="city" v-if="displayCity">
          <marquee :duration="5" :repeat="8">
            {{ weather.name }}
            <span class="spacer-desc"></span>
          </marquee>
        </div>
        <div>{{ dateTime[0] }}</div>
      </div>
    </div>

    <div class="list-weather">
      <div v-for="(weather, index) in weatherList" :key="index">
        {{ dateTextRemove(weather.dt_txt) }}
        <div
          class="description"
          :class="fitClass(info.description)"
          v-for="(info, index) in weather.weather"
          :key="index"
        >{{ info.description }}</div>
        <div class="temperature">{{ Math.round(weather.main.temp) }}°</div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import marquee from "vue-marquee-text-component";

export default {
  name: "Meteo",
  components: {
    marquee
  },
  data() {
    return {
      requete: "",
      api_code: process.env.VUE_APP_WEATHER_API,
      url_search: "https://api.openweathermap.org/data/2.5/forecast?",
      url_search_weather: "https://api.openweathermap.org/data/2.5/weather?",
      weatherList: [],
      weatherListToday: [],
      weatherCurrent: [],
      displayCity: false,
      dateTime: [],
      description: [],
      classDescription: ""
    };
  },

  methods: {
    capitalize: function(str) {
      if (typeof str === "string") {
        return str.replace(/^\w/, c => c.toUpperCase());
      } else {
        return "";
      }
    },

    fitClass: function(str) {
      return str.replace(" ", "-");
    },

    dateTextRemove: function(text) {
      let res = text.substring(text.indexOf(" ") + 1);
      let re = new RegExp(".{" + 5 + "}", "i");
      return res.match(re)[0];
    },

    goMeteo(e) {
      if (e.key == "Enter") {
        let modale = document.querySelector(".modale");

        // Appelle le temps des heures / jours suivants
        axios
          .get(
            `${this.url_search}q=${this.requete}&units=metric&APPID=${this.api_code}&lang=fr`
          )
          .then(response => {
            for (let i = 0; i < 6; i++) {
              this.weatherList.push(response.data.list[i]);
            }
          });

        // Appelle le temps actuel
        axios
          .get(
            `${this.url_search_weather}q=${this.requete}&units=metric&APPID=${this.api_code}&lang=fr`
          )
          .then(response => {
            this.weatherCurrent.push(response.data);

            for (let weather of this.weatherCurrent) {
              for (let w of weather.weather) {
                this.description.push(w.description);
              }
            }

            let classDescription = this.description[0].replace(" ", "-");
            this.classDescription = classDescription;
          });

        // Appelle le l'heure actuelle
        // axios
        //   .get(`http://worldtimeapi.org/api/timezone/Europe/${this.requete}`)
        //   .then(response => {
        //     let date = response.data.datetime.split("");
        //     date.splice(0, 11);
        //     date.splice(5, 16);
        //     this.dateTime.push(date.join(""));
        //   });

        this.displayCity = true;
        modale.classList.remove("visible");
      }
    },

    changeInfoCircle: function() {
      // let elems = document.querySelectorAll(".click-visible");
    }
  }
};
</script>

<style lang='scss'>
@import "Meteo.scss";
</style>