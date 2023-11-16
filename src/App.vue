<template>
  <v-app>
    <v-app-bar
      app
      color="primary lighten-2"
      dark
    >
      <v-icon color="green darken-2">mdi-currency-usd</v-icon>
      <v-toolbar-title>Max out my 401k! (for 2024)</v-toolbar-title>

      <v-spacer></v-spacer>

    </v-app-bar>

    <v-main>
      <Calculator/>
    </v-main>
  </v-app>
</template>

<script>
import Calculator from './components/Calculator';

export default {
  name: 'App',

  components: {
    Calculator,
  },

  data: () => ({
    //
  }),
};
</script>
