<template>
  <select v-model="selectedCityId" @change="emitSelectedCityId" class="form-select" aria-label="Default select example">
    <option selected :value="0">Vali linn</option>
    <option v-for="city in cities" :value="city.cityId" :key="city.cityId">
      {{ city.cityName }}
    </option>
  </select>
</template>
<script>
export default {
  name: 'CitiesDropdown',
  props: {
    cities: {},
  },
  data() {
    return {
      selectedCityId: 0
    }
  },

  methods: {
    emitSelectedCityId() {
      this.$emit('event-update-selected-city-id', this.selectedCityId)
    },
  }
}
</script>