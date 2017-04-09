<template>
    <div id="progress-bar">
        <p class="progress-bg"></p>
        <div class="progress-box" :style="{'width':pWidth+'%'}">
            <p id="progress-icon" :class="playState"></p>
        </div>
    </div>
</template>
<script>
    export default {
        props: {
            pWidth: {
                type: Number,
                default: '0'
            },
            playState: {
                type: String,
                default: 'pause'
            }
        },
        mounted() {
            const self = this;
            const oBar = document.querySelector('#progress-bar');
            const oBox = document.querySelector('.progress-box');
            oBar.onmousedown = function(evnet){
                event.stopPropagation();
                self.$emit('control',{type:'down'});
                const w = oBar.clientWidth;
                const left = oBar.getBoundingClientRect().left;
                let x = event.offsetX;
                oBox.style.width = x+'px';
                document.onmousemove = ev =>{
                    ev.stopPropagation();
                    x = ev.pageX-left;
                    x = Math.min(x, w);
                    oBox.style.width = x+'px';
                };
                document.onmouseup = e =>{
                    e.stopPropagation();
                    x = e.pageX-left;
                    x = Math.min(x, w);
                    //oBox.style.width = x+'px';
                    self.$emit('control',{type:'up',data:(x*100/w).toFixed(2)});
                    //self.pWidth = (x*100/w).toFixed(2);
                    document.onmouseup = document.onmousemove = null;
                }
            }
            oBar.ontouchstart = function(event){
                self.$emit('control',{type:'down'});
                const w = oBar.clientWidth;
                const left = oBar.getBoundingClientRect().left;
                let x = event.targetTouches[0].offsetX;
                oBox.style.width = x+'px';

                document.ontouchmove = ev => {
                    x = ev.targetTouches[0].pageX-left;
                    x = Math.min(x, w);
                    oBox.style.width = x+'px';
                };
                document.ontouchend = e =>{
                    x = e.changedTouches[0].pageX-left;
                    x = Math.min(x, w);
                    self.$emit('control',{type:'up',data:(x*100/w).toFixed(2)});
                    document.ontouchend = null;
                };
                return false;
            }
        }
    }
</script>

<style lang="scss">
    $barHeight: 3px;
    $barPadding: 3px;
    $boxShadow: 5px;
    $barIcon: 12px;
    $barIconShadow: 5px;
    
    @mixin progress($n:1){
        height: $barHeight*$n;
        padding: $barPadding*$n 0;

        .progress-bg {
            border-radius: $barHeight*$n/2;
        }
        .progress-box {
            top: $barPadding*$n;
            height: $barHeight*$n;
            border-radius: $barHeight*$n/2;
            box-shadow: 0 0 $boxShadow*$n rgba(255,255,255,0.5);

        }
        
        #progress-icon {
            width: $barIcon*$n;
            height: $barIcon*$n;
            right: -$barIcon*$n/2;
            margin-top: -$barIcon*$n/2;
            box-shadow: 0 0 ($boxShadow+3)*$n #fff;
        }
    }
    @mixin iconShadow($n:1){
        @keyframes shadow_#{$n} {
            0% {
                box-shadow: 0 0 $barIconShadow*$n rgba(255,255,255,0.5), 0 0 ($barIconShadow+3px)*$n #fff;
            }
            50% {
                box-shadow: 0 0 ($barIconShadow+3px)*$n rgba(255,255,255,0.5), 0 0 ($barIconShadow+10px)*$n #fff;
            }
            100% {
                box-shadow: 0 0 $barIconShadow*$n rgba(255,255,255,0.5), 0 0 ($barIconShadow+3px)*$n #fff;
            }
        }
    }

    #progress-bar {
        width: 100%;
        position: absolute;
        top: 0;
        bottom: 0;
        left: 0;
        top: 0;
        margin: auto;
        cursor: pointer;
        @include progress;

        .progress-bg {
            width: 100%;
            height: 100%;
            background: rgba(120,120,120,.5);
        }
        .progress-box {
            position: absolute;
            left: 0;
            width: 0;
            background: rgba(255,255,255,0.8);

        }

        #progress-icon {
            position: absolute;
            top: 50%;
            background: #fff;
            border-radius: 50%;
            cursor: pointer;

            &.play {
                animation: shadow_1 3s linear infinite;
            }
            &.waite {
                animation: iconRotate 2s linear infinite;
                &:after{
                    content: '';
                    display: block;
                    width: 30%;
                    height: 30%;
                    background: rgba(100,100,100,.8);
                    position: absolute;
                    right: 20%;
                    bottom: 20%;
                    border-radius: 50%;
                }
            }
        }
    }
    @include iconShadow;
    @include iconShadow(2);
    @include iconShadow(3);

    @keyframes iconRotate {
        from {
            transform: rotate(0deg);
        }
        to {
            transform: rotate(360deg);
        }
    }

    [data-dpr="2"] #progress-bar {
        @include progress(2);

        #progress-icon {
            &.play {
                animation: shadow_2 3s linear infinite;
            }
        }
    }
    [data-dpr="3"] #progress-bar {
        @include progress(3);

        #progress-icon {
            &.play {
                animation: shadow_3 3s linear infinite;
            }
        }
    }
</style>