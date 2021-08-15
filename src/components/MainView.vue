<template>

  <KnowledgeView ></KnowledgeView>
  
</template>

<script>
import KnowledgeView from './knowledge/index';


export default {
  name: 'MainView',
  props: {
    global: Object
  },
  components:{
    KnowledgeView
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
