<script src="https://unpkg.com/rxjs/bundles/rxjs.umd.js"></script>
<script src="https://unpkg.com/vue/dist/vue.js"></script>
<script src="../dist/vue-rx.js"></script>

<template>
  <div id="app">
    <PricesView />
  </div>
</template>

<script lang="ts">
import Vue from "vue";
import VueRx from "vue-rx";

Vue.use(VueRx);

import PricesView from "./components/PricesView.vue";

export default Vue.extend({
  name: "App",
  components: {
    PricesView
  }
});
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
