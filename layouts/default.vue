<template>
  <v-app>
    <appBar />
    <v-content>
      <v-container fluid fill-height grey lighten-2>
        <nuxt />
        <div class="page-border-top"></div>
        <div class="page-border-left"></div>
        <div class="page-border-right"></div>
        <div class="page-border-bottom"></div>
      </v-container>
    </v-content>
  </v-app>
</template>

<script>
import appBar from '~/components/appbar.vue'

export default {
  components: {
    appBar
  }
}
</script>

<style></style>
