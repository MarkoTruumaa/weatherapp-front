<template>
  <div class="container">
    <div class="row justify-content-center">
      <div class="col col-3">
        <AlertDanger :alert-message="errorMessage" />
        <AlertSuccess :alert-message="successMessage" />
      </div>
    </div>
    <div class="row justify-content-center">
      <div class="col col-4">
        <label for="">Sisesta siia linna nimi, mille kohta soovid andmeid koguma hakata</label>
        <input v-model="cityName" type="text">
        <button @click="addCity">Sisesta</button>
      </div>
      <div class="col col-4">
        <label for="">Sisesta linn mida soovis kustutada</label>
        <CitiesDropdown @event-update-selected-city-id="setSelectedCityId" :cities="cities"/>
        <button @click="deleteCity">Kustuta</button>
      </div>
    </div>
    <div class="row justify-content-center">
      <div class="col col-4">
        <label for="">Sisesta linn mille kohta soovid andmeid</label>
        <CitiesDropdown @event-update-selected-city-id="setSelectedCityId" :cities="cities"/>
        <button @click="getCityMeasurementData">Saada info</button>
      </div>
    </div>
    <div class="row justify-content-center">
      <div class="col col-6">
        <h2>Linna nimi</h2>
        <table class="table">
          <thead>
          <tr>
            <th scope="col"></th>
            <th scope="col">Kuupäev</th>
            <th scope="col">Tempertatuur </th>
            <th scope="col">Tuulekiirus m/s</th>
            <th scope="col">Õhuniiskus %</th>
          </tr>
          </thead>
          <tbody>
          <tr v-for="(cityData, index) in citiesMeasurementData" :key="index">
            <th scope="row">{{ index + 1 }}</th>
            <td>{{ cityData.dateTime }}</td>
            <td>{{ cityData.temperature }}</td>
            <td>{{ cityData.windSpeed}}</td>
            <td>{{ cityData.humidity}}</td>
          </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script>

import AlertSuccess from "@/components/AlertSuccess.vue";
import AlertDanger from "@/components/AlertDanger.vue";
import {
  CITY_ADDED_TO_SYSTEM,
  CITY_DELETED_FROM_SYSTEM,
  NO_CITY_NAME,
  NO_CITY_SELECTED
} from "@/assets/script/AlertMessage";
import {CITY_ALREADY_IN_SYSTEM, NO_DATA_FOR_THAT_CITY} from "@/assets/script/ErrorCode";
import router from "@/router";
import CitiesDropdown from "@/components/CitiesDropdown.vue";

export default {
  name: 'HomeView',
  components: {CitiesDropdown, AlertDanger, AlertSuccess},
  data() {
    return {
      cityName: '',
      selectedCityId: 0,
      errorMessage: '',
      successMessage: '',
      errorResponse: {
        message: '',
        errorCode: 0,
      },
      cities: [
        {
          cityId: 0,
          cityName: ''
        }
      ],
      citiesMeasurementData: [
        {
          temperature: 0,
          windSpeed: 0,
          humidity: 0,
          dateTime: ''
        }
      ]
    }
  },

  methods: {

    addCity() {
      if (this.cityName.length > 0) {
        this.sendAddCityRequest();
      } else {
        this.errorMessage = NO_CITY_NAME
        this.resetErrorMessage()
      }
    },

    resetErrorMessage() {
      setTimeout(() => {this.errorMessage = ''}, 2000)
    },

    sendAddCityRequest() {
      this.$http.post("/city", null, {
            params: {
              cityName: this.cityName
            }
          }
      ).then(response => {
        this.successMessage = CITY_ADDED_TO_SYSTEM
        setTimeout(() => {this.resetFields()}, 2000)
        this.getCityInfo()
      }).catch(error => {
        this.errorResponse = error.response.data
        this.handleErrorResponse()
      })
    },

    resetFields() {
      this.successMessage = ''
      this.cityName = ''
    },

    getCityInfo() {
      this.$http.get("/cities")
          .then(response => {
            this.cities = response.data
          })
          .catch(error => {
            router.push({name: 'errorRoute'})
          })
    },

    handleErrorResponse() {
      if (this.errorResponse.errorCode === CITY_ALREADY_IN_SYSTEM) {
        this.errorMessage = this.errorResponse.message
        this.resetErrorMessage()
      } else if (this.errorResponse.errorCode === NO_DATA_FOR_THAT_CITY) {
        this.errorMessage = this.errorResponse.message
        this.resetErrorMessage()
      } else {
        router.push({name: 'errorRoute'})
      }
    },

    deleteCity() {
      if (this.cityId > 0) {
        this.sendDeleteCityRequest();
      } else {
        this.errorMessage = NO_CITY_SELECTED
        this.resetErrorMessage()
      }
    },

    sendDeleteCityRequest() {
      this.$http.delete("/city", {
            params: {
              cityId: this.selectedCityId,
            }
          }
      ).then(response => {
        this.successMessage = CITY_DELETED_FROM_SYSTEM
        setTimeout(() => {this.successMessage = ''}, 2000)
        this.getCityInfo()
      }).catch(error => {
        router.push({name: 'errorRoute'})
      })
    },

    setSelectedCityId(selectedCityId) {
      this.selectedCityId = selectedCityId
    },

    getCityMeasurementData() {
      this.$http.get("/city", {
            params: {
              cityId: this.selectedCityId
            }
          }
      ).then(response => {
        this.citiesMeasurementData = response.data
      }).catch(error => {
        router.push({name: 'errorRoute'})
      })
    },
  },

  mounted() {
    this.getCityInfo()
  }
}
</script>

<style>
.row {
  margin-bottom: 50px;
}

.col {
  margin-right: 50px;
}
</style>

