<template>
  <section style="position: absolute; height: 100%; width: 100%; background: #fff">
    <!--头部-->
    <van-nav-bar title="我的资产" left-text="返回" left-arrow @click-left="$back()">
      <van-icon class="iconfont icon-fanhui" slot="left" />
    </van-nav-bar>
    <!--内容-->
    <div class="content">
      <ul class="tradingTab">
        <li v-for="item in tradingTab" :class="{'active': item.id === tradingTabA}" @click="changeTab(item.id)">{{item.name}}</li>
      </ul>
      <div class="details">
        <ul class="total">
          <li>
            <h6>{{tradingTab[tradingTabA].name}}账户</h6>
            <span>总资产折合（BTC）</span>
          </li>
          <li>
            <p>{{total}}</p>
          </li>
          <li>
            <span>≈ {{changeH}} CNY</span>
          </li>
        </ul>
        <ul class="currency">
          <li v-for="item in currency">
            <router-link :to="tradingTabA === 0 ? {path:'assetsDetails', query:{cur: JSON.stringify(item)}} : ''">
              <p>
                <span>{{item.currencyCode}}</span>
                <em class="iconfont icon-jiantouarrow487 fr" v-if="tradingTabA === 0"> </em>
              </p>
              <ul class="currencyDetails">
                <li class="title">
                  <span>可用</span>
                  <span>冻结</span>
                  <span>折合(CNY)</span>
                </li>
                <li>
                  <span>{{item.amount}}</span>
                  <span>{{item.frozenAmount}}</span>
                  <span>{{item.cny}}</span>
                </li>
              </ul>
            </router-link>
          </li>
        </ul>
      </div>
    </div>
  </section>
</template>

<script>
  export default {
    name: "index",
    data() {
      return {
        tradingTab: [
          {name: 'C2C', id: 0},
          {name: '币币', id: 1},
          {name: '合约', id: 2},
        ],
        tradingTabA: 0,
        currency: [], //币种列表
        total: 0.00,
        changeH: 0.00,
      }
    },
    mounted() {
      this.getSurveyQuery();
      this.getQuery();
    },
    methods: {
      changeTab(id) { //切换tab
        this.tradingTabA = id;
        this.getSurveyQuery();
        this.getQuery();
      },
      getSurveyQuery(){//获取资产概况查询
        this.$axios({
          url: '/api/assets/survey/query',
          method: 'post',
        })
          .then(res => {
            if(res.data.errorCode === 0){
              if(res.data.data.length !== 0){
                res.data.data.forEach((item,index) => {
                  if(item.assetsType == this.tradingTabA){
                    this.changeH = item.cny.toFixed(2);
                    this.getBtc(item.cny)
                  }
                })
              }else {
                this.$toast('资产概况数据为空');
              }
            }else {
              this.$toast(res.data.errorMsg);
            }
          })
          .catch(err => {
            this.$toast(err.data.errorMsg);
          })
      },
      getBtc(cny) { //通过人民币换算BTC
        this.$axios({
          url: '/api/market/price',
          method: 'post',
          data: {
            codes: 'BTC'
          }
        }).then(res => {
          if(!res.data.errorCode) {
            this.total = (cny/res.data.data.BTC).toFixed(2);
          }
        })
      },
      getQuery() { //获取账户币种信息
        this.$axios({
          url: '/api/assets/query',
          method: 'post',
          data: {
            type: this.tradingTabA,
          }
        }).then(res => {
          if(!res.data.errorCode) {
            this.currency = res.data.data;
          }
        })
      }
    },
  }
</script>

<style scoped lang="less">
  .content {
    padding: 0.5rem;
    .tradingTab {
      height: 2.6rem;
      line-height: 2.6rem;
      li {
        display: inline-block;
        width: calc(100% / 3);
        text-align: center;
      }
      li.active {
        color: #1882d6;
        border-bottom: 1px solid #4277a1;
      }
    }
    .details {
      .total {
        margin: 1rem 0;
        border: 1px solid #ddd;
        li {
          padding: 0 1rem;
          line-height: 2.4rem;
          h6 {
            display: inline-block;
            font-size: 1.4rem;
          }
          span {
            display: inline-block;
            font-size: 1rem;
          }
          p {
            display: inline-block;
            font-size: 1.4rem;
            font-weight: bold
          }
        }
      }
      .currency {
        font-size: 0.8rem;
        li {
          margin: 1rem 0;
          border: 1px solid #ddd;
          a {
            p {
              line-height: 2.2rem;
              padding: 0 1rem;
              font-size: 1.1rem;
              font-weight: bold;
            }
            .currencyDetails {
              padding: 0 1rem;
              li {
                border: none;
                span {
                  display: inline-block;
                  width: calc((100% - 0.7rem) / 3);
                  text-align: center;
                }
                span:first-child {
                  text-align: left;
                }
                span:last-child {
                  text-align: right;
                }
              }
            }
          }
        }
      }
    }
  }
</style>
