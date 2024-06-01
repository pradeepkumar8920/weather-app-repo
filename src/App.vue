<!-- App.vue -->
<template>
  <div class="container">
    <div class="header">
      <h1>WEATHER APP</h1>
      <div class="search-bar">
        <input
          type="text"
          v-model="city"
          placeholder="Enter city name"
          class="search-input"
        />
        <button @click="searchByCity"
                class="search-button">Search</button>
      </div>
    </div>

    <main class="main-section">
      <div v-if="weatherData" class="weather">
        <h2>{{ weatherData.name }}, 
            {{ weatherData.sys.country }}</h2>
        <div class="temp-box">
          <img :src="iconUrl" alt="Weather Icon"
                  class="weather-icon" />
          <p class="temperature">{{ temperature }} °C</p>
        </div>
        <span class="clouds">{{ weatherData.weather[0].description }}</span>
      </div>
      <div v-else class="loading">Loading...</div>
      <div class="divider"></div>

      <div class="forecast">
        <div class="cast-header">Upcoming forecast</div>
        <div class="forecast-list">
          <div
            class="next"
            v-for="(forecast, index) in hourlyForecast"
            :key="index"
          >
            <div>
              <p class="time">{{ forecast.time }}</p>
              <p class="temp-max">{{ forecast.temp_max }} °C</p>
              <p class="temp-min">{{ forecast.temp_min }} °C</p>
            </div>
            <p class="desc">{{ forecast.description }}</p>
          </div>
        </div>
      </div>
    </main>

    <div v-if="dailyForecast.length" class="forecast">
      <div class="cast-header">Next 4 days forecast</div>
      <div class="forecast-list">
        <div
          class="day"
          v-for="(forecast, index) in dailyForecast"
          :key="index"
        >
          <p class="date">{{ forecast.date }}</p>
          <p class="temp-max">{{ forecast.temp_max }} °C</p>
          <p class="temp-min">{{ forecast.temp_min }} °C</p>
          <p class="desc">{{ forecast.description }}</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";

const apikey = "feff206daa60b539abe8fae8f2ab7f29";

export default {
  name: "App",
  data() {
    return {
      city: "",
      weatherData: null,
      hourlyForecast: [],
      dailyForecast: [],
    };
  },
  computed: {
    temperature() {
      return this.weatherData
        ? Math.floor(this.weatherData.main.temp - 273)
        : null;
    },
    iconUrl() {
      return this.weatherData
        ? 
`http://api.openweathermap.org/img/w/${this.weatherData.weather[0].icon}.png`
        : null;
    },
  },
  mounted() {
    this.fetchCurrentLocationWeather();
  },
  methods: {
    async fetchCurrentLocationWeather() {
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(async (position) => {
          const { latitude, longitude } = position.coords;
          const url = `ht
tp://api.openweathermap.org/data/2.5/weather?lat=${latitude}&lon=${longitude}&appid=${apikey}`;
          await this.fetchWeatherData(url);
        });
      }
    },
    async fetchWeatherData(url) {
      try {
        const response = await axios.get(url);
        this.weatherData = response.data;
        // Fetch forecast data
        await this.fetchForecast(this.weatherData.name);
      } catch (error) {
        console.error("Error fetching weather data:", error);
      }
    },
    async fetchForecast(city) {
      const urlcast =
 `http://api.openweathermap.org/data/2.5/forecast?q=${city}&appid=${apikey}`;
      try {
        const response = await axios.get(urlcast);
        const forecast = response.data;
        this.hourForecast(forecast);
        this.dayForecast(forecast);
      } catch (error) {
        console.error("Error fetching forecast data:", error);
      }
    },
    async searchByCity() {
      if (!this.city) return;
      try {
        const urlsearch = 
`http://api.openweathermap.org/data/2.5/weather?q=${this.city}&appid=${apikey}`;
        const response = await axios.get(urlsearch);
        this.weatherData = response.data;
        // Fetch forecast data
        await this.fetchForecast(this.weatherData.name);
      } catch (error) {
        console.error("Error fetching weather data:", error);
      }
      this.city = "";
    },
    hourForecast(forecast) {
      this.hourlyForecast = [];
      for (let i = 0; i < 5; i++) {
        const date = new Date(forecast.list[i].dt * 1000);
        this.hourlyForecast.push({
          time: date
            .toLocaleTimeString(undefined, "Asia/Kolkata")
            .replace(":00", ""),
          temp_max: Math.floor(forecast.list[i].main.temp_max - 273),
          temp_min: Math.floor(forecast.list[i].main.temp_min - 273),
          description: forecast.list[i].weather[0].description,
        });
      }
    },
    dayForecast(forecast) {
      this.dailyForecast = [];
      for (let i = 8; i < forecast.list.length; i += 8) {
        const date = new Date(forecast.list[i].dt * 1000);
        this.dailyForecast.push({
          date: date.toDateString(undefined, "Asia/Kolkata"),
          temp_max: Math.floor(forecast.list[i].main.temp_max - 273),
          temp_min: Math.floor(forecast.list[i].main.temp_min - 273),
          description: forecast.list[i].weather[0].description,
        });
      }
    },
  },
};
</script>

<style scoped src="../src/styles.css">
/* @import '../src/styles.css'; */
</style>
