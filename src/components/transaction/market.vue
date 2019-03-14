<template>
  <div class="main">
    <van-nav-bar :title="symbol.name" left-text="返回" left-arrow @click-left="$back()" class="market_title">
        <van-icon class="iconfont icon-fanhui" slot="left" />
    </van-nav-bar>
      <div class="market">
          <div class="m_left" :class="symbol.diff<0?'down':'up'">
              <span class="m_price">{{symbol.price||'----'}}</span>
              <span>{{symbol.diffRate?((+symbol.diffRate)*100).toFixed(2):'--'}}%<em>≈ {{symbol.symbolCny?(+symbol.symbolCny).toFixed((+symbol.precision)):'--'}} CNY</em></span>
          </div>
          <div class="m_right">
              <span>最高价格<i>{{symbol.high||'--'}}</i></span>
              <span>最低价格<i>{{symbol.low||'--'}}</i></span>
          </div>
      </div>
    <ul class="interval_tab">
        <li :class="{'active':interval_active===i.code }" v-for="(i,index) in interval" :key="index" @click="interval_active=i.code">
            {{i.name}}
        </li>
    </ul>
    <div class="charts">
        <div  v-if="!market.code" style="height:21rem;"></div>
        <trading-view v-if="market.code" :config="{line:'#efefef',bg:'#fff'}"  :code='code' :market='market' height='21rem' :interval='interval_active'></trading-view> 
    </div> 
    <div class="trade_list">
        <ul class="t_title"><li>买盘 数量({{symbol.exchangeCode}})</li><li>价格({{symbol.unitsCode}})</li><li>数量({{symbol.exchangeCode}}) 卖盘</li></ul>
        <div class="t_list">
            <ul class="t_buy">
                <li v-for="(i,index) in 5" :key="i"><span>{{index+1}}<i style="margin-left:1rem">{{symbol[`bv${i}`]}}</i></span><span class="down">{{symbol[`bp${i}`]}}</span></li>
            </ul>
            <ul class="t_sell">
                <li v-for="(i,index) in 5" :key="i"><span class="up">{{symbol[`sp${i}`]}}</span><span>{{symbol[`sv${i}`] }} <i style="margin-right:1rem;margin-left: 1rem;">{{index+1}}</i></span></li>
            </ul>
        </div>
    </div>
    <div class="trade_btn">
        <button @click="$router.push(`/contract?symbol=${code}`)">合约交易</button>
    </div>
  </div>
</template>

<script type="">
import tradingView from "../tradingView/index";
export default {
  name:"market",
  data() {
    return {
        interval:[//k线区间 
            {code:1,name:'分时'},
            {code:'1',name:'1分钟'},
            {code:'5',name:'5分钟'},
            // {code:'15',name:'15分钟'},
            {code:'30',name:'30分钟'},
            {code:'1h',name:'1小时'},
            {code:'1d',name:'日线'}],
        interval_active:'1',//k线默认选择
        symbolList:[],
        symbol:{},//当前行情列表
        market:{},//行情
        marketTimer:null,
    }
  },
  computed: {
    code(){
        return this.$route.query.symbol
    }  
  },
  components: {
    tradingView  
  },
  created () {
      this.getSymbol()
      this.setMarketTimer()
  },
destroyed () {
      clearInterval(this.marketTimer)
  },
  methods: {
        //获取币种
      getSymbol(){
         return this.$axios.post('api/trade/contract/query').then(res=>{
              if (!res.data.errorCode) {
                  this.symbolList=res.data.data
              }
          })
      },
      //获取行情定时
      setMarketTimer(){
          if (this.marketTimer) {
              clearInterval(this.marketTimer)
              this.marketTimer=null
          }
          this.getmarket()
          this.marketTimer=setInterval(() => {
              this.getmarket()
          }, 2000);
      },
      //获取行情
        getmarket(){//获取行情
            let marketCode=this.code
            this.$axios.get(`api/market/latest_market?codes=${marketCode}`).then(res=>{
                if (!res.data.errorCode) {
                    res.data.data.map((v,index)=>{
                        this.symbolList.map(i=>{
                            if (v.code==i.marketCode) {
                                Object.assign(i, v)
                                // this.symbolList[index] = Object.assign({}, i, v)
                            }
                            if (v.code==this.code){
                                this.market=v
                            }
                            if (i.marketCode==this.code) {
                                this.symbol = Object.assign({}, i, v)                                
                            }
                        })
                    })
                }
            }).catch(err=>{

            })
        },

  },
}
</script>

<style scoped lang="less">
@bg:#fff;//#131f30
@font-color:#888f94;//#6d87a8
@border-color:#efefef;//#081724
.main{
    padding-bottom: 80px;
    background-color: @border-color;
}
.down{
    color:#03ad8f;
}
.up{
    color:#d14b64;
}
.market{
    display: flex;
    padding: 1rem 1rem 1.5rem;
    // border-bottom: 1rem solid #efefef;
    justify-content: space-between;
    height: 4rem;
    font-size: 1.1rem;
    background-color: @bg;
    .m_left,.m_right{
        display: flex;
        flex-direction: column;
        justify-content: space-between;
    }
    .m_left{
        em{
            margin-left: .8rem;
        color: @font-color;            
        }
    }
    .m_right{
        color: @font-color;
        i{
            margin-left: .5rem;
            color: #4a69b5;//cfd3e9
        }
    }
    .m_price{
        font-size: 1.5rem;
        font-weight: 600;
    }
}
.interval_tab{
    display: flex;
    justify-content: space-between;
    padding: 0 1rem;
    background-color: @bg;
    color: @font-color;
    li{
        padding-bottom: .7rem;
        &.active{
            color: #1882d4;
            border-bottom: 2.5px solid #1882d4;
        }
    }
}
.charts{
    border-bottom: 0.7rem solid @border-color;
}
.trade_list{
    padding: 1rem;
    background: @bg;
    .t_title{
        display: flex;
        justify-content: space-between;
        font-size: 1rem;
        color: #3e536c;
        margin-bottom: .5rem;
    }
    .t_list{
        display: flex;
        font-size: 1rem;
        >ul{
            flex: 0 0 50%;
            padding: 0 .3rem;
            li{
                display: flex;
                justify-content: space-between;
                margin: .5rem 0;
            }
        }
        .t_buy{

        }
        .t_sell{
        }
    }
}
.trade_btn{
    padding: 1.5rem 1rem;
    background: #fff;//172841
    position: fixed;
    width: 100%;
    bottom: 0;
    box-sizing: border-box;
    button{
        background-color: #d04a63;
        border-radius: .3rem;
        color: #fff;
        line-height: 2rem;
        padding: .5rem;
        font-size: 1.05rem;
        text-align: center;
        outline: none;
        display: block;
        width: 100%;
    }
}
</style>
