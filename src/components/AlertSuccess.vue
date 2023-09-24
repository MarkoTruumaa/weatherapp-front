<template>
  <div v-show="alertMessage != null && alertMessage.length > 0" class="alert alert-success" role="alert">
    {{ alertMessage }}
  </div>
</template>
<script>
export default {
  name: 'AlertSuccess',
  props: {
    alertMessage: String
  }
}
</script>