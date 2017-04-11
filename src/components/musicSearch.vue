<template>
    <div id="music-search">
        <div class="music-search-header">
            <span class="back" @click="close">&times;</span>
            <input type="search" v-model="searchStr" id="search" placeholder="搜索歌曲">
            <label for="search"><i class="iconfont">&#xe6f1;</i></label>
        </div>
        <div class="music-search-body">
            <ul class="music-search-list">
                <li class="flex-h" v-for="(item, index) of searchList" :class="{'check': isCheck(item)}"><p>{{ item.name }}<em> - {{ item.ar.name }}</em></p><span @click="addSong(item)">+</span></li>
            </ul>
        </div>
        <div class="search-result">{{ searchStr }} : {{ getStr }}</div>
    </div>
</template>

<script>
    import axios from 'axios'
    export default {
        props:{
            musicList: {
                type: Array,
                default: []
            }
        },
        data () {
            return {
                searchList: [
                    {
                        name:'xxx',
                        ar:{
                            name:'xxx'
                        }
                    },
                    {
                        name:'xxx',
                        ar:{
                            name:'xxx'
                        }
                    },
                    {
                        name:'xxx',
                        ar:{
                            name:'xxx'
                        }
                    }
                ],
                searchStr: '',
            }
        },
        computed: {
            getStr (){
                this.searchStr = this.searchStr.trim();
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
                this.$emit('event-closeSearch');
            },
            // 搜索歌曲
            searchSong (str) {
                let self = this;
                if( str == '' ) return;
                let url = 'https://api.imjad.cn/cloudmusic/?type=search&s='+str;
                axios.get(window.location.origin.replace(/[0-9]+$/,'8081')+'/proxy.php',{
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
            padding: 45px*$n 0 0 10px*$n;
            margin-top: -45px*$n;
            .music-search-list {
                padding-right: 10px*$n;
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