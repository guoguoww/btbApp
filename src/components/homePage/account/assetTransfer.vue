<template>
    <div class="changeBG">
      <van-nav-bar
        title="资产划转"
        left-arrow
        @click-left="back"
      >
        <van-icon class="iconfont icon-fanhui" slot="left" />
      </van-nav-bar>
      <div class="transContent">
        <ul>
          <li @click="hidType">
            <div>
              <em>币种选择</em>
              <span class="line"></span>
            </div>
            <p ref="selType">USDT</p>
            <i class="iconfont icon-xialasanjiao"></i>
          </li>
          <!--<li @click="hidMode">-->
          <li @click="hidChoose">
            <div>
              <em>从</em>
              <span class="line"></span>
            </div>
            <p ref="selModS">{{initFromType}}</p>
            <i class="iconfont icon-xialasanjiao"></i>
          </li>
          <li @click="hidMode">
            <div>
              <em>划转至</em>
              <span class="line"></span>
            </div>
            <p ref="selModEnd">{{intToType}}</p>
            <i class="iconfont icon-xialasanjiao"></i>
          </li>
          <li>
            <div>
              <em>数量</em>
              <span class="line"></span>
            </div>
            <input type="number" v-model="hzNum">
            <span class="all" @click="allAccount">全部</span>
          </li>
        </ul>
      </div>
      <div>
        <p class="transExp"><span>可转额度 <i class="iconfont icon-gantanhao"></i>{{hzNumx}}</span><span class="confirmOut" @click="confirmZC">确认转出</span></p>
      </div>
      <div>
        <ul class="detailTit">
          <li><span>方向</span><span>币种</span><span>时间</span><span>数量</span></li>
        </ul>
        <div>
          <van-list v-model="loading" :finished="finished" :immediate-check="immediateCheck"  @load="onLoad">
            <van-cell v-for="(item,index) in transferLists" :key="index">
              <ul class="transferInfos">
                <li>
                  <span>{{item.assetsType[0]=='0'?'C2C':item.assetsType[0]=='1'?'币币':item.assetsType[0]=='2'?'合约':item.assetsType[0]=='3'?'佣金':'无效'}} =>
                    {{item.assetsType.split(',')[1]=='0'?'C2C':item.assetsType.split(',')[1]=='1'?'币币':item.assetsType.split(',')[1]=='2'?'合约':item.assetsType.split(',')[1]=='3'?'佣金':'无效'}}</span>
                  <span>{{item.currencyCode}}</span>
                  <span>{{item.createTime }}</span>
                  <span>{{item.difMoney}}</span>
                </li>
              </ul>
            </van-cell>
          </van-list>
        </div>
        <div class="more" v-show="noData">

          <span>没有更多数据</span>

        </div>
      </div>
      <!--币种选择框-->
      <van-popup v-model="hideBg">
        <ul class="hideTypeInfo">
          <li v-for="(item, index) of $store.getters.currency.params" :key="index" @click="curTypTxt(item.code)">{{item.code}}</li>
        </ul>
      </van-popup>
      <!--划转选择框-->
      <van-popup v-model="hideChooseT">
        <ul class="hideTypeInfo">
          <li v-for="(item, index) of chooseInfos" :key="index" @click="curChooseTxt(item)">{{item.name}}</li>
        </ul>
      </van-popup>
      <!--划转方向框-->
      <van-popup v-model="hideModS">
        <ul class="hideModeInfo">
          <li v-for="(item, index) of modeInfos" :key="index" @click="curModTxt(item)">{{item.name}}</li>
        </ul>
      </van-popup>
    </div>
</template>

<script>
    export default {
      name: "assetTransfer",
      data(){
        return {
          loading: false,
          finished: false,
          startPage: -10, //起始页码
          rows: 10,
          immediateCheck: false,
          totalPageNum: 0,//数据总条数据
          hideBg: false, //币种模态
          hideChooseT: false, //账户选择模态
          hideModS: false, //账户转至模态
          chooseInfos: [{name: 'C2C',id: '0'},{name: '币币', id: '1'},{name: '合约',id: '2'}],
          modeInfos: [{name: 'C2C',id: '0'},{name: '币币', id: '1'},{name: '合约',id: '2'}],
          noData: true,
          initFromType:'',//初始化账户选择名称
          intToType: '',//初始化账户转至名称
          curIndex: '',//当前账户选择下标
          transferred: '', //账户转至下标
          hzNumx: '',//账户数量差
          hzNum:'',//划转数量
          transferLists: [],//资金划转列表数据
        }
      },
      mounted(){
        //获取动态路由带过来的参数
        this.initFromType = this.chooseInfos[this.$route.query.curType].name;
        this.curIndex = this.$route.query.curType;

        this.chooseInfos.forEach((val,index) => {
          if(val.id == this.curIndex){
            this.chooseInfos.splice(index,1);
          }
        })
        this.modeInfos.forEach((val,index) => {
          if(val.id == this.curIndex){
            this.modeInfos.splice(index,1);
          }
        })
        this.intToType = this.modeInfos[0].name;
        this.transferred = this.modeInfos[0].id;

        this.assetDetail();

        this.getTransferQuery();//获取进入资金划转页面是的初始列表数据
      },
      methods: {
        back(){//返回上一级
          this.$router.go(-1);
        },
        hidType(){
          this.hideBg = true;
        },
        hidChoose(){
          this.hideChooseT = true;
        },
        hidMode(){
          this.hideModS = true;
        },
        curTypTxt(txt){ //币种选择
          this.$refs.selType.innerHTML = txt;
          this.assetDetail();
          this.hzNum = '';
          this.hideBg = false;
        },
        curChooseTxt(txt) { //划转入口选择

          console.log(txt)


          this.$refs.selModS.innerHTML = txt.name;
          this.initFromType = txt.name;
          this.curIndex = txt.id;
          this.hideChooseT = false;
          this.chooseInfos = [{name: 'C2C',id: '0'},{name: '币币', id: '1'},{name: '合约',id: '2'}];
          this.modeInfos = [{name: 'C2C',id: '0'},{name: '币币', id: '1'},{name: '合约',id: '2'}];
          this.chooseInfos.forEach((val,index) => {
            if(val.id == this.curIndex){
              this.chooseInfos.splice(index,1);
            }
          })
          this.modeInfos.forEach((val,index) => {
            if(val.id == this.curIndex){
              this.modeInfos.splice(index,1);
            }
          })

          // 判断一下要选择的是否跟转到的币种一致，一致，则下面的换
          if(txt.name == this.intToType) {
            this.$refs.selModEnd.innerHTML = this.modeInfos[0].name;
            this.intToType = this.modeInfos[0].name;
            this.transferred = this.modeInfos[0].id;
          }

          this.assetDetail();
          this.hzNum = '';
          this.startPage = -10;
          this.transferLists = [];
          this.totalPageNum = 0;
          this.getTransferQuery();
        },
        curModTxt(txt){ //划转目的选择

          this.$refs.selModEnd.innerHTML = txt.name;
          this.intToType = txt.name;
          this.transferred = txt.id;
          this.hideModS = false;
          this.chooseInfos = [{name: 'C2C',id: '0'},{name: '币币', id: '1'},{name: '合约',id: '2'}];
          this.modeInfos = [{name: 'C2C',id: '0'},{name: '币币', id: '1'},{name: '合约',id: '2'}];
          this.chooseInfos.forEach((val,index) => {
            if(val.id == this.transferred){
              this.chooseInfos.splice(index,1);
            }
          })
          this.modeInfos.forEach((val,index) => {
            if(val.id == this.transferred){
              this.modeInfos.splice(index,1);
            }
          })

          if(txt.name == this.initFromType) {
            this.initFromType = this.chooseInfos[0].name;
            this.curIndex = this.chooseInfos[0].id;
            this.$refs.selModS.innerHTML = this.chooseInfos[0].name;
          }

          this.assetDetail();
          this.hzNum = '';
          this.startPage = -10;
          this.transferLists = [];
          this.totalPageNum = 0;
          this.getTransferQuery();

        },
        assetDetail(){//获取当前账户资产明细
          this.$axios({
            url: 'api/assets/query',
            method: 'post',
            headers: {"Content-Type": "application/json;charset=utf-8"},
            params: {
              type: this.curIndex,
            }
          })
            .then(res => {
              if(res.data.errorCode === 0){
                res.data.data.forEach((val, index) => {
                  if(val.currencyCode == this.$refs.selType.innerHTML){
                    this.hzNumx = Number(val.amount - val.frozenAmount).toFixed(2);
                  }
                })
              }else {
                this.$toast(res.data.errorMsg);
              }
            })
            .catch(err => {
              this.$toast(err);
            })
        },
        allAccount(){//获取全部余额，总资产减去冻结金
          this.hzNum = this.hzNumx;
        },
        confirmZC(){//确认转出
          // 判断输入的转出数量不能超出自有的最大数量
          if(this.hzNum){
            this.assetDetail();
          }

         if(this.hzNum == 0){
            this.$toast('请输入大于0的数量');
          }else {
            this.getTransfer();
          }
        },
        getTransfer(){//资金划转的接口
          // let targetAddress = this.intToType.charCodeAt(0) == 'C2C'.charCodeAt(0)? 0 :
          //                     this.intToType.charCodeAt(0) == '币币'.charCodeAt(0)? 1 :
          //                     this.intToType.charCodeAt(0) == '合约'.charCodeAt(0)? 2 :
          //                     this.intToType.charCodeAt(0) == '佣金'.charCodeAt(0)? 3 : -1;
          //
          // console.log(targetAddress);
          this.$axios({
            url: '/api/assets/transfer',
            method: 'post',
            data: {
              rawAddress: this.curIndex,
              targetAddress: this.transferred,
              exchangeCode: this.$refs.selType.innerHTML,
              amount: this.hzNum,
            }
          })
            .then(res => {
              if(res.data.errorCode === 0){
                this.$toast(res.data.errorMsg);
                this.hzNum = '';
                this.startPage = -10;
                this.transferLists = [];
                this.getTransferQuery();

                this.assetDetail();

              }else {
                this.$toast(res.data.errorMsg);
              }
            })
            .catch(err => {
              this.$toast(err.data.errorMsg);
            })
        },
        getTransferQuery(){//获取进入资金划转页面是的初始列表数据
          this.startPage += this.rows;
          this.$axios({
            url: '/api/assets/transfer/query',
            method: 'post',
            params: {
              start: this.startPage,
              rows: this.rows,
              type: this.curIndex,
            }
          })
            .then(res => {
              if(res.data.errorCode === 0){
                if(res.data.page.data.length !== 0){

                  this.loading = false;

                  this.transferLists = this.transferLists.concat(res.data.page.data);
                  this.totalPageNum = res.data.page.total;

                  if (this.transferLists.length >= this.totalPageNum) {
                    this.finished = true;
                    // this.$toast('没有更多数据');
                  }
                }else {
                  // this.$toast('列表数据暂时为空');
                }

              }else {
                this.$toast(res.data.errorMsg);
              }
            })
            .catch(err => {
              this.$toast(err.data.errorMsg);
            })
        },
        onLoad() {//上拉加载,下拉刷新
          setTimeout(() => {

            this.getTransferQuery();

          }, 500);
        },
      }
    }
</script>

<style scoped>
  .transContent {
    background: #fff;
    border: 1px solid transparent;
  }
  .transContent ul {
    padding: 0 1.2rem;
  }
  .transContent ul li {
    display: flex;
    height: 3.5rem;
    line-height: 3.5rem;
    border: 1px solid #bfbfbf;
    margin-top: 1.5rem;
    border-radius: .3rem;
    justify-content: space-between;
    font-size: 1.1rem;
  }
  .transContent ul li span.line {
    display: inline-block;
    height: 1.5rem;
    width: 1px;
    border-right: 1px solid #787878;
    vertical-align: middle;
  }
  .transContent ul li em {
    display: inline-block;
    text-align: center;
    width: 6rem;
    font-size: 1rem;
    height: 1.7rem;
  }
  .transContent ul li i {
    padding-right: .5rem;
  }
  .hideTypeInfo,.hideModeInfo {
    margin: auto;
    padding: 1rem 2rem;
  }
  .van-popup {
    width: 70%;
  }
  .hideTypeInfo li {
    line-height: 3.5rem;
    border-bottom: 1px solid #ddd;
    text-indent: 1rem;
    font-size: 1.1rem;
  }
  .hideTypeInfo li:last-child {
    border-bottom: none;
  }

  .hideModeInfo li {
    line-height: 3.5rem;
    border-bottom: 1px solid #ddd;
    text-indent: 1rem;
    font-size: 1.1rem;
  }
  .hideModeInfo li:last-child{
    border-bottom: none;
  }
  .all {
    margin-right: .5rem;
    color: #00a8ff;
    font-size: 1.1rem;
  }
  .transExp {
    padding-top: 1rem;
    display: flex;
    height: 2rem;
    line-height: 2rem;
    justify-content: space-around;
    color: #787878;
    font-size: 1rem;
  }
  .transExp i {
    vertical-align: middle;
    margin-right: .3rem;
  }
  .transExp .confirmOut {
    color: #00a8ff;
  }
  .detailHZ {
    padding: 0 1.2rem;
    line-height: 4rem;
    color: #787878;
    font-size: 1rem;
    display: flex;
    justify-content: space-between;
  }
  .detailHZ span:hover {
    cursor: pointer;
  }
  .detailHZ span.active {
    color: #00a8ff;
  }
  .detailTit, .transferInfos{
    padding: 0 1rem;
    font-size: 1rem;
  }
  .transferInfos{
    padding: 0;
  }
  .detailTit li {
    padding: 0 .5rem;
    display: flex;
    justify-content: space-between;
    border-bottom: 1px solid #a0a0a0;
    line-height: 2rem;
    align-items: center;
  }
  .transferInfos li {
    line-height: 1.5;
    padding: .5rem 0;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
  .detailTit li span, .transferInfos li span {
    flex: 1;
    text-align: left;
  }
  .detailTit li span:last-child, .transferInfos li span:last-child{
    text-align: right;
  }
  .more {
    font-size: 1rem;
    text-align: center;
    padding-top: .5rem;
  }
</style>
