<template>
  <div class="meteo-container" :class="[currentTemp, classDescription]">
    <div class="modale visible">
      <div class="title-wrapper">
        <img src="../assets/giphy.gif" alt srcset />
        <h1 class="title">Weather.App</h1>
      </div>
      <div class="form">
        <label for="position">Entrez le nom d'une ville</label>
        <br />
        <input
          v-model="requete"
          v-on:keypress="goMeteo"
          placeholder="Paris, Marseille, New-York..."
          type="text"
          id="position"
        />
      </div>
    </div>

    <div class="current-weather" v-for="(weather, index) in weatherCurrent" :key="index">
      <div class="description" v-for="(info, index) in weather.weather" :key="index">
        <div class="icon-weather"></div>
      </div>

      <div class="cercle-temp">
        <div class="temperature" @click="changeInfoCircle">
          <div class="sub-temp-infos">Température</div>
          {{ Math.round(weather.main.temp) }}
          <div class="sub-temp-infos">°C</div>
        </div>
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
      classDescription: "",
      dataInfoCirle: 0,
      currentTemp: ""
    };
  },

  methods: {
    capitalize: function(str) {
      // Ajoute une majuscule à la première lettre
      if (typeof str === "string") {
        return str.replace(/^\w/, c => c.toUpperCase());
      } else {
        return "";
      }
    },

    fitClass: function(str) {
      // Enlève l'espace et ajoute "-" pour les classes
      return str.replace(" ", "-");
    },

    dateTextRemove: function(text) {
      // Permet d'afficher les heures au format "HH:MM"
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
            // ajout de la réponse des données à this.weatherCurrent
            this.weatherCurrent.push(response.data);

            for (let weather of this.weatherCurrent) {
              this.currentTemp = weather.main.temp;

              // Ajout d'une classe au conteneur selon la température
              if (this.currentTemp > 30) {
                this.currentTemp = "temp-plus-30";
              } else if (this.currentTemp > 25) {
                this.currentTemp = "temp-plus-25";
              } else if (this.currentTemp > 20) {
                this.currentTemp = "temp-plus-20";
              } else if (this.currentTemp > 15) {
                this.currentTemp = "temp-plus-15";
              } else if (this.currentTemp > 10) {
                this.currentTemp = "temp-plus-10";
              } else if (this.currentTemp < 10) {
                this.currentTemp = "temp-moins-10";
              }

              for (let w of weather.weather) {
                // Ajout de description au tableau this.description
                this.description.push(w.description);
              }
            }

            // Rajoute un tiret à la place de l'espace pour ajout de description en classe
            let classDescription = this.description[0].replace(" ", "-");
            this.classDescription = classDescription;
          });

        // Affiche l'écran météo avec la ville séléctionnée
        this.displayCity = true;

        // Retire l'écran avec l'input
        modale.classList.remove("visible");
      }
    },

    changeInfoCircle: function(e) {
      // Au click sur le cercle, change l'info affichée
      let dataWeather = this.weatherCurrent[0].main;
      let dataWeatherWind = this.weatherCurrent[0].wind.speed;

      // Données affichées au moment du click
      let arrOfDatas = [
        {
          descr: "Ressenti",
          unity: "°C",
          data: Math.round(dataWeather.feels_like)
        },
        {
          descr: "Minimum",
          unity: "°C",
          data: Math.round(dataWeather.temp_min)
        },
        {
          descr: "Maximum",
          unity: "°C",
          data: Math.round(dataWeather.temp_max)
        },
        {
          descr: "Humidité",
          unity: "%",
          data: dataWeather.humidity,
          class: "percent"
        },
        {
          descr: "Température",
          unity: "°C",
          data: Math.round(dataWeather.temp)
        },
        { descr: "Vent", unity: "km/h", data: Math.round(dataWeatherWind) }
      ];

      // Remet à 0 l'index du tableau quand on arrive au dernier index
      if (this.dataInfoCirle === arrOfDatas.length) {
        this.dataInfoCirle = 0;
      }

      // ajout des données au DOM
      e.path[0].innerHTML = `
      <div class="sub-temp-infos">
      ${arrOfDatas[this.dataInfoCirle].descr}
      </div>
      ${arrOfDatas[this.dataInfoCirle].data}
      <div class="sub-temp-infos">${arrOfDatas[this.dataInfoCirle].unity}</div>
      `;

      // Incrémente de 1 infoCircle
      this.dataInfoCirle++;
    }
  }
};
</script>

<style lang='scss'>
@import "Meteo.scss";
</style>