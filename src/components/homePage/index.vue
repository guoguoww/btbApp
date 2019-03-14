<template>
  <div>
    <div class="changeBG">
      <div class="loginSucess">
        <div>
          <i class="iconfont icon-mine" @click="poprightShow()"></i>
          <span>BlockChain Terminal</span>
        </div>
      </div>
      <div class="carouselC">
        <swiper :options="swiperOption" ref="mySwiper" id="swiper1">
          <!-- slides -->
          <swiper-slide v-for="(item, index) of images" :key="index"><img :src="item.src" alt=""></swiper-slide>
          <!-- Optional controls -->
          <!--<div class="swiper-pagination"  slot="pagination"></div>-->
        </swiper>
      </div>
      <div class="upDownInfo">
        <p><img src="./images/tb05.png"></p>
        <vue-seamless-scroll   :data="noticeLists" class="verticalScroll" :class-option="classOption">

          <ul>
            <li v-for="(list, index) of noticeLists" :key="index">
              <router-link :to="{path:'announceDetail',query:{msgId: list.id}}" tag="p"><span v-text='list.title'></span></router-link>
            </li>
          </ul>
        </vue-seamless-scroll>
      </div>
      <div class="dataContainer">
        <ul>
          <li>
            <span v-for="(item, index) of marketData" :key="index">{{item.exchangeCode}}/{{item.unitsCode}}</span>
          </li>
          <li>
            <span v-for="(item, index) of diffRatePrice" :key="index" :class="item.price>=0?'increaseUp':'increaseDown'">{{item.price | numTwo}}</span>
          </li>
          <li>
            <span v-for="(item, index) of diffRatePrice" :key="index" :class="item.diffRate>=0?'increaseUp':'increaseDown'">{{(item.diffRate) | numTwo}}%</span>
          </li>
          <li>
            <span v-for="(item, index) of diffRatePrice" :key="index">≈ {{item.symbolCny | numTwo}} CNY</span>
          </li>
        </ul>
      </div>
      <div class="helpNav">
        <router-link to="/c2c" tag="div">
          <div>
            <p>C2C交易</p>
            <span>轻松购买数字产品</span>
          </div>
          <div>
            <img src="./images/tb04.png" alt="">
          </div>
        </router-link>
        <div>
          <router-link to="/announceList" tag="p"><span><img src="./images/tb01.png" alt=""></span><span>公告列表</span></router-link>
          <router-link to="/helpCenter" tag="p"><span><img src="./images/tb02.png" alt=""></span><span>帮助中心</span></router-link>
        </div>
      </div>
      <div class="marketLists">
        <h3>行情</h3>
        <ul>
          <router-link to="/homeMarket" tag="li" v-for="(item,index) of diffRatePrice" :key="index" v-if="index < 10">
            <div><p><span class="weightCode">{{item.exchangeCode}}</span><span class="smallCode">/{{item.unitCode}}</span></p><p class="smallCode"><span>24H量</span><span> {{item.totalVol  | numTwo}}</span></p></div>
            <div><span class="weightCode">{{item.price | numTwo}}</span><span class="smallCode">￥{{item.symbolCny | numTwo}}</span></div>
            <div><span :class="[item.diffRate>0?'up':'down']">{{item.diffRate | numTwo}}%</span></div>
          </router-link>
        </ul>
      </div>
    </div>
    <!-- 右边弹出 菜单 -->
    <van-popup v-model="poprightshow" position="left" class="popright" @click-overlay="poprightClose">
      <div class="swit-head" v-if="!unLogin">
        <p class="account"> {{userData.nickName }} </p>
        <span class="UID"> 邀请码：{{userData.inviteCode}} </span>
      </div>
      <div class="swit-head" v-if="unLogin" @click="loginType">
        <h6>点击登陆</h6>
        <p>欢迎来到BCT</p>
      </div>
      <div class="acc_tab ">
        <ul>
          <li v-for="(item,index) of accTabs" :key="index" @click="tiaoZhuan(item.txt)">
            <img :src="item.url" alt="">
            <p>{{item.txt}}</p>
          </li>
        </ul>
      </div>
      <div class="swit-list">
        <ul>
          <li>
            <router-link to="/assets">
              <img src="../homePage/images/inviteIcon.png" alt="">
              <span>我的资产</span>
              <em class="iconfont icon-jiantouarrow487"> </em>
            </router-link>
          </li>
          <li>
            <router-link to="/c2cMenu">
              <img src="../homePage/images/c2cIcon.png" alt="">
              <span>C2C管理</span>
              <em class="iconfont icon-jiantouarrow487"> </em>
            </router-link>
          </li>
          <li>
            <router-link to="/walletAddress">
              <img src="../homePage/images/walletIcon.png" alt="">
              <span>钱包地址管理</span>
              <em class="iconfont icon-jiantouarrow487"> </em>
            </router-link>
          </li>
          <li>
            <router-link to="/promote">
              <img src="../homePage/images/assetsIcon.png" alt="">
              <span>邀请返佣</span>
              <em class="iconfont icon-jiantouarrow487"> </em>
            </router-link>
          </li>
          <li>
            <router-link to="/security">
              <img src="../homePage/images/securityIcon.png" alt="">
              <span>安全中心</span>
              <em class="iconfont icon-jiantouarrow487"> </em>
            </router-link>
          </li>
          <li>
            <router-link to="/about">
              <img src="../homePage/images/aboutIcon.png" alt="">
              <span>关于我们</span>
              <em class="iconfont icon-jiantouarrow487"> </em>
            </router-link>
          </li>
          <li>
            <a @click="kf">
              <img src="../homePage/images/serviceIcon.png" alt="">
              <span>在线客服</span>
              <em class="iconfont icon-jiantouarrow487"> </em>
            </a>
          </li>
        </ul>
      </div>
      <!--退出按钮-->
      <div class="exit" @click="logout()" v-if="!unLogin">
        <span>退出当前帐号</span>
      </div>
    </van-popup>
  </div>

</template>

<script>
  import Vue from "vue";
  import vueSeamlessScroll from "vue-seamless-scroll";
  // import cookie from "../../../static/js/cookie";
  import { Swipe, SwipeItem, Lazyload } from "vant";
  Vue.use(Swipe).use(SwipeItem);

  Vue.use(Lazyload);


  export default {
    name: "index",
    // directives : {
    //     loadimg (el,binding,vnode){
    //         //el就是img
    //         // console.log(el,binding,vnode);
    //         let src = el.src
    //         let newImg = new Image()
    //         newImg.src = src
    //         // el.src='http://iph.href.lu/1040x435?text=%E5%8A%A0%E8%BD%BD%E4%B8%AD&fg=ccc&bg=ccc'
    //         newImg.onload = function(){
    //             //doSomething
    //             el.src=src
    //         }
    //     }
    // },
    data() {
      return {
        poprightshow: JSON.parse(this.$cookie.get('poprightshow')), //左侧个人中心显示
        accTabs: [
          {url: require('./images/t.png'), txt: '充币'},
          {url: require('./images/cc.png'), txt: '提币'},
          {url: require('./images/zhuan.png'), txt: '划转'},
        ],
        show: true,
        unLogin: true,
        images: [
          { src: require("./images/carousel1.jpg") },
          { src: require("./images/carousel2.jpg") },
          { src: require("./images/carousel3.jpg") }
        ],
        noticeLists: [

        ],
        userData: {}, //用户信息
        marketData: [{exchangeCode:'---',unitsCode:'---'},{exchangeCode:'---',unitsCode:'---'},{exchangeCode:'---',unitsCode:'---'}], //行情数据
        strMarketPar: "", //行情数据中的行情代码
        diffRatePrice: [{price:0,diffRate:0,price:0},{price:0,diffRate:0,price:0},{price:0,diffRate:0,price:0},], //行情数据涨跌率和价格
        unitsCodes: [], //单位集合
        _timeOut: null,
        repeatData: [],
        swiperOption: {
          // 如果需要分页器
          pagination: {
            el: '.swiper-pagination',
          },
          loop:true,
          autoplay:{
            disableOnInteraction: false,
          },
          loopAdditionalSlides: 1,
          effect:'coverflow',
          coverflowEffect: {
            rotate: 20,
            stretch: 10,
            depth: 10,
            modifier: 2,
            slideShadows : true
          },
          spaceBetween: 8,
          //滑动速度
          speed:800,
        },
      };
    },
    created() {
      console.log()
      this.getAllMarket();

    },
    mounted() {
      this.swiper.slideTo(1, 1000, false);
      this.loadimg()
      //首先判断页面是否是登录状态
      var userInfo = JSON.parse(this.$cookie.get("_auth"));
      if (!userInfo) {
        this.unLogin = true;
      } else {
        this.unLogin = false;
        this.getUserData(); //获取登录用户的基本信息
      }

      this._timeOut = setInterval(() => {
        //获取首页中的所有行情数据
        this.getLatestMarket(this.strMarketPar, this.unitsCodes);
      }, 3000);
      this.announce(); //首页公告查询

    },
    methods: {
      //打开客服
      kf(){
           if (!window.plus) {
                  location.href='https://tb.53kf.com/code/client/10196276/1'
                  return
           }
              // plus.webview.open('https://tb.53kf.com/code/client/10196276/1', "baidu");
              plus.runtime.openURL("https://tb.53kf.com/code/client/10196276/1");
              // {
              //   'titleNView':{
              //     'backgroundColor':'#fb8c00',
              //     'titleText':'在线充值',
              //     'titleColor':'#fff',
              //     buttons:[{type:'back',float:'left',onclick:this.close}]
              //     }
              //   }
      },
      close(){
        plus.webview.close( plus.webview.all()[1])
      },
      loadimg(){
        let imgs = document.querySelector('.carouselC').querySelectorAll('img')
        // console.log(imgs)
        Array.from(imgs).forEach((item)=>{
          let img = new Image()
          img.src=item.getAttribute('src')
          item.src='http://iph.href.lu/640x260?text=%E5%8A%A0%E8%BD%BD%E4%B8%AD&fg=ccc&bg=ccc'
          img.onload = ()=>{
            item.src=img.src
          }

        })
      },
      getUserData() {
        //获取用户信息
        this.$axios
          .post("/api/user/query_active_user")
          .then(res => {
            if (res.data.errorCode === 0) {
              this.userData = res.data.data;
            } else if (res.data.errorCode === -2) {
              this.$cookie.clear(); //清除cookie
              this.$router.push("/login"); //跳转登录页
            } else {
              this.$toast(res.data.errorMsg);
            }
          })
          .catch(err => {
            this.$toast(err.data.errorMsg);
          });
      },
      getAllMarket() {
        //获取首页中的所有行情数据
        this.$axios
          .post("/api/market/all_market")
          .then(res => {
            if (res.data.errorCode === 0) {
              this.marketData = res.data.data;
              this.strMarketPar = [];
              this.unitsCodes = [];
              res.data.data.forEach((item, index) => {
                this.strMarketPar += `${item.marketCode},`;
                this.unitsCodes.push(item.unitsCode);

                // this.codesCNY = [...this.codesCNY,...item.marketCode];
                // this.codesCNY.push(item.exchangeCode);

              });

              this.getLatestMarket(
                this.strMarketPar,
                this.unitsCodes
              ); //获取最新行情数据
            } else if (res.data.errorCode === -2) {
              this.$cookie.clear(); //清除cookie
              this.$router.push("/login"); //跳转登录页
            } else {
              this.$toast(res.data.errorMsg);
            }
          })
          .catch(err => {
            this.$toast(err.data.errorMsg);
          });
      },

      getLatestMarket(latestData, unitVal) {
        //获取首页最新行情数据
        this.$axios({
          url: "/api/market/latest_market",
          method: "get",
          params: {
            codes: latestData
          }
        })
          .then(res => {
            if (res.data.errorCode === 0) {
              this.diffRatePrice = res.data.data;
              this.diffRatePrice.forEach((val, index) => {

                this.marketData.map(marketVal => {
                  if(marketVal.marketCode == val.code) {
                    this.$set(val,'exchangeCode',marketVal.exchangeCode);
                    this.$set(val,'unitCode',marketVal.unitsCode);
                  }
                })

              });
              this.$nextTick(()=>this.$store.commit('onready'))
            } else {
              this.$toast(res.data.errorMsg);
            }
          })
          .catch(err => {
            this.$toast(err.data.errorMsg);
          });
      },

      announce() {
        //首页公告查询
        this.$axios({
          url: "/api/user/announce",
          method: "post",
          params: {
            start: 0,
            rows: 100
          }
        })
          .then(res => {
            if (res.data.errorCode === 0) {
              if (res.data.page.data.length !== 0) {
                this.noticeLists = res.data.page.data;
              } else {
                // this.$toast('公告数据为空');
              }
            } else {
              this.$toast(res.data.errorMsg);
            }
          })
          .catch(err => {
            this.$toast(err.data.errorMsg);
          });
      },
      queryDetailInfo(listId) {
        //查询公告明细
        this.$axios({
          url: "/api/user/announce/details",
          method: "post",
          params: {
            announceId: listId
          }
        })
          .then(res => {
            if (res.data.errorCode === 0) {
              this.$router.push("announceDetail");
            } else {
              this.$toast(res.data.errorMsg);
            }
          })
          .catch(err => {
            this.$toast(err.data.errorMsg);
          });
      },

      poprightShow() { //打开侧边栏
        this.poprightshow = true;
        this.$cookie.set('poprightshow', this.poprightshow)
      },
      loginType() {
        if(!localStorage.getItem('loginType') || localStorage.getItem('loginType') == '0') {
          this.$router.push('/login')
        }else {
          this.$router.push('/patternLogin')
        }
      },
      poprightClose() { //点击蒙版 关闭侧边栏
        this.$cookie.delete('poprightshow')
      },
      tiaoZhuan(txt){
        switch (txt) {
          case '充币':
            this.$router.push('moneyIn');
            return;
          case '提币':
            this.$router.push('moneyOut');
            return;
          case '划转':
            this.$router.push({path: '/assetTransfer',query: {curType: '0'}});
            return;
        }
      },
      logout(){  //退出登录
        this.$dialog.confirm({
          title: '退出登录',
          message: '确认退出?'
        }).then(() => {
          if(localStorage.getItem('loginType') == '1' && localStorage.getItem('userMob')) {
            this.$cookie.delete('_auth'); //清除cookie
            this.$router.push('/patternLogin')
          }else {
            this.$cookie.delete('_auth'); //清除cookie
            this.$router.push('/login')
          }
        })
      },
    },
    beforeDestroy() {
      this.$store.commit('unready')
      clearInterval(this._timeOut);
      this._timeOut = null;
    },
    watch: {

      marketData: {
        handler: function(newVal, oldVal) {},
        deep: true
      },
      immediate: true
    },
    computed: {
      classOption() {
        return {
          step: 0.5, //步长 越大滚动速度越快
          limitMoveNum: 2, //启动无缝滚动最小数据量 this.dataList.length
          hoverStop: false, //是否启用鼠标hover控制
          direction: 1, //1 往上 0 往下
          openWatch: true, //开启data实时监听
          singleHeight: 20, //单条数据高度有值hoverStop关闭
          waitTime: 2500, //单步停止等待时间
        };
      },
      swiper() {
        return this.$refs.mySwiper.swiper
      }
    },
    filters: {
      //保留2位小数点过滤器 不四舍五入
      numTwo(value) {
        var toFixedNum = Number(value).toFixed(3);
        var realVal = toFixedNum.substring(
          0,
          toFixedNum.toString().length - 1
        );
        return realVal;
      },
      number4(value) {
        var toFixedNum = Number(value).toFixed(5);
        var realVal = toFixedNum.substring(
          0,
          toFixedNum.toString().length - 1
        );
        return realVal;
      }
    },
    components: {
      vueSeamlessScroll,
    }
  };
</script>

<style scoped lang="less">
  .mint-header {
    background-color: #fff;
    color: #787878;
  }
  img {
    max-width: 100%;
  }
  .loginSucess {
    padding: 0 1.2rem;
    display: flex;
    justify-content: space-between;
    /*color: #00b4ff;*/
    height: 4rem;
    line-height: 4rem;
  }
  .loginSucess i,
  .loginSucess em {
    color: #00b4ff;
    vertical-align: middle;
    font-size: 2.6rem;
  }
  .loginSucess span {
    font-size: 1.7rem;
    vertical-align: middle;
  }
  .loginSucess > div {
    width: 100%;
    position: relative;
    text-align: center;
  }
  .loginSucess i {
    position: absolute;
    left: .5rem;
  }
  .header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    height: 3.5rem;
    padding: 0 1.2rem;
    font-size: 1.25rem;
    color: #787878;
  }
  .carouselC {
    padding: 0 2rem;
    overflow: hidden;
  }
  .carouselC > div {
    border-radius: 0.5rem;
  }

  .header img {
    height: 2rem;
  }
  .verticalScroll {
    font-size: 1rem;
    color: #8c8c8c;
    height: 1.8rem;
    line-height: 1.8rem;
    overflow: hidden;
    text-align: left;
    padding-left: 4rem;
    padding-top: .2rem;
  }
  .verticalScroll li p {
  }
  .verticalScroll li span {
    display: inline-block;
    width: 60%;
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
    vertical-align: middle;
  }
  .verticalScroll i {
    display: inline-block;
    width: 1.3rem;
    height: 1.3rem;
    background: url("./images/notice.png") center center no-repeat;
    background-size: cover;
    margin-right: 0.5rem;
    vertical-align: middle;
  }
  .dataContainer ul {
    overflow-x: scroll;
    padding: 0 1.2rem;
    padding-top: 0.5rem;
  }
  .dataContainer ul li {
    line-height: 1.4rem;
    color: #787878;
    white-space: nowrap;
  }
  .dataContainer ul li span {
    display: inline-block;
    width: 31%;
    padding: 0 0.5rem;
    text-align: center;
    font-size: 1rem;
  }
  .dataContainer ul li .increaseUp {
    color: #05c19e;
  }
  .dataContainer ul li .increaseDown {
    color: #ff3b3b;
  }
  .index_type {
    padding: 0 1.2rem 0 1.2rem;
  }
  .index_type li {
    margin-top: 1rem;
    background: url("./images/rt_arrow_bg.png") no-repeat 98% 0.3rem;
    border: 1px solid #f0f0f0;
    height: 3.5rem;
    line-height: 3.5rem;
    border-radius: 0.5rem;
    box-shadow: 0 0.3rem 0.6rem #edecec;
    background-size: 3.8rem 2.9rem;
    text-align: left;
  }
  .index_type li span {
    display: inline-block;
    padding-left: 3rem;
    vertical-align: middle;
    color: #bbb;
    font-size: 1rem;
  }
  .index_type li:first-child span {
    background: url("./images/sy_c2c.png") no-repeat 1rem 0.9rem;
    background-size: 1.6rem 1.6rem;
  }
  .index_type li:nth-child(2) span {
    background: url("./images/sy_hy.png") no-repeat 1rem 0.9rem;
    background-size: 1.6rem 1.6rem;
  }
  .index_type li:nth-child(3) span {
    background: url("./images/sy_bb.png") no-repeat 1rem 0.9rem;
    background-size: 1.6rem 1.6rem;
  }
  .index_type ul li b {
    font-size: 1rem;
    font-weight: bold;
    padding: 0 1.2rem 0 2rem;
    color: #00b4ff;
    vertical-align: middle;
  }
  .index_type ul li em {
    font-size: 1rem;
    vertical-align: middle;
    color: #00b4ff;
  }
  .index_bottom {
    position: absolute;
    bottom: 4rem;
    font-size: 1rem;
    width: 100%;
    height: 2.5rem;
    text-align: center;
    color: #fff;
    left: 0;
    background: url("./images/index_bottom_bg.png") no-repeat 0 0;
    line-height: 2.5rem;
    background-size: 100% 100%;
  }
  .helpNav {
    background: #f5f5f5;
    padding: .6rem;
    display: flex;
  }
  .helpNav > div:first-child {
    width: 60%;
    background: #fff;
    margin-right: 1rem;
    padding: 0 1rem;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
  .helpNav > div:first-child > div:last-child {
    width: 40%;
  }
  .helpNav > div:first-child p {
    color: #000;
    font-size: 1.6rem;
    padding-bottom: .3rem;
  }
  .helpNav > div:last-child {
    width: 40%;
  }
  .helpNav > div:last-child p {
    display: flex;
    background: #fff;
    padding: .8rem;
    text-align: center;
    align-items: center;
  }
  .helpNav > div:last-child p span:first-child {
    width: 22%;
    max-width: 100%;
    padding-right: 1rem;
  }
  .helpNav > div:last-child p:first-child {
    margin-bottom: .5rem;
  }

  .marketLists > h3 {
    padding: 1.2rem;
    border-bottom: 1px solid #eee;
  }
  .marketLists > ul li {
    display: flex;
    align-items: center;
    border-bottom: 1px solid #e5e5e5;
    padding: 1rem 1.2rem;
  }

  .marketLists > ul li > div {
    width: 40%;
    display: flex;
    flex-direction: column;
  }
  .marketLists > ul li > div:first-child {

  }
  .marketLists > ul li > div:last-child {
    width: 20%;
    text-align: center;
  }
  .marketLists > ul li > div:last-child span {
    background: #05c19e ;
    padding: .6rem .8rem;
    color: #fff;
    border-radius: .2rem;
  }



  // 右边弹出 菜单
  .popright {
    background-color: #fff;
    width: 76%;
    height: 100%;
    .swit-head {
      padding: 1.5rem 2rem 1.2rem;
      .account {
        font-size: 1.5rem;
        font-weight: bold;
        line-height: 2.6rem;
      }
      .UID {
        font-size: 1.1rem;
      }
      h6 {
        font-size: 1.8rem;
        font-weight: normal;
      }
      border-bottom: 1px solid #f1f1f1;
    }
    .acc_tab {
      padding: 0 2rem;
      background: #fff;
      border-bottom: 1px solid #f1f1f1;
      ul {
        display: flex;
        justify-content: space-between;
        li {
          padding: 1.4rem 0;
          flex: 1;
          text-align: center;
          img {
            max-width: 100%;
            height: 2.8rem;
            vertical-align: bottom;
          }
          p {
            padding-top: 0.4rem;
            color: #787878;
          }
        }
        li:first-child {
          text-align: left;
        }
        li:last-child {
          text-align: right;
        }
      }
    }
    .swit-list {
      ul li {
        background: #fff;
        padding: 0 2rem;
        height: 3.93rem;
        line-height: 3.93rem;
        border-bottom: 0.66px solid #f1f1f1;
        color: #000;
        /*position: relative;*/
        a {
          display: block;
          height: 100%;
          color: #000;
          line-height: 4rem;
          position: relative;
          img {
            position: absolute;
            height: 1.8rem;
            top: 1rem;
          }
          span {
            position: absolute;
            font-size: 1.2rem;
            left: 2.4rem;
            color: #787878;
          }
          em {
            position: absolute;
            right: 0;
            color: #b4b4b4;
          }
        }
      }
    }
    /*退出按钮*/
    .exit {
      width: 70%;
      height: 4rem;
      line-height: 4rem;
      background: #00b4ff;
      text-align: center;
      color: #fff;
      font-size: 1.4rem;
      margin: 1.5rem auto 0;
      border-radius: 5%;
    }
  }
  .upDownInfo {
    width: 100%;
    overflow: hidden;
    position: relative;
  }
  .upDownInfo > p {
    position: absolute;
    top: .2rem;
    left: 1.6rem;
    width: 6%;
  }
  .upDownInfo > p img {
    max-width: 100%;
    vertical-align: middle;
  }
  .swiper-container {
    width: 100%;
    overflow: visible;
    position: relative;
  }
  .swiper-container img {
    border-radius: .5rem;
  }
  .up {
    background: #d14b63 !important;
  }
  .down {
    background: #04ac8e !important;
  }
  .weightCode {
    color: #2d5073;
    font-size: 1.45rem;
    padding-right: .6rem;
    font-weight: 600;
  }
  .smallCode {
    font-size: .9rem;
    color: #b0bebf;
    font-weight: lighter;
  }
</style>
