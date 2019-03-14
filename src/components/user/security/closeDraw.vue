<template>
  <section style="position: absolute; height: 100%; width: 100%; background: #fff">
    <!--头部-->
    <van-nav-bar title="" left-text="返回" left-arrow @click-left="$back()">
      <van-icon class="iconfont icon-fanhui" slot="left" />
    </van-nav-bar>
    <!--内容-->
    <div class="content">
      <div class="gesturePwd">
        <div class="box">
          <h4 class="gestureTitle">手势密码</h4>
          <p ref="gestureTitle">请输入手势密码</p>
          <canvas ref="canvas"></canvas>
        </div>
      </div>
    </div>
  </section>
</template>

<script>
  export default {
    name: "closeDraw",
    data() {
      return {
        ctx: '',
        width: 0,
        height: 0,
        devicePixelRatio: 0,
        chooseType: '',
        r: '',// 公式计算
        lastPoint: [],
        arr: [],
        restPoint: [],
        canvas: ''
      }
    },
    mounted() {
      this.setChooseType(3);
    },
    methods: {
      drawCle(x, y) { // 初始化解锁密码面板 小圆圈
        this.ctx.strokeStyle = '#87888a';//密码的点点默认的颜色
        this.ctx.lineWidth = 2;
        this.ctx.beginPath();
        this.ctx.arc(x, y, this.r, 0, Math.PI * 2, true);
        this.ctx.closePath();
        this.ctx.stroke();
      },
      drawPoint(style) { // 初始化圆心
        for (var i = 0; i < this.lastPoint.length; i++) {
          this.ctx.fillStyle = style;
          this.ctx.beginPath();
          this.ctx.arc(this.lastPoint[i].x, this.lastPoint[i].y, this.r / 2.5, 0, Math.PI * 2, true);
          this.ctx.closePath();
          this.ctx.fill();
        }
      },
      drawStatusPoint(type) { // 初始化状态线条
        for (var i = 0; i < this.lastPoint.length; i++) {
          this.ctx.strokeStyle = type;
          this.ctx.beginPath();
          this.ctx.arc(this.lastPoint[i].x, this.lastPoint[i].y, this.r, 0, Math.PI * 2, true);
          this.ctx.closePath();
          this.ctx.stroke();
        }
      },
      drawLine(style, po, lastPoint) {//style:颜色 解锁轨迹
        this.ctx.beginPath();
        this.ctx.strokeStyle = style;
        this.ctx.lineWidth = 3;
        this.ctx.moveTo(this.lastPoint[0].x, this.lastPoint[0].y);

        for (var i = 1; i < this.lastPoint.length; i++) {
          this.ctx.lineTo(this.lastPoint[i].x, this.lastPoint[i].y);
        }
        this.ctx.lineTo(po.x, po.y);
        this.ctx.stroke();
        this.ctx.closePath();

      },
      createCircle() {// 创建解锁点的坐标，根据canvas的大小来平均分配半径
        var n = this.chooseType;
        var count = 0;
        this.r = this.ctx.canvas.width / (2 + 4 * n);// 公式计算
        this.lastPoint = [];
        this.arr = [];
        this.restPoint = [];
        var r = this.r;
        for (var i = 0; i < n; i++) {
          for (var j = 0; j < n; j++) {
            count++;
            var obj = {
              x: j * 4 * r + 3 * r,
              y: i * 4 * r + 3 * r,
              index: count
            };
            this.arr.push(obj);
            this.restPoint.push(obj);
          }
        }
        this.ctx.clearRect(0, 0, this.ctx.canvas.width, this.ctx.canvas.height);
        for (var i = 0; i < this.arr.length; i++) {
          this.drawCle(this.arr[i].x, this.arr[i].y);

        }
      },
      getPosition(e) {// 获取touch点相对于canvas的坐标
        var rect = e.currentTarget.getBoundingClientRect();
        var po = {
          x: (e.touches[0].clientX - rect.left) * this.devicePixelRatio,
          y: (e.touches[0].clientY - rect.top) * this.devicePixelRatio
        };
        return po;
      },
      update(po) {// 核心变换方法在touchmove时候调用
        this.ctx.clearRect(0, 0, this.ctx.canvas.width, this.ctx.canvas.height);
        for (var i = 0; i < this.arr.length; i++) { // 每帧先把面板画出来
          this.drawCle(this.arr[i].x, this.arr[i].y);
        }
        this.drawPoint('#27AED5');// 每帧花轨迹
        this.drawStatusPoint('#27AED5');// 每帧花轨迹
        this.drawLine('#27AED5', po, this.lastPoint);// 每帧画圆心
        for (var i = 0; i < this.restPoint.length; i++) {
          if (Math.abs(po.x - this.restPoint[i].x) < this.r && Math.abs(po.y - this.restPoint[i].y) < this.r) {
            this.drawPoint(this.restPoint[i].x, this.restPoint[i].y);
            this.lastPoint.push(this.restPoint[i]);
            this.restPoint.splice(i, 1);
            break;
          }
        }
      },

      storePass(psw) {// touchend结束之后对密码和状态的处理
        if (psw.length < 4) {
          this.drawStatusPoint('red');
          this.drawPoint('red');
          this.drawLine('red', this.lastPoint[this.lastPoint.length - 1], this.lastPoint);// 每帧画圆心
          var gestureTitle = this.$refs.gestureTitle;
          gestureTitle.style.color = "red";
          gestureTitle.innerTEXT = '请连接4个点';
          this.resetUpdate();
        }
        var str = '';
        psw.forEach((item)=>{
          str += item.index;
        })
        this.$axios({
          url: '/api/user/close/pattern',
          method: 'post',
          data: {
            patternPassword: str
          }
        }).then(res => {
          if(!res.data.errorCode) {
            this.$toast.success('密码关闭成功');
            localStorage.removeItem('userMob'); //手势解锁手机号
            localStorage.removeItem('loginType'); //登录方式
            this.$back();
          }else {
            this.$toast.fail(res.data.errorMsg);
            this.resetUpdate();
          }
        })
      },

      resetUpdate() {
        var self = this
        setTimeout(function () {
          self.reset();
        }, 500);
      },
      makeState() {
        var gestureTitle = this.$refs.gestureTitle;
        gestureTitle.style.color = "#87888a";
        gestureTitle.innerHTML = '请输入手势密码';
      },
      setChooseType(type) {
        this.chooseType = type;
        this.init();
      },
      initDom() {
        this.chooseType = Number(window.localStorage.getItem('chooseType')) || 3;
        this.devicePixelRatio = window.devicePixelRatio || 1;
        var canvas = this.$refs.canvas;
        var width = this.width || 320;
        var height = this.height || 320;
        // 高清屏锁放
        canvas.style.width = width + "px";
        canvas.style.height = height + "px";
        canvas.height = height * this.devicePixelRatio;
        canvas.width = width * this.devicePixelRatio;
      },
      init() {
        this.initDom();
        this.lastPoint = [];
        this.makeState();
        this.touchFlag = false;
        this.canvas = this.$refs.canvas;
        this.ctx = this.canvas.getContext('2d');
        this.createCircle();
        this.bindEvent();
      },
      reset() {
        this.makeState();
        this.createCircle();
      },
      bindEvent() {
        var self = this;
        this.canvas = this.$refs.canvas;
        this.canvas.addEventListener("touchstart", function (e) {
          e.preventDefault();// 某些android 的 touchmove不宜触发 所以增加此行代码
          var po = self.getPosition(e);
          for (var i = 0; i < self.arr.length; i++) {
            if (Math.abs(po.x - self.arr[i].x) < self.r && Math.abs(po.y - self.arr[i].y) < self.r) {
              self.touchFlag = true;
              self.drawPoint(self.arr[i].x, self.arr[i].y);
              self.lastPoint.push(self.arr[i]);
              self.restPoint.splice(i, 1);
              break;
            }
          }
        }, false);
        this.canvas.addEventListener("touchmove", function (e) {
          if (self.touchFlag) {
            self.update(self.getPosition(e));
          }
        }, false);
        this.canvas.addEventListener("touchend", function (e) {
          if (self.touchFlag) {
            self.touchFlag = false;
            self.storePass(self.lastPoint);
          }
        }, false);
      }
    }
  }
</script>

<style scoped>
  .gesturePwd {
    position: fixed;
    /*z-index: 5000;*/
    width: 100%;
    height: 88%;
    top: 50%;
    left: 50%;
    -webkit-transform: translate(-50%, -50%);
    transform: translate(-50%, -50%);
    text-align: center;
    border-radius: 3px;
    overflow: hidden;
    background-color: #fff;
  }
  .gestureTitle {
    color: #87888a;
    margin-top: 2rem;
    font-size: 20px;
    font-weight: normal;
  }
  .box canvas{
    background-color: #fff;
    display: inline-block;
    margin-top: 1rem;
    width: 320px;
    height: 320px;
  }
</style>
