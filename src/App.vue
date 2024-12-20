<script>
import axios from "axios";

export default {
  data() {
    return {
      city: "Tashkent",
      error: "",
      todayInfo: null,
      dailyForecast: [],
      specificDayInfo: null,
      isDarkMode: false,
      loading: false, // Track loading state
    };
  },
  mounted() {
    // Sahifa yuklanganda ob-havoni darhol olish
    this.getWeather();
  },
  methods: {
    async getWeather() {
      if (this.city.trim().length < 2) {
        alert("ERROR! Write the name of the city with more than 2 letters");
        return;
      }
      this.error = "";
      this.todayInfo = null;
      this.dailyForecast = [];
      this.specificDayInfo = null;
      this.loading = true; // Set loading state to true

      try {
        // Fetch today's weather
        const todayResponse = await axios.get(
          `https://api.openweathermap.org/data/2.5/weather?q=${this.city}&units=metric&appid=1f0a447e37781df1d4c898814da66680`
        );

        this.todayInfo = todayResponse.data;

        // Fetch forecast for the next 7 days
        const forecastResponse = await axios.get(
          `https://api.openweathermap.org/data/2.5/forecast?q=${this.city}&units=metric&appid=1f0a447e37781df1d4c898814da66680`
        );

        const forecastList = forecastResponse.data.list;
        const today = new Date().toISOString().split("T")[0];
        const daysMap = {};

        forecastList.forEach((item) => {
          const date = item.dt_txt.split(" ")[0];
          if (!daysMap[date]) {
            daysMap[date] = item;
          }
          if (date === "2024-12-26") {
            this.specificDayInfo = item;
          }
        });

        this.dailyForecast = Object.values(daysMap)
          .filter((item) => item.dt_txt.split(" ")[0] !== today)
          .slice(0, 7);
      } catch (error) {
        this.error = "Could not fetch weather data. Please try again.";
      } finally {
        this.loading = false; // Set loading state to false after data is fetched
      }
    },
    toggleDarkMode() {
      this.isDarkMode = !this.isDarkMode;
      if (this.isDarkMode) {
        document.body.classList.add("dark-mode");
        localStorage.setItem("theme", "dark");
      } else {
        document.body.classList.remove("dark-mode");
        localStorage.setItem("theme", "light");
      }
    },
  },
};
</script>

<template>
  <div :class="{ 'dark-mode': isDarkMode }" class="wrapper">
    <!-- Navbar Section -->
    <header class="navbar">
      <h2 class="navbar-title">The Weather App</h2>
      <p class="navbar-text">
        Find out the weather in {{ city || "YOUR CITY" }}
      </p>
      <input
        class="navbar-input"
        type="text"
        v-model="city"
        placeholder="Enter city"
      />
      <button class="navbar-button" v-if="city" @click="getWeather">
        Get the Weather
      </button>
      <button class="navbar-button" disabled v-else>Enter name of city</button>
      <!-- Dark/Light Mode Switch Button with Icons -->
      <button @click="toggleDarkMode" class="mode-toggle-btn">
        <span v-if="isDarkMode">🌞</span>
        <span v-else>🌙</span>
      </button>
    </header>

    <!-- Loader when data is loading -->
    <div v-if="loading" class="loader">Loading weather data...</div>

    <!-- Display Results when data is fetched -->
    <div v-if="!loading && (todayInfo || dailyForecast.length)" class="results">
      <!-- Today's weather -->
      <div class="card">
        <h3>Today</h3>
        <p>Temperature: {{ todayInfo?.main?.temp }}°C</p>
        <p>Feels Like: {{ todayInfo?.main?.feels_like }}°C</p>
        <p>Min Temperature: {{ todayInfo?.main?.temp_min }}°C</p>
        <p>Max Temperature: {{ todayInfo?.main?.temp_max }}°C</p>
      </div>

      <!-- Next 6 days forecast -->
      <div class="forecast">
        <div class="card" v-for="(day, index) in dailyForecast" :key="index">
          <h3>{{ day.dt_txt.split(" ")[0] }}</h3>
          <p>Temperature: {{ day?.main?.temp }}°C</p>
          <p>Min Temperature: {{ day?.main?.temp_min }}°C</p>
          <p>Max Temperature: {{ day?.main?.temp_max }}°C</p>
          <p>Weather: {{ day?.weather[0]?.description }}</p>
        </div>
      </div>

      <!-- Weather for specific day (2024-12-26) -->
      <div v-if="specificDayInfo" class="card">
        <h3>2024-12-26</h3>
        <p>Temperature: {{ specificDayInfo?.main?.temp }}°C</p>
        <p>Min Temperature: {{ specificDayInfo?.main?.temp_min }}°C</p>
        <p>Max Temperature: {{ specificDayInfo?.main?.temp_max }}°C</p>
        <p>Weather: {{ specificDayInfo?.weather[0]?.description }}</p>
      </div>
    </div>

    <!-- Footer with creator info -->
    <footer class="footer">
      <p>&copy Created by Iskandarov Firdavs</p>
    </footer>
  </div>
</template>
