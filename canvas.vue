<template>
  <div class="des-sign">
    <p class="sign-title">请在此区域手绘签名，确认后不可修改</p>
    <div class="canvasBox" ref="canvasHW">
      <canvas :w="'99%'" :h="'60%'" @touchstart="touchStart" @touchmove="touchMove" @touchend="touchEnd" ref="canvasF" @mousedown="mouseDown" @mousemove="mouseMove" @mouseup="mouseUp"></canvas>
      <div class="sign-btn">
        <div class="clear-btn" @click="clear">清除</div>
        <div class="save-btn" @click="save">保存</div>
      </div>
    </div>
  </div>
</template>

<script>
import Api from '../config/api.js'
import Util from '../utils/utils'
export default {
  name: 'demo',
  data () {
    return {
      points: [],
      canvasTxt: null,
      startX: 0,
      startY: 0,
      moveY: 0,
      moveX: 0,
      endY: 0,
      endX: 0,
      w: null,
      h: null,
      isDown: false
    }
  },
  created () {

  },
  mounted () {
    let canvas = this.$refs.canvasF
    // canvas.height = this.$refs.canvasHW.offsetHeight - 60
    // canvas.width = this.$refs.canvasHW.offsetWidth - 10
    this.canvasTxt = canvas.getContext('2d')
  },
  components: {
    // NavPublic
  },
  methods: {
    getData () {
      this.getToken()
      this.getRoomId()
    },
    getToken () {
      if (!Util.getCookie('token')) {
        this.token = Util.pageUrlGetValue('token')
        Util.setCookie('token', this.token, 'aylives.cn')
      } else {
        this.token = Util.getCookie('token')
      }
    },
    getRoomId () {
      if (!Util.getCookie('currentRoomId')) {
        this.roomId = Util.pageUrlGetValue('currentRoomId')
        Util.setCookie('currentRoomId', this.roomId, 'aylives.cn')
      } else {
        this.roomId = Util.getCookie('currentRoomId')
      }
    },
    save () {
      var _this = this
      var png = _this.$refs.canvasF.toDataURL()
      // var jpeg = _this.$refs.signature.save("image/jpeg");
      // var svg = _this.$refs.signature.save("image/svg+xml");
      this.uploadName(png)
    },

    convertBase64UrlToBlob (urlData) {
      var bytes = window.atob(urlData.split(',')[1])       //去掉url的头，并转换为byte
      //处理异常,将ascii码小于0的转换为大于0
      var ab = new ArrayBuffer(bytes.length)
      var ia = new Uint8Array(ab)
      for (var i = 0; i < bytes.length; i++) {
        ia[i] = bytes.charCodeAt(i)
      }
      return new Blob([ab], {type: 'image/png'})
    },

    uploadName (filebase64) {
      let file = this.convertBase64UrlToBlob(filebase64)
      let formdata = new FormData();
      formdata.append("img", file);
      Api.Axios({
        url: Api.UPLOAD,
        method: 'post',
        data: formdata,
        headers: {
          'Content-Type': 'multipart/form-data'
        }
      }).then(res => {
        if (res.data.code === '200') {
          this.signSrc = res.data.data.url
          this.confirmData()
        } else {
          this.$toast(res.data.msg, 1500)
        }
      })
    },
    confirmData () {
      Api.Axios.get(Api.CONFIRM(this.token, this.signSrc, this.roomId)).then(res => {
        if (res.data && (res.data.code === 200)) {
          this.$router.push({path: `/confirm?signSrc=${this.signSrc}`})
          // this.rank = res.data.data.rank
          // this.signUrl = res.data.data.signUrl            // 是否已经签过名了
          // this.signTime = res.data.data.createdDate
        } else {
          this.$toast(res.data.msg, 1500)
        }
      })
    },
    // 电脑设备事件
    mouseDown (ev) {
      ev = ev || event
      ev.preventDefault()
      let obj = {
        x: ev.offsetX,
        y: ev.offsetY
      }
      this.startX = obj.x
      this.startY = obj.y
      this.canvasTxt.beginPath()
      this.canvasTxt.moveTo(this.startX, this.startY)
      this.canvasTxt.lineTo(obj.x, obj.y)
      this.canvasTxt.stroke()
      this.canvasTxt.closePath()
      this.points.push(obj)
      this.isDown = true
    },
    // 移动设备事件
    touchStart (ev) {
      ev = ev || event
      ev.preventDefault()
      if (ev.touches.length === 1) {
        let obj = {
          x: ev.targetTouches[0].clientX,
          y: ev.targetTouches[0].clientY - 48
        }
        this.startX = obj.x
        this.startY = obj.y
        this.canvasTxt.beginPath()
        this.canvasTxt.moveTo(this.startX, this.startY)
        this.canvasTxt.lineTo(obj.x, obj.y)
        this.canvasTxt.stroke()
        this.canvasTxt.closePath()
        this.points.push(obj)
      }
    },
    // 电脑设备事件
    mouseMove (ev) {
      ev = ev || event
      ev.preventDefault()
      if (this.isDown) {
        let obj = {
          x: ev.offsetX,
          y: ev.offsetY
        }
        this.moveY = obj.y
        this.moveX = obj.x
        this.canvasTxt.beginPath()
        this.canvasTxt.moveTo(this.startX, this.startY)
        this.canvasTxt.lineTo(obj.x, obj.y)
        this.canvasTxt.stroke()
        this.canvasTxt.closePath()
        this.startY = obj.y
        this.startX = obj.x
        this.points.push(obj)
      }
    },
    // 移动设备事件
    touchMove (ev) {
      ev = ev || event
      ev.preventDefault()
      if (ev.touches.length === 1) {
        let obj = {
          x: ev.targetTouches[0].clientX,
          y: ev.targetTouches[0].clientY - 48
        }
        this.moveY = obj.y
        this.moveX = obj.x
        this.canvasTxt.beginPath()
        this.canvasTxt.moveTo(this.startX, this.startY)
        this.canvasTxt.lineTo(obj.x, obj.y)
        this.canvasTxt.stroke()
        this.canvasTxt.closePath()
        this.startY = obj.y
        this.startX = obj.x
        this.points.push(obj)
      }
    },
    // 电脑设备事件
    mouseUp (ev) {
      ev = ev || event
      ev.preventDefault()
      let obj = {
        x: ev.offsetX,
        y: ev.offsetY
      }
      this.canvasTxt.beginPath()
      this.canvasTxt.moveTo(this.startX, this.startY)
      this.canvasTxt.lineTo(obj.x, obj.y)
      this.canvasTxt.stroke()
      this.canvasTxt.closePath()
      this.points.push(obj)
      this.points.push({ x: -1, y: -1 })
      this.isDown = false
    },
    // 移动设备事件
    touchEnd (ev) {
      ev = ev || event
      ev.preventDefault()
      console.log(ev)
      if (ev.touches.length === 1) {
        let obj = {
          x: ev.targetTouches[0].clientX,
          y: ev.targetTouches[0].clientY - 48
        }
        this.canvasTxt.beginPath()
        this.canvasTxt.moveTo(this.startX, this.startY)
        this.canvasTxt.lineTo(obj.x, obj.y)
        this.canvasTxt.stroke()
        this.canvasTxt.closePath()
        this.points.push(obj)
        this.points.push({ x: -1, y: -1 })
      }
    },
    // 重写
    clear () {
      this.canvasTxt.clearRect(0, 0, this.$refs.canvasF.width, this.$refs.canvasF.height)
      this.points = []
    }
  }
}
</script>

<style lang="less" scoped>
  .des-sign {
    z-index: 999;
    width: 336px;
    height: 336px;
    background: #ffffff;
    font-size: 18px;
    // border: 1px solid red;
    color: #FCCA26;
    .sign-title {
      font-size: 16px;
      height: 12%;
      margin-top: 5%;
      border-bottom: 1px solid #CECECE;
      @media only screen and (device-width: 375px) and (device-height: 812px) {
        height: 8%;
        font-size: 18px;
      }
      @media only screen and (width: 768px) {
        font-size: 32px;
      }

      @media only screen and (width: 1024px) {
        font-size: 38px;
      }
    }
    // .canvasBox {
    //   padding: 10px 5px;
    //   box-sizing: border-box;
    //   flex: 1;
    // }
    .sign-btn {
      // border: 1px solid red;
      text-align: center;
      // list-style: none;
      display: flex;
      justify-content: space-between;
      // position: relative;
      font-size: 18px;
      height: 44px;
      .clear-btn {
        flex: 1;
        -ms-flex: 1;
        -webkit-box-flex: 1;
        justify-content: convert;
        overflow: hidden;
        width: 108px;
        // height: 80%;
        line-height: 44px;
        border-radius: 20px;
        background: linear-gradient(360deg, rgba(249, 164, 105, 1) 0%, rgba(251, 203, 156, 1) 100%);
        display: inline-block;
        margin-right: 28px;
        margin-left: 10px;
        color: #FFFFFF;
        @media only screen and (width: 768px) {
          border-radius: 60px;
          margin-right: 10%;
          margin-left: 8%;
          line-height: 80px;
        }

        @media only screen and (width: 1024px) {
          border-radius: 60px;
          margin-right: 10%;
          margin-left: 8%;
          line-height: 80px;
        }
      }
      .save-btn {
        margin-left: 28px;
        margin-right: 10px;
        flex: 1;
        -ms-flex: 1;
        -webkit-box-flex: 1;
        justify-content: convert;
        overflow: hidden;
        width: 108px;
        line-height: 44px;
        // height: 80%;
        border-radius: 20px;
        background: linear-gradient(360deg, rgba(249, 164, 105, 1) 0%, rgba(251, 203, 156, 1) 100%);
        display: inline-block;
        color: #FFFFFF;
        @media only screen and (width: 768px) {
          border-radius: 60px;
          margin-right: 8%;
          margin-left: 10%;
          line-height: 80px;
        }

        @media only screen and (width: 1024px) {
          border-radius: 60px;
          margin-right: 8%;
          margin-left: 10%;
          line-height: 80px;
        }
      }
      @media only screen and (device-width: 375px) and (device-height: 812px) {
        font-size: 18px;
      }
      @media only screen and (width: 768px) {
        font-size: 40px;
        height: 80px;
      }

      @media only screen and (width: 1024px) {
        font-size: 40px;
        height: 80px;
      }
    }
  }
</style>

