<template>
    <div id="music-bar">
        <div class="bar-top flex-h">
            <p class="currentTime" v-text="convertTime(time.current)">00:00</p>
            <div id="progress">
                <progress-bar :pWidth="getProgressWidth" :playState="playState"></progress-bar>
            </div>
            <p class="durationTime" v-text="convertTime(time.duration)">00:00</p>
        </div>
        <div class="bar-bottom flex-h">
            <div id="loopType" class="icon"><i class="iconfont" @click="changeLoop" v-html="loopType[loop]"></i></div>
            <div id="control-prev" class="icon"><i class="iconfont" @click="prev">&#xe706;</i></div>
            <div id="control-play" class="icon"><i class="iconfont" @click="play" v-html="controlPlay[playState]"></i></div>
            <div id="control-next" class="icon"><i class="iconfont" @click="next">&#xe706;</i></div>
            <div id="songList" class="icon"><i class="iconfont">&#xe6ec;</i></div>
        </div>
    </div>
</template>

<script>
    import progressBar from './progressBar'
    export default {
        props: {
            loop: {
                tpye: String,
                default: 'loop'
            },
            playState: {
                type: String,
                default: 'pause'
            },
            time: {
                type: Object,
                default: {'current':0,'duration':1}
            }
        },
        components: {
            progressBar
        },
        data () {
            return {
                loopType: {
                    'loop': '&#xe6f9;',
                    'random': '&#xe716;'
                },
                controlPlay: {
                    'play': '&#xe6f7;',
                    'pause': '&#xe713;',
                    'waite': '&#xe6f7;'
                },
            }
        },
        computed: {
            // 计算进度条长度
            getProgressWidth () {
                let c = this.time.current;
                let d = this.time.duration;
                if(typeof c !== 'number') c = 0;
                if(typeof d !== 'number' || d == 0) d = 1;
                let percent = (c*100/d).toFixed(2);
                percent = Math.min(percent, 100);
                return percent+'%';
            }
        },
        methods: {
            // 播放
            play () {
                this.$emit('event-play');
            },
            // 上一首
            prev () {
                this.$emit('event-change', 'prev');
            },
            // 下一首
            next () {
                this.$emit('event-change', 'next');
            },
            // 改变循环模式
            changeLoop () {
                this.$emit('event-loop');
            },
            // 显示播放时间
            convertTime: (num) => {
                if(typeof num !== 'number') return '--:--';
                let m = Math.floor(num/60);
                m = m<10 ? '0'+m : ''+m;
                let s = Math.floor(num%60);
                s = s<10 ? '0'+s : ''+s;
                return m+':'+s;
            }
        }
    }
</script>

<style lang="scss">
    $height: 120px;
    $barTopHeight: 30px;
    $timeWidth: 1.5rem;

    @mixin musicBar($n:1){
        height:$height*$n;

        &:after {
            height: $n;
        }

        .bar-top {
            height: $barTopHeight*$n;
            line-height: $barTopHeight*$n;
        }
        .bar-bottom {
            height: ($height - $barTopHeight)*$n;
            line-height: ($height - $barTopHeight)*$n;

            .iconfont {
                font-size: 36px*$n;

                &:hover {
                    text-shadow: 0 0 10px*$n #fff;
                }
            }
        }

        #songType, #songList {
            .iconfont {
                font-size: 26px*$n;
            }
        }
        #control-play {
            .iconfont {
                font-size: 46px*$n;
            }
        }
    }

    #music-bar {
        @include musicBar;
        width: 100%;
        position: absolute;
        z-index: 10;
        bottom: 0px;
        user-select: none;

        .bar-top {
            overflow:hidden;
            text-align: center;

            .currentTime, .durationTime {
                width: $timeWidth;
            }
        }
        .bar-bottom {
            width: 100%;
            overflow: hidden;
            text-align: center;

            .icon {
                width: 20%;
                height: 100%;
            }
            .iconfont {
                cursor: pointer;
            }
        }
        #control-prev {
            transform: rotate(180deg);
        }
    }
    
    #progress {
        width: 10rem - 2*$timeWidth;
        position: relative;
    }

    [data-dpr="2"] #music-bar {
        @include musicBar(2);
    }
    [data-dpr="3"] #music-bar {
        @include musicBar(3);
    }
</style>