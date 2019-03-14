<template>
    <div>
      <div class="head">
        <i class="iconfont icon-fanhui" @click="$back"></i>
        <span>公告</span>
      </div>
      <div v-if="listInfos.length != 0">
        <ul class="listInfo">
          <router-link :to="{path: '/announceDetail',query: {msgId: item.id}}" tag="li" v-for="(item, index) of listInfos" :key="index">
            <span>{{item.title}}</span><span>{{item.createTime}}</span>
          </router-link>
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
      name: "announceLists",
      data() {
          return {
            start: 0,
            rows: 10,
            total: '',
            listInfos: [],
          }
        },
        mounted() {
          this.announceList();
        },
        methods: {
          announceList() {
            this.$axios.post('/api/user/announce',{start: this.start,rows: this.rows}).then(res => {
              if(res.data.errorCode == 0) {
                if(res.data.page.data.length !=0) {

                  this.listInfos = [...this.listInfos, ...res.data.page.data];
                  this.total = res.data.page.total;

                }
              }
            })
          },
          loadMord() {
            if(this.listInfos.length != this.total ) {
              this.reqParams.start += this.reqParams.rows;
              this.helpLists();
            }
          }
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
  .listInfo li > span:first-child {
    width: 70%;
  }
  .listInfo li > span:last-child {
    width: 25%;
    text-align: center;
  }
  .more, .noMore {
    background: #fff;
    text-align: center;
    padding: 1rem;
  }
  .noData {
    background: #fff;
    padding: 2rem 0;
    text-align: center;
  }
</style>
