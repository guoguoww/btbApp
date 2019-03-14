<template>
    <!-- <div class="changeBG">
        <div class="sign-head" style="pointer-events: auto;">
          <router-link to="/"><span>取消</span></router-link>
          <router-link v-if="patternLoginShow" class="fr" to="/patternLogin"><span>手势登录</span></router-link>
        </div>
        <div class="sign-logo" style="pointer-events: auto;">
          <img src="../../../static/images/GHE.png">
        </div>
        <div class="sign-li">
            <ul>
                <li>
                    <input placeholder="请输入手机号" type="text" v-model="user.mobile">
                    <i class="iconfont icon-ai54" v-if="user.mobile" @click="user.mobile=''"></i>
                </li>
                <li>
                    <input placeholder="请输入密码" :type="pwdshow?'text':'password'" v-model="user.password">
                    <i class="iconfont icon-kaiicon" v-if="pwdshow" @click="pwdshow=!pwdshow"></i>
                    <i class="iconfont icon-guanicon" v-else @click="pwdshow=!pwdshow"></i>
                </li>
            </ul>
            <div class="sign-but">
                <button @click="login">登录</button>
            </div>
            <div class="sign-reg">
                <dl>
                    <dt @click="$router.push('/register')">
                        <em>注册</em>
                    </dt>
                    <dd @click="$router.push('/forget')">
                        <em>忘记密码</em>
                    </dd>
                </dl>
            </div>

            <div class=""></div>
        </div> -->


    <div class="login" id="login">
      <div class="user-top" style="    padding: 0 20px;">
        <a @click="$router.push('/')" class="fl" style="color: #c5cfd5;">取消</a>
      </div>
      <dl class="user-title">
          <!-- https://m.huobi.co/image/logo.4f36ed3.svg -->
        <dt><img style="width: 200px;" src="../../../static/images/GHE.png" /></dt>
        <dd>登录BCT</dd>
      </dl>
      <div class="user-form">
        <div class="user-input-group">
          <input placeholder="邮箱或手机号码" v-model="user.mobile"/>
        </div>
        <div class="user-input-group">
          <input placeholder="密码" type="password" v-model="user.password" />
        </div>
        <div id="captcha_wrap"></div>
        <div class="user-submit-group">
          <button class="user-submit" @click="login">
            <span>登录</span ><img
              src="data:image/svg+xml;base64,PHN2ZyB2ZXJzaW9uPSIxLjEiIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyIgeG1sbnM6eGxpbms9Imh0dHA6Ly93d3cudzMub3JnLzE5OTkveGxpbmsiICB3aWR0aD0iMjAwcHgiIGhlaWdodD0iMjAwcHgiICB2aWV3Qm94PSIwIDAgMjAwIDIwMCIKIHhtbDpzcGFjZT0icHJlc2VydmUiPgo8c3R5bGUgdHlwZT0idGV4dC9jc3MiPgoubGVmdHtmaWxsOnVybCgjbGVmdCk7fQoucmlnaHR7ZmlsbDp1cmwoI3JpZ2h0KTt9Ci50b3B7ZmlsbDojN0E5OEY3O30KQGtleWZyYW1lcyBsb2FkewowJXt0cmFuc2Zvcm06cm90YXRlKDApfQoxMDAle3RyYW5zZm9ybTpyb3RhdGUoLTM2MGRlZyl9Cn0KI2xvYWR7YW5pbWF0aW9uOmxvYWQgMXMgIGxpbmVhciBpbmZpbml0ZTsgdHJhbnNmb3JtLW9yaWdpbjpjZW50ZXIgY2VudGVyOyB9Cjwvc3R5bGU+CjxnIGlkPSJsb2FkIj4KPGxpbmVhckdyYWRpZW50IGlkPSJyaWdodCIgZ3JhZGllbnRVbml0cz0idXNlclNwYWNlT25Vc2UiIHgxPSIxNTAiIHkxPSIyMCIgeDI9IjE1MCIgeTI9IjE4MCI+CjxzdG9wICBvZmZzZXQ9IjAiIHN0eWxlPSJzdG9wLWNvbG9yOiM3QTk4RjciLz4KPHN0b3AgIG9mZnNldD0iMSIgc3R5bGU9InN0b3AtY29sb3I6IzdBOThGNztzdG9wLW9wYWNpdHk6MC41Ii8+PCEtLeapmeWIsOa1heapmea4kOWPmC0tPgo8L2xpbmVhckdyYWRpZW50Pgo8cGF0aCBjbGFzcz0icmlnaHQiIGQ9Ik0xMDAsMHYyMGM0NC4xLDAsODAsMzUuOSw4MCw4MGMwLDQ0LjEtMzUuOSw4MC04MCw4MHYyMGM1NS4yLDAsMTAwLTQ0LjgsMTAwLTEwMFMxNTUuMiwwLDEwMCwweiIvPjwhLS3lj7PljYrlnIbnjq8tLT4KPGxpbmVhckdyYWRpZW50IGlkPSJsZWZ0IiBncmFkaWVudFVuaXRzPSJ1c2VyU3BhY2VPblVzZSIgeDE9IjUwIiB5MT0iMCIgeDI9IjUwIiB5Mj0iMTgwIj4KPHN0b3AgIG9mZnNldD0iMCIgc3R5bGU9InN0b3AtY29sb3I6IzdBOThGNztzdG9wLW9wYWNpdHk6MCIvPgo8c3RvcCAgb2Zmc2V0PSIxIiBzdHlsZT0ic3RvcC1jb2xvcjojN0E5OEY3O3N0b3Atb3BhY2l0eTowLjUiLz48IS0t5rWF5qmZ5Yiw55m96Imy5riQ5Y+YLS0+CjwvbGluZWFyR3JhZGllbnQ+CjxwYXRoIGNsYXNzPSJsZWZ0IiBkPSJNMjAsMTAwYzAtNDQuMSwzNS45LTgwLDgwLTgwVjBDNDQuOCwwLDAsNDQuOCwwLDEwMHM0NC44LDEwMCwxMDAsMTAwdi0yMEM1NS45LDE4MCwyMCwxNDQuMSwyMCwxMDB6Ii8+PCEtLeW3puWNiuWchueOry0tPgo8Y2lyY2xlIGNsYXNzPSJ0b3AiIGN4PSIxMDAiIGN5PSIxMCIgcj0iMTAiLz4KPC9nPgo8L3N2Zz4K"
              class="loading"
            />
          </button>
          <div class="user-top">
            <a @click="$router.push('/forget')" class="fl">忘记密码</a>
        </div>
        </div>
      </div>
      <p class="snm">还没有BCT账号? <span  @click="$router.push('/register')">注册</span></p>
    </div>

</template>

<script type="">
export default {
    name: "login",
    data() {
        return {
            user:{
                mobile:'',
                password:'',
                terminal:'1',//设备类型
                type: 0, //登录类型
            },
            pwdshow:false,
            patternLoginShow: false, //是否可以手势登录
        };
    },
    created() {
      if(localStorage.getItem('userMob')) {
        this.patternLoginShow = true;
      }
    },
    methods: {
        login(){
            if (!this.user.mobile) {
                this.$toast('请输入手机号码')
                return
            }
            if (!this.user.password) {
                this.$toast('请输入密码')
                return
            }
            this.$axios.post('api/user/login',this.user).then(res=>{
                if (!res.data.errorCode) {
                    this.$cookie.set('_auth',res.data.data.cookie)
                    localStorage.setItem('eyeState','false');
                    localStorage.setItem('eyeState1','false');
                    localStorage.setItem('eyeState2','false');
                    this.$toast('登录成功')
                    localStorage.setItem('loginType', this.user.type) //登录方式
                    this.$router.replace(this.$route.query.redirect||'/')
                }
            }).catch(err=>{

            })
        }
    }
};
</script>

<style scoped lang="less">
// div {
//   .sign-head {
//     height: 9.1rem;
//     background: url(../../../static/images/sign-head.png);
//     background-size: 100% 100%;
//     position: relative;
//     span {
//       display: inline-block;
//       padding: 1.4rem;
//       font-size: 1.2rem;
//       color: #fff;
//     }
//   }
//   .sign-logo {
//     padding: 3.2rem 0 2rem;
//     text-align: center;
//     img {
//       height: 2.8rem;
//     }
//   }
//     .sign-li {
//         /* float: left; */
//         // width: 50%;
//         padding: 0 3rem;
//         -webkit-box-sizing: border-box;
//         box-sizing: border-box;
//         li {
//             border-bottom: 0.333px solid #dcdcdc;
//             padding: 2.01rem 0 0.1rem;
//             position: relative;
//             input {
//                 width: 60%;
//                 height: 2.7rem;
//                 // font-size: 35px;
//                 padding-left: 1rem;
//                 outline: none;
//                 background-color: #fff;
//             }
//             > i {
//                 font-size: 1.5rem;
//                 color: #e3e3e3;
//                 position: absolute;
//                 top: 3rem;
//                 right: 1rem;
//             }
//         }
//         .sign-but {
//             padding-top: 4.5rem;
//             button {
//                 width: 100%;
//                 height: 3rem;
//                 font-size: 1.12rem;
//                 background: #0088f1;
//                 outline: none;
//                 border-radius: 3px;
//                 color: #fff;
//                 border: 0;
//                 -webkit-box-shadow: 0 0.4rem 0.35rem #f3f3f3;
//                 box-shadow: 0 0.4rem 0.35rem #f3f3f3;
//             }
//         }
//         .sign-reg {
//             padding-top: 1.5rem;
//             color: #b4b4b4;
//             dl {
//                 display: flex;
//                 dt {
//                     flex: 1;
//                 }
//                 em {
//                     color: #0088f1;
//                 }
//             }
//         }
//     }
// }
.snm{
    // position: fixed;
    margin-top: 10rem;
    margin-left: 2rem;
    color: #c5cfd5;
    span{
        color: #1882d4;
    }
}
@b:0.5;
.login{
    background-color: #fff;
}
.user-top {
    height: 110px*@b;
    line-height: 110px*@b;
    padding: 0 60px*@b;
}
.user-title {
    padding: 80px*@b 60px*@b 0;
    height: 184px*@b;
}
.user-title dt {
    height: 80px*@b;
    font-size: 60px*@b;
}
.user-title dd {
    color: #c5cfd5;
    font-size: 32px*@b;
}
.user-input-group {
    height: 52px*@b;
    // padding: 60px*@b 60px*@b 0;
    padding-top: 30/12rem;
    text-align: center;

}
.user-input-group input {
    border: 0 none;
    border-bottom: 1px solid #c5cfd5;
    width: 84.5%;
    height: 32px*@b;
    font-size: 32px*@b;
    color: #1f3f59;
    padding-bottom: 20px*@b;
}
#captcha_wrap {
    padding-top: 60px*@b;
}
.user-submit-group {
    height: 44px;
    padding-top: 30/12rem;
    text-align: center;
}
.user-submit-group button {
    width: 84.5%;
    height: 88px*@b;
    line-height: 88px*@b;
    text-align: center;
    background: #1882d4;
    font-size: 32px*@b;
    color: #fff;
}
 .user-submit-group button:disabled {
    background: #c5cfd5;
}
.user-submit-group button .loading {
    display: none;
}
.user-top a {
    color: #1882d4;
}
</style>
