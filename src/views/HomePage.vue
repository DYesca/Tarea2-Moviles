<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Clima en tu Ubicación</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content class="ion-padding">
      <div id="container">
        <!-- Campo para ingresar la ubicación manualmente -->
        <ion-item>
          <ion-input v-model="location" label="Toca aquí" label-placement="floating"
            placeholder="Escriba una Ciudad"></ion-input>
        </ion-item>

        <!-- Botones -->
        <ion-button shape="round" @click="fetchWeatherByLocation">Consultar Clima</ion-button>
        <ion-button shape="round" color="secondary" @click="fetchWeatherByGeolocation">Usar mi Ubicación</ion-button>

        <!-- Mensaje de error -->
        <p v-if="errorMessage" style="color: red;">{{ errorMessage }}</p>

        <!-- Mostrar información del clima si está disponible -->
        <ion-card v-if="weatherData">
          <ion-card-header>
            <!-- Con este h2 lo que hice es tratar de que las cordenadas del lugar no sean tan grandes -->
            <h2 v-if="weatherData.latitude && weatherData.longitude">
              <strong>Coordenadas:</strong> {{ formatCoordinates(weatherData.latitude, weatherData.longitude) }}
            </h2>
            <!-- Este h4 lo agrego mayormente para que cuando se busque por nombre salga el nombre del pais o lugar donde se busque-->
            <h4>{{ weatherData.resolvedAddress }}</h4>
            <p>{{ weatherData.description }}</p>
          </ion-card-header>
          <ion-card-content>
            <p><strong>Temperatura:</strong> {{ weatherData.currentConditions.temp }}°C</p>
            <p><strong>Viento:</strong> {{ weatherData.currentConditions.windspeed }} km/h</p>
            <p><strong>Lluvia:</strong> {{ weatherData.currentConditions.precipprob }}%</p>
            <p><strong>Clima:</strong> {{ weatherData.currentConditions.conditions }}</p>
          </ion-card-content>
        </ion-card>
        <div id="organizar">
          <!-- Pronóstico de horas anteriores -->
          <ion-card v-if="pastHourlyForecast.length">
            <ion-card-header>
              <h2>Clima en las últimas 24 horas</h2>
            </ion-card-header>
            <ion-card-content>
              <div v-for="hour in pastHourlyForecast" :key="hour.datetime">
                <p><strong>{{ hour.datetime }}</strong>: {{ hour.temp }}°C, {{ hour.conditions }}</p>
              </div>
            </ion-card-content>
          </ion-card>

          <!-- Pronóstico de horas futuras -->
          <ion-card v-if="futureHourlyForecast.length">
            <ion-card-header>
              <h2>Pronóstico para las próximas 24 horas</h2>
            </ion-card-header>
            <ion-card-content>
              <div v-for="hour in futureHourlyForecast" :key="hour.datetime">
                <p><strong>{{ hour.datetime }}</strong>: {{ hour.temp }}°C, {{ hour.conditions }}</p>
              </div>
            </ion-card-content>
          </ion-card>
        </div>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import { Geolocation } from '@capacitor/geolocation';
import axios from 'axios';
import { IonButton, IonInput, IonCard, IonCardHeader, IonCardContent, IonItem, IonContent, IonHeader, IonToolbar, IonTitle, IonPage } from '@ionic/vue';

// API Key de Visual Crossing Weather
const API_KEY = 'Q6HWA3QWGNSEVA4KTF6GYPJ62';

// Variables reactivas para la interfaz
const location = ref('');
const weatherData = ref<any>(null);
const pastHourlyForecast = ref<any[]>([]);
const futureHourlyForecast = ref<any[]>([]);
const errorMessage = ref('');

// Función para obtener datos climáticos
const fetchWeather = async (query: string) => {
  try {
    const response = await axios.get(`https://weather.visualcrossing.com/VisualCrossingWebServices/rest/services/timeline/${query}?unitGroup=metric&key=${API_KEY}`);
    weatherData.value = response.data;
   

    // Separar datos de las últimas 24 horas y las futuras
    const now = new Date().getHours();
    pastHourlyForecast.value = response.data.days[0].hours.filter((h: any) => parseInt(h.datetime.split(':')[0]) < now);
    futureHourlyForecast.value = response.data.days[0].hours.filter((h: any) => parseInt(h.datetime.split(':')[0]) >= now);
  } catch (error) {
    errorMessage.value = 'No se pudo obtener el clima. Revisa la ubicación ingresada.';
  }
};

// Función para obtener clima por ciudad ingresada
const fetchWeatherByLocation = async () => {
  if (!location.value) {
    errorMessage.value = 'Por favor, ingresa una ubicación.';
    return;
  }
  errorMessage.value = '';
  await fetchWeather(location.value);
};

// Función para obtener clima con la ubicación actual
const fetchWeatherByGeolocation = async () => {
  try {
    const position = await Geolocation.getCurrentPosition();
    const { latitude, longitude } = position.coords;
    await fetchWeather(`${latitude},${longitude}`);
  } catch (error) {
    errorMessage.value = 'No se pudo obtener la ubicación.';
  }
};

// Función para reducir la cantidad de decimales
const formatCoordinates = (lat: number, lon: number) => {
  return `${lat.toFixed(4)}, ${lon.toFixed(4)}`;
};

</script>

<style scoped>
#container {
  display: flex;
  justify-content: center;
  align-items: center;
  height: auto;
  width: 80%;
  flex-direction: column;
  border-radius: 10px;
  padding: 20px;
  margin: auto;
}

ion-content {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100%;
}

ion-input {
  margin-bottom: 10px;
  border-radius: 10px;
  --border-radius: 10px; /* Intenté ponerle borde redondeado a el input pero no pude */
}
</style>
