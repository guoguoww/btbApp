<template>
    <div class="changeBG">
      <!--头部-->
      <van-nav-bar title="地址管理" left-text="返回" left-arrow @click-left="$back()">
        <van-icon class="iconfont icon-fanhui" slot="left" />
      </van-nav-bar>
      <!--绑定钱包内容-->
      <div class="address_con">
        <ul class="addressAdd_list">
          <li>
            <select v-model="currency" >
              <!--<option v-for="item in currencyList"  :value="item.name">{{item.name}}</option>-->
              <option v-for="item in $store.getters.currency.params"  :value="item.code">{{item.code}}</option>
            </select>
          </li>
          <li class="name">
            <p>钱包地址</p>
            <input type="text" name="" value="" v-model="subData.address" >
          </li>
          <li class="name">
            <p>备注</p>
            <input type="text" name="" value="" v-model="subData.remark" >
          </li>
        </ul>
        <div class="binding" @click="submitAddress()">
          <span>绑定</span>
        </div>
      </div>
      <!--钱包列表-->
      <ul class="address_list">
        <li class="address_list_title">
          <span>币种</span>
          <span>钱包地址</span>
          <span>备注</span>
          <span>操作</span>
        </li>
        <li v-for="item in addressList.data" v-if="addressList.data.length > 0">
          <span>{{item.currencyCode}}</span>
          <span>{{item.address}}</span>
          <span>{{!!item.remark? item.remark : '无'}}</span>
          <span>
            <b class="btns" @click="removeAddress(item.id)">解绑</b>
          </span>
        </li>
        <li v-if="addressList.data.length === 0" style="width: 100%;text-align: center;height: 5rem;line-height: 5rem;">
          暂未绑定地址...
        </li>
      </ul>
      <van-pagination
        v-if="addressList.total > pages.rows"
        v-model="currentPage"
        :total-items="addressList.total"
        :items-per-page="pages.rows"
        :show-page-size="3"
        @change="change()"
      />

    </div>
</template>

<script>
  export default {
    name: "walletAddress",
    data() {
      return {
        currency: 'USDT',
        // currencyList: [
        //   {
        //     name: 'USDT',
        //     id: 1,
        //   },
        //   {
        //     name: 'BTC',
        //     id: 2,
        //   },
        //   {
        //     name: 'ETH',
        //     id: 3,
        //   },
        // ],
        addressList: {
          data: [],
          total: 0
        },
        pages: {
          start: 0,
          rows: 5,
        },
        currentPage: 1,
        subData: {
          currencyCode: '', //币种
          address: '', //地址
          remark: '', //备注
        }
      }
    },
    created() {
      this.getAddressList()
    },
    methods: {
      getAddressList() { //查询地址列表
        this.pages.start =  this.pages.rows * (this.currentPage-1);
        this.$axios({
          url: '/api/wallet/queryaddress',
          method: "post",
          data: this.pages
        }).then(res => {
          if(res.data.errorCode === 0) {
            this.addressList.data = res.data.page.data;
            this.addressList.total =  res.data.page.total;
          }
        })
      },
      change() { //查询地址分页
        this.getAddressList()
      },
      submitAddress() { //绑定地址
        if(this.subData.address === '') {
          this.$toast('请填写钱包地址');
          return;
        }
        this.subData.currencyCode = this.currency
        this.$axios({
          url: '/api/wallet/saveaddress',
          method: "post",
          data: this.subData
        }).then(res => {
          if(res.data.errorCode === 0) {
            this.$toast('绑定成功');
            this.getAddressList();
          }
        })
      },
      removeAddress(id) { //解除绑定地址
        this.$axios({
          url: '/api/wallet/deladdress',
          method: "post",
          data: {
            id: id
          }
        }).then(res => {
          if(res.data.errorCode === 0) {
            this.$toast('解除成功');
            this.getAddressList();
          }
        })
      },
    }
  }
</script>

<style scoped>
  /*增加钱包地址*/
  .address_con{
    padding: 0 1rem;
    border-top: 2px solid #f0f0f0;
    border-bottom: 2px solid #f0f0f0;
  }
  .address_con .addressAdd_list li {
    margin: 1rem 0;
  }
  .address_con .addressAdd_list li select {
    width: 100%;
    height: 2.4rem;
    outline: 0;
    border: 1px solid #d9d9d9;
  }
  .address_con .addressAdd_list li select:focus {
    outline: 0;
  }
  .address_con .addressAdd_list li p {
    height: 3rem;
    line-height: 3rem;
    font-size: 1.2rem;
  }
  .address_con .addressAdd_list li input {
    border: 1px solid #d9d9d9;
    width: calc(100% - 0.5rem);
    height: 2.4rem;
    padding-left: 0.4rem;
  }
  .address_con .binding {
    width: 60%;
    height: 3rem;
    line-height: 3rem;
    background: #0088f1;
    text-align: center;
    color: #fff;
    font-size: 1.2rem;
    border: 1px solid #0088f1;
    margin: 1rem auto;
  }
/*钱包列表*/
  .address_list {
    padding: 0 1rem;
  }
  .address_list li {
    height: 3rem;
    line-height: 3rem;
    margin: 0.5rem 0;
  }
  .address_list li.address_list_title {
    background: #f4f4f4;
    border-bottom: 1px solid #a0a0a0;
  }
  .address_list li span {
    display: inline-block;
    width: 23%;
    height: 100%;
    text-align: center;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }
  .address_list li span .btns {
    display: inline-block;
    width: 70%;
    height: 2rem;
    line-height: 2rem;
    vertical-align: middle;
    background: #0088f1;
    border-radius: 10px;
    color: #fff;
    font-weight: normal;
  }
</style>
