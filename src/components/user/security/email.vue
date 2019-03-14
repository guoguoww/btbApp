<template>
  <section style="position: absolute; height: 100%; width: 100%; background: #fff">
    <!--头部-->
    <van-nav-bar title="邮箱认证" left-text="返回" left-arrow @click-left="$back()">
      <van-icon class="iconfont icon-fanhui" slot="left" />
    </van-nav-bar>
    <!--内容-->
    <div class="content">
      <ul>
        <li>
          <input placeholder="邮箱" type="email" v-model="email">
        </li>
        <li>
          <input maxlength="6" placeholder="验证码" type="text" v-model="emailValidCode">
          <button class="ver-but" @click="getCode()" :disabled='disabled'>
            {{codetext}}
          </button>
        </li>
      </ul>
      <div class="sign-but">
        <button @click="submit">认证</button>
      </div>
    </div>
  </section>
</template>

<script>
  export default {
    name: "email",
    data() {
      return {
        email: '', //邮箱
        emailValidCode: '', //邮箱验证码
        codetext: "获取验证码", //验证码按钮文字
        disabled: false, //验证码按钮状态
        timer: null, //验证码定时器
      }
    },
    methods: {
      getCode() { //获取邮箱验证码
        if (!this.email) {
          this.$toast('请输入邮箱');
          return;
        }
        this.$axios({
          url: "/api/email/validate_code",
          method: "post",
          params: {
            email: this.email,
            type: 'auth',
          }
        }).then(res=>{
          if (res.data.errorCode === 0) {
            this.$toast('发送成功');
            this.disabled = true;
            this.codetext = 60;
            this.timer = setInterval(()=>{
              if(this.codetext > 0 && this.codetext <= 60 > 0 && this.codetext <= 60){
                this.codetext--
              }else{
                this.disabled = false
                this.codetext = '获取验证码'
                clearInterval(this.timer)
                this.timer = null
              }
            },1000)
          }
        })
      },

      submit() {
        if (!this.email) {
          this.$toast('请输入邮箱');
          return;
        }
        if (!this.emailValidCode) {
          this.$toast('请输入验证码');
          return;
        }
        this.$axios({
          url: "/api/user/mail/auth",
          method: "post",
          data: {
            email: this.email,
            emailValidCode: this.emailValidCode,
          }
        }).then(res=> {
          if(!res.data.errorCode) {
            this.$toast('认证成功');
            this.$router.push('/security')
          }
        })
      },
    }
  }
</script>

<style scoped lang="less">
  .content {
    padding: 1.4rem 1rem 0;
    -webkit-box-sizing: border-box;
    box-sizing: border-box;
    ul {
      li {
        border-bottom: 0.333px solid #dcdcdc;
        padding: 2rem 0 0.1rem;
        position: relative;
        input {
          width: 60%;
          height: 2.7rem;
          padding-left: 1rem;
          outline: none;
          background-color: #fff;
        }
        > button {
          height: 1.5rem;
          outline: none;
          border: 0;
          width: 8rem;
          position: absolute;
          top: 2.7rem;
          right: 0.7rem;
          z-index: 1;
          color: #0088f1;
          background-color: #fff;
          border-left: 1px solid #dcdcdc;
        }
      }
    }
    .sign-but {
      padding-top: 4rem;
      button {
        width: 100%;
        height: 3rem;
        font-size: 1.12rem;
        background: #0088f1;
        outline: none;
        border-radius: 3px;
        color: #fff;
        border: 0;
        -webkit-box-shadow: 0 0.4rem 0.35rem #f3f3f3;
        box-shadow: 0 0.4rem 0.35rem #f3f3f3;
      }
    }
  }
</style>
