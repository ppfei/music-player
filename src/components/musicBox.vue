<template>
	<div id="music-box">
		<transition name="fade">
			<div class="song-main" v-show="!isShowLyric">
				<div class="song-icon" @click="isShowLyric = true"><img :class="{'play':isplay=='play'}" :src="activeMusic.al.picUrl" alt="song-icon"></div>
				<div class="song-info">
					<ul class="flex-h">
						<li class="song-ar" @click="showPage('artistIsShow')"><i class="iconfont">&#xe6f4;</i> {{ activeMusic.ar.name }}</li>
						<li class="song-al" @click="showPage('albumIsShow')"><i class="iconfont">&#xe6e4;</i> {{ activeMusic.al.name }}</li>
						<li class="song-com" @click="showPage('commentIsShow')"><i class="iconfont">&#xe702;</i> 评论</li>
					</ul>
				</div>
			</div>
		</transition>
		<transition name="fade">
			<div class="lyric-box" v-show="isShowLyric" @click="isShowLyric = false">
				<div class="lyric-body">
					<ul class="lyric-list" :style="{'top': -activeIndex*26*getDpr()+'px'}">
						<li v-for="(value,index) of lyric" :class="{'active': index==activeIndex}">{{ value }}</li>
					</ul>
				</div>
				<div style="position:absolute;left:200%">{{ updataLrc }}</div>
			</div>
		</transition>
	</div>
</template>

<script>
	import axios from 'axios'
	export default {
		props: {
			activeMusic: Object,
			isplay: String,
			currentTime: Number
		},
		data () {
			return {
				isShowLyric: false,
				time: [],
				lyric: []
			}
		},
		computed: {
			// 当歌曲id变化时自动获取歌词
			updataLrc () {
				let id = this.activeMusic.id ? this.activeMusic.id : 0;
				if(!id) return id;
				this.getLyric(id);
				return id;
			},
			// 计算当前高亮歌词的索引
			activeIndex () {
				let cTime = this.currentTime-200; // -200是歌词提前200毫秒
				let leng = this.time.length;
				if(!leng) return 0;
				let index = this.time.findIndex(value=>{
					return value >= cTime;
				});
				if(index<=0){
					index = cTime>0 && cTime>this.time[0] ? leng : 1;
				}
				return index-1;
			}
		},
		methods: {
			showPage (str) {
				this.$emit('event-showPage', str);
			},
			// 获取歌词
			getLyric (id) {
				this.time = [];
				this.lyric = [];
				let self = this;
				if( !(typeof id === 'number') ) id = this.musicList[this.index].id;
				let url = 'https://api.imjad.cn/cloudmusic/?type=lyric&id='+id;
				axios.get(window.location.origin.replace(/[0-9]+$/,'8081')+'/proxy.php',{
						params:{ url: url }
					})
					.then(function (response) {
						if(response.data.code != 200){
							self.lyric = ['歌词获取失败，静心聆听音乐'];
						}
						let str = response.data.lrc.lyric;
						let sArr = str.split('\n');
						sArr.pop();
						sArr.forEach(value=>{
							let vArr = value.split(']');
							self.lyric.push(vArr[1]);
							let timeStr = vArr[0].replace('[','');
							self.time.push(self.toTime(timeStr));
						});

					})
					.catch(function (error) {
						self.lyric = ['歌词获取失败，静心聆听音乐1'];
					});
			},
			// 转换成时间毫秒数
			toTime (timeStr) {
				if(!timeStr) return 0;
				let tArr = timeStr.split(':');
				let tNum = Math.floor(tArr[0]*60*1000 + tArr[1]*1000);
				return tNum;
			},
			// 获取像素dpr
			getDpr () {
				let dpr = document.documentElement.getAttribute('data-dpr');
				if(!dpr) dpr = 1;
				return dpr;
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
		.lyric-box {
			padding: 20px*$n 0;

			.lyric-list {
				li {
					height: 26px*$n;
					line-height: 26px*$n;
					font-size: 16px*$n;
				}
				li.active {
					height: 36px*$n;
					line-height: 36px*$n;
					font-size: 18px*$n;
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

		.fade-enter-active, .fade-leave-active {
			transition: opacity 0.3s;
		}
		.fade-enter, .fade-leave-active {
			opacity: 0;
		}

		.song-main{
			width: 100%;
			height: 100%;
			position: absolute;
			top: 0;
			left: 0;
		}
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
		.lyric-box {
			width: 100%;
			height: 100%;
			box-sizing: border-box;
			position: absolute;
			top: 0;
			left: 0;
			
			.lyric-body {
				width: 100%;
				height: 100%;
				position: relative;
				overflow: hidden;
			}
			.lyric-list {
				width: 100%;
				height: 9999px;
				position: absolute;
				left: 0;
				top: 0;
				text-align: center;
				margin-top: 28%;
				transition: top .3s;

				li {
					color: #ccc;
				}
				li.active {
					color: #fff;
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