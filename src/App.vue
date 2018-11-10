<template lang="pug">
  #app
    Search
</template>


<script>
  import Search from './components/Search';

  export default {
    components: {
      Search,
      Chart,
    }
  }
</script>

<style lang="sass">
  body
    width: 100vw
    height: 100vh
    font-family: "Roboto", sans-seri

  #app
    width: 100%
    height: 100%
</style>
