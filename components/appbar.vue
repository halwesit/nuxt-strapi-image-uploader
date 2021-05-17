<template>
  <div>
    <v-app-bar
      :clipped-left="$vuetify.breakpoint.lgAndUp"
      app
      color="primary"
      dark
    >
      <v-spacer></v-spacer>
      <h2>Image Uploader - Advanced Database homework</h2>
      <v-spacer></v-spacer>
    </v-app-bar>
  </div>
</template>

<script>
export default {
  data: () => ({
    dialog: false,
    drawer: null
  })
}
</script>

<style scoped></style>
