<template>
    <div>
      <div class="head">
        <i class="iconfont icon-fanhui" @click="$back"></i>
        <span>详情</span>
      </div>
      <div class="content">
        <h3 v-html="title"></h3>
        <div v-html="context"></div>
      </div>
    </div>
</template>

<script>
    export default {
      name: "helpDetail",
      data() {
        return {
          context: '',
          title: '',
        }
      },
      mounted() {
        this.msgDetail();
      },
      methods: {
        msgDetail() {
          this.$axios.post('/api/user/msg/details',{msgId: this.$route.query.msgId}).then(res => {
            console.log(res)
            if(!res.data.errorCode) {
              this.title = res.data.data.title;
              this.context = res.data.data.content;
            }
          })
        }
      },

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
  .content {
    background: #fff;
    padding: 1rem;
  }
  .content h3 {
    text-align: center;
    padding-bottom: 1rem;
    font-weight: normal;
    font-size: 1.3rem;
  }
</style>
