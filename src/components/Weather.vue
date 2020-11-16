<template>
  <div id="weather" class="card card-body shadow pb-2">
    <div
      v-if="loading"
      class="w-100 h-100 d-flex justify-content-center align-items-center"
    >
      <div class="spinner-border text-primary" role="status"></div>
    </div>

    <div v-else>
      <div v-if="!hasPermission">
        <hr />

        <div class="alert alert-danger" role="alert">
          This widget requires location permissions
        </div>
      </div>

      <div v-else>
        <div class="row">
          <div class="col">
            <weather-icons
              v-if="!gettingData"
              :icon="weatherData.data.weather[0].icon"
            />

            <div
              v-else
              class="iconLoader d-flex align-items-center justify-content-end"
            >
              <div class="spinner-grow text-muted" role="status">
                <span class="visually-hidden">Loading...</span>
              </div>
            </div>

            <p class="m-0">
              {{ capitalize(weatherData.data.weather[0].description) }}
            </p>
          </div>

          <div class="col-auto">
            <h1 class="m-0">{{ KtoC(weatherData.data.main.temp) }}&deg;</h1>

            <small class="text-muted"
              >Feels like
              {{ KtoC(weatherData.data.main.feels_like) }}&deg;</small
            >
          </div>
        </div>

        <hr class="mb-2" />

        <div class="row">
          <div class="col">
            <p class="small m-0">
              <span class="text-muted">{{ weatherData.data.name }}. </span>

              <span class="text-muted">{{ timestampToDate(timestamp) }}</span>
            </p>
          </div>

          <div class="col-auto">
            <p class="small m-0">
              <a
                href="#"
                v-if="!gettingData && !error"
                @click="getData()"
                class="ml-2"
              >
                <svg
                  width="18"
                  height="18"
                  fill="none"
                  stroke="currentColor"
                  stroke-width="2"
                  stroke-linecap="round"
                  stroke-linejoin="round"
                >
                  <use
                    xlink:href="@/assets/icons/feather-sprite.svg#refresh-cw"
                  />
                </svg>
              </a>

              <a
                href="#"
                v-if="!gettingData && error"
                @click="getData()"
                class="ml-2 text-danger"
              >
                <svg
                  width="18"
                  height="18"
                  fill="none"
                  stroke="currentColor"
                  stroke-width="2"
                  stroke-linecap="round"
                  stroke-linejoin="round"
                >
                  <use
                    xlink:href="@/assets/icons/feather-sprite.svg#alert-circle"
                  />
                </svg>
              </a>

              <span
                v-if="gettingData"
                class="spinner-border spinner-border-sm text-primary"
                role="status"
                aria-hidden="true"
              ></span>
            </p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";

import WeatherIcons from "../components/WeatherIcons.vue";

export default {
  name: "Weather",

  components: {
    WeatherIcons,
  },

  data() {
    return {
      loading: true,
      gettingData: false,
      error: false,
      apiURL: "http://api.openweathermap.org/data/2.5/weather?",
      hasPermission: false,
      pos: {
        lat: null,
        lon: null,
      },
      weatherData: null,
      timestamp: null,
    };
  },

  methods: {
    // Converts Kelvin to Celsius
    KtoC(val) {
      return Math.round(val - 273.15);
    },

    // Converts timestamp to date string
    timestampToDate(val) {
      const D = new Date(val);
      const months = [
        "Jan",
        "Feb",
        "Mar",
        "Apr",
        "May",
        "Jun",
        "Jul",
        "Aug",
        "Sep",
        "Oct",
        "Nov",
        "Dec",
      ];

      let hours = D.getHours();
      let minutes = D.getMinutes();

      if (hours < 10) hours = "0" + hours;
      if (minutes < 10) minutes = "0" + minutes;

      return `${months[D.getMonth()]} ${D.getDate()}, ${hours}:${minutes}`;
    },

    // capitalizes first letter of all words in string
    capitalize(str) {
      let words = str
        .trim()
        .toLowerCase()
        .split(" ");

      let output = "";

      words.forEach((word) => {
        output += word[0].toUpperCase() + word.slice(1) + " ";
      });

      return output.trim();
    },

    // Get weather data from the Open Weather API
    async getData(lat, lon) {
      try {
        this.gettingData = true;
        this.error = false;

        const url =
          this.apiURL +
          `lat=${lat || this.pos.lat}&lon=${lon || this.pos.lon}&appid=${
            process.env.VUE_APP_OPEN_WEATHER_API_KEY
          }`;

        let response = await axios.get(url);
        this.weatherData = response;
        this.timestamp = new Date().getTime();

        // Cache weather data in local storage
        await new Promise((resolve) => {
          chrome.storage.local.set(
            {
              weatherData: {
                weather: response,
                timestamp: new Date().getTime(),
              },
            },
            (value) => resolve(value)
          );
        });

        this.gettingData = false;
      } catch (e) {
        console.error(e);
        this.gettingData = false;
        this.error = true;
      }
    },

    // Set the current geolocation
    async getLocation() {
      if (navigator.geolocation) {
        try {
          let position = await new Promise((resolve) =>
            navigator.geolocation.getCurrentPosition((pos) => {
              resolve(pos);
            })
          );

          this.pos.lat = position.coords.latitude;
          this.pos.lon = position.coords.longitude;
        } catch (e) {
          console.error(e);
        }
      }
    },
  },

  async mounted() {
    // Check that geolocation is working
    await new Promise((resolve) => {
      chrome.permissions.contains(
        {
          permissions: ["geolocation"],
          origins: ["http://www.google.com/"],
        },
        (result) => {
          if (result) {
            this.hasPermission = true;
          }

          resolve();
        }
      );
    });

    // Get cached data from local storage
    let localData = await new Promise((resolve) => {
      chrome.storage.local.get(["weatherData"], (result) => resolve(result));
    });

    // Calculate time difference in seconds from cache date and current date
    const cacheD = new Date(localData.weatherData.timestamp);
    const currentD = new Date();
    const DDiff = (currentD.getTime() - cacheD.getTime()) / 1000;

    // Set data to cahced data, unless cache is older than one hour
    if (localData && DDiff < 3600) {
      this.weatherData = localData.weatherData.weather;
      this.timestamp = localData.weatherData.timestamp;
      this.pos.lat = localData.weatherData.weather.data.coord.lat;
      this.pos.lon = localData.weatherData.weather.data.coord.lon;
    } else {
      await this.getLocation();
      await this.getData();
    }

    this.loading = false;
  },
};
</script>

<style>
#weather {
  max-width: 20rem;
}

.iconLoader {
  width: 56px;
  height: 56px;
}
</style>
