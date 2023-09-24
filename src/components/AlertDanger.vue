<template>
  <div v-show="alertMessage != null && alertMessage.length > 0" class="alert alert-danger" role="alert">
    {{ alertMessage }}
  </div>
</template>
<script>
export default {
  name: 'AlertDanger',
  props: {
    alertMessage: String
  }
}
</script>