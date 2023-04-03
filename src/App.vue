<template>
  <div id="app">
    <h1>Mia Bot</h1>
    <InputBox />
  </div>
</template>

<script>
import InputBox from './components/InputBox.vue'

export default {
  name: 'App',
  components: {
    InputBox
  }
}
</script>

<style>
#app {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100vh;
  font-family: Arial, Helvetica, sans-serif;
}

h1 {
  font-size: 3rem;
  margin-bottom: 2rem;
}

input {
  padding: 0.5rem 1rem;
  font-size: 1.2rem;
  border: 2px solid #0074d9;
  border-radius: 4px;
  width: 60vw;
}

div {
  margin-top: 1rem;
  text-align: center;
}

p {
  margin: 0.5rem;
  font-size: 1.2rem;
}

#results {
  display: flex;
  flex-direction: column;
  text-align: left;
}
</style>
