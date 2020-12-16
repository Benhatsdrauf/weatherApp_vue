<template>
  <div id="app">
    <main v-touch:swipe.bottom="fetchWeatherData">
      <div class="search-box">
        <input
          autocomplete="off"
          id="searchBar"
          type="text"
          class="search-bar"
          placeholder="Search..."
          @keypress.enter="fetchWeatherData"
        />
      </div>
      <div
        class="weather-wrap"
        v-if="typeof currentWeather.main != 'undefined'"
      >
        <div class="location-box">
          <div class="location">
            {{ currentWeather.name }}, {{ currentWeather.sys.country }}
          </div>
          <div class="date">{{ dateBuilder() }}</div>
        </div>
        <div class="weather-box">
          <div class="temp">{{ Math.round(currentWeather.main.temp) }}°c</div>
          <div class="weather">
            {{ currentWeather.weather[0].description }}
          </div>
        </div>
        <div class="forcast-wrap">
          <div
            class="forcast-row"
            v-for="day in forcastWeather.daily"
            v-bind:key="day.dt"
          >
            <p class="item-left">{{ day.weather[1] }}</p>
            <p class="item-right">
              {{ Math.round(day.temp.min) }}°c/{{ Math.round(day.temp.max) }}°c
            </p>
          </div>
        </div>
      </div>
      <div v-else>
        <h1 class="error-message" id="errorMessage">
          Ort konnte nicht gefunden werden!
        </h1>
      </div>
    </main>
  </div>
</template>

<script>
export default {
  name: "App",
  data: () => ({
    api_key: process.env.VUE_APP_API_KEY,
    api_base: "https://api.openweathermap.org/data/2.5/weather?q=",
    currentWeather: {},
    forcastWeather: {},
    locationData: {},
  }),
  methods: {
    // fetching the weather data for the location the user put in the search bar
    fetchWeatherData() {
      this.fetchCurrentWeather();
    },
    fetchCurrentWeather() {
      let query = document.getElementById("searchBar").value;
      if (query != "") {
        fetch(
          `https://api.openweathermap.org/data/2.5/weather?q=${query}&lang=de&units=metric&appid=${this.api_key}`
        )
          .then((res) => {
            return res.json();
          })
          .then(this.setCurrentWeatherData); // calling the function
      } else {
        this.startingLocation();
      }
    },
    fetchForcastWeather() {
      if (this.locationData != "undefined") {
        fetch(
          `https://api.openweathermap.org/data/2.5/onecall?lat=${this.locationData.lat}&lon=${this.locationData.lon}&exclude=hourly,minutely,alerts&lang=de&units=metric&appid=${this.api_key}`
        )
          .then((res) => {
            return res.json();
          })
          .then(this.setForcastWeatherData); // calling the function
      }
    },
    setCurrentWeatherData(results) {
      let searchBar = document.getElementById("searchBar");
      this.currentWeather = results;
      this.locationData = results.coord;
      searchBar.value = this.currentWeather.name ?? "";
      this.changeBGImage();
      this.fetchForcastWeather();
    },
    setForcastWeatherData(results) {
      this.forcastWeather = results;
      this.forcastWeather.daily.splice(5, 7);
      this.getDayOfTheWeek();
    },
    changeBGImage() {
      if (typeof this.currentWeather.main != "undefined") {
        let temp = this.currentWeather.main.temp;
        let element = document.getElementById("app").classList;
        if (temp < 0) {
          element.remove(...element);
          element.add("cold");
        } else if (temp > 15 && temp <= 25) {
          element.remove(...element);
          element.add("warm");
        } else if (temp > 25) {
          element.remove(...element);
          element.add("hot");
        } else {
          element.remove(...element);
        }
      }
    },
    dateBuilder() {
      let timestemp = this.currentWeather.dt;
      let d = new Date(timestemp * 1000);
      let months = [
        "Januar",
        "Februar",
        "März",
        "April",
        "Mai",
        "Juni",
        "Juli",
        "August",
        "September",
        "Oktober",
        "November",
        "Dezember",
      ];
      let days = [
        "Sonntag",
        "Montag",
        "Dienstag",
        "Mittwoch",
        "Donnerstag",
        "Freitag",
        "Samstag",
      ];

      let day = days[d.getDay()]; // getting the day with the index from the day array
      let date = d.getDate();
      let month = months[d.getMonth()]; // getting the month with the index of the months array
      let year = d.getFullYear();

      return `${day} ${date} ${month} ${year}`;
    },
    startingLocation() {
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(this.startingWeather);
      }
    },
    startingWeather(position) {
      fetch(
        `https://api.openweathermap.org/data/2.5/weather?lat=${position.coords.latitude}&lon=${position.coords.longitude}&lang=de&units=metric&appid=${this.api_key}`
      )
        .then((res) => {
          return res.json();
        })
        .then(this.setCurrentWeatherData); // calling the function
    },
    getDayOfTheWeek() {
      if (this.forcastWeather) {
        let i = 0;
        this.forcastWeather.daily.forEach((day) => {
          let timestamp = day.dt;
          let a = new Date(timestamp * 1000);
          let days = [
            "Sonntag",
            "Montag",
            "Dienstag",
            "Mittwoch",
            "Donnerstag",
            "Freitag",
            "Samstag",
          ];
          let dayOfWeek = days[a.getDay()];
          if (i == 0) {
            this.forcastWeather.daily[i].weather.push("Heute");
          } else {
            this.forcastWeather.daily[i].weather.push(dayOfWeek);
          }
          i += 1;
        });
      }
    },
  },
  created() {
    this.startingLocation();
  },
};
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: "montserrat", sans-serif;
}

#app {
  background-image: url("./assets/default-bg.jpg");
  background-size: cover;
  background-position: top center;
  background-repeat: no-repeat;
  background-attachment: fixed;
  transition: ease-in-out 1.5s;
}

#app.cold {
  background-image: url("./assets/cold-bg.jpg");
}

#app.warm {
  background-image: url("./assets/warm-bg.jpg");
}

#app.hot {
  background-image: url("./assets/hot-bg.jpg");
}

main {
  min-height: 100vh;
  height: auto;
  padding: 25px;
  background-image: linear-gradient(
    to bottom,
    rgba(0, 0, 0, 0.25),
    rgba(0, 0, 0, 0.75)
  );
}

.error-message {
  color: #fff;
  font-size: 1.5rem;
  font-weight: 700;
  font-style: italic;
  text-shadow: 3px 6px rgba(0, 0, 0, 0.25);
}

.search-box {
  width: 100%;
  margin-bottom: 1rem;
}

.search-box .search-bar {
  display: block;
  width: 100%;
  padding: 15px;
  color: #313131;
  font-size: 1rem;
  appearance: none;
  border: none;
  outline: none;
  background: none;
  box-shadow: 0px 0px 8px rgba(0, 0, 0, 0.25);
  background-color: rgba(255, 255, 255, 0.3);
  transition: 0.4s;
}

.search-box .search-bar:focus {
  box-shadow: 0px 0px 16px rgba(0, 0, 0, 0.25);
  background-color: rgba(255, 255, 255, 0.75);
  border-radius: 5px 5px 5px 5px;
}

.location-box .location {
  color: #fff;
  font-size: 1.8rem;
  font-weight: 500;
  text-align: center;
  text-shadow: 1px 3px rgba(0, 0, 0, 0.25);
}

.location-box .date {
  color: #fff;
  font-size: 1.25rem;
  font-weight: 300;
  font-style: italic;
  text-align: center;
}

.weather-box {
  text-align: center;
}

.weather-box .temp {
  display: inline-block;
  padding: 10px 25px;
  color: #fff;
  font-size: 5rem;
  font-weight: 900;
  text-shadow: 3px 6px rgba(0, 0, 0, 0.25);
  background-color: rgba(255, 255, 255, 0.25);
  border-radius: 20px;
  margin: 30px 0px;
  box-shadow: 3px 6px rgba(0, 0, 0, 0.25);
}

.weather-box .weather {
  color: #fff;
  font-size: 1.5rem;
  font-weight: 700;
  font-style: italic;
  text-shadow: 3px 6px rgba(0, 0, 0, 0.25);
}

.forcast-wrap {
  font-size: 1.2rem;
  text-shadow: 3px 6px rgba(0, 0, 0, 0.25);
  margin-top: 1rem;
  display: grid;
  color: #fff;
}

.forcast-row {
  margin: 1.2rem 10px;
}

.item-left {
  float: left;
}

.item-right {
  float: right;
}

@media screen and (min-width: 480px) {
  body {
    height: 100vh;
    background: linear-gradient(to bottom, #2c3e50, #bdc3c7);
  }

  #app {
    max-width: 600px;
    margin: auto;
    background-image: url("./assets/default-bg.jpg");
    background-size: cover;
    background-position: bottom;
    background-repeat: no-repeat;
    background-attachment: scroll;
    transition: ease-in-out 1.5s;
  }
  main {
    padding: 25px;
    background-image: linear-gradient(
      to bottom,
      rgba(0, 0, 0, 0.25),
      rgba(0, 0, 0, 0.75)
    );
  }
}
</style>
