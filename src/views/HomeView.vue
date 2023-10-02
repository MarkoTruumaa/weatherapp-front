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
        <MeasurementsDataTable :cities-measurement-data="cityMeasurementData"/>
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
import MeasurementsDataTable from "@/components/MeasurementsDataTable.vue";

export default {
  name: 'HomeView',
  components: {MeasurementsDataTable, CitiesDropdown, AlertDanger, AlertSuccess},
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
      cityMeasurementData: [
        {
          temperature: 0,
          windSpeed: 0,
          humidity: 0,
          time: ''
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
        this.getAllCitiesInfo()
      }).catch(error => {
        this.errorResponse = error.response.data
        this.handleErrorResponse()
      })
    },

    resetFields() {
      this.successMessage = ''
      this.cityName = ''
    },

    getAllCitiesInfo() {
      this.$http.get("/city/all")
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
      if (this.selectedCityId > 0) {
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
        this.getAllCitiesInfo()
      }).catch(error => {
        router.push({name: 'errorRoute'})
      })
    },

    setSelectedCityId(selectedCityId) {
      this.selectedCityId = selectedCityId
    },

    getCityMeasurementData() {
      if (this.selectedCityId > 0) {
        this.sendCityMeasurementDataRequest();
      } else {
        this.errorMessage = NO_CITY_SELECTED
        this.resetErrorMessage()
      }
    },

    sendCityMeasurementDataRequest() {
      this.$http.get("/city", {
            params: {
              cityId: this.selectedCityId
            }
          }
      ).then(response => {
        this.cityMeasurementData = response.data
      }).catch(error => {
        router.push({name: 'errorRoute'})
      })
    },
  },

  mounted() {
    this.getAllCitiesInfo()
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

