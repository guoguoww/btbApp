<template>
    <div style="position: absolute; height: 100%; width: 100%; background: #fff">
      <!--头部-->
      <van-nav-bar title="邀请推广" left-text="返回" left-arrow @click-left="$back()">
        <van-icon class="iconfont icon-fanhui" slot="left" />
      </van-nav-bar>
      <!--内容-->
      <!-- <div class="promote_con">
        <img src="../image/fanyong.png" alt="">
        <ul>
          <li class="clearfix">
            <b>链接&nbsp&nbsp&nbsp&nbsp</b>
            <input type="text" id="link" value="" v-model="shareLink" readonly>
            <span  @click="copytxt('link')">
              <i class="iconfont icon-fuzhi"></i>
              复制
            </span>
          </li>
          <li class="clearfix">
            <b>邀请码&nbsp</b>
            <input type="text" id="invite" value="" v-model="userData.inviteCode" readonly>
            <span @click="copytxt('invite')">
              <i class="iconfont icon-fuzhi"></i>
              复制
            </span>
          </li>
        </ul>
      </div> -->
      <!--<div style="position: relative;height:100%;">-->
        <!--<img  style="width:100%;height:100%;" src="../../../../static/images/t.jpg" alt="">-->
        <!--<div class="a"  style="text-align:center">-->
          <!--<div id="qrcode" ref="qrcode"></div>-->
        <!--</div>-->
      <!--</div>-->
      <div class="promoteAddress">
        <div class="copyAddress">
          <h6>推广链接</h6>
          <input id="shareLink" type="text" readonly :value="router + 'register/' + userData.inviteCode">
          <span @click="copytxt('shareLink')">复制</span>
        </div>
        <div class="copyAddress">
          <h6>邀请码</h6>
          <input id="shareCode" type="text" readonly :value="userData.inviteCode">
          <span @click="copytxt('shareCode')">复制</span>
        </div>
      </div>
      <div class="shareDetails">
        <h6>我的用户</h6>
        <ul class="shareList">
          <li>
            <span>姓名</span>
            <span>日期</span>
            <span>返佣数量</span>
          </li>
          <!--<li v-for="item in 10">-->
            <!--<span>嗷吼猴</span>-->
            <!--<span>2019.2.18 10:21:00</span>-->
            <!--<span>2019</span>-->
          <!--</li>-->

          <van-list v-if="promotion.total > 0" class="shareList" v-model="loading" :finished="finished" :immediate-check="immediateCheck" :offset="10" @load="onLoad(operation)" check>
            <!--<van-list v-model="loading" :finished="finished" @load="onLoad">-->
            <van-cell v-for="(item,i) in promotion.data" :key="i">
              <span class="date">{{item.mobile}}</span>
              <span class="">{{item.lastTradeTime }}</span>
              <span class="">{{item.tradeTotal }}</span>
            </van-cell>
            <!--<van-cell v-for="item in list" :key="item" :title="item + ''" />-->
          </van-list>

          <li class="null" v-if="promotion.total === 0">
            <span>暂无分享人数</span>
          </li>
        </ul>
      </div>
    </div>
</template>

<script>
  export default {
    name: "promote",
    data() {
      return {
        userData: {},
        pages: {
          start: -10,
          rows: 10,
        },
        promotion: {
          data: [],
          total: 0,
        },
        loading: false, //是否显示加载中提示，加载过程中不触发load事件
        finished: false, //是否已加载完成，加载完成后不再触发load事件
        immediateCheck: true, //是否在初始化时立即执行滚动位置检查
        router:''
      }
    },
    computed: {

    },
    created() {
      this.getUserData();
      this.getPromotion();
      this.geturl()
    },
    methods: {
      //获取推广链接
      geturl(){
        this.$axios.post('api/user/download_link').then(res=>{
          if(!res.data.errorCode){
           this.router=res.data.data.H5
          }
        })
      },
      getUserData() {  //获取用户信息
        this.$axios.post("/api/user/query_active_user")
          .then(res => {
            if(res.data.errorCode === 0) {
              this.userData = res.data.data;
            }
          })
      },
      copytxt(id) { //复制
        try{
            if(document.execCommand('copy', false, null)){
                //success info
                var input = document.createElement("input");
                input.value = document.getElementById(id).value;
                document.body.appendChild(input);
                input.select();
                input.setSelectionRange(0, input.value.length), document.execCommand('Copy');
                document.body.removeChild(input);
                this.$toast('已复制到剪贴板');
            } else{
                //fail info
            }
          } catch(err){
              //fail info
          }
      },

      getPromotion() {
        this.pages.start += this.pages.rows;
        this.$axios({
          url: '/api/assets/promotion/details',
          method: 'post',
          data: this.pages,
        }).then(res => {
          if(!res.data.errorCode) {
            this.loading = false;
            this.finished = false;

            this.promotion.data = this.promotion.data.concat(res.data.page.data)
            this.promotion.total = res.data.page.total

            if (this.promotion.data.length >= this.promotion.total) {
              this.finished = true;
            }
          }
        })
       },
      onLoad() { //滚动加载
        setTimeout(() => {
          this.getPromotion()
        }, 500);
      },
    }
  }
</script>

<style scoped lang="less">
  /*.a{*/
      /*position: absolute;*/
      /*left: 27%;*/
      /*transform: translateX(-50%);*/
      /*bottom: 31%;*/
  /*}*/
  /*img{*/
      /*-webkit-touch-callout:default;*/
    /*}*/
  /*.promote_con {*/
    /*border-top: 2px solid #f0f0f0;*/
    /*text-align: center;*/
  /*}*/
  /*.promote_con img{*/
    /*margin: 4rem 0;*/
    /*width: 21rem;*/
    /*height: 21rem;*/
  /*}*/
  /*.promote_con ul li {*/
    /*height: 3rem;*/
    /*margin: 1rem 0;*/
  /*}*/
  /*.promote_con ul li input {*/
    /*width: 50%;*/
    /*height: 100%;*/
    /*border: 1px solid #ccc;*/
    /*padding-left: 1rem;*/
  /*}*/
  /*.promote_con ul li span {*/
    /*display: inline-block;*/
    /*width: 18%;*/
    /*text-align: center;*/
    /*height: 100%;*/
    /*line-height: 3rem;*/
    /*border: 1px solid #ccc;*/
    /*background: #30b8ff;*/
    /*color: #fff;*/
  /*}*/
  .promoteAddress {
    padding: 0.5rem 1rem 0;
    .copyAddress {
      margin: 10px 0;
      position: relative;
      h6 {
        font-size: 1.2rem;
        font-weight: normal;
        line-height: 2.6rem;
      }
      input {
        width: calc(100% - 4rem);
        height: 2rem;
        border: 1px solid #7f828b;
        padding-left: 0.5rem;
      }
      span {
        position: absolute;
        right: 0;
        width: 3rem;
        height: 2rem;
        text-align: center;
        line-height: 2rem;
        background: #f29700;
        color: #fff;
        font-size: 0.6rem;
        border: 1px solid #f29700;
      }
    }
  }
  .shareDetails {
    margin-top: 1rem;
    h6 {
      font-size: 1.2rem;
      padding-left: 0.5rem;
      /*font-weight: normal;*/
      line-height: 3rem;
    }
    .shareList {
      li {
        height: 2rem;
        line-height: 2rem;
        span{
          display: inline-block;
          width: calc((100% - 0.7rem) / 3);
          text-align: center;
          font-size: 0.7rem;
        }
      }
      .van-list {
        .van-cell {
          padding: 10px 0;
          .van-cell__value {
            height: 2rem;
            line-height: 2rem;
            span {
              display: inline-block;
              width: calc((100% - 0.7rem) / 3);
              text-align: center;
              font-size: 0.7rem;
            }
          }
        }
      }
      /*没有推荐账户*/
      .null {
        text-align: center;
        padding: 30% 0;
        span {
          display: block;
          width: 100%;
          font-size: 0.6rem;
          color: #f29700;
        }
      }
    }
  }

</style>
