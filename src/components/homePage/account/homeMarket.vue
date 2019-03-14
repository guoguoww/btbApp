<template>
    <div class="marketWrap">
      <div class="head">
        <i class="iconfont icon-fanhui" @click="$back"></i>
        <span>行情</span>
      </div>
      <ul>
        <li v-for="(item,index) of diffRatePrice" :key="index" @click="runTo(item.code)">
          <div><p><span class="weightCode">{{item.exchangeCode}}</span><span class="smallCode">/{{item.unitCode}}</span></p><p class="smallCode"><span>24H量</span><span> {{item.totalVol  | numTwo}}</span></p></div>
          <div><span class="weightCode">{{item.price | numTwo}}</span><span class="smallCode">￥{{item.symbolCny | numTwo}}</span></div>
          <div><span :class="[item.diffRate>0?'up':'down']">{{item.diffRate | numTwo}}%</span></div>
        </li>
      </ul>
    </div>
</template>

<script>
    export default {
      name: "homeMarket",
      data() {
        return {
          marketData: [{exchangeCode:'---',unitsCode:'---'},{exchangeCode:'---',unitsCode:'---'},{exchangeCode:'---',unitsCode:'---'}], //行情数据
          diffRatePrice: [{price:0,diffRate:0,price:0},{price:0,diffRate:0,price:0},{price:0,diffRate:0,price:0},], //行情数据涨跌率和价格
          _timeOut: null,
          reqCodes: [],
        }
      },
      created() {
        this.allMarket().then((res) => {
          this.latestMarket(res);
        });
      },
      mounted() {

        this._timeOut = setInterval(() => {
          this.latestMarket(this.reqCodes.join());
        }, 3000);

      },
      methods: {
        allMarket() {//所有行情
          return this.$axios.post('/api/market/all_market').then(res => {
            if(!res.data.errorCode) {
              if(res.data.data.length != 0) {
                this.marketData = res.data.data;
                let reqMarketCode = [];
                res.data.data.map(marketCodeVal => {
                  this.reqCodes.push(marketCodeVal.marketCode);
                  reqMarketCode.push(marketCodeVal.marketCode);
                })
                return reqMarketCode.join();
              }
            }
          })
        },
        latestMarket(codes) {//最新行情
          this.$axios({
            url: '/api/market/latest_market',
            method: 'get',
            params: {
              codes,
            }
          }).then(res => {
              if(!res.data.errorCode) {
                if(res.data.data.length!= 0) {

                  this.diffRatePrice = res.data.data;

                  res.data.data.forEach((val) => {

                    this.marketData.map(marketVal => {
                      if(marketVal.marketCode == val.code) {
                        this.$set(val,'exchangeCode',marketVal.exchangeCode);
                        this.$set(val,'unitCode',marketVal.unitsCode);
                      }
                    })
                  });
                }
              }
          })
        },
        //获取币种
        getSymbol(){//先获取合约中的交易币种，和当前的行情中的币种做比较，有的就跳转
          return this.$axios.post('api/trade/contract/query').then(res=>{
            if (!res.data.errorCode) {
              return res.data.data;
            }
          })
        },
        runTo(curMarketCode) {//跳转到对应的合约交易
          this.getSymbol().then(res => {
            res.map(val => {
              if(val.marketCode == curMarketCode) {
                this.$router.push(`/market?symbol=${curMarketCode}`);
              }
            })
          });
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
      },
      beforeDestroy() {
        clearInterval(this._timeOut);
        this._timeOut = null;
      },
    }
</script>

<style scoped>
  .head {
    height: 4rem;
    color: #0088f1;
    text-align: center;
    padding: 0 1.2rem;
    line-height: 4rem;
    font-size: 1rem;
    background: #fff;
    border-bottom: 1px solid #efefef;
  }
  .head i {
    position: absolute;
    color: #000;
    left: 1.1rem;
    font-size: 1.4rem;
  }
  .head span {
   font-size: 1.4rem;
  }
  .marketWrap {
    padding-bottom: 4rem;
  }
  .marketWrap > ul li {
    display: flex;
    align-items: center;
    border-bottom: 1px solid #e5e5e5;
    padding: 1rem 1.2rem;
    background: #fff;
  }

  .marketWrap > ul li > div {
    width: 40%;
    display: flex;
    flex-direction: column;
  }
  .marketWrap > ul li > div:first-child {

  }
  .marketWrap > ul li > div:last-child {
    width: 20%;
    text-align: center;
  }
  .marketWrap > ul li > div:last-child span {
    background: #05c19e ;
    padding: .6rem .8rem;
    color: #fff;
    border-radius: .2rem;
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
