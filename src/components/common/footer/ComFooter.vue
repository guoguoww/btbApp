<template>
    <div id="Foot">
      <ul class="footerC">
        <!-- <li><router-link to="/" exact><i class="iconfont icon-shouyeicon"></i><span>首页</span></router-link></li> -->
        <li><router-link to="/" exact><img :src="`./static/images/logo${num}.png`" alt="" style="width: 24px;height: 26px;"><span>首页</span></router-link></li>
        <li><router-link to="/homeMarket"><i class="iconfont icon-hangqingicon"></i><span>行情</span></router-link></li>
        <li><router-link to="/c2c"><i class="iconfont icon-CCicon"></i><span>C2C</span></router-link></li>
        <li><router-link to="/contract"><i class="iconfont icon-heyueicon"></i><span>合约</span></router-link></li>
        <li><router-link to="/coin"><i class="iconfont icon-bibiicon"></i><span>币币</span></router-link></li>
      </ul>
    </div>
</template>

<script>
    export default {
        name: "Comfooter",
        data () {
          return {
          }
        },
        computed: {
          num(){
            var num=0
            console.log(this.$route);
            this.$route.path=='/'?num=1:num=2
            return num
          }
        }
    }
</script>

<style scoped>
  #Foot {
    position: fixed;
    right: 0;
    left: 0;
    bottom: 0;
    z-index: 99;
    width: 100%;
    background: #ffffff;
    box-shadow: 0px -2px 3px 0px #e9e9e9;
  }

  .footerC {
    display: flex;
    /* justify-content: space-around; */
    padding: .4rem 0;
  }
  .footerC li{
    width: 25%;
  }
  .footerC li a{
    flex: 1;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
  }
  .footerC li a i{
    color: #b4b4b4;
    font-size: 2rem;
    /* transform: scale(1); */
    /* transition-duration: 0.2s; */
  }
    @keyframes popup {
      0% {
          transform: scale(0.9);
      }
      50%{
          transform: scale(1.1);
      }
      to {
          transform: scale(1);
      }
  }
  .footerC li span {
    padding-top: 0.2rem;
    font-size: 1rem;
    color: #787878;
    line-height: 1rem;
  }
  .router-link-active i,.router-link-active img {
    color: #0088f1 !important;
    animation: popup 0.2s linear;
  }
  .router-link-active span{
    color: #0189fd !important;
  }
  /*.router-link-exact-active i[data-v-22069cac] {*/
    /*color: #0088f1 !important;*/
  /*}*/
</style>
