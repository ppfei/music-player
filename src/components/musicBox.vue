<template>
  <div id="music-box">
    <div class="song-icon"><img :class="{'play':isplay=='play'}" :src="songInfo.al.picUrl" alt="song-icon"></div>
    <div class="song-info">
      <ul class="flex-h">
        <li class="song-ar" @click="showPage('artistIsShow')"><i class="iconfont">&#xe6f4;</i> {{ songInfo.ar.name }}</li>
        <li class="song-al" @click="showPage('albumIsShow')"><i class="iconfont">&#xe6e4;</i> {{ songInfo.al.name }}</li>
        <li class="song-com" @click="showPage('commentIsShow')"><i class="iconfont">&#xe702;</i> 评论</li>
      </ul>
    </div>
  </div>
</template>

<script>
  export default {
    props: {
      imgsrc: String,
      songInfo: Object,
      isplay: String
    },
    data () {
      return {
        
      }
    },
    methods: {
      showPage (str) {
        this.$emit('event-showPage', str);
      }
    }
  }
</script>

<style lang="scss">
  $songIconBottom: 60px;
  $imgBorder: 3px;
  $liFontSize: 14px;
  
  @mixin musicBox($n: 1){
    bottom: 120px*$n;
    top: 80px*$n;
    .song-icon {
      bottom: $songIconBottom*$n;
      img {
        border: $imgBorder*$n solid #eee;
      }
    }
    .song-info {
      height: $songIconBottom*$n;

      ul {
         li {
           font-size: $liFontSize*$n;
         }
         .iconfont {
            font-size: 22px*$n;
            top: 3px*$n;
         }
      }
    }
  }

  #music-box {
    width: 100%;
    position: absolute;
    z-index: 10;
    left: 0;
    @include musicBox();

    .song-icon {
      width: 100%;
      position: absolute;
      top: 0;
      left: 0;

      img{
        width: 6rem;
        height: 6rem;
        border-radius: 50%;
        overflow: hidden;
        display: block;
        position: absolute;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        margin: auto;
        animation: imgRotate 30s linear infinite;
        animation-play-state: paused;

        &.play {
          animation-play-state: running;
        }
      }
    }
    .song-info {
      position: absolute;
      bottom: 0;
      left: 0;

      ul {
        width: 100%;
        height: 100%;
        flex-flow: row nowrap;
        text-align: center;
        overflow: hidden;
         
         li {
           width: 3rem;
           overflow: hidden;
           text-overflow: ellipsis;
           white-space: nowrap;
           cursor: pointer;
         }
         li.song-al, li.song-ar {
           width: 3.5rem;
         }
         .iconfont {
           position:relative;
         }
      }
    }
  }

  @keyframes imgRotate {
    from {
      transform: rotate(0deg);
    }
    to {
      transform: rotate(360deg);
    }
  }

  [data-dpr="2"] #music-box {
    @include musicBox(2);
    
  }
  [data-dpr="3"] #music-box {
    @include musicBox(3);
  }
</style>