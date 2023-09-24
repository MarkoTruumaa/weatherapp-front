<template>
  <table class="table">
    <thead>
    <tr>
      <th scope="col"></th>
      <th scope="col">Kellaaeg</th>
      <th scope="col">Tempertatuur</th>
      <th scope="col">Tuulekiirus m/s</th>
      <th scope="col">Õhuniiskus %</th>
    </tr>
    </thead>
    <tbody>
    <tr v-for="(cityData, index) in citiesMeasurementData" :key="index">
      <th scope="row">{{ index + 1 }}</th>
      <td>{{ cityData.time }}</td>
      <td>{{ cityData.temperature }}</td>
      <td>{{ cityData.windSpeed}}</td>
      <td>{{ cityData.humidity}}</td>
    </tr>
    </tbody>
  </table>
</template>
<script>
export default {
  name: 'MeasurementsDataTable',
  props: {
    citiesMeasurementData: {}
  }
}
</script>