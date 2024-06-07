<script>
  import { getContext } from "svelte";
  import { onMount } from "svelte";

  export let APIkey;
  export let conditions;
  export let humidity;
  export let windSpeed;
  export let units;
  export let icon
  let symbol = units === "metric" ? "C" : "F";

  const { styleable } = getContext("sdk");
  const component = getContext("component");

  $: dynamicUnits = units;

  let weatherData = null;
  let loading = true;
  let error = null;
  let lat = 0;
  let lon = 0;
  let currentUnits = "";

  async function getWeatherData(lat, lon, units) {
    const URL = `https://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&appid=${APIkey}&units=${units}`;
    try {
      const response = await fetch(URL);
      if (!response.ok) {
        throw new Error("Failed to fetch weather data");
      }
      return response.json();
    } catch (err) {
      throw new Error(err.message);
    }
  }

  function getLocation() {
    return new Promise((resolve, reject) => {
      navigator.geolocation.getCurrentPosition(resolve, reject);
    });
  }
  
  async function fetchWeather() {
    loading = true;
    error = null;
    try {
      const position = await getLocation();
      const { latitude, longitude } = position.coords;
      weatherData = await getWeatherData(latitude, longitude, units);
    } catch (err) {
      error = err.message;
    } finally {
      loading = false;
    }
  }

  onMount(fetchWeather);

  $: if (units) {
    symbol = units === "metric" ? "C" : "F";
    fetchWeather();
  }
</script>


{#if loading}
  <div class="loading">Loading...</div>
{:else if error}
  <div class="error">{error}</div>
{:else}
  <div class="weather" >
    <p>{(weatherData.main.temp).toFixed(1)}°{symbol}</p>
    {#if conditions}
    <p>{weatherData.weather[0].description}</p>
    {/if}
    {#if humidity}
    <p>{weatherData.main.humidity}%</p>
    {/if}
    {#if windSpeed}
    <p>{weatherData.wind.speed} m/s</p>
    {/if}
    {#if icon}
      <img class="weather-icon" src="https://openweathermap.org/img/wn/{weatherData.weather[0].icon}.png" alt="{weatherData.weather[0].description}">
    {/if}
  </div>
{/if}

<style>
.weather {
  display: flex;
  align-items: center;
}

  .weather p {
    display: inline-block;
    margin-right: 10px;
}

.weather-icon {
  filter: drop-shadow(0px 0px 10px #7e7e7e);
}
</style>