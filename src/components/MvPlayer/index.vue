<template>
  <div>
    <div class="mv-player">
      <div class="container">
        <div class="back" @click="back"><Icon type="ios-arrow-back" /></div>
        <div class="video-box">
          <video class="video" :poster="pic" controls ref="video" playsinline="" webkit-playsinline="" :src="url"></video>
        </div>
        <div class="video-info  border-half-bottom">
          <div class="title">{{playMvData.title}}</div>
          <div class="count"><Icon type="ios-videocam" /> {{playMvData.listenCount}} 万</div>
        </div>
        <div class='head'>大部分人还爱看</div>
        <div class="swiper" @touchstart="swiperTouchStr($event)" @touchend="swiperTouchEnd($event)">
          <div class="swiper-item" v-for="(value, key) in otherList" :key="key" :style="{ transform: `translate(${key-current}00%,0)`}">
            <div class="item" v-if="value.length==3" v-for="(item, index) in value" :key="index">
              <img :src="item.cover_pic" alt="" @click="changePlay(item.vid)">
              <div class='title text-line'>
                <span class="pix">{{item.name}}</span>
              </div>
              <div class='date'><Icon type="ios-videocam" /> {{item.playcnt}} 万</div>
            </div>
          </div>
          <div class="dots">
            <div class="dot" v-for="(value,index) in otherList" :class="current==index?'active':''" :key="index"></div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
export default {
  name: 'mv-player',
  data () {
    return {
      playMvData: {},
      video: null,
      otherList: [],
      url: '',
      pic: '',

      current: 0,
      timer: null,
      startX: 0,
      startY: 0
    }
  },
  mounted () {
    this.playMvData = this.$route.params.data
    this.video = this.$refs.video
    this.getData(this.playMvData.vid)
  },
  methods: {
    back () {
      this.$router.go(-1)
    },
    changePlay (vid) {
      this.$refs.video.pause()
      this.getData(vid)
    },
    getData (vid) {
      let getMvUrl = new Promise(resolve => {
        this.axios.get(this.X.path.u, {
          params: {
            api: this.X.api.getMvUrl,
            inCharset: 'utf8',
            outCharset: 'GB2312',
            param: {
              vids: [vid]
            }
          }
        }).then((res) => {
          resolve(res.data.url)
        })
      })
      let getMvInfo = new Promise(resolve => {
        this.axios.get(this.X.path.u, {
          params: {
            api: this.X.api.getMvInfo,
            param: {
              vidlist: [vid]
            }
          }
        }).then((res) => {
          resolve(res.data.url)
        })
      })
      let getMvOther = new Promise(resolve => {
        this.axios.get(this.X.path.u, {
          params: {
            api: this.X.api.getMvOther,
            param: {
              vid: vid
            }
          }
        }).then((res) => {
          res.data.forEach((item, index) => {
            let s = item.playcnt
            item.playcnt = (s / 10000).toFixed(2)
          })
          resolve(res.data)
        })
      })
      Promise.all([getMvUrl, getMvInfo, getMvOther]).then((values) => {
        this.url = values[0]
        this.pic = values[1]
        this.otherList = this.subArr(values[2])
        let timer = setTimeout(() => {
          this.video = this.$refs.video
          clearTimeout(timer)
        }, 500)
      })
    },
    // 分隔成三个一组
    subArr (data) {
      let subArrayNum = 3
      let dataArr = new Array(Math.ceil(data.length / subArrayNum))
      for (let i = 0; i < dataArr.length; i++) {
        dataArr[i] = []
        for (let j = 0; j < subArrayNum; j++) {
          dataArr[i][j] = ''
        }
      }
      for (let i = 0; i < data.length; i++) {
        dataArr[parseInt(i / subArrayNum)][i % subArrayNum] = data[i]
      }
      return dataArr
    },
    swiperTouchStr (e) {
      clearInterval(this.timer)
      this.startX = e.changedTouches[0].pageX
      this.startY = e.changedTouches[0].pageY
    },
    swiperTouchEnd (e) {
      let x = e.changedTouches[0].pageX - this.startX
      let y = e.changedTouches[0].pageY - this.startY
      if (Math.abs(x) > Math.abs(y) && x > 0) {
        if (this.current !== 0) {
          this.current -= 1
        }
      } else if (Math.abs(x) > Math.abs(y) && x < 0) {
        if (this.current < this.otherList.length - 1) {
          this.current += 1
        }
      } else if (Math.abs(y) > Math.abs(x) && y > 0) {
      } else if (Math.abs(y) > Math.abs(x) && y < 0) {
      } else {
      }
    }
  },
  destroyed () {
    clearInterval(this.timer)
  }
}
</script>
<style lang="stylus" scoped>
@import '../../assets/css/common'
.mv-player
  height 100%
  width 100%
  .container
    position absolute
    top 0
    left 0
    right 0
    bottom 0
    z-index 12
    background $bgColor
    .swiper
      width 100%
      height 150px
      overflow hidden
      position relative
      .img
        visibility hidden
      .swiper-item
        position absolute
        top 0
        padding 5px
        width 100%
        height 100%
        transition all 0.5s ease
        display flex
        .item
          flex 1
          color #999
          margin 5px
          .title
            height 25px
            line-height 25px
            position relative
          .date
            font-size 10px
            i
              font-size 16px
              position relative
              top -2px
      .dots
        position absolute
        left 0
        right 0
        bottom 0
        height 20px
        text-align center
        .dot
          display inline-block
          width 8px
          height 8px
          border-radius 50%
          margin 0 3px
          background rgba(0,0,0,0.5)
          &.active
            background rgba(255,255,255,0.7)
      img
        width 100%
        display block
    .back
      width 30px
      height 30px
      background rgba(0,0,0,0.5)
      position fixed
      top 90px
      left 5px
      border-radius: 50%
      z-index 100
      font-size 22px
      color #fff;
      display flex
      justify-content center
      align-items center
      i
        position relative
        left -1px
    .head
      font-size 15px
      padding 0 10px
      line-height 50px
      color $themeColor
    .video-info
      position relative
      line-height 25px
      padding 5px 10px
      color #999
      font-size 14px
      .count
        color #999
        i
          font-size 23px
          position relative
          top -1px
    .video-box
      width 100%
      height 220px
      overflow hidden
      .video
        width 100%
        height 100%
        object-fit fill
@media screen and (max-width:480px)
  .mv-player .container
    .video-box
      height 232px
    .item
      img
        height 70px
@media screen and (max-width:375px)
  .mv-player .container
    .video-box
      height 210px
    .item
      img
        height 63px
@media screen and (max-width:320px)
  .mv-player .container
    .video-box
      height 180px
    .item
      img
        height 53px
</style>
