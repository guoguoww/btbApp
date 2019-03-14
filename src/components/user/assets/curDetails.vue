<template>
  <section style="position: absolute; height: 100%; width: 100%; background: #fff">
    <!--头部-->
    <van-nav-bar title="资产详情" left-text="返回" left-arrow @click-left="$back()">
      <van-icon class="iconfont icon-fanhui" slot="left" />
    </van-nav-bar>
    <!--内容-->
    <div class="content">
      <div class="currency">
        <p>
          <span>{{currency.currencyCode}}</span>
        </p>
        <ul class="currencyDetails">
          <li class="title">
            <span>可用</span>
            <span>冻结</span>
            <span>折合(CNY)</span>
          </li>
          <li>
            <span>{{currency.amount}}</span>
            <span>{{currency.frozenAmount}}</span>
            <span>{{currency.cny}}</span>
          </li>
        </ul>
      </div>
      <div class="acc_tab">
        <ul>
          <li v-for="(item,index) of accTabs" :key="index" @click="tiaoZhuan(item.txt)">
            <img :src="item.url" alt="">
            <p>{{item.txt}}</p>
          </li>
        </ul>
      </div>
      <div class="financial">
        <h6>财务记录</h6>
        <ul>
          <li>
            <span>数量</span>
            <span>类型</span>
            <span>时间</span>
          </li>
          <li v-for="item in financial">
            <span>{{item.number}}</span>
            <span>{{item.type == 0? '充值': '提现'}}</span>
            <span>{{item.updateTime}}</span>
          </li>
          <li v-if="financialTotal === 0" class="null"> 暂无记录 </li>
        </ul>
        <van-pagination
          v-model="currentPage"
          v-if="financialTotal > pages.rows"
          :total-items="financialTotal"
          :items-per-page="pages.rows"
          @change="change()"
        />
      </div>
    </div>
  </section>
</template>

<script>
  export default {
    name: "curDetails",
    data() {
      return {
        currency: {},
        currentPage: 1,
        accTabs: [
          {url: require('../../homePage/images/buy.png'), txt: '充币'},
          {url: require('../../homePage/images/sale.png'), txt: '提币'},
          {url: require('../../homePage/images/account.png'), txt: '划转'},
        ],
        financial: [], //财务记录
        financialTotal: 0, //总条数
        pages: {
          start: 0,
          rows: 10,
          type: 'Z',
          currencyCode: '',
        }
      }
    },
    created() {
      this.currency = JSON.parse(this.$route.query.cur);
    },
    mounted() {
      this.getFinancial();
    },
    methods: {
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
      getFinancial() { //获取财务记录
        this.pages.start =  this.pages.rows * (this.currentPage-1);
        this.pages.currencyCode = this.currency.currencyCode;
        this.$axios({
          url: '/api/wallet/financialRecord',
          method: 'post',
          data: this.pages
        }).then(res => {
          this.financial = res.data.page.data.reverse()
          
          this.financialTotal = res.data.page.total
        })
      },
      change() { //查询地址分页
        this.getFinancial()
      },
    }
  }
</script>

<style scoped lang="less">
  .content {
    padding: 0.5rem;
    .currency {
      font-size: 0.8rem;
      border-top: 1px solid #ddd;
      border-bottom: 1px solid #ddd;
      p {
        line-height: 2.2rem;
        padding: 0 1rem;
        font-size: 1.1rem;
        font-weight: bold;
      }
      .currencyDetails {
        padding: 0 1rem;
        li {
          margin: 1rem 0;
          span {
            display: inline-block;
            width: calc((100% - 0.7rem) /3);
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
    .financial {
      margin: 0.5rem 0;
      h6 {
        font-size: 1.4rem;
        border-bottom: 1px solid #ddd;
        line-height: 2.4rem;
      }
      ul {
        li {
          height: 3rem;
          line-height: 3rem;
          span {
            display: inline-block;
            width: 29%;
            text-align: center;
            font-size: 1.1rem;
          }
          span:last-child {
            width: 39%;
          }
        }
        .null {
          text-align: center;
          height: 10rem;
          line-height: 10rem;
        }
      }
    }
    .acc_tab {
      padding: 0 1.2rem;
      background: #fff;
      ul {
        display: flex;
        justify-content: space-between;
        li {
          padding: 1.4rem 0;
          flex: 1;
          text-align: center;
          img {
            max-width: 100%;
            height: 2.1rem;
            vertical-align: bottom;
          }
          p {
            font-size: 1rem;
            padding-top: 0.4rem;
            color: #787878;
          }
        }
      }
    }
  }
</style>
