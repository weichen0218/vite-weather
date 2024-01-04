<script>
import debounce from "lodash.debounce";
export default {
  data() {
    return {
      location: "",
      preLocation: "",
      weather: {},
      weatherType: ["Clouds", "Clear", "Rain", "Snow", "Drizzle", "Mist"],
      temperature: 0,
      notFound: false,
      isShow: false,
    };
  },
  computed: {
    weatherIcon() {
      if (this.weatherType.includes(this.weather.weather[0].main)) {
        const weatherName = this.weather.weather[0].main.toLowerCase();
        return `src/assets/images/${weatherName}.png`;
      } else {
        return `src/assets/images/clear.png`;
      }
    },
  },
  methods: {
    filterInput() {
      this.location = this.location.trim();
    },
    checkLocation() {
      const cityNameRegex = /^[a-zA-Z\u0080-\u024F]+(?:([\ \-\']|(\.\ ))[a-zA-Z\u0080-\u024F]+)*$/;
      if (!this.location) {
        alert("Please enter a location");
        return false;
      }
      if (this.location === this.preLocation) {
        alert("Input is the same as before");
        return false;
      }
      if (!cityNameRegex.test(this.location)) {
        alert("Please enter a valid location");
        return false;
      }
      this.preLocation = this.location;
      return true;
    },
    getWeather() {
      this.isShow = false;
      this.notFound = false;
      const url = `${import.meta.env.VITE_WEATHER}/weather?q=${this.location}&units=metric&appid=${import.meta.env.VITE_API_KEY}`;
      this.$http
        .get(url)
        .then((res) => {
          this.weather = res.data;
          this.isShow = true;
          this.temperature = Math.round(this.weather.main.temp);
        })
        .catch((error) => {
          console.error("Error: ", error);
          this.notFound = true;
        });
    },
    debounceWeather: debounce(function () {
      this.filterInput();
      if (this.checkLocation()) {
        this.getWeather();
      }
    }, 500),
  },
  created() {},
};
</script>

<template>
  <div class="card" :style="{ 'background-position': temperature < 20 ? 'left' : 'right' }">
    <div class="search-box">
      <input type="text" class="search-bar" placeholder="Enter Location" v-model="location" @keyup.enter="debounceWeather" pattern="[A-Za-z]*" />
    </div>
    <div class="wrap">
      <transition name="slide" mode="out-in">
        <div class="weather" v-if="weather.main && !this.notFound">
          <div class="weather-box">
            <div class="weather-info" :class="{ fadeIn: isShow }">
              <img class="weather-icon" :src="weatherIcon" alt="" />
              <h1 class="temp">{{ weather.main.temp.toFixed() }}°C</h1>
              <h2 class="city">{{ weather.name }}</h2>
            </div>
          </div>

          <div class="weather-details">
            <div class="col" :class="{ fadeIn: isShow }">
              <img src="./assets/images/humidity.png" alt="" />
              <div class="humidity">
                <p>{{ weather.main.humidity }}%</p>
                <p>Humidity</p>
              </div>
            </div>
            <div class="col" :class="{ fadeIn: isShow }">
              <img src="./assets/images/wind.png" alt="" />
              <div class="wind">
                <p>{{ weather.wind.speed.toFixed(1) }} km/h</p>
                <p>Wind Speed</p>
              </div>
            </div>
          </div>
        </div>

        <div class="error" v-else-if="this.notFound">
          <div class="error-box">
            <img src="./assets/images/404.png" alt="" />
            <p>Oops! Location not found!</p>
          </div>
        </div>
      </transition>
    </div>
  </div>
</template>
