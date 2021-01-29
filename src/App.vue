<template>
  <div id="app">
    <deckBuilder></deckBuilder>
  </div>
</template>


<script>


import DeckBuilder from "./components/deckbuilder-dbs/deckBuilder.vue";

export default {
  name: "App",
  components: { DeckBuilder },
};
</script>

<style>
#app {
  padding: 10px;
  height: calc(100% - 20px);
  font-family: Avenir, Helvetica, Arial, sans-serif;
}
</style>
