<template>
    <div class="">
        <div class="none">
            <img src="../../../static/images/none.png" alt="">
            <p>没有记录</p>
        </div>
  </div>
</template>

<script type="">
export default {
  name:"nodata",
  data() {
    return {

    }
  },
  created () {

  },
  methods: {

  },
}
</script>

<style scoped lang="less">
.none{
    padding: 3rem 1.5rem;
    text-align: center;
    font-size: 1rem;
    color: silver;
    >img{
        width: 50%;
    }
    >p{
        margin-top: 1rem;
    }
}
</style>
