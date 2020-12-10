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
      <div class="weather-wrap" v-if="typeof weather.main != 'undefined'">
        <div class="location-box">
          <div class="location">
            {{ weather.name }}, {{ weather.sys.country }}
          </div>
          <div class="date">{{ dateBuilder() }}</div>
        </div>
        <div class="weather-box">
          <div class="temp">{{ Math.round(weather.main.temp) }}°c</div>
          <div class="weather">{{ weather.weather[0].description }}</div>
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
    weather: {},
    // coords = {},
  }),
  methods: {
    // fetching the weather data for the location the user put in the search bar
    fetchWeatherData() {
      let query = document.getElementById("searchBar").value;
      if(query != '') {
              fetch(
        `${this.api_base}${query}&units=metric&lang=de&APPID=${this.api_key}`
      )
        .then((res) => {
          return res.json();
        })
        .then(this.setResults); // calling the function
      } else {
        this.startingLocation();
      }
    },
    setResults(results) {
      let searchBar = document.getElementById("searchBar");
      this.weather = results; // setting the value of the weather object to the date we get from the api
      searchBar.value = this.weather.name ?? "";
      this.changeBGImage();
    },
    changeBGImage() {
      if(typeof(this.weather.main) != 'undefined' ){      
      let temp = this.weather.main.temp;
      let element = document.getElementById("app").classList;
        if (temp < 0) {
          element.remove(...element);
          element.add("cold");
        } else if (temp > 15 && temp <= 25) {
          element.remove(...element);
          element.add("warm");
        } else if(temp > 25){
          element.remove(...element);
          element.add('hot')
        } else {
          element.remove(...element);
        }
        }
    },
    dateBuilder() {
      let d = new Date();
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
      if (window.navigator.geolocation) {
        window.navigator.geolocation.getCurrentPosition(this.showPosition);
      }
    },
    showPosition(position) {
      fetch(
        `https://api.openweathermap.org/data/2.5/weather?lat=${position.coords.latitude}&lon=${position.coords.longitude}&units=metric&lang=de&APPID=${this.api_key}`
      )
        .then((res) => {
          return res.json();
        })
        .then(this.setResults); // calling the function
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
  background-position: bottom;
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
  height: 100vh;
  padding: 25px;
  background-image: linear-gradient(
    to bottom,
    rgba(0, 0, 0, 0.25),
    rgba(0, 0, 0, 0.75)
  );
}

@media screen and (min-width: 480px) {
  body {
    height: 100vh;
    background: linear-gradient(to bottom, #2c3e50, #bdc3c7);
  }

  #app {
    max-width: 600px;
    margin: auto;
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

.error-message {
  color: #fff;
  font-size: 1.5rem;
  font-weight: 700;
  font-style: italic;
  text-shadow: 3px 6px rgba(0, 0, 0, 0.25);
}

.search-box {
  width: 100%;
  margin-bottom: 30px;
}

.search-box .search-bar {
  display: block;
  width: 100%;
  padding: 15px;
  color: #313131;
  font-size: 20px;
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
  font-size: 2rem;
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
  font-size: 6rem;
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
</style>
