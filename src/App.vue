<template>
  <div class="header">
    <h1>PokeDex</h1>
    <img src="./assets/pokedex.png" alt="pokedex" />
  </div>
  <router-view />
</template>

<style>
body {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  background-color: lightgrey;
  color: #2c3e50;
  margin: 0 !important;
}

.header {
  display: flex;
  text-align: center;
  justify-content: center;
  margin-top: 25px;
}
.header img {
  height: 100px;
  width: 100px;
}
</style>
