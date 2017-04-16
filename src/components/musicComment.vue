<template>
    <div id="music-comment">
        <div class="music-header">
            <span class="back" @click="close">&times;</span>
            <b class="title">歌曲: {{ staticMusic.name }}</b>
        </div>
        <div class="music-body">
            <div class="music-des">
                <img :src="staticMusic.al.picUrl" :alt="staticMusic.name">
                <div class="des-box">
                    <p class="des-title">{{ staticMusic.name }}</p>
                    <p class="des-info">{{ staticMusic.ar.name }}</p>
                </div>
            </div>
            <h3>精彩评论</h3>
            <ul class="comment-list">
                <li v-for="(item, index) of hotComments">
                    <img class="userImg" :src="item.user.avatarUrl">
                    <div class="comment-box">
                        <div class="comment-header">
                            <p class="comment-title flex-h">
                                <span>{{ item.user.nickname }}</span>
                                <span class="comment-like">{{ item.likedCount }} <em>赞</em></span>
                            </p>
                            <p class="comment-time">{{ toTime(item.time) }}</p>
                        </div>
                        <div class="comment-body">
                            <p class="comment-con"><span v-if="item.beReplied.length">回复<em>@{{ item.beReplied.length ? item.beReplied[0].user.nickname : '' }}</em>：</span>{{ item.content }}</p>
                            <p class="comment-beReplied" v-if="item.beReplied.length"><span><em>@{{ item.beReplied.length ? item.beReplied[0].user.nickname : '' }}</em>：</span>{{ item.beReplied.length ? item.beReplied[0].content : '' }}</p>
                        </div>
                    </div>
                </li>
            </ul>
            <h3>最新评论</h3>
            <ul class="comment-list">
                <li class="flex" v-for="(item, index) of comments">
                    <img class="userImg" :src="item.user.avatarUrl">
                    <div class="comment-box">
                        <div class="comment-header">
                            <p class="comment-title flex-h">
                                <span>{{ item.user.nickname }}</span>
                                <span class="comment-like">{{ item.likedCount }} <em>赞</em></span>
                            </p>
                            <p class="comment-time">{{ toTime(item.time) }}</p>
                        </div>
                        <div class="comment-body">
                            <p class="comment-con"><span v-if="item.beReplied.length">回复<em>@{{ item.beReplied.length ? item.beReplied[0].user.nickname : '' }}</em>：</span>{{ item.content }}</p>
                            <p class="comment-beReplied" v-if="item.beReplied.length"><span><em>@{{ item.beReplied.length ? item.beReplied[0].user.nickname : '' }}</em>：</span>{{ item.beReplied.length ? item.beReplied[0].content : '' }}</p>
                        </div>
                    </div>
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
                comments: [{
                    beReplied: [{
                        content: 'xxx',
                        user: {
                            avatarUrl: '',
                            nickname: 'xxx',
                            userId: 0
                        }
                    }],
                    commentId: 0,
                    content: 'xxx',
                    likedCount: 0,
                    time: 0,
                    user: {
                        avatarUrl: '',
                        nickname: 'xxx',
                        userId: 0
                    }
                }],
                hotComments: [],
                commentId: 0,
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
                if(this.commentId == this.activeMusic.id) return;
                this.staticMusic = this.activeMusic;
                this.commentId = this.activeMusic.id;
                this.getComment(this.commentId);
            },
            close () {
                this.$emit('event-closePage');
            },
            toTime (time) {
                let timeStr = '--';
                if(!time) return '--';
                let date = new Date(time);
                let yyyy = date.getFullYear(),
                    MM = date.getMonth()+1,
                    dd = date.getDate(),
                    HH = date.getHours(),
                    mm = date.getMinutes(),
                    ss = date.getSeconds();
                MM = MM<9 ? '0'+MM : MM;
                dd = dd<9 ? '0'+dd : dd;
                HH = HH<9 ? '0'+HH : HH;
                mm = mm<9 ? '0'+mm : mm;
                ss = ss<9 ? '0'+ss : ss;

                let now = new Date().getTime()-time;
                let n_dd = Math.floor(now/1000/60/60/24);
                if( n_dd<1 ){
                    timeStr = HH+':'+mm;
                }else if( n_dd==1 ){
                    timeStr = '昨天'+HH+':'+mm;
                }else if( n_dd==2 ){
                    timeStr = '前天'+HH+':'+mm;
                }else{
                    timeStr = yyyy+'年'+MM+'月'+dd+'日';
                };
                return timeStr;
            },
            // 评论详情
            getComment (id) {
                this.isLoading = true;
                let self = this;
                let url = 'https://api.imjad.cn/cloudmusic/?type=comments&id='+id;
                axios.get(window.location.origin+'/proxy.php',{
                        params:{ url: url }
                    })
                    .then(function (response) {
                        if(response.data.code != 200){
                            console.log(response.data);
                            console.log('搜索失败');
                            return false;
                        };
                        let data = response.data;
                        self.comments = data.comments;
                        self.hotComments = data.hotComments;
                        setTimeout(()=>{self.isLoading = false},500);
                    })
                    .catch(function (error) {
                        alert(error);
                    });
            }
        }
    }
</script>

<style lang="scss">
    @mixin musicComment ($n:1){
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
            h3 {
                font-size: 14px*$n;
                padding: 5px*$n 0;
            }
            .music-des {
                height: 60px*$n;
                padding: 10px*$n 0;
                img {
                    height: 60px*$n;
                    margin-right: 10px*$n;
                }
                .des-box {
                    height: 60px*$n;
                    line-height: 20px*$n;
                    font-size: 12px*$n;
                    .des-title {
                        height: 20px*$n;
                        font-size: 16px*$n;
                    }
                    .des-info {
                        height: 40px*$n;
                    }
                }
            }
            .comment-list {
                font-size: 14px*$n;
                li {
                    padding: 5px*$n 0;
                    .userImg {
                        width: 40px*$n;
                        height: 40px*$n;
                    }
                    .comment-box {
                        margin-left: 50px*$n;
                        .comment-header{
                            height: 40px*$n;
                            .comment-title {
                                line-height: 22px*$n;
                                height: 22px*$n;
                                justify-content: space-between;
                                .comment-like {
                                    font-size:12px*$n;
                                }
                                em {
                                    border-radius: 3px*$n;
                                    padding: 2px*$n 3px*$n;
                                }
                            }
                            .comment-time {
                                height: 18px*$n;
                                line-height: 18px*$n;
                                font-size: 12px*$n;
                            }
                        }
                        .comment-body {
                            padding: 5px*$n 0;
                            .comment-con {
                                padding: 5px*$n 0;
                                line-height: 22px*$n;
                            }
                            .comment-beReplied {
                                line-height: 16px*$n;
                                padding: 5px*$n !important;
                                border-radius: 3px*$n;
                                font-size: 12px*$n !important;
                            }
                        }
                    }
                }
            }
        }
    }

    #music-comment {
        width: 100%;
        height: 100%;
        position: absolute;
        top: 0;
        left: 0;
        z-index: 20;
        color: #333;
        background: #fff;

        @include musicComment;

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
            h3 {
                background: #eee;
                text-indent: 1em;
            }
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
            .comment-list {
                width: 100%;

                li {
                    position: relative;
                    border-bottom: 1px solid #efefef;
                    .userImg {
                        border-radius: 50%;
                        float: left;
                    }
                    .comment-box {
                        .comment-header {
                            width: 100%;
                            color: #666;
                            .comment-title {
                                em {
                                    color: #f66;
                                    border: 1px solid #f66;
                                    vertical-align: baseline;
                                }
                            }
                            .comment-time {
                                color: #aaa;
                            }
                        }
                        .comment-body {
                            .comment-con {
                                em {
                                    color: #6cf;
                                }
                            }
                            .comment-beReplied {
                                @extend .comment-con;
                                color: #999;
                                background: #f6f6f6;
                            }
                        }
                    }
                }
            }
        }
    }
    [data-dpr="2"] #music-comment {
        @include musicComment(2);
    }
    [data-dpr="3"] #music-comment {
        @include musicComment(3);
    }
</style>