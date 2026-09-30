<template>
  <div class="app">
    <div class="weather-card">

      <h1>Weather App</h1>

      <div class="search-box">
        <input
          v-model="location"
          type="text"
          placeholder="Enter location"
          @keyup.enter="getWeather"
        />

        <button @click="getWeather">
          Search
        </button>
      </div>

      <div v-if="weather" class="weather">

        <h2>
          {{ weather.name }}
        </h2>

        <img
          :src="weatherIcon"
          alt="Weather icon"
        />

        <div class="temperature">
          {{ Math.round(weather.main.temp) }}°C
        </div>

        <p class="condition">
          {{ weather.weather[0].description }}
        </p>

        <div class="details">

          <div>
            <span>Feels Like</span>
            <strong>
              {{ Math.round(weather.main.feels_like) }}°C
            </strong>
          </div>

          <div>
            <span>Humidity</span>
            <strong>
              {{ weather.main.humidity }}%
            </strong>
          </div>

       

        </div>

      </div>

      <p v-if="loading">
        Loading...
      </p>

      <p v-if="error" class="error">
        {{ error }}
      </p>

    </div>
  </div>
</template>


<script>
export default {

  data() {
    return {

      location: '',

      weather: null,

      loading: false,

      error: '',

      apiKey: 'a26c901cd75698e5049c29f6c903452f'

    }
  },


  computed: {

    weatherIcon() {

      if (!this.weather) {
        return ''
      }

      return `https://openweathermap.org/img/wn/${this.weather.weather[0].icon}@2x.png`

    }

  },


  methods: {

    async getWeather() {

      if (!this.location.trim()) {
        return
      }

      this.loading = true

      this.error = ''

      const url =
        `https://api.openweathermap.org/data/2.5/weather?q=${encodeURIComponent(this.location)}&units=metric&appid=${this.apiKey}`

      try {

        const response = await fetch(url)

        if (!response.ok) {
          throw new Error('Location not found')
        }

        const data = await response.json()

        this.weather = data

      } catch (error) {

        this.weather = null

        this.error = error.message

      } finally {

        this.loading = false

      }

    }

  }

}
</script>


<style>
* {
  box-sizing: border-box;
}

body {
  margin: 0;
  font-family: Arial, sans-serif;
}

.app {
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  background: #e8f3ff;
  padding: 20px;
}

.weather-card {
  width: 100%;
  max-width: 500px;
  background: white;
  padding: 30px;
  border-radius: 20px;
  text-align: center;
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.15);
}

h1 {
  margin-top: 0;
}

.search-box {
  display: flex;
  gap: 10px;
  margin: 25px 0;
}

.search-box input {
  flex: 1;
  padding: 12px;
  font-size: 16px;
  border: 1px solid #ccc;
  border-radius: 8px;
}

.search-box button {
  padding: 12px 20px;
  border: none;
  border-radius: 8px;
  background: #007bff;
  color: white;
  cursor: pointer;
}

.weather img {
  width: 120px;
}

.temperature {
  font-size: 60px;
  font-weight: bold;
}

.condition {
  font-size: 20px;
  text-transform: capitalize;
  color: #666;
}

.details {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 15px;
  margin-top: 25px;
}

.details div {
  background: #f2f6fa;
  padding: 15px;
  border-radius: 10px;
}

.details span {
  display: block;
  color: #777;
  font-size: 14px;
  margin-bottom: 5px;
}

.details strong {
  font-size: 20px;
}

.error {
  color: red;
}
</style>