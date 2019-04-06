<template>
  <div id="wrapper">
    <main class="mainbox" :class="{'isloading': loading}" :style="{backgroundImage: 'url('+imgData.cdnurl+')'}"></main>
    <div class="left-bar">
      <div class="icon-area" @click="onClickPre">
        <i class="iconfont icon-left"></i>
      </div>
    </div>
    <div class="right-bar">
      <div class="icon-area" @click="onClickNext">
        <i class="iconfont icon-right"></i>
      </div>
    </div>
    <div class="copyright">
      <p class="title">{{imgData.title}} [{{imgData.date ? imgData.date.substring(0, 10) : ''}}]</p>
      <p class="story">{{imgData.story}}</p>
    </div>
    <footer class="footer">
      <div title="今天" class="icon" @click="onClickHome"><i class="iconfont icon-home"></i></div>
      <!-- <div class="icon"><i class="iconfont icon-image"></i></div> -->
      <div title="壁纸" class="icon" @click="onClickBg"><i class="iconfont icon-skin"></i></div>
      <div title="浏览器打开" class="icon" @click="onClickOpenInBro"><i class="iconfont icon-desktop"></i></div>
      <div title="下载" class="icon" @click="onClickDownload"><i class="iconfont icon-download"></i></div>
      <div title="Github" class="icon" @click="onClickGithub"><i class="iconfont icon-star"></i></div>
    </footer>
  </div>
</template>

<script>
  import dayjs from 'dayjs'
  const wallpaper = require('wallpaper')
  export default {
    name: 'landing-page',
    data () {
      return {
        loading: true,
        imgData: {},
        dateStr: '',
        timer: null,
        issetting: false
      }
    },
    created () {
      this.dateStr = dayjs().format('YYYY-MM-DD')
      this.getImageData()
    },
    methods: {
      loadImg (link) {
        this.$electron.shell.openExternal(link)
      },
      getImageData () {
        this.loading = true
        this.$http.get('http://bing.ioleo.cn/bing/getbydate?d=' + this.dateStr)
          .then(response => {
            if (response.data.errno === 0) {
              this.imgData = response.data.data
              // this.imgData.cdnurl = response.data.data.cdnurl + '?imageView2/1/w/1920/h/1080/q/50'
            }
            this.timer = setTimeout(() => {
              this.loading = false
            }, 1000)
          })
          .catch(error => console.log(error))
      },
      onClickPre () {
        if (this.dateStr === '2015-05-12') {
          this.dateStr = dayjs().format('YYYY-MM-DD')
        } else {
          this.dateStr = dayjs(this.dateStr).add(-1, 'day').format('YYYY-MM-DD')
        }
        this.getImageData()
      },
      onClickNext () {
        const todayDate = dayjs().format('YYYY-MM-DD')
        if (todayDate === this.dateStr) {
          this.dateStr = '2015-05-12'
        } else {
          this.dateStr = dayjs(this.dateStr).add(1, 'day').format('YYYY-MM-DD')
        }
        this.getImageData()
      },
      onClickHome () {
        const todayDate = dayjs().format('YYYY-MM-DD')
        if (this.dateStr === todayDate) return false
        this.dateStr = todayDate
        this.getImageData()
      },
      onClickOpenInBro () {
        this.$electron.shell.openExternal(this.imgData.cdnurl)
      },
      onClickDownload () {
        this.$electron.ipcRenderer.send('download', this.imgData.cdnurl)
      },
      onClickGithub () {
        this.$electron.shell.openExternal('https://github.com/qhx0807/')
      },
      async onClickBg () {
        const a = await wallpaper.get()
        this.$electron.ipcRenderer.send('download', a)
        // if (this.issetting) return false
        // this.issetting = true
      }
    }
  }
</script>

<style lang="less">
  * {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
    transition: all .3s linear;
  }

  body { 
    font-family: '微软雅黑',
    sans-serif;
    overflow: hidden;
  }
  #wrapper {
    background:
      radial-gradient(
        ellipse at top left,
        rgba(255, 255, 255, 1) 40%,
        rgba(229, 229, 229, .9) 100%
      );
    height: 100vh;
    width: 100vw;
    overflow: hidden;
    position: relative;
  }
  .mainbox{
    height: 100%;
    width: 100%;
    overflow: hidden;
    background-size: cover;
    background-position: center;
    &.isloading{
      filter: blur(10px);
      transform: scale(1.1);
    }
  }
  .image{
    width: 100%;
    height: 100%;
    filter: blur(30px);
    transform: scale(1.1);
  }
  .copyright{
    position: fixed;
    bottom: 55px;
    color: #fff;
    left: 0;
    right: 0;
    text-align: right;
    padding-right: 12px;
    font-weight: normal;
    font-size: 12px;
    transition: 0s;
    &:hover{
      color: #aaa;
    }
  }
  .footer{
    position: fixed;
    bottom: 0;
    left: 0;
    right: 0;
    height: 50px;
    background-color: rgba(222, 222, 222, .2);
    display: flex;
    flex-direction: row;
    justify-content: flex-end;
    color: #fff;
    .icon{
      width: 50px;
      cursor: pointer;
      height: 100%;
      display: flex;
      align-items: center;
      justify-content: center;
      &:hover{
        background-color: rgba(50, 50, 50, .3);
      }
      i{
        font-size: 18px;
      }
    }
  }
  .bar{
    position: fixed;
    top: 0;
    bottom: 0;
    width: 20px;
    display: flex;
    align-items: center;
    justify-content: center;
    color: #fff;
  }
  .icon-area{
    height: 120px;
    width: 20px;
    background-color: rgba(100, 100, 100, 0.2);
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    &:hover{
      background-color: rgba(240, 240, 240, .2);
    }
  }
  .left-bar{
    .bar;
    left: 0;
  }
  .right-bar{
    .bar;
    right: 0;
  }

</style>
