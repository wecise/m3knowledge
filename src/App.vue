<template>
  <div class="m3">
    <Header :auth="auth" v-if="auth" class="header"></Header>
    <div class="main">
      <SideBar v-if="auth" class="sidebar" :auth="auth" :global="global"></SideBar>
      <MainView :global="global" v-if="global" class="content"></MainView>
    </div>
    <Footer :auth="auth" v-if="auth" class="footer"></Footer>
  </div>
</template>

<script>

import MainView from './components/MainView';
import Header from './components/layout/Header';
import Footer from './components/layout/Footer';
import SideBar from './components/layout/SideBar';

export default {
  name: 'app',
  components: {
    Header,
    MainView,
    Footer,
    SideBar
  },
  data(){
    return {
      global:null,
      auth: null
    }
  },
  mounted(){
    setTimeout(()=>{
      this.global = this.m3.global;
      this.auth = this.m3.auth.signedUser;
    },500)
  }
}
</script>

<style>
  body{
    font-size: 12px;
    font-family: "PingFang SC",Arial,"Microsoft YaHei",sans-serif;
    margin: 0px;
    padding: 0px;
    overflow: hidden;
  }
  
  .el-menu .svg-icon{
    width: 1.2em!important;
    height: 1.2em!important;
    padding: 0px 5px 0 0;
  }

  .main{
    padding-top: 50px;
    display: flex;
  }
</style>
