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
        <select v-model="selectedCityId" class="form-select" aria-label="Default select example">
          <option selected :value="0">Vali linn</option>
          <option v-for="city in cities" :value="city.cityId" :key="city.cityId">
            {{ city.cityName }}
          </option>
        </select>
        <button @click="deleteCity">Kustuta</button>
      </div>
    </div>
    <div class="row justify-content-center">
      <div>

      </div>
    </div>
  </div>
</template>

<script>

import AlertSuccess from "@/components/AlertSuccess.vue";
import AlertDanger from "@/components/AlertDanger.vue";
import {CITY_ADDED_TO_SYSTEM, CITY_DELETED_FROM_SYSTEM} from "@/assets/script/AlertMessage";
import {CITY_ALREADY_IN_SYSTEM, NO_DATA_FOR_THAT_CITY} from "@/assets/script/ErrorCode";
import router from "@/router";

export default {
  name: 'HomeView',
  components: {AlertDanger, AlertSuccess},
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
      ]
    }
  },

  methods: {

    addCity() {
      this.errorMessage = ''
      this.$http.post("/city", null, {
            params: {
              cityName: this.cityName
            }
          }
      ).then(response => {
        this.successMessage = CITY_ADDED_TO_SYSTEM
        setTimeout(() => {this.resetFields()}, 2000)
      }).catch(error => {
        this.errorResponse = error.response.data
        this.handleErrorResponse()
      })
    },

    resetFields() {
      this.successMessage = ''
      this.cityName = ''
    },

    handleErrorResponse() {
      if (this.errorResponse.errorCode === CITY_ALREADY_IN_SYSTEM) {
        this.errorMessage = this.errorResponse.message
      } else if (this.errorResponse.errorCode === NO_DATA_FOR_THAT_CITY) {
        this.errorMessage = this.errorResponse.message
      } else {
        router.push({name: 'errorRoute'})
      }
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

    deleteCity() {
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
  },

  mounted() {
    this.getCityInfo()
  }
}
</script>

<style>

</style>
