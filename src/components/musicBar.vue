<template>
    <div id="music-bar">music-bar
    </div>
</template>

<script>
    export default {
        data () {
            return {
              
            }
        }
    }
</script>

<style lang="scss">
    $height: 100px;
    #music-bar {
        width: 100%;
        height: $height;
        box-shadow: 0 0 5px blue;
        position: absolute;
        z-index: 10;
        bottom: 0px;
    }
    [data-dpr="2"] #music-bar {
        height:$height*2;
    }
    [data-dpr="3"] #music-bar {
        height:$height*3;
    }
</style>