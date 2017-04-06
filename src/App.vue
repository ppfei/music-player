<template>
  <div id="app">
    <div id="music-header">{{ getSongInfo.name }}</div>
    <audio src="###" style="display:none" id="audio"></audio>
    <music-box :songInfo="getSongInfo" :isplay="state"></music-box>
    <music-bar @event-play="play" @event-change="changeMusic" @event-loop="changeLoop" :loop="loopType" :playState="state" :time="getTime"></music-bar>
    <div id="back-img" :style="{'background-image':'url('+getSongInfo.al.picUrl+')'}"></div>
    <div style="position:fixed;z-index:999;">
      <p>state: {{ state }}</p>
      <p>index: {{getIndex}}</p>
      <p>currentTime: {{currentTime}}</p>
      <p>duration: {{duration}}</p>
      <p>loopType: {{loopType}}</p>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
import musicBox from './components/musicBox'
import musicBar from './components/musicBar'

export default {
  name: 'app',
  mounted () {
    this.timer = null;
    this.myAudio = document.querySelector('#audio');
    // 当音乐播放时
    this.myAudio.onplay = ()=> {
      this.timer = setInterval(()=>{
        this.currentTime = this.myAudio.currentTime;
      },500);
      this.state = 'play';
    };
    // 当音乐暂停时
    this.myAudio.onpause = ()=> {
      if(this.timer) clearInterval(this.timer);
      this.state = 'pause';
    };
    // 当音乐缓冲时
    this.myAudio.onwaiting = ()=> {
      this.state = 'waite';
    };
    // 当音乐缓冲完成时
    this.myAudio.onplaying = ()=> {
      this.myAudio.play();
      this.state = 'play';
    };
    // 当音乐元数据加载完成时
    this.myAudio.onloadedmetadata = ()=> {
      this.duration = this.myAudio.duration;
    }
    // 当音乐当前播放时间改变时
    // this.myAudio.ontimeupdate = ()=> {
    //   this.currentTime = this.myAudio.currentTime;
    // };
    // 当音乐播放结束时
    this.myAudio.onended = ()=> {
      this.changeMusic();
    };
    this.getSong();
  },
  data () {
    return {
      // audio元素节点
      myAudio: null,
      // 音乐索引
      index: 0,
      // 播放状态
      state: 'pause',
      // 循环模式(loop:列表循环；one:单曲循环；random:随机循环)
      loopType: 'loop',
      // 音量
      volum: 50,
      // 当前播放位置
      currentTime: 0,
      // 当前音频总长度
      duration: 1,
      // 初始音乐列表
      musicList: [
        {
          id: 25727803,
          name: '修炼爱情',
          al: {
            id: 2301158,
            name: '因你而在',
            picUrl: 'https://p3.music.126.net/6miaRW-_QT81R-gsj4MCLg==/4431031859953290.jpg',
          },
          ar: {
            id: 3684,
            name: '林俊杰',
          },
          mv: 125,
          br: 128000
        },
        {
          id: 25727804,
          name: '修炼爱情',
          al: {
            id: 2301158,
            name: '因你而在',
            picUrl: 'https://p3.music.126.net/6miaRW-_QT81R-gsj4MCLg==/4431031859953290.jpg',
          },
          ar: {
            id: 3684,
            name: '林俊杰',
          },
          mv: 125,
          br: 128000
        },
      ]
    }
  },
  components: {
    musicBox,
    musicBar
  },
  computed: {
    // 获取当前的播放序号
    getIndex () {
      return this.index;
    },
    /*// 获取封面图片
    getImgSrc () {
      return this.musicList[this.index].al.picUrl;
    },
    // 获取歌曲名称
    getName () {
      return this.musicList[this.index].name;
    },*/
    // 获取歌曲信息
    getSongInfo () {
      return this.musicList[this.index];
    },
    //
    getTime () {
      return {
        'current': Math.floor(this.currentTime),
        'duration': Math.floor(this.duration)
      }
    }
  },
  methods: {
    // 播放暂停
    play () {
      if(this.state === 'play'){
        this.myAudio.pause();
      }else{
        this.myAudio.play();
      };
    },
    // 切换音乐
    changeMusic (type) {
      let _index = 0;
      let leng = this.musicList.length;
      if(type === 'prev'){
        _index = this.index-1;
        if(_index<0) _index = leng-1;
      }else if(type === 'next'){
        _index = this.index +1;
        if(_index >= leng) _index = 0;
      }else {
        if(this.loopType === 'random'){
          _index = Math.floor(Math.random()*leng);
        }else if(this.loopType === 'one'){
          this.myAudio.play();
          return false;
        }else {
          _index = this.index +1;
          if(_index >= leng) _index = 0;
        }
      }
      this.index = _index;
      this.getSong();
    },
    // 更改循环模式
    changeLoop () {
      if(this.loopType === 'loop'){
        this.loopType = 'random';
      }else{
        this.loopType = 'loop';
      }
    },
    // 获取音乐
    getSong (id) {
      if( !(typeof id === 'number') ) id = this.musicList[this.index].id;
      let self = this;
      let url = 'https://api.imjad.cn/cloudmusic/?type=song&id='+id+'&br=128000';
      axios.get(window.location.origin.replace(/[0-9]+$/,'8081')+'/proxy.php',{
          params:{ url: url }
        })
        .then(function (response) {
          let data = response.data;
          if(data){
            let src = data.data[0].url;
            if(src){
              src = src.replace('https','http');
              self.myAudio.src = src;
              self.myAudio.play();
            }
          }
        })
        .catch(function (error) {
          console.log(window.location.origin.replace(/[0-9]+$/,'8081')+'/proxy.php');
          alert(error);
        });
    },
    // 获取音乐详情
    getDetail (id) {
      if( !(typeof id === 'number') ) id = this.musicList[this.index].id;
      let url = 'https://api.imjad.cn/cloudmusic/?type=detail&id='+id;
      axios.get(window.location.origin.replace(/[0-9]+$/,'8081')+'/proxy.php',{
          params:{ url: url }
        })
        .then(function (response) {
          console.log(response.data);
        })
        .catch(function (error) {
          console.log(error);
        });
    },
    // 获取MV
    getMv (id) {
      if( !(typeof id === 'number') ) id = this.musicList[this.index].mv;
      let url = 'https://api.imjad.cn/cloudmusic/?type=mv&id='+id;
      axios.get(window.location.origin.replace(/[0-9]+$/,'8081')+'/proxy.php',{
          params:{ url: url }
        })
        .then(function (response) {
          console.log(response.data);
        })
        .catch(function (error) {
          console.log(error);
        });
    },
    // 获取音乐评论
    getComment (id) {
      if( !(typeof id === 'number') ) id = this.musicList[this.index].id;
      let url = 'https://api.imjad.cn/cloudmusic/?type=comments&id='+id;
      axios.get(window.location.origin.replace(/[0-9]+$/,'8081')+'/proxy.php',{
          params:{ url: url }
        })
        .then(function (response) {
          console.log(response.data);
        })
        .catch(function (error) {
          console.log(error);
        });
    },
    // 获取歌词
    getLyric (id) {
      if( !(typeof id === 'number') ) id = this.musicList[this.index].id;
      let url = 'https://api.imjad.cn/cloudmusic/?type=lyric&id='+id;
      axios.get(window.location.origin.replace(/[0-9]+$/,'8081')+'/proxy.php',{
          params:{ url: url }
        })
        .then(function (response) {
          console.log(response.data);
        })
        .catch(function (error) {
          console.log(error);
        });
    }
  }
}
</script>

<style lang="scss">
.flex-h {
  display: flex;
  flex-flow: row wrap;
}
.flex-v {
  display: flex;
  flex-flow: column nowrap;
}
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #fff;
  margin: 0 auto;
  box-shadow: 0 0 10px red inset;
  width: 100%;
  height: 100%;
  overflow: hidden;
  position: relative;
}
@mixin headerHeight($n: 1){
  height: 80px*$n;
  font-size: 20px*$n;
  line-height: 80px*$n;
}
#music-header {
  @include headerHeight();
  width: 100%;
  text-align: center;
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
  position: absolute;
  top: 0;
  z-index: 10;

  &:after {
    position: absolute;
    bottom: 0;
    content: '';
    display: block;
    width: 100%;
    height: 1px;
    background: linear-gradient( left, rgba(255,255,255,0), rgba(255,255,255,0.5), rgba(255,255,255,0) );
  }
}
[data-dpr="2"] #music-header {
  @include headerHeight(2);

  &:after {
    height: 2px;
  }
}
[data-dpr="3"] #music-header {
  @include headerHeight(3);

  &:after {
    height: 3px;
  }
}

#back-img {
  position: absolute;
  z-index: 1;
  top: -10px;
  left: -10px;
  bottom: -10px;
  right: -10px;
  background-size: cover;
  filter: blur(10px);

  &:after {
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    content: '';
    display: block;
    background: black;
    opacity: 0.4;
  }
}
</style>
