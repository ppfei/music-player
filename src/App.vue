<template>
  <div id="app">
    <div id="music-header">{{ getSongInfo.name }}<i class="music-search-btn iconfont" @click="searchIsShow=true">&#xe6f1;</i></div>
    <audio src="###" style="display:none" id="audio"></audio>
    <music-box :songInfo="getSongInfo" :isplay="state" @event-showPage="showPage"></music-box>
    <music-bar @event-play="play" @event-change="changeMusic" @event-loop="changeLoop" @event-control="eventControl" @event-showList="showList('show')" :loop="loopType" :playState="state" :time="getTime"></music-bar>
    <div id="back-img" :style="{'background-image':'url('+getSongInfo.al.picUrl+')'}"></div>
    <!-- 歌曲列表 -->
    <div id="music-list" :class="{'show': listIsShow=='show'}">
      <div id="music-list-bg" @click="showList('hide')"></div>
      <ul class="music-list-box">
        <li class="flex-h" v-for="(item, index) of getMusicList"><p @click="changeMusicByIndex(index)">{{ item.name }}<em> - {{ item.ar.name }}</em></p><span @click="removeSong(index)">&times;</span></li>
      </ul>
    </div>
    <!-- 搜索和详情 -->
    <transition name="page-fade">
      <music-search :musicList="getMusicList" v-show="searchIsShow" @event-closePage="closePage('searchIsShow')" @event-addSong="addSong" @event-changeMusic="changeMusicByIndex"></music-search>
    </transition>
    <transition name="page-fade">
      <music-artist ref="artistIsShow" :musicList="getMusicList" :activeMusic="getSongInfo" v-show="artistIsShow" @event-closePage="closePage('artistIsShow')" @event-addSong="addSong" @event-changeMusic="changeMusicByIndex"></music-artist>
    </transition>
    <transition name="page-fade">
      <music-album ref="albumIsShow" :musicList="getMusicList" :activeMusic="getSongInfo" v-show="albumIsShow" @event-closePage="closePage('albumIsShow')" @event-addSong="addSong" @event-changeMusic="changeMusicByIndex"></music-album>
    </transition>
    <transition name="page-fade">
      <music-comment ref="commentIsShow" :activeMusic="getSongInfo" v-show="commentIsShow" @event-closePage="closePage('commentIsShow')"></music-comment>
    </transition>
    <!--<div class="info" style="position:fixed;z-index:999;">
      <p>state: {{ state }}</p>
      <p>index: {{index}}</p>
      <p>currentTime: {{currentTime}}</p>
      <p>duration: {{duration}}</p>
      <p>loopType: {{loopType}}</p>
    </div>-->
  </div>
</template>

<script>
import axios from 'axios'
import musicBox from './components/musicBox'
import musicBar from './components/musicBar'
import musicSearch from './components/musicSearch'
import musicArtist from './components/musicArtist'
import musicAlbum from './components/musicAlbum'
import musicComment from './components/musicComment'

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
    };
    // 当音乐播放结束时
    this.myAudio.onended = ()=> {
      this.changeMusic('next');
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
          mv: 125
        },
        {
          id: 368727,
          name: '明天，你好',
          al: {
            id: 36520,
            name: '去寻找',
            picUrl: 'https://p4.music.126.net/LQ2iUKlZwqGMysGkeCR4ww==/27487790697969.jpg',
          },
          ar: {
            id: 12437,
            name: '牛奶咖啡',
          },
          mv: 480070
        },
        {
          id: 28661564,
          name: 'Maps',
          al: {
            id: 2866276,
            name: 'Maps',
            picUrl: 'https://p3.music.126.net/OC3XXrblVQPgXrkZyLaPow==/3394192437682072.jpg',
          },
          ar: {
            id: 96266,
            name: 'Maroon 5',
          },
          mv: 285043
        },
        {
          id: 418603077,
          name: '告白气球',
          al: {
            id: 34720827,
            name: '周杰伦的床边故事',
            picUrl: 'https://p4.music.126.net/cUTk0ewrQtYGP2YpPZoUng==/3265549553028224.jpg',
          },
          ar: {
            id: 6452,
            name: '周杰伦',
          },
          mv: 5382080
        }
      ],
      // list显示状态
      listIsShow: 'hide',
      // 搜索页显示状态
      searchIsShow: false,
      // 歌手页显示状态
      artistIsShow: false,
      // 专辑页显示状态
      albumIsShow: false,
      // 评论页显示状态
      commentIsShow: false,
      // mv页显示状态
      mvIsShow: false,
    }
  },
  components: {
    musicBox,
    musicBar,
    musicSearch,
    musicArtist,
    musicAlbum,
    musicComment
  },
  computed: {
    // 获取歌曲信息
    getSongInfo () {
      return this.musicList[this.index];
    },
    // 获取歌曲时间
    getTime () {
      return {
        'current': Math.floor(this.currentTime),
        'duration': Math.floor(this.duration)
      }
    },
    // 获取歌曲列表
    getMusicList () {
      if(this.musicList.length){
        return this.musicList;
      }else{
        return []
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
      if(leng==0) return false;

      if(this.loopType === 'random'){
        _index = Math.floor(Math.random()*leng);
      }else if(this.loopType === 'one'){
        this.myAudio.play();
        return false;
      }else {
        if(type === 'prev'){
          _index = this.index-1;
          if(_index<0) _index = leng-1;
        }else{
          _index = this.index +1;
          if(_index >= leng) _index = 0;
        }
      }
      this.index = _index;
      this.getSong();
    },
    // 通过索引切歌
    changeMusicByIndex (_index) {
      let leng = this.musicList.length;
      if(leng==0) return false;
      if(typeof _index ==='number'){
        if(_index<0){
          _index = 0;
        }else if(_index>=leng){
          _index = leng-1;
        }
      }else{
        _index = 0;
      }
      this.index = _index;
      //this.listIsShow = 'hide';
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
          alert(error);
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
    },
    // 进度条控制
    eventControl (data) {
      if(this.state==='play'){
        if(data.type==='down'){
          clearInterval(this.timer);
        }
        else if(data.type==='up'){
          this.myAudio.currentTime = this.duration*data.data/100;
          this.timer = setInterval(()=>{
            this.currentTime = this.myAudio.currentTime;
          },500);
        }
      }else{
        if(data.type==='up') this.myAudio.currentTime = this.duration*data.data/100;
      }
    },
    // 显示和隐藏歌曲列表
    showList (str) {
      if(str === 'show'){
        this.listIsShow = 'show';
      }else{
        this.listIsShow = 'hide';
      }
    },
    // 移除歌曲
    removeSong (_index) {
      let leng = this.musicList.length;
      if(leng == 1){
        this.index = 0;
        alert('请保留一首歌曲');
        return false;
      }
      this.musicList.splice(_index,1);
      if(_index < this.index){
        this.index --;
      }else if(_index == this.index){
        _index --;
        if(_index<0) _index = 0;
        this.index = _index;
        this.getSong();
      }
    },
    // 显示页面
    showPage (str) {
      if(!str) return;
      this[str] = true;
      this.$refs[str].beforeShow();
    },
    // 关闭页
    closePage (str) {
      if(!str) return;
      this[str] = false;
    },
    // 添加歌曲到列表
    addSong (data){
      if(!data) return;
      let test = this.musicList.some(value=>{
        return value.id == data.id;
      });
      if(test) return;
      this.musicList.push(data);
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
    width: 10rem;
    height: 100%;
    overflow: hidden;
    position: relative;
    .page-fade-enter-active, .page-fade-leave-active {
      transition: opacity 0.3s;
    }
    .page-fade-enter, .page-fade-leave-active {
      opacity: 0;
    }
  }
  @mixin headerHeight($n: 1){
    height: 80px*$n;
    font-size: 20px*$n;
    line-height: 80px*$n;

    .music-search-btn {
      width: 40px*$n;
      height: 40px*$n;
      line-height: 40px*$n;
      font-size: 30px*$n;
      top: 20px*$n;
      right: 10px*$n;
    }
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

    .music-search-btn {
      display: block;
      position: absolute;
      text-align: center;
      cursor: pointer;
    }

    &:after {
      position: absolute;
      bottom: 0;
      content: '';
      display: block;
      width: 100%;
      height: 1px;
      background: linear-gradient( left, rgba(255,255,255,0), rgba(255,255,255,0.3), rgba(255,255,255,0) );
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
    top: -0.5rem;
    left: -0.5rem;
    bottom: -0.5rem;
    right: -0.5rem;
    background-size: cover;
    background-position: center center;
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

  /* 歌曲列表 */
  @mixin musicList($n: 1){
    .music-list-box {
      border-width: 2px*$n;
      padding: 10px*$n 0;
      font-size: 14px*$n;

      li {
        height: 40px*$n;
        line-height: 40px*$n;
        padding: 0 20px*$n;

        em {
          font-size: 12px*$n;
        }
        span {
          width: 40px*$n;
          height: 40px*$n;
          right: 10px*$n;
          font-size: 20px*$n;
        }
      }
    }
  }
  #music-list{
      width: 100%;
      height: 100%;
      position: absolute;
      bottom: -100%;
      left: 0;
      z-index: 20;
      color: #333;
      @include musicList;

      .music-list-box {
          width: 100%;
          height: 60%;
          background: #fff;
          position: absolute;
          bottom: -100%;
          left: 0;
          overflow-y: auto;
          border-top: 2px solid #f66;
          box-shadow: 0 0 10px #333;
          transition: bottom .3s ease-out;

          li {
            position: relative;
            border-bottom: 1px solid #efefef;

            p {
              width: 100%;
              height: 100%;
              overflow: hidden;
            }
            em {
              color: #aaa;
            }
            span {
              position: absolute;
              text-align: center;
              color: red;
              top: 0;
              background: #fff;
              cursor: pointer;
            }
          }
      }
  }
  #music-list.show {
    bottom: 0;

    .music-list-box {
      bottom: 0;
    }
    #music-list-bg {
      opacity: 1;
    }
  }
  #music-list-bg {
      width: 100%;
      height: 100%;
      background: rgba(0,0,0,.4);
      opacity: 0;
      transition: opacity 0.3s;
  }
  [data-dpr="2"] #music-list {
    @include musicList(2);
  }
  [data-dpr="3"] #music-list {
    @include musicList(3);
  }
</style>
