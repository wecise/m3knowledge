<template>

  <AppView ></AppView>
  
</template>

<script>
import AppView from './app/AppView';


export default {
  name: 'MainView',
  props: {
    global: Object
  },
  components:{
    AppView
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
