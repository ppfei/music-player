<template>
    <div id="music-album">
        <div class="music-header">
            <span class="back" @click="close">&times;</span>
            <b class="title">专辑: {{ staticMusic.al.name }}</b>
        </div>
        <div class="music-body">
            <div class="music-des">
                <img :src="staticMusic.al.picUrl" :alt="staticMusic.al.name">
                <div class="des-box">
                    <p class="des-title">{{ album.name }}</p>
                    <p class="des-info">{{ album.description }}</p>
                </div>
            </div>
            <ul ref="list" class="music-list">
                <li class="flex-h" v-for="(item, index) of searchList" :class="{'check': isCheck(item)}">
                    <i v-if="activeMusic.id == item.id" class="iconfont">&#xe6f4;</i>
                    <p @click="addAndChangeMusic(item)">{{ item.name }}<em> - {{ item.ar.name }}</em></p>
                    <span @click="addSong(item)">+</span>
                </li>
            </ul>
        </div>
        <transition name="fade-out">
            <loading v-if="isLoading"></loading>
        </transition>
    </div>
</template>

<script>
    import axios from 'axios'
    import loading from './loading'
    export default {
        props:{
            musicList: {
                type: Array,
                default: []
            },
            activeMusic: {
                type: Object,
                default: {
                    id: 0,
                    name: '',
                    al: {
                        id: 0,
                        name: '',
                        picUrl: '',
                    },
                    ar: {
                        id: 0,
                        name: ''
                    },
                    mv: 0
                }
            }
        },
        components: {
            loading
        },
        data () {
            return {
                searchList: [],
                albumId: 0,
                album: {
                    id: 0,
                    name: '',
                    picUrl: '',
                    description: ''
                },
                staticMusic: {
                    id: 0,
                    name: '',
                    al: {
                        id: 0,
                        name: '',
                        picUrl: '',
                    },
                    ar: {
                        id: 0,
                        name: ''
                    },
                    mv: 0
                },
                isLoading: true
            }
        },
        methods: {
            beforeShow () {
                if(this.albumId == this.activeMusic.al.id) return;
                this.albumId = this.activeMusic.al.id;
                this.staticMusic = this.activeMusic;
                this.getAlbum(this.albumId);
            },
            close () {
                this.$emit('event-closePage');
            },
            // 歌手详情
            getAlbum (id) {
                this.isLoading = true;
                let self = this;
                let url = 'https://api.imjad.cn/cloudmusic/?type=album&id='+id;
                axios.get('http://xiaodidiao.com/proxy.php',{
                        params:{ url: url }
                    })
                    .then(function (response) {
                        if(response.data.code != 200){
                            console.log(response.data);
                            console.log('搜索失败');
                            return false;
                        };
						self.$refs.list.scrollTop = 0; // 返回顶部
                        let data = response.data;
                        self.album = {
                            'id': data.album.id,
                            'name': data.album.name,
                            'picUrl': data.album.picUrl,
                            'description': data.album.description
                        };
                        self.searchList = [];
                        data.songs.forEach((value,index)=>{
                            let result = {
                                'id': value.id,
                                'name': value.name,
                                'al': {
                                    'id': value.al.id,
                                    'name': value.al.name,
                                    'picUrl': value.al.picUrl
                                },
                                'ar': {
                                    'id': value.ar[0].id,
                                    'name': value.ar[0].name
                                },
                                'mv': value.mv
                            };
                            self.searchList.push(result);
                        });
                        setTimeout(()=>{self.isLoading = false},500);
                    })
                    .catch(function (error) {
                        alert(error);
                    });
            },
            // 添加歌曲
            addSong (data){
                this.$emit('event-addSong',data);
            },
            // 是否在播放列表中存在该歌曲
            isCheck (item){
                if(!this.musicList.length) return false;
                return this.musicList.some(value=>{
                    return value.id == item.id;
                });
            },
            //
            addAndChangeMusic (data) {
                this.$emit('event-addSong',data);
                let len = this.musicList.length;
                let _index = 0;
                this.musicList.some((value,index)=>{
                    _index = index;
                    return value.id == data.id;
                });
                this.$emit('event-changeMusic', _index);
            }
        }
    }
</script>

<style lang="scss">
    @mixin musicAlbum ($n:1){
        font-size: 16px*$n;
        .music-header {
            height: 45px*$n;
            line-height: 45px*$n;

            .back {
                padding: 0 20px*$n;
                font-size: 20px*$n;
            }
            b {
                font-size: 18px*$n;
                margin-left: -50px*$n;
            }
        }
        .music-body {
            padding: 45px*$n 20px*$n 0;
            margin-top: -45px*$n;
            .music-des {
                height: 120px*$n;
                padding: 10px*$n 0;
                img {
                    height: 120px*$n;
                    margin-right: 10px*$n;
                }
                .des-box {
                    height: 120px*$n;
                    line-height: 20px*$n;
                    font-size: 12px*$n;
                    .des-title {
                        height: 20px*$n;
                        font-size: 16px*$n;
                    }
                    .des-info {
                        height: 100px*$n;
                    }
                }
            }
            .music-list {
                li {
                    height: 40px*$n;
                    padding: 0 10px*$n;
                    line-height: 40px*$n;

                    em {
                        font-size: 12px*$n;
                    }
                    .iconfont {
                        font-size: 12px*$n;
                        left: -10px*$n;
                    }
                    span {
                        width: 40px*$n;
                        height: 40px*$n;
                        font-size: 20px*$n;
                    }
                }
            }
        }
    }

    #music-album {
        width: 100%;
        height: 100%;
        position: absolute;
        top: 0;
        left: 0;
        z-index: 20;
        color: #333;
        background: #fff;

        @include musicAlbum;

        .fade-out-leave-active {
            transition: opacity 0.3s;
        }
        .fade-out-leave-active {
            opacity: 0;
        }

        .music-header {
            width: 100%;
            position: relative;
            z-index: 20;
            text-align: center;
            overflow: hidden;
            border-bottom: 1px solid #eee;
            background: #fff;

            .back {
                font-family: 'Microsoft Yahei';
                float: left;
                cursor: pointer;
                color: red;
            }
        }
        .music-body {
            width: 100%;
            box-sizing: border-box;
            height: 100%;
            overflow-y: auto;
            .music-des {
                width: 100%;
                img {
                    width: auto;
                    float: left;
                }
                .des-box {
                    overflow: hidden;
                    .des-info {
                        color: #888;
                        overflow-y: auto;
                    }
                }
            }
            .music-list {
                width: 100%;

                li {
                    position: relative;
                    border-bottom: 1px solid #efefef;
                    &.check {
                        color: #f66;
                        em {
                            color: #faa;
                        }
                    }

                    p {
                        width: 100%;
                        height: 100%;
                        overflow: hidden;
                    }
                    em {
                        color: #aaa;
                    }
                    .iconfont {
                        position: absolute;
                        color: #f66;
                    }
                    span {
                        position: absolute;
                        text-align: center;
                        color: #999;
                        top: 0;
                        right: 0;
                        background: #fff;
                        cursor: pointer;
                    }
                }
            }
        }
        .music-result {
            position: absolute;
            left: -100%;
            top: -100%;
        }
    }
    [data-dpr="2"] #music-album {
        @include musicAlbum(2);
    }
    [data-dpr="3"] #music-album {
        @include musicAlbum(3);
    }
</style>