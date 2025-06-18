<template>
  <div class="page-container" :class="weatherClass">
    <div class="app">
      <!-- Główna zawartość w kontenerze -->
      <div class="app-content">
        <div class="logo-row">
          <div class="logo-container">
            <img src="./assets/avg-weather.png" alt="AVG WEATHER" class="logo" />
          </div>
          <div class="hero-text">
            <h1>Twoja prognoza pogody z różnych źródeł </h1>
            <h2>Porównaj średnie wyniki z wielu API</h2>
          </div>
        </div>

        <!-- Komponent wyszukiwania -->
        <SearchBox
            v-model="cityName"
            :loading="loading"
            @search="getWeather"
        />

        <!-- Komponent ładowania -->
        <LoadingIndicator
            v-if="loading"
            :total-apis="totalApis"
            :completed-apis="completedApis"
            :api-status="apiStatus"
            :is-visible="loading"
        />

        <!-- Komponent błędu -->
        <div v-if="error" class="error">
          ❌ {{ error }}
        </div>

        <!-- Sekcja wyników pogody -->
        <div v-if="weatherData" class="weather-results">
          <MainWeatherCard
              :city-name="displayCityName"
              :temperature="weatherData.averageTemperature"
              :weather-description="weatherData.weatherDescription"
              :temperature-range="weatherData.temperatureRange"
              :successful-sources="weatherData.successfulSources"
              :description="weatherData.description"
          />

          <WeatherDetails
              :humidity="weatherData.humidity"
              :pressure="weatherData.pressure"
              :wind-speed="weatherData.windSpeed"
              :feels-like="weatherData.feels_like"
          />

          <ApiResults
              :api-results="weatherData.apiResults"
          />

          <Statistics
              :average-temperature="weatherData.averageTemperature"
              :min-temperature="weatherData.minTemperature"
              :max-temperature="weatherData.maxTemperature"
              :successful-sources="weatherData.successfulSources"
              :total-apis="totalApis"
          />

          <ApiList />
        </div>
      </div>

      <!-- Footer -->
      <Footer v-if="!weatherData || (weatherData && showFooter)" />
    </div>
  </div>
</template>

<script>
import { ref, computed, watch } from 'vue'

// Import wszystkich komponentów
import SearchBox from './components/SearchBox.vue'
import LoadingIndicator from './components/LoadingIndicator.vue'
import WeatherIcon from './components/WeatherIcon.vue'
import MainWeatherCard from './components/MainWeatherCard.vue'
import WeatherDetails from './components/WeatherDetails.vue'
import ApiCard from './components/ApiCard.vue'
import ApiResults from './components/ApiResults.vue'
import StatCard from './components/StatCard.vue'
import Statistics from './components/Statistics.vue'
import ApiList from './components/ApiList.vue'
import Footer from './components/Footer.vue'

export default {
  name: 'App',
  components: {
    SearchBox,
    LoadingIndicator,
    WeatherIcon,
    MainWeatherCard,
    WeatherDetails,
    ApiCard,
    ApiResults,
    StatCard,
    Statistics,
    ApiList,
    Footer
  },
  setup() {
    const cityName = ref('')
    const displayCityName = ref('')
    const weatherData = ref(null)
    const loading = ref(false)
    const error = ref('')
    const totalApis = ref(9)
    const completedApis = ref(0)
    const apiStatus = ref({})
    const showFooter = ref(false)

    const API_KEYS = {
      VISUAL_CROSSING: import.meta.env.VITE_VISUAL_CROSSING,
      TOMORROW_IO: import.meta.env.VITE_TOMORROW_IO,
      ACCUWEATHER: import.meta.env.VITE_ACCUWEATHER,
      AIRLY: import.meta.env.VITE_AIRLY
    }

    // Polskie miasta do stacji IMGW
    const polishCityToStation = {
      'warszawa': 'Warszawa',
      'krakow': 'Kraków',
      'gdansk': 'Gdańsk',
      'wroclaw': 'Wrocław',
      'poznan': 'Poznań',
      'lodz': 'Łódź',
      'katowice': 'Katowice',
      'bydgoszcz': 'Bydgoszcz',
      'lublin': 'Lublin',
      'szczecin': 'Szczecin',
      'olsztyn': 'Olsztyn',
      'rzeszow': 'Rzeszów',
      'torun': 'Toruń',
      'kielce': 'Kielce',
      'gorzow': 'Gorzów Wielkopolski',
      'opole': 'Opole',
      'zielona gora': 'Zielona Góra',
      'bialystok': 'Białystok',
      'czestochowa': 'Częstochowa',
      'radom': 'Radom',
      'sosnowiec': 'Sosnowiec',
      'tychy': 'Tychy',
      'gliwice': 'Gliwice',
      'zabrze': 'Zabrze',
      'bytom': 'Bytom',
      'ruda slaska': 'Ruda Śląska',
      'rybnik': 'Rybnik',
      'dabrowa gornicza': 'Dąbrowa Górnicza'
    }

    const weatherClass = computed(() => {
      if (!weatherData.value || !weatherData.value.weatherDescription) {
        return 'default'
      }

      const description = weatherData.value.weatherDescription.toLowerCase()

      // Więcej słów kluczowych dla lepszego dopasowania
      if (description.includes('bezchmurnie') ||
          description.includes('słonecznie') ||
          description.includes('clear') ||
          description.includes('sunny')) {
        return 'sunny'
      }

      if (description.includes('deszcz') ||
          description.includes('rain') ||
          description.includes('mżawka') ||
          description.includes('drizzle')) {
        return 'rainy'
      }

      if (description.includes('śnieg') ||
          description.includes('snow')) {
        return 'snowy'
      }

      if (description.includes('burza') ||
          description.includes('storm') ||
          description.includes('thunder')) {
        return 'stormy'
      }

      if (description.includes('mgła') ||
          description.includes('fog') ||
          description.includes('mist')) {
        return 'foggy'
      }

      if (description.includes('zachmurzenie') ||
          description.includes('pochmurno') ||
          description.includes('cloud') ||
          description.includes('overcast')) {
        return 'cloudy'
      }

      return 'default'
    })

    // Watch dla kontroli Footer
    watch(weatherData, (newData) => {
      if (newData) {
        // Pokaż footer po 600ms
        setTimeout(() => {
          showFooter.value = true
        }, 600)
      } else {
        showFooter.value = false
      }
    })

    const getCoordinates = async (city) => {
      try {
        const response = await fetch(`https://nominatim.openstreetmap.org/search?format=json&q=${encodeURIComponent(city)}&countrycodes=pl&limit=1`)
        const data = await response.json()
        if (data.length > 0) {
          return {
            lat: parseFloat(data[0].lat),
            lon: parseFloat(data[0].lon),
            displayName: data[0].display_name.split(',')[0]
          }
        }
        return null
      } catch (error) {
        console.error('Błąd pobierania współrzędnych:', error)
        return null
      }
    }

    const fetchIMGWData = async (city) => {
      try {
        apiStatus.value['IMGW-PIB'] = 'Pobieranie danych...'

        const stationName = polishCityToStation[city.toLowerCase()]
        if (!stationName) {
          throw new Error('Miasto nie jest obsługiwane przez IMGW')
        }

        const response = await fetch('https://danepubliczne.imgw.pl/api/data/synop')
        const data = await response.json()

        const stationData = data.find(station =>
            station.stacja && station.stacja.toLowerCase().includes(stationName.toLowerCase())
        )

        if (!stationData) {
          throw new Error('Nie znaleziono stacji dla tego miasta')
        }

        apiStatus.value['IMGW-PIB'] = 'Gotowe ✅'
        return {
          source: 'IMGW-PIB',
          success: true,
          temperature: parseFloat(stationData.temperatura) || null,
          humidity: parseFloat(stationData.wilgotnosc_wzgledna) || null,
          pressure: parseFloat(stationData.cisnienie) || null,
          windSpeed: parseFloat(stationData.predkosc_wiatru) || null,
          description: `Stacja: ${stationData.stacja}`
        }
      } catch (error) {
        apiStatus.value['IMGW-PIB'] = `Błąd: ${error.message} ❌`
        return {
          source: 'IMGW-PIB',
          success: false,
          error: error.message
        }
      }
    }

    const fetchOpenMeteoData = async (lat, lon) => {
      try {
        apiStatus.value['Open-Meteo'] = 'Pobieranie danych...'

        const response = await fetch(`https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lon}&current_weather=true&hourly=temperature_2m,relativehumidity_2m,surface_pressure,windspeed_10m&timezone=Europe/Warsaw`)
        const data = await response.json()

        if (!data.current_weather) {
          throw new Error('Brak danych pogodowych')
        }

        const weatherCodeMap = {
          0: 'Bezchmurnie',
          1: 'Głównie bezchmurnie',
          2: 'Częściowe zachmurzenie',
          3: 'Zachmurzenie',
          45: 'Mgła',
          48: 'Mgła z szronem',
          51: 'Lekka mżawka',
          53: 'Umiarkowana mżawka',
          55: 'Intensywna mżawka',
          61: 'Lekki deszcz',
          63: 'Umiarkowany deszcz',
          65: 'Intensywny deszcz',
          71: 'Lekki śnieg',
          73: 'Umiarkowany śnieg',
          75: 'Intensywny śnieg',
          80: 'Lekkie opady deszczu',
          81: 'Umiarkowane opady deszczu',
          82: 'Intensywne opady deszczu',
          95: 'Burza',
          96: 'Burza z gradem',
          99: 'Silna burza z gradem'
        }

        const weatherDescription = weatherCodeMap[data.current_weather.weathercode] || `Kod pogody: ${data.current_weather.weathercode}`

        apiStatus.value['Open-Meteo'] = 'Gotowe ✅'
        return {
          source: 'Open-Meteo',
          success: true,
          temperature: data.current_weather.temperature,
          humidity: data.hourly.relativehumidity_2m[0],
          pressure: data.hourly.surface_pressure[0],
          windSpeed: data.current_weather.windspeed,
          description: weatherDescription
        }
      } catch (error) {
        apiStatus.value['Open-Meteo'] = `Błąd: ${error.message} ❌`
        return {
          source: 'Open-Meteo',
          success: false,
          error: error.message
        }
      }
    }

    const fetchYrNoData = async (lat, lon) => {
      try {
        apiStatus.value['Yr.no'] = 'Pobieranie danych...'

        const response = await fetch(`https://api.met.no/weatherapi/locationforecast/2.0/compact?lat=${lat}&lon=${lon}`, {
          headers: {
            'User-Agent': 'AVG-Weather-App/1.0'
          }
        })
        const data = await response.json()

        if (!data.properties || !data.properties.timeseries || data.properties.timeseries.length === 0) {
          throw new Error('Brak danych pogodowych')
        }

        const current = data.properties.timeseries[0].data.instant.details

        apiStatus.value['Yr.no'] = 'Gotowe ✅'
        return {
          source: 'Yr.no (MET Norway)',
          success: true,
          temperature: current.air_temperature,
          humidity: current.relative_humidity,
          pressure: current.air_pressure_at_sea_level,
          windSpeed: current.wind_speed,
          description: 'Norweski Instytut Meteorologiczny'
        }
      } catch (error) {
        apiStatus.value['Yr.no'] = `Błąd: ${error.message} ❌`
        return {
          source: 'Yr.no (MET Norway)',
          success: false,
          error: error.message
        }
      }
    }

    const fetchVisualCrossingData = async (city) => {
      try {
        apiStatus.value['Visual Crossing'] = 'Pobieranie danych...'

        if (!API_KEYS.VISUAL_CROSSING) {
          throw new Error('Brak klucza API - zarejestruj się na visualcrossing.com')
        }

        const response = await fetch(
            `https://weather.visualcrossing.com/VisualCrossingWebServices/rest/services/timeline/${encodeURIComponent(city)}?key=${API_KEYS.VISUAL_CROSSING}&include=current&unitGroup=metric`
        )

        if (!response.ok) {
          throw new Error(`HTTP ${response.status}`)
        }

        const data = await response.json()

        if (!data.currentConditions) {
          throw new Error('Brak aktualnych danych pogodowych')
        }

        const current = data.currentConditions

        apiStatus.value['Visual Crossing'] = 'Gotowe ✅'
        return {
          source: 'Visual Crossing',
          success: true,
          temperature: current.temp,
          humidity: current.humidity,
          pressure: current.pressure,
          windSpeed: current.windspeed / 3.6,
          description: current.conditions
        }
      } catch (error) {
        apiStatus.value['Visual Crossing'] = `Błąd: ${error.message} ❌`
        return {
          source: 'Visual Crossing',
          success: false,
          error: error.message
        }
      }
    }

    const fetchTomorrowIOData = async (lat, lon) => {
      try {
        apiStatus.value['Tomorrow.io'] = 'Pobieranie danych...'

        if (!API_KEYS.TOMORROW_IO) {
          throw new Error('Brak klucza API - zarejestruj się na tomorrow.io')
        }

        const response = await fetch(
            `https://api.tomorrow.io/v4/weather/realtime?location=${lat},${lon}&apikey=${API_KEYS.TOMORROW_IO}`
        )

        if (!response.ok) {
          throw new Error(`HTTP ${response.status}`)
        }

        const data = await response.json()

        if (!data.data || !data.data.values) {
          throw new Error('Brak danych pogodowych')
        }

        const values = data.data.values

        const weatherCodeMap = {
          0: 'Nieznana pogoda',
          1000: 'Bezchmurnie',
          1001: 'Zachmurzenie',
          1100: 'Głównie bezchmurnie',
          1101: 'Częściowe zachmurzenie',
          1102: 'Głównie zachmurzenie',
          2000: 'Mgła',
          2100: 'Lekka mgła',
          4000: 'Mżawka',
          4001: 'Deszcz',
          4200: 'Lekki deszcz',
          4201: 'Umiarkowany deszcz',
          5000: 'Śnieg',
          5001: 'Płatki śniegu',
          5100: 'Lekki śnieg',
          5101: 'Umiarkowany śnieg',
          6000: 'Deszcz ze śniegiem',
          6001: 'Deszcz ze śniegiem',
          6200: 'Lekki deszcz ze śniegiem',
          6201: 'Umiarkowany deszcz ze śniegiem',
          7000: 'Grad',
          7101: 'Umiarkowany grad',
          7102: 'Intensywny grad',
          8000: 'Burza'
        }

        const weatherDescription = weatherCodeMap[values.weatherCode] || `Kod pogody: ${values.weatherCode}`

        apiStatus.value['Tomorrow.io'] = 'Gotowe ✅'
        return {
          source: 'Tomorrow.io',
          success: true,
          temperature: values.temperature,
          humidity: values.humidity,
          pressure: values.pressureSeaLevel,
          windSpeed: values.windSpeed,
          description: weatherDescription
        }
      } catch (error) {
        apiStatus.value['Tomorrow.io'] = `Błąd: ${error.message} ❌`
        return {
          source: 'Tomorrow.io',
          success: false,
          error: error.message
        }
      }
    }

    const fetchAccuWeatherData = async (city) => {
      try {
        apiStatus.value['AccuWeather'] = 'Pobieranie danych...'

        if (!API_KEYS.ACCUWEATHER) {
          throw new Error('Brak klucza API - zarejestruj się na developer.accuweather.com')
        }

        const locationResponse = await fetch(
            `https://dataservice.accuweather.com/locations/v1/cities/search?apikey=${API_KEYS.ACCUWEATHER}&q=${encodeURIComponent(city)}`
        )

        if (!locationResponse.ok) {
          throw new Error(`HTTP ${locationResponse.status}`)
        }

        const locations = await locationResponse.json()

        if (!locations || locations.length === 0) {
          throw new Error('Nie znaleziono lokalizacji')
        }

        const locationKey = locations[0].Key

        const weatherResponse = await fetch(
            `https://dataservice.accuweather.com/currentconditions/v1/${locationKey}?apikey=${API_KEYS.ACCUWEATHER}&details=true`
        )

        if (!weatherResponse.ok) {
          throw new Error(`HTTP ${weatherResponse.status}`)
        }

        const weatherData = await weatherResponse.json()

        if (!weatherData || weatherData.length === 0) {
          throw new Error('Brak danych pogodowych')
        }

        const current = weatherData[0]

        apiStatus.value['AccuWeather'] = 'Gotowe ✅'
        return {
          source: 'AccuWeather',
          success: true,
          temperature: current.Temperature.Metric.Value,
          humidity: current.RelativeHumidity,
          pressure: current.Pressure?.Metric?.Value || null,
          windSpeed: current.Wind?.Speed?.Metric?.Value ? current.Wind.Speed.Metric.Value / 3.6 : null,
          description: current.WeatherText
        }
      } catch (error) {
        apiStatus.value['AccuWeather'] = `Błąd: ${error.message} ❌`
        return {
          source: 'AccuWeather',
          success: false,
          error: error.message
        }
      }
    }

    const fetchAirlyData = async (lat, lon) => {
      try {
        apiStatus.value['Airly'] = 'Pobieranie danych...'

        if (!API_KEYS.AIRLY) {
          throw new Error('Brak klucza API - zarejestruj się na developer.airly.eu')
        }

        const installationsResponse = await fetch(
            `https://airapi.airly.eu/v2/installations/nearest?lat=${lat}&lng=${lon}&maxDistanceKM=50&maxResults=1`,
            {
              headers: {
                'Accept': 'application/json',
                'apikey': API_KEYS.AIRLY
              }
            }
        )

        if (!installationsResponse.ok) {
          throw new Error(`HTTP ${installationsResponse.status}`)
        }

        const installations = await installationsResponse.json()

        if (!installations || installations.length === 0) {
          throw new Error('Brak instalacji Airly w okolicy')
        }

        const installationId = installations[0].id

        const measurementsResponse = await fetch(
            `https://airapi.airly.eu/v2/measurements/installation?installationId=${installationId}`,
            {
              headers: {
                'Accept': 'application/json',
                'apikey': API_KEYS.AIRLY
              }
            }
        )

        if (!measurementsResponse.ok) {
          throw new Error(`HTTP ${measurementsResponse.status}`)
        }

        const data = await measurementsResponse.json()

        if (!data.current || !data.current.values) {
          throw new Error('Brak aktualnych danych')
        }

        const tempValue = data.current.values.find(v => v.name === 'TEMPERATURE')
        const humidityValue = data.current.values.find(v => v.name === 'HUMIDITY')
        const pressureValue = data.current.values.find(v => v.name === 'PRESSURE')

        apiStatus.value['Airly'] = 'Gotowe ✅'
        return {
          source: 'Airly',
          success: true,
          temperature: tempValue ? tempValue.value : null,
          humidity: humidityValue ? humidityValue.value : null,
          pressure: pressureValue ? pressureValue.value : null,
          description: `Instalacja ${installationId} - ${installations[0].address?.city || 'Nieznana lokalizacja'}`
        }

      } catch (error) {
        apiStatus.value['Airly'] = `Błąd: ${error.message} ❌`
        return {
          source: 'Airly',
          success: false,
          error: error.message
        }
      }
    }

    const fetch7TimerData = async (lat, lon) => {
      try {
        apiStatus.value['7Timer!'] = 'Pobieranie danych...'

        const response = await fetch(`https://www.7timer.info/bin/api.pl?lon=${lon}&lat=${lat}&product=astro&output=json`)

        if (!response.ok) {
          throw new Error(`HTTP ${response.status}`)
        }

        const text = await response.text()

        if (!text || text.trim() === '') {
          throw new Error('Pusta odpowiedź z serwera')
        }

        let data
        try {
          data = JSON.parse(text)
        } catch (jsonError) {
          throw new Error('Nieprawidłowy format JSON')
        }

        if (!data.dataseries || data.dataseries.length === 0) {
          throw new Error('Brak danych pogodowych')
        }

        const current = data.dataseries[0]

        if (typeof current.temp2m !== 'number') {
          throw new Error('Brak danych temperatury')
        }

        const tempC = current.temp2m

        apiStatus.value['7Timer!'] = 'Gotowe ✅'
        return {
          source: '7Timer!',
          success: true,
          temperature: Math.round(tempC * 10) / 10,
          description: 'Prognoza astronomiczna'
        }
      } catch (error) {
        apiStatus.value['7Timer!'] = `Błąd: ${error.message} ❌`
        return {
          source: '7Timer!',
          success: false,
          error: error.message
        }
      }
    }

    const fetchWeatherAPIData = async (city) => {
      try {
        apiStatus.value['WeatherAPI'] = 'Pobieranie danych...'

        const response = await fetch(`https://wttr.in/${encodeURIComponent(city)}?format=j1`)
        const data = await response.json()

        if (!data.current_condition || data.current_condition.length === 0) {
          throw new Error('Brak danych pogodowych')
        }

        const current = data.current_condition[0]

        apiStatus.value['WeatherAPI'] = 'Gotowe ✅'
        return {
          source: 'WeatherAPI (wttr.in)',
          success: true,
          temperature: parseFloat(current.temp_C),
          humidity: parseFloat(current.humidity),
          pressure: parseFloat(current.pressure),
          windSpeed: parseFloat(current.windspeedKmph) / 3.6,
          description: current.weatherDesc[0].value
        }
      } catch (error) {
        apiStatus.value['WeatherAPI'] = `Błąd: ${error.message} ❌`
        return {
          source: 'WeatherAPI (wttr.in)',
          success: false,
          error: error.message
        }
      }
    }

    const getWeather = async (searchCity) => {
      if (!searchCity || !searchCity.trim()) {
        error.value = 'Proszę wpisać nazwę miasta'
        return
      }

      loading.value = true
      error.value = ''
      weatherData.value = null
      showFooter.value = false
      completedApis.value = 0
      apiStatus.value = {}

      try {
        displayCityName.value = searchCity

        const coords = await getCoordinates(searchCity)
        if (!coords) {
          throw new Error('Nie znaleziono miasta. Sprawdź pisownię.')
        }

        const apiPromises = [
          fetchIMGWData(searchCity),
          fetchOpenMeteoData(coords.lat, coords.lon),
          fetchYrNoData(coords.lat, coords.lon),
          fetchVisualCrossingData(searchCity),
          fetchTomorrowIOData(coords.lat, coords.lon),
          fetchAccuWeatherData(searchCity),
          fetchAirlyData(coords.lat, coords.lon),
          fetch7TimerData(coords.lat, coords.lon),
          fetchWeatherAPIData(searchCity)
        ]

        const results = await Promise.all(apiPromises.map(promise =>
            promise.then(result => {
              completedApis.value++
              return result
            }).catch(error => {
              completedApis.value++
              return {
                success: false,
                error: error.message
              }
            })
        ))

        const successfulResults = results.filter(result => result.success && result.temperature !== null)

        if (successfulResults.length === 0) {
          throw new Error('Nie udało się pobrać danych z żadnego źródła')
        }

        const temperatures = successfulResults.map(result => result.temperature)
        const avgTemp = temperatures.reduce((sum, temp) => sum + temp, 0) / temperatures.length
        const minTemp = Math.min(...temperatures)
        const maxTemp = Math.max(...temperatures)

        const humidities = successfulResults.filter(r => r.humidity).map(r => r.humidity)
        const pressures = successfulResults.filter(r => r.pressure).map(r => r.pressure)
        const windSpeeds = successfulResults.filter(r => r.windSpeed).map(r => r.windSpeed)

        const avgHumidity = humidities.length > 0 ? Math.round(humidities.reduce((sum, h) => sum + h, 0) / humidities.length) : 'Brak danych'
        const avgPressure = pressures.length > 0 ? Math.round(pressures.reduce((sum, p) => sum + p, 0) / pressures.length) : 'Brak danych'
        const avgWindSpeed = windSpeeds.length > 0 ? Math.round(windSpeeds.reduce((sum, w) => sum + w, 0) / windSpeeds.length * 10) / 10 : 'Brak danych'

        const descriptions = successfulResults.filter(r => r.description && !r.description.includes('Kod pogody') && !r.description.includes('Stacja') && !r.description.includes('Instalacja')).map(r => r.description)
        const mostCommonDescription = descriptions.length > 0 ? descriptions[0] : 'Brak opisu pogody'

        weatherData.value = {
          averageTemperature: Math.round(avgTemp * 10) / 10,
          minTemperature: Math.round(minTemp * 10) / 10,
          maxTemperature: Math.round(maxTemp * 10) / 10,
          temperatureRange: `${Math.round(minTemp * 10) / 10}°C - ${Math.round(maxTemp * 10) / 10}°C`,
          humidity: avgHumidity,
          pressure: avgPressure,
          windSpeed: avgWindSpeed,
          feels_like: Math.round(avgTemp * 10) / 10,
          weatherDescription: mostCommonDescription,
          successfulSources: successfulResults.length,
          description: `Dane z ${successfulResults.length} źródeł pogodowych`,
          apiResults: results
        }

      } catch (err) {
        error.value = err.message
        showFooter.value = true
      } finally {
        loading.value = false
      }
    }

    return {
      cityName,
      displayCityName,
      weatherData,
      loading,
      error,
      totalApis,
      completedApis,
      apiStatus,
      weatherClass,
      showFooter,
      getWeather
    }
  }
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;500;600;700;800&display=swap');

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

/* Tło całej strony z dynamicznymi kolorami */
.page-container {
  font-family: 'Poppins', sans-serif;
  min-height: 100vh;
  padding: 20px;
  transition: background 0.3s ease;
  display: flex;
  justify-content: center;
  align-items: flex-start;
}

.page-container.default {
  background: linear-gradient(135deg, #74b9ff 0%, #0984e3 100%);
}

.page-container.sunny {
  background: linear-gradient(135deg, #FFE066 0%, #FF9500 50%, #FF6B35 100%);
}

.page-container.rainy {
  background: linear-gradient(135deg, #5DADE2 0%, #3498DB 50%, #2980B9 100%);
}

.page-container.cloudy {
  background: linear-gradient(135deg, #BDC3C7 0%, #95A5A6 50%, #7F8C8D 100%);
}

.page-container.snowy {
  background: linear-gradient(135deg, #EBF4F6 0%, #D5DBDB 50%, #AEB6BF 100%);
}

.page-container.stormy {
  background: linear-gradient(135deg, #34495E 0%, #2C3E50 50%, #1B2631 100%);
}

.page-container.foggy {
  background: linear-gradient(135deg, #F8F9FA 0%, #E9ECEF 50%, #DEE2E6 100%);
}

/* Główny kontener aplikacji */
.app {
  width: 100%;
  max-width: 1400px;
  min-height: calc(100vh - 40px);
  display: flex;
  flex-direction: column;
  background: rgba(248, 249, 250, 0.98);
  border-radius: 20px;
  padding: 30px;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.15);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.2);
  animation: app-entrance 0.6s ease-out;
}

/* Kontener dla głównej zawartości */
.app-content {
  flex: 1;
  display: flex;
  flex-direction: column;
}

/* Różne odcienie dla różnych stanów pogody */
.page-container.sunny .app {
  background: rgba(255, 248, 225, 0.98);
  border: 1px solid rgba(255, 193, 7, 0.2);
}

.page-container.rainy .app {
  background: rgba(240, 248, 255, 0.98);
  border: 1px solid rgba(52, 152, 219, 0.2);
}

.page-container.cloudy .app {
  background: rgba(245, 245, 245, 0.98);
  border: 1px solid rgba(149, 165, 166, 0.2);
}

.page-container.snowy .app {
  background: rgba(250, 250, 255, 0.98);
  border: 1px solid rgba(174, 182, 191, 0.2);
}

.page-container.stormy .app {
  background: rgba(240, 240, 245, 0.98);
  border: 1px solid rgba(52, 73, 94, 0.2);
}

.page-container.foggy .app {
  background: rgba(249, 249, 249, 0.98);
  border: 1px solid rgba(222, 226, 230, 0.2);
}

h1 {
  text-align: center;
  color: #2d3436;
  margin-bottom: 30px;
  font-size: 2.5em;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.1);
  animation: title-glow 2s ease-in-out infinite;
}

.weather-results {
  display: grid;
  gap: 25px;
  animation: results-appear 0.6s ease-out;
  margin-bottom: 30px;
}

.logo-row {
  display: flex;
  align-items: center;
  gap: 40px;
  margin: 30px 0;
}
.logo-container {
  display: flex;
  align-items: center;
  justify-content: center;
}
.logo {
  max-width: 320px;
  max-height: 320px;
  display: block;
  margin: 10px 10px 10px 5px;
}

.hero-text h1 {
  font-size: 2.5em;
  font-weight: 700;
  margin: 0;
  color: #222;
  letter-spacing: 1px;
}
.error {
  background: linear-gradient(135deg, #e74c3c, #c0392b);
  color: white;
  padding: 20px;
  border-radius: 15px;
  margin-bottom: 20px;
  text-align: center;
  font-size: 1.1em;
  box-shadow: 0 8px 16px rgba(231, 76, 60, 0.3);
  animation: error-entrance 0.4s ease-out;
}

/* Animacje */
@keyframes app-entrance {
  from { transform: translateY(20px); opacity: 0; }
  to { transform: translateY(0); opacity: 1; }
}

@keyframes title-glow {
  0%, 100% { text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.1); }
  50% { text-shadow: 2px 2px 8px rgba(116, 185, 255, 0.3); }
}

@keyframes slide-in-left {
  from { transform: translateX(-20px); opacity: 0; }
  to { transform: translateX(0); opacity: 1; }
}

@keyframes results-appear {
  from { transform: translateY(30px); opacity: 0; }
  to { transform: translateY(0); opacity: 1; }
}

@keyframes error-entrance {
  from { transform: scale(0.95); opacity: 0; }
  to { transform: scale(1); opacity: 1; }
}

/* Responsive design */
@media (max-width: 768px) {
  .page-container {
    padding: 15px;
  }

  .app {
    padding: 20px;
    min-height: calc(100vh - 30px);
  }

  h1 {
    font-size: 2em;
  }

  .weather-results {
    margin-bottom: 20px;
  }
}

@media (max-width: 480px) {
  .page-container {
    padding: 10px;
  }

  .app {
    padding: 15px;
    min-height: calc(100vh - 20px);
  }

  h1 {
    font-size: 1.8em;
    margin-bottom: 20px;
  }


  .weather-results {
    gap: 20px;
    margin-bottom: 15px;
  }
}

/* Accessibility */
@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
</style>