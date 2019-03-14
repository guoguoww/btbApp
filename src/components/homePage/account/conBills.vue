<template>
  <div class="main">
    <van-nav-bar title="合约订单" left-text="返回" left-arrow @click-left="$back()">
        <van-icon class="iconfont icon-fanhui" slot="left" />
    </van-nav-bar>
      <!-- 订单列表 -->
      <van-tabs v-model="active">
        <van-tab v-for="(i,index) in tab" :key="index" :title="i">
            <ul class="o_list">
                <li v-for="i in orders" :key="i.id">
                    <div class="item_title">
                        <span :class="i.direction==0?'up':'down'">{{i.direction==0?'做多':'做空'}} <s>{{i.symbol}}</s></span>
                        <time>{{timeFormat(i.buyTime).t}} {{timeFormat(i.buyTime).d}}</time>
                    </div>
                    <div class="item_content">
                        <ul class="i_c_left">
                            <li>{{active==0?'持仓价':'委托价'}} <span>{{i.buyPoint}}</span></li>
                            <li>持仓量 <span>{{i.number}}</span></li>
                            <li>保证金 <span>{{i.orderPrice}}</span></li>
                            <li v-if="active!=1">现价 <span>{{i.price}}</span></li>
                            <li v-if="active==1">平仓价 <span>{{i.sellPoint||'--'}}</span></li>
                            <li>手续费 <span>{{i.sellFee}}</span></li>
                            <li>利息 <span>--</span></li>
                            <li>止盈价格 <span>{{i.profitPoints||'--'}}</span></li>
                            <li>止损价格 <span>{{i.lossPoints||'--'}}</span></li>
                            <li v-if="active==0">浮动盈亏 <span :class="float(i)<0?'down':'up'">{{float(i)}}</span></li>
                            <li v-if="active==1">盈亏 <span :class="i.difMoney<0?'down':i.difMoney>0?'up':''">{{i.difMoney?(+i.difMoney).toFixed(4):'--'}}</span></li>
                            <li v-show="i.status!=1">订单状态 <span>{{status[i.status]||'--'}}</span></li>
                            <li v-show="i.status==1">订单状态 <span>{{sellType[i.sellType]||'--'}}</span></li>
                          <li>杠杆倍数 <span>{{i.lever}}</span></li>
                        </ul>
                        <div class="i_c_right" v-if="active==0">
                            <button class="border_1" @click="sell(i.id)">平仓</button>
                            <button @click="stopShow=true;stop.tradeId=i.id;stop.profitPoints=i.profitPoints;stop.lossPoints=i.lossPoints">修改</button>
                        </div>
                        <div class="i_c_right" v-if="active==2">
                            <button @click="recall(i.id)">撤单</button>
                        </div>
                    </div>
                </li>
            </ul>

        </van-tab>
    </van-tabs>
    <div class="more" @click="loadmore" v-show='!nodata'>
        <span v-if="total==orders.length">没有更多数据</span>
        <span v-else>加载更多</span>
    </div>
    <nodata v-show='nodata' />
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
    <div  v-if="loading" class="m">
        <van-loading class="loading" color="white" />
        <div class="mask"></div>
    </div>      
  </div>
</template>

<script type="">
import nodata from "@/components/common/nodata";
export default {
  name:"",
  data() {
    return {
        marketTimer:null,//行情定时器
        orders:[],//订单
        stopShow:false,//修改止盈止损
        stop:{
            tradeId:'',
            lossPoints:null,
            profitPoints:null
        },
        tab:['持仓订单','历史订单','委托订单'],
        active:0,//tab栏当前选择
        params:{//订单参数
            start:0,
            rows:2,
            status:0
        },
        total:null,//订单总数
        status:['持仓', '平仓', '取消', '委托中'],
        sellType:['手动平仓','止盈平仓','止损平仓','强制平仓'],
        loading:false,
        nodata:false,

    }
  },
  components: {
    nodata  
  },
  created () {
      this.query_order().then(()=>this.setMarketTimer())

  },
  destroyed () {
      clearInterval(this.marketTimer)
  },
  watch: {
     active(){
        this.orders=[]
        this.query_order() 
     } 
  },
  methods: {
      //查询持仓订单
      query_order(){
          this.params.status=this.active
          this.active==2?this.params.status=3:null
        return   this.$axios.post('api/trade/contract/trade_query',this.params).then(res=>{
              if (!res.data.errorCode) {
                //   console.log(res.data.page.data);
                  this.orders=[...this.orders,...res.data.page.data]
                  this.total=res.data.page.total  
                  if (!this.orders.length) {
                      console.log(this.orders.length);
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
                  this.orders=[]
                  this.query_order()
              }
          })
        }).catch(() => {
        // on cancel
        });
          
      },
      //撤回委托
      recall(id){
          this.$dialog.confirm({
            title: '撤回委托',
            message: '是否撤回委托'
        }).then(() => {
        // on confirm
        this.$axios.post(`api/trade/contract/revoke?tradeIds=${id}`).then(res=>{
              if (!res.data.errorCode) {
                  this.$toast('撤回成功')
                  this.orders=[]
                  this.query_order()
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
          this.orders=[]
          this.query_order();
          this.$toast('修改成功!')
        }
      })
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
            if (!this.orders.length) {
                return 
            }
            let arr=[]
            this.orders.map(i=>{
                arr.push(i.marketCode)
            })
            let marketCode=[...new Set(arr)].join(',')
            this.$axios.get(`api/market/latest_market?codes=${marketCode}`).then(res=>{
                if (!res.data.errorCode) {
                    res.data.data.map((v,index)=>{
                        this.orders.map(i=>{
                            if (i.marketCode==v.code) {
                                //  Object.assign(i, v)
                                 this.$set(i,'price',v.price)
                            }
                        })
                    })

                }
            }).catch(err=>{

            })
        },
        //加载更多
        loadmore(){
            if (this.total==this.orders.length) {
                return
            }
            // this.mloading=true
            this.params.start+=this.params.rows
            this.query_order()
        },        

  },
}
</script>

<style scoped lang="less">
.main{
    padding-bottom:60px; 
    background-color: #fff;
}
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
                        &.border_1{
                            border-bottom: 1px solid #efefef;
                        }
                    }
                }
            }
        }
    }
.more {
    font-size: 1rem;
    text-align: center;
    padding-top: 0.5rem;
}
</style>
