<template>
    <div>
      <div class="head">
        <i class="iconfont icon-fanhui" @click="$back"></i>
        <span>帮助中心</span>
      </div>
      <div class="dataInfo" v-if="listInfos.length != 0">
        <ul class="listInfo">
          <!--<van-list-->
            <!--v-model="loading"-->
            <!--:finished="finished"-->
            <!--:finished-text="noMore"-->
            <!--@load="onLoad"-->
          <!--&gt;-->
            <!--<van-cell-->
              <!--v-for="(item,index) in listInfos"-->
              <!--:key="index"-->
              <!--:title="item.title"-->
              <!--@click="detail(item.id)"-->
            <!--/>-->
          <!--</van-list>-->
          <router-link :to="{path: '/helpDetail',query: {msgId: item.id}}" tag="li" v-for="(item,index) of listInfos" :key="index"><span>{{item.title}}</span></router-link>

        </ul>
        <p class="more" v-if="listInfos.length != total" @click="loadMord">加载更多~</p>
        <p class="noMore" v-else>没有更多了~</p>
      </div>
      <div class="noData" v-else>
        <p>暂无数据</p>
      </div>
    </div>
</template>

<script>
    export default {
      name: "helpCenter",
      data() {
        return {
          reqParams: {
            start: 0,
            rows: 10,
          },
          total: '',
          listInfos: [],
          loading: false,
          finished: false,
        }
      },
      mounted() {
        this.helpLists();
      },
      methods: {
        helpLists() {
          this.$axios.post('/api/user/help/center',{start: this.reqParams.start,rows: this.reqParams.rows}).then(res => {
            if(res.data.errorCode == 0) {
              if(res.data.page.data.length !=0) {
                this.listInfos = [...this.listInfos,...res.data.page.data];
                this.total = res.data.page.total;
                // 加载状态结束
                // this.loading = false;
                //
                // if(this.listInfos.length == res.data.page.total) {
                //   this.finished = true;
                // }
              }
            }
          });

        },
        loadMord() {
          if(this.listInfos.length != this.total ) {
            this.reqParams.start += this.reqParams.rows;
            this.helpLists();
          }
        }
        // onLoad() {
        //
        //   this.reqParams.start += this.reqParams.rows;
        //
        //   this.helpLists();
        // },
        // detail(id) {
        //   this.$router.push({path: '/helpDetail',query: {msgId: id}});
        // }
      }
    }
</script>

<style scoped>
  .head {
    height: 4rem;
    /*background: #0088f1;*/
    color: #000;
    text-align: center;
    padding: 0 1.2rem;
    line-height: 4rem;
    font-size: 1.5rem;
  }
  .head i {
    position: absolute;
    color: #000;
    left: 1.2rem;
    font-size: 1.4rem;
  }
  .more, .noMore {
    background: #fff;
    text-align: center;
    padding: 1rem;
  }
  .dataInfo {
    margin-bottom: 1rem;
  }
  .listInfo {
    background: #fff;
  }
  .listInfo li {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 1rem 1.2rem;
    border-bottom: 1px solid #f5f5f5;
  }
  .noData {
    background: #fff;
    padding: 2rem 0;
    text-align: center;
  }
</style>
