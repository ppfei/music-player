<template>
  <div id="music-box">
    <div class="song-icon"><img :src="imgsrc" alt="song-icon"></div>

  </div>
</template>

<script>
  export default {
    props: {
      imgsrc: String,
    },
    data () {
      return {
        
      }
    }
  }
</script>

<style lang="scss">
  @mixin boxMargin($n: 1){
    bottom: 100px*$n;
    top: 45px*$n;
  }
  $songIconBottom: 200px;
  $imgBorder: 3px;

  #music-box {
    width: 100%;
    position: absolute;
    z-index: 10;
    left: 0;
    background: red;
    @include boxMargin();

    .song-icon {
      width: 100%;
      position: absolute;
      top: 0;
      left: 0;
      bottom: $songIconBottom;
      background: blue;

      img{
        width: 6rem;
        height: 6rem;
        border-radius: 50%;
        overflow: hidden;
        background: lightgreen;
        position: absolute;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        margin: auto;
        border: $imgBorder solid #eee;
      }
    }
  }
  [data-dpr="2"] #music-box {
    @include boxMargin(2);
    .song-icon {
      bottom: $songIconBottom*2;
      img {
        border: $imgBorder*2 solid #eee;
      }
    }
  }
  [data-dpr="3"] #music-box {
    @include boxMargin(3);
    .song-icon {
      bottom: $songIconBottom*3;
      img {
        border: $imgBorder*3 solid #eee;
      }
    }
  }
</style>