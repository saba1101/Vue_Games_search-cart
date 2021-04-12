<template>
  <div id="app">
    <Data />
  </div>
</template>

<script>
import Data from './components/Data'
export default {
  name: 'App',
  components: {
    Data,
  }
}
</script>

<style lang="scss">


@import url("https://fonts.google.com/specimen/Roboto?preview.text_type=custom");
  *{
    margin: 0;
    padding: 0;
    box-sizing: border-box;
    font-family: 'Roboto', sans-serif;
  }

  body{
  background: #243949;
  background-repeat: no-repeat;
  background-size: cover;
  }


</style>
