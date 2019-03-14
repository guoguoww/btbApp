<template>
  <div class="main">
      <header>
          <div class="h_left" @click="coin_show=true"><img src="../../../static/images/menu.png" alt=""></div>
          <div class="h_symbol"><span>{{symbol.name||'--/--'}}</span></div>
          <div class="h_right" @click="$router.push(`/market?symbol=${symbol.marketCode}`)"><img style="width: 1.8rem;" src="../../../static/images/market.png" alt=""></div>
      </header>
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
      <div class="trade">
          <ul class="t_nav">
              <li :class="params.direction==0?'active':''" @click="params.direction=0">做多</li>
              <li :class="params.direction==1?'active':''" @click="params.direction=1">做空</li>
          </ul>
          <div class="t_type">
              <select name="" id="" v-model="params.type">
                <option value="0">市价</option>
                <option value="1">限价</option>
            </select>
          </div>
          <div>
              
          </div>
          <div class="t_type_input">
              <input type="number" class='type1' disabled placeholder="以当前最优价格交易" v-if="params.type==0"> 
            <div class="van-stepper w50" v-else> 
                <button class="van-stepper__minus" @click="moneychange"></button>
                    <input type="number" class="van-stepper__input" v-model="params.price">
                <button class="van-stepper__plus" @click="moneychange(1)"></button>
            </div>
          </div>
          <div class="num_input">
              <input type="number" placeholder='购买数量' v-model="params.volume" @blur="limit">
              <span>{{symbol.exchangeCode||'--'}}</span>
          </div>
          <div class="t_base">
            <ul>
                <li :class="params.lever==i?'active':''"  v-for="i in base" :key="i" @click="params.lever=i">{{i}}倍</li>
            </ul>
            <span class="risk">风险率 : {{(riskRate*100).toFixed(2)}}%</span>
          </div>
          <div class="t_money">
              <span>可用: {{userinfo.amount?(+userinfo.amount).toFixed(2):'--'}} {{symbol.unitsCode}}</span><span>保证金: {{earnest_money}} {{symbol.unitsCode}}</span>
          </div>
      </div>
      <div class="buy">
          <div class="_stop">
            <div class="s_profit">
                <h5 class="s_title">止盈价格:</h5>
                <input type="number" placeholder="请输入止盈价格" v-model="params.profitPoints">
            </div>
            <div class="s_loss">
                <h5 class="s_title">止损价格:</h5>
                <input type="number" placeholder="请输入止损价格" v-model="params.lossPoints">
            </div>
        </div>
        <div class="buy_btn">
            <button :class="params.direction==0?'t0':'t1'" @click="buy">{{params.direction==0?'做多':'做空'}}</button>
        </div>
      </div>
      <!-- 订单列表 -->
      <div class="orders">
          <div class="o_title">
              <span>持仓订单</span>
              <i @click="$router.push('/conBills')">全部</i>
          </div>
          <ul class="o_list">
              <li v-for="i in orders" :key="i.id">
                  <div class="item_title">
                      <span :class="i.direction==0?'up':'down'">{{i.direction==0?'做多':'做空'}} <s>{{i.symbol}}</s></span>
                      <time>{{timeFormat(i.buyTime).t}} {{timeFormat(i.buyTime).d}}</time>
                  </div>
                  <div class="item_content">
                      <ul class="i_c_left">
                          <li>持仓价 <span>{{i.buyPoint}}</span></li>
                          <li>持仓量 <span>{{i.number}}</span></li>
                          <li>保证金 <span>{{i.orderPrice}}</span></li>
                          <li>现价 <span>{{i.price}}</span></li>
                          <li>手续费 <span>{{i.sellFee}}</span></li>
                          <li>利息 <span>--</span></li>
                          <li>止盈价格 <span>{{i.profitPoints||'--'}}</span></li>
                          <li>止损价格 <span>{{i.lossPoints||'--'}}</span></li>
                          <li>浮动盈亏 <span :class="float(i)<0?'down':'up'">{{float(i)}}</span></li>
                          <li>杠杆倍数 <span>{{i.lever}}</span></li>
                      </ul>
                      <div class="i_c_right">
                          <button @click="sell(i.id)">平仓</button>
                          <button @click="stopShow=true;stop.tradeId=i.id;stop.profitPoints=i.profitPoints;stop.lossPoints=i.lossPoints">修改</button>
                      </div>
                  </div>
              </li>
          </ul>
            <nodata v-show='nodata' />
      </div>
      <van-popup v-model="coin_show" position="left" class="popup">
        <div class="p_title">
            <p>合约</p>
        </div>
        <ul class="p_list">
            <li v-for="i in symbolList" :key="i.id" :class="i.id==symbol.id?'active':''" @click="symbol=i;coin_show=false;">
                <span class="p_symbol"><b>{{i.exchangeCode}}</b>/{{i.unitsCode}}</span>
                <span class="p_price" :class="i.diff<0?'down':'up'">{{i.price}}</span>
            </li>
        </ul>
      </van-popup>
        <!-- 下单后弹出 -->
        <!-- <van-popup v-model="odshow" class="buy-popup">
            <van-cell-group>
                <h3>订单详情</h3>
                <van-cell title="合约" :value="od.symbol" />
                <van-cell title="订单号" :value="od.serialNo"  />
                <van-cell title="建仓时间" :value="od.buyTime" />
                <van-cell title="平仓时间" v-if="od.sellTime" :value="od.sellTime" />
                <van-cell title="平仓类型" v-if="od.sellTime" :value="od.sellType=='0'?'手动平仓':od.sellType=='1'?'止盈平仓':'止损平仓'" />
                <van-cell title="手数" :value="od.hands"  />
                <van-cell title="保证金" :value="od.orderPrice" />
                <van-cell title="手续费" :value="od.fee" />
                <van-cell title="开仓价" :value="od.buyPoint"  />
                <van-cell title="止盈价" :value="od.profitPoints" />
                <van-cell title="止损价" :value="od.lossPoints"  />
                <van-cell title="方向" :value="od.direction=='1'?'买跌':'买涨'"  />
                <van-cell title="浮动盈亏" v-if="od.fdyk" :value="od.fdyk" />
                <van-cell title="盈亏" v-if="od.difMoney" :value="od.difMoney" />
                <div class="btn1">
                    <button class="btn2" @click='odshow=false'>确定</button>
                </div>
            </van-cell-group>
        </van-popup> -->
    <van-dialog v-model="stopShow" show-cancel-button :before-close="stopClose" title='修改止盈止损'>
        <div class="dialog">
          <p class="xj">
            <label>止盈</label>
            <input type="text" placeholder="修改止盈" v-model="stop.profitPoints">
          </p>
          <p class="xj">
            <label>止损</label>
            <input type="text" placeholder="修改止损" v-model="stop.lossPoints">
          </p>
        </div>
    </van-dialog>     

    </div>
</template>

<script type="">
import nodata from "@/components/common/nodata";
export default {
  name:"contract",
  data() {
    return {
        coin_show:false,//切换币种
        symbolList:[],//币种代码
        symbol:{},//当前选择币种
        riskRate:'',//风险率
        base:[],//杠杆倍数
        params:{
            contractId:null,//合约ID
            direction:0,//方向 0:买涨 1:买跌
            type:0,//类型：0-市价，1-限价
            price:null,//购买价格
            volume:null,//购买量
            lever:null,//杠杆比例（如20倍为20）
            lossPoints:null,//止损值
            profitPoints:null,//止盈值
        },
        userinfo:{},//余额
        marketTimer:null,//行情定时器
        orders:[],//订单
        stopShow:false,//修改止盈止损
        stop:{
            tradeId:'',
            lossPoints:null,
            profitPoints:null
        },
        nodata:false,//没有更多数据
    }
  },
  components: {
    nodata  
  },
  computed: {
      //保证金
    earnest_money(){
        let price
        this.params.type==0?price=this.symbol.price:price=this.params.price        
        let m=price*this.params.volume*(1/this.params.lever)
        return m?(m).toFixed(2):0
    },
    code(){
        return this.$route.query.symbol
    }

  },
  watch: {
      'symbol.exchangeCode'(){
        this.params.price=this.symbol.price
        this.getUserinfo()
      },
  },
  created () {
      this.getSymbol().then(()=>{this.setMarketTimer()})
      this.get_riskRate()
      this.query_order()
      this.getlever()
  },
  destroyed () {
    clearInterval(this.marketTimer)
    this.marketTimer=null
  },
  methods: {
    //获取倍数
   async getlever(){
       let data=await this.$axios.get('api/trade/contract/leverage')
      this.base=data.data.data.split(',')
       this.params.lever=this.base[0]
    },
    // 委托价格变化
    moneychange(type){
        if (type==1) {
            this.params.price=((+this.params.price)+0.0001).toFixed(4)
        }else{
            this.params.price=((+this.params.price)-0.0001).toFixed(4)
            this.params.price<0?this.params.price=0:this.params.price
        }
    },
    //浮动盈亏
    float(i){
        if (i.direction==0) {
            if ((i.price-i.buyPoint)*i.number) {
                return ((i.price-i.buyPoint)*i.number).toFixed(2)
            }
            return '--'
        }
        if ((i.buyPoint-i.price)*i.number) {
            return ((i.buyPoint-i.price)*i.number).toFixed(2)                    
        }
        return '--'
    },      
      //格式化时间
      timeFormat(time){
         let d= time.split(' ')[0].split('-')
         let t=time.split(' ')[1].split(':')
         return {
             d:`${d[1]}/${d[2]}`,
             t:`${t[0]}:${t[1]}`
         }
      },
      //获取币种
      getSymbol(){
         return this.$axios.post('api/trade/contract/query').then(res=>{
              if (!res.data.errorCode) {
                  this.symbolList=res.data.data
                  if (this.code) {
                      this.symbolList.forEach((v) => {
                          if (v.marketCode==this.code) {
                            this.symbol=v
                          }
                      });
                  }else{
                    this.symbol=this.symbolList[0]
                  }
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
            if (!this.symbolList.length) {
                return
            }
            let arr=[]
            this.symbolList.map(i=>{
                arr.push(i.marketCode)
            })
            let marketCode=[...new Set(arr)].join(',')
            this.$axios.get(`api/market/latest_market?codes=${marketCode}`).then(res=>{
                if (!res.data.errorCode) {
                    res.data.data.map((v,index)=>{
                        this.symbolList.map(i=>{
                            if (v.code==i.marketCode) {
                                Object.assign(i, v)
                                // this.symbolList[index] = Object.assign({}, i, v)
                            }
                            if (v.code==this.symbol.marketCode){
                                this.symbol = Object.assign({}, this.symbol, v)
                            }
                        })
                        if (!this.orders.length) {
                            return 
                        }
                        this.orders.map(i=>{
                            if (i.marketCode==v.code) {
                                 Object.assign(i, v)
                            }
                        })
                    })
                    //下单的价格等于当前现价
                    if (!this.params.price) {
                        this.params.price=this.symbol.price
                        this.getUserinfo()
                    }
                }
            }).catch(err=>{

            })
        },
      //查询风险值
      get_riskRate(){
          if(!this.$cookie.get('_auth')){
              return 
          }                    
          this.$axios.post('api/trade/contract/riskRate').then(res=>{
              if (!res.data.errorCode) {
                  this.riskRate=res.data.data
              }
          })
      },
      //获取用户信息
      getUserinfo(){
          if(!this.$cookie.get('_auth')){
              return 
          }
          this.$axios.post('api/assets/query?type=2').then(res=>{
              if (!res.data.errorCode) {
                  res.data.data.map(i=>{
                      if(i.currencyCode==this.symbol.unitsCode){
                          this.userinfo=i
                      }
                  })
              }
          })
      },
      //限额      
      limit(){
          if (this.params.volume>this.symbol.maxVolume) {
              this.$toast(`最大限额为 ${this.symbol.maxVolume} ${this.symbol.exchangeCode}`)
              this.params.volume=this.symbol.maxVolume
          }
          if (this.params.volume<0) {
              this.$toast(`最小限额为 ${this.symbol.minVolume} ${this.symbol.exchangeCode}`)
              this.params.volume=this.symbol.minVolume
          }
      },
      //下单
      buy(){
          if (!this.params.volume) {
              this.$toast('请输入购买数量')
              return
          }
        this.$dialog.confirm({
            title: '确认下单',
            message: `确认购买${this.symbol.exchangeCode}*${this.params.volume}`
        }).then(() => {
            // on confirm
            this.params.contractId=this.symbol.id
            this.$axios.post('api/trade/contract/place_order',this.params).then(res=>{
                if (!res.data.errorCode) {
                    this.getUserinfo()
                    this.query_order()
                    this.$toast('下单成功!')
                    setTimeout(() => {
                        this.get_riskRate()                        
                    }, 1000);
                }
            })
        }).catch(() => {
        // on cancel
        });
      },
      //查询持仓订单
      query_order(){
          if(!this.$cookie.get('_auth')){
              return 
          }          
          this.$axios.post('api/trade/contract/trade_query?rows=3&status=0').then(res=>{
              if (!res.data.errorCode) {
                //   console.log(res.data.page.data);
                  this.orders=res.data.page.data
                  if (!this.orders.length) {
                      this.nodata=true
                  }else{
                      this.nodata=false
                  }
              }
          })
      },
      //平仓
      sell(id){
          this.$dialog.confirm({
            title: '确认平仓',
            message: '是否确认手动平仓'
        }).then(() => {
        // on confirm
        this.$axios.post(`api/trade/contract/do_sell?tradeId=${id}`).then(res=>{
              if (!res.data.errorCode) {
                  this.$toast('平仓成功')
                  this.query_order()
                  setTimeout(() => {
                        this.get_riskRate()                        
                    }, 1000);
              }
          })
        }).catch(() => {
        // on cancel
        });
          
      },
    //确认修改止盈止损
    stopClose(action,done){
        if (action === 'confirm'){
            this.updateStop(done).finally(()=>{done()})
        }else{
            // this.stop.stopProfit=null
            // this.stop.stopLoss=null
            done()
        }
    },
    //修改止盈止损
    updateStop(done){
      if (!this.stop.profitPoints&&!this.stop.lossPoints) {
        this.$toast('请至少修改一个')
        done()
        return
      }
     return this.$axios.post('api/trade/contract/setUpStop',this.stop).then(res=>{
        if (!res.data.errorCode) {
          this.query_order();
          this.$toast('修改成功!')
        }
      })
    },    
  },
}
</script>

<style scoped lang="less">
.dialog{
    padding: 1rem 1.5rem;
    .xj{
        margin-top: .5rem;
        display: flex;
        align-items: center;
        font-size: .8rem;
        color: dimgrey;
        label{
            margin-right: .5rem;
        }
        input{
            border: 1px solid #ddd;
            padding: .2rem .4rem;
            border-radius: .2rem;
        }
    }
}
select{
        background-color: transparent;
    border: 0;
    outline: none;
    color: #888f94;
}
.main{
    background-color: #fff;
    padding-bottom: 60px;
}
header{
    display: flex;
    justify-content: space-between;
    padding: 1.3rem 1rem;
    border-bottom: 1px solid #efefef;
    img{
        width: 1.5rem;
    }
}
.market{
    display: flex;
    padding: 1rem 1rem 1.5rem;
    border-bottom: 1rem solid #efefef;
    justify-content: space-between;
    height: 4rem;
    font-size: 1.1rem;
    .m_left,.m_right{
        display: flex;
        flex-direction: column;
        justify-content: space-between;
    }
    .m_left{
        em{
            margin-left: .8rem;
            color: #9c9e9c;
        }
    }
    .m_right{
        color: #4a69b5;
        i{
            margin-left: .5rem;
        }
    }
    .m_price{
        font-size: 1.5rem;
    }
}
.trade{
    padding: 0.7rem 1rem;
    border-bottom:1px solid #efefef;
    .t_nav{
        display: flex;
        justify-content: space-between;
        // padding: .7rem 1rem;
        li{
            flex: 0 0 45%;
            // padding: .5rem 0;
            line-height: 2.5rem;
            text-align: center;
            position: relative;
            background-color: #f7f7fb;
            color: #8c9fad;
            font-weight: 600;
            &:first-child::before{
                content: '';
                position: absolute;
                display: block;
                width: 0;
                height: 0;
                border-top: 2.5rem solid #f7f7fb;
                border-right: 1rem solid transparent;
                right: -1rem;
            }
            &:last-child{
                background-color: #f7f7fb;
                &::before{
                    content: '';
                    position: absolute;
                    display: block;
                    width: 0;
                    height: 0;
                    border-bottom: 2.5rem solid #f7f7fb;
                    border-left: 1rem solid transparent;
                    left: -1rem;
                }
            }
            &.active:first-child{
                background-color: #e76d42;
                color: #fff;
                &::before{
                    border-top: 2.5rem solid #e76d42;
                }
            }
            &.active:last-child{
                background-color: #00c384;
                color: #fff;
                &::before{
                    border-bottom: 2.5rem solid #00c384;
                }
            }

        }
    }
    .t_type{
        margin-top: .7rem;
    }
    .t_type_input{
        margin: 0.7rem 0;
        .type1{
            padding: .7rem;
            box-sizing: border-box;
            width: 50%;
        }
        .van-stepper__input{
            width: 25%;
        }
    }
    .price_input,.num_input{
        border: 1px solid #e7e7e7;
        padding: .7rem;
        display: flex;
        justify-content: space-between;
    }
    .price_input{
        margin: .7rem 0;
        span{
            color: #4a69b6;
        }
    }
    .t_base{
        display: flex;
        justify-content: space-between;
        margin: .7rem 0;
        align-items: center;
        font-size: 1rem;
        >ul{
            display: flex;
            justify-content: space-between;
            flex:0 0 70%;
            flex-wrap: wrap;
            li{
                padding: .4rem 1rem;
                font-size: 1rem;
                border:1px solid #e7e7e7;
                border-radius: 2px;
                margin-bottom: 0.1rem;
                &.active{
                    border-color: #4a69b5;
                    color: #4a69b5;
                }
            }
        }
        .risk{
            color: #9c9e9c;
        }
    }
    .t_money{
        display: flex;
        justify-content: space-between;
        color: #888f94;
    }
}
.buy{
    padding: 1rem;
    border-bottom:.5rem solid #efefef;
    ._stop{
        display: flex;
        justify-content: space-between;
        margin-bottom: 1rem;
        >div{
            flex:0 0 45%;
        }
        .s_title{
            font-size: 1.25rem;
            margin-bottom: 2rem;
        }
        input{
            font-size: 1rem;
            border-bottom: 1px solid #c6c7ce;
            padding-bottom: .2rem;
            &::-webkit-input-placeholder {
             color: #c6c7ce   
            }
        }
    }
    .buy_btn{
        button{
            width: 100%;
            padding: 1rem 0;
            color: #fff;
            border-radius: 2px;
            &.t0{
                background-color: #e76d42;
            }
             &.t1{
                background-color: #00c384;
            }
        }
    }
}
.orders{
    .o_title{
        display: flex;
        justify-content: space-between;
        align-items: center;
        padding: .8rem 1rem;
        border-bottom: 1px solid #efefef;
        span{
            font-size: 1.35rem;
            font-weight: 700;
        }
        i{
            color: #8c9fad;
            font-size: 1rem;
        }
    }
    .o_list{
        >li{
            padding: 1rem;
            border-bottom: 1px solid #efefef;
            color: #39556b;
            .item_title{
                display: flex;
                justify-content: space-between;
                align-items: center;
                margin-bottom: 1rem;
                >span{
                    font-size: 1.3rem;
                    font-weight: 600;
                    &.up{
                        color: #f7595a;
                    }
                    &.down{
                        color: #00c384;
                    }
                    s{
                        font-weight: 600;
                        font-size: 1.2rem;
                        text-decoration: none;
                        color: #39556b;
                    }
                }
                time{
                    font-size: 1.1rem;
                    font-weight: 600;
                }
            }
            .item_content{
                display: flex;
                .i_c_left{
                    display: flex;
                    flex-wrap: wrap;
                    flex: 0 0 70%;
                    font-size: 1rem;
                    border-right: 1px solid #efefef;
                    >li{
                        margin-bottom: 0.6rem;
                        flex: 0 0 50%;
                        span{
                            margin-left: .3rem;
                        }
                    }
                }
                .i_c_right{
                    display: flex;
                    flex-direction: column;
                    flex: 0 0 30%;
                    justify-content: space-between;
                    button{
                        flex: 1;
                        background-color: transparent;
                        font-size: 1.3rem;
                        font-weight: 600;
                        color: #4a69b5;
                        padding: 0;
                        margin: 0 .2rem;
                        &:first-child{
                            border-bottom: 1px solid #efefef;
                        }
                    }
                }
            }
        }
    }
}
//
.popup{
    height: 100%;
    width: 65%;
    .p_title{
        padding: 1rem;
        p{
            font-size: 1.3rem;
            font-weight: 700;
        }
    }
    .p_list{
        font-size: 1rem;
        li{
            display: flex;
            justify-content: space-between;
            padding: .8rem 1rem;
            border-bottom: 1px solid #efefef;
            &.active{
                background-color: #f7f7fb;
            }
            .p_symbol{
                color: #c5cfd5;
                b{
                    font-size: 1.28rem;
                    color: #1f3f59;
                }
            }
        }
    }
}
// 下单后弹出详情
// .btn2{
//         height: 30px;
//     width: 15rem;
//     border-radius: 7px;
//     background: #3283fa;
//     color: #fff;
//     font-weight: 500;
// }
// .buy-popup{
//     width: 70%;
//     border-radius: 7px;
//     padding: 1rem 0;
//     .van-cell{
//         padding: 5px 10px;
//         color: #828282;
//     }
//     .btn1{
//         width: 100%;
//         text-align: center;
//         margin-top: 1rem;
//     }
//     h3{
//         text-align: center;
//     }
// }

</style>
