<template>
    <div id="music-search">
        <div class="music-search-header">
            <span class="back" @click="close">&times;</span>
            <input type="search" v-model="searchStr" id="search" placeholder="搜索歌曲">
            <label for="search"><i class="iconfont">&#xe6f1;</i></label>
        </div>
        <div class="music-search-body">
            <ul class="music-search-list">
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
        <div class="search-result">{{ searchStr }} : {{ getStr }}</div>
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
                searchList: [
                    {
                        name:'xxx',
                        ar:{
                            name:'xxx'
                        }
                    }
                ],
                searchStr: '',
                isLoading: false
            }
        },
        computed: {
            getStr (){
                this.searchStr = this.searchStr.replace(/^\s*/,'');
                if(this.searchTimer) clearTimeout(this.searchTimer);
                if(this.searchStr == ''){
                    this.searchList = [];
                    return false;
                }
                this.searchTimer = setTimeout(()=>{
                    this.searchSong(this.searchStr);
                },300);
                return true;
            } 
        },
        methods: {
            close () {
                this.$emit('event-closePage');
            },
            // 搜索歌曲
            searchSong (str) {
                let self = this;
                this.searchList = [];
                if( str == '' ) return;
                this.isLoading = true;
                let url = 'https://api.imjad.cn/cloudmusic/?type=search&s='+str;
                axios.get('http://xiaodidiao.com/proxy.php',{
                        params:{ url: url }
                    })
                    .then(function (response) {
                        if(response.data.code != 200){
                            console.log(response.data);
                            console.log('搜索失败');
                            return false;
                        };
                        if(!response.data.result.songCount) return false;
                        response.data.result.songs.forEach((value,index)=>{
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
                            self.isLoading = false;
                        });
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
    @mixin musicSearch ($n:1){
        font-size: 16px*$n;
        .music-search-header {
            height: 45px*$n;
            line-height: 45px*$n;

            .back {
                padding: 0 20px*$n;
                font-size: 20px*$n;
            }
            #search {
                padding: 0 10px*$n;
            }
            label {
                padding: 0 20px*$n;
                .iconfont {
                    font-size: 22px*$n;
                }
            }
        }
        .music-search-body {
            padding: 45px*$n 0 0 0;
            margin-top: -45px*$n;
            .music-search-list {
                li {
                    height: 40px*$n;
                    line-height: 40px*$n;
                    padding: 0 30px*$n;

                    em {
                        font-size: 12px*$n;
                    }
                    .iconfont {
                        font-size: 12px*$n;
                        left: 10px*$n;
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
    }

    #music-search {
        width: 100%;
        height: 100%;
        position: absolute;
        top: 0;
        left: 0;
        z-index: 20;
        color: #333;
        background: #fff;

        .fade-out-leave-active {
            transition: opacity 0.3s;
        }
        .fade-out-leave-active {
            opacity: 0;
        }

        @include musicSearch;

        .music-search-header {
            width: 100%;
            position: relative;
            text-align: center;
            overflow: hidden;
            border-bottom: 1px solid #eee;

            .back {
                font-family: 'Microsoft Yahei';
                float: left;
                cursor: pointer;
                color: red;
            }
            #search {
                width: 70%;
                height: 70%;
                border: none;
                outline: none;
                border-bottom: 1px solid transparent;
                transition: border-color 0.3s linear;
                
                &:focus {
                    border-color: #aaa;
                }
            }
            label {
                float: right;
                cursor: pointer;
            }
        }
        .music-search-body {
            width: 100%;
            box-sizing: border-box;
            height: 100%;
            .music-search-list {
                width: 100%;
                height: 100%;
                overflow-y: auto;

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
                        background: #fff;
                        cursor: pointer;
                    }
                }
            }
        }
        .search-result {
            position: absolute;
            left: -100%;
            top: -100%;
        }
    }
    [data-dpr="2"] #music-search {
        @include musicSearch(2);
    }
    [data-dpr="3"] #music-search {
        @include musicSearch(3);
    }
</style>