<template>
    <div class="router-music-list">
      <top-title :titleName="topTitle" @back="back"></top-title>
      <!--      // 吸顶播放-->
     <!-- <div v-show="playTopShow" class="play playTop" @click="playAll">
        <svg class="icon" aria-hidden="true">
          <use xlink:href="#icon-bofang"></use>
        </svg>
        <span class="play-all">播放全部 <i>(共{{trackCount}}首)</i></span>
      </div>-->
      <!--      背景虚化-->
      <div class="bg-img" ref="bgImg">
        <div class="bg-mask"></div>
        <div class="dim-bg">
          <img :src="bgStyle" alt="">
        </div>
      </div>
      <!--       滚动区域-->
      <scroll class="song-list-scroll"
              :data="tracks"
              @scroll="scroll"
              :listen-scroll="listenScroll"
              :probeType="probeType">
        <div class="flex-contain">
          <!--     头部图片-->
          <div class="business-card">
            <div class="image">
              <div class="image-left">
                <img :src="musicList.picUrl || musicList.coverImgUrl" alt="">
                <div class="play-number">
                  <svg class="icon" aria-hidden="true">
                    <use xlink:href="#icon-z"></use>
                  </svg>
                  <span>{{musicList.playCount}}</span>
                </div>
              </div>
              <div class="introduce">
                <p class="title text-ellipsis-two-line" ref="titleName">{{musicList.name}}</p>
                <div>
                  <img class="sm-icon" :src="avatarUrl" alt="">
                  <span class="text-ellipsis-one-line">{{nickname}}</span>
                </div>
                <p class="description text-ellipsis-two-line">{{description}}</p>
              </div>
            </div>
            <div class="business-card-bottom">
              <div class="song-icons">
                <svg class="icon" aria-hidden="true">
                  <use xlink:href="#icon-pinglun"></use>
                </svg>
                <span>{{commentCount}}</span>
              </div>
              <div class="song-icons">
                <svg class="icon" aria-hidden="true">
                  <use xlink:href="#icon-fenxiang"></use>
                </svg>
                <span>{{shareCount}}</span>
              </div>
              <div class="song-icons">
                <svg class="icon" aria-hidden="true">
                  <use xlink:href="#icon-iconset0425"></use>
                </svg>
                <span>下载</span>
              </div>
              <div class="song-icons">
                <svg class="icon" aria-hidden="true">
                  <use xlink:href="#icon-faxian1"></use>
                </svg>
                <span>多选</span>
              </div>
            </div>
          </div>
          <!--          歌曲列表-->
          <song-list :songLists="tracks">
            <div v-show="!playTopShow" class="play" ref="play" @click="playAll">
              <svg class="icon" aria-hidden="true">
                <use xlink:href="#icon-bofang"></use>
              </svg>
              <span class="play-all">播放全部 <i>(共{{trackCount}}首)</i></span>
            </div>
          </song-list>
        </div>
      </scroll>
    </div>
</template>

<script>
import { getPersonalizedDetail, getDailySong } from 'common/api/discover'
import { mapGetters, mapActions } from 'vuex'
import { ERR_OK } from 'common/js/config'
import Scroll from '@/baseComponent/scroll/scroll'
import SongList from '@/baseComponent/songList/songList'
import TopTitle from 'baseComponent/topTitle/topTitle'
export default {
  name: 'musicList',
  components: {
    TopTitle,
    Scroll,
    SongList
  },
  data () {
    return {
      // 歌曲列表数据
      tracks: [],
      // 专辑描述
      description: '',
      // 专辑小图标
      avatarUrl: '',
      // 小图标右边标题
      nickname: '',
      // 分享数量
      shareCount: '',
      // 评论数
      commentCount: '',
      // 歌曲数
      trackCount: '',
      probeType: 3,
      listenScroll: true,
      playTopShow: false,
      topTitle: '歌单',
      bgImgHeight: null,
      playTop: 0
    }
  },
  mounted () {
    this._getPersonalizedDetail()
    // 获取初始播放按钮距离顶部高度
    // this.playTop = this.$refs.play.offsetTop - this.$refs.play.clientHeight
    // 获取头部虚化图片高度
    this.bgImgHeight = this.$refs.bgImg.clientHeight
  },
  methods: {
    // 获取滚动坐标
    scroll (pos) {
      let posY = pos.y
      posY >= 0 && (this.$refs.bgImg.style.height = (this.bgImgHeight + posY) + 'px')
      let titleNameTop = this.$refs.titleName.clientHeight
      this.changeTitle(posY, titleNameTop)
      // this.floatingCover(posY, this.playTop)
    },
    // 滑动吊顶
    /* floatingCover (posY, offsetTop) {
      if (Math.abs(posY) >= offsetTop) {
        this.playTopShow = true
      } else {
        this.playTopShow = false
      }
    },*/
    back () {
      this.$router.go(-1)
    },
    // 滑动改变标题
    changeTitle (posY, offsetTop) {
      if (posY <= -offsetTop) {
        this.topTitle = this.musicList.name
      } else {
        this.topTitle = '歌单'
      }
    },
    _getPersonalizedDetail () {
      let id = this.$route.params.id
      if (this.$route.params.id === 'dailySong') {
        getDailySong().then(res => {
          if (res.data.code === ERR_OK) {
            this.tracks = res.data.result
          }
        })
      } else {
        getPersonalizedDetail(id).then(res => {
          let data = res.data.result
          if (res.data.code === ERR_OK) {
            let { tracks, description, creator, shareCount, commentCount, trackCount } = data
            this.tracks = tracks
            this.description = description
            this.creator = creator
            this.shareCount = shareCount
            this.commentCount = commentCount
            this.trackCount = trackCount
            this.nickname = creator.nickname
            this.avatarUrl = creator.avatarUrl
          }
        })
      }
    },
    playAll () {
      if (this.tracks.length > 0) {
        this.selectPlay({ list: this.tracks, index: 0 })
      }
    },
    ...mapActions([
      'selectPlay'
    ])
  },
  computed: {
    bgStyle () {
      return this.musicList.picUrl || this.musicList.coverImgUrl
    },
    ...mapGetters([
      'musicList'
    ])
  }
}
</script>

<style lang="scss" scoped>
@import "~common/scss/variable.scss";
@import "~common/scss/common.scss";
.router-music-list{
    .bg-img{
      position: fixed;
      height: 300px;
      z-index: -1;
      top: 0;
      left: 0;
      overflow: hidden;
      .bg-mask{
        background-color: rgba(0, 0, 0, 0.62);
        opacity: 0.3;
        position: absolute;
        top: 0;
        left: 0;
        height: 100%;
        z-index: 1;
      }
      .dim-bg{
        height: 100%;
        position: absolute;
        top: 0;
        left: 0;
        -webkit-filter: blur(20px); /* Chrome, Opera */
        -moz-filter: blur(20px);
        -ms-filter: blur(20px);
        filter: blur(20px);
        background-size:cover;
        background-position:center top;
        img{
          width: 100%;
          height: 100%;
        }
      }
    }
    .business-card{
      color: #ffffff;
      .image{
        height: 140px;
        padding-left: 15px;
        padding-top: 5px;
      }
      .business-card-bottom{
        height: 60px;
        display: flex;
        justify-content: space-around;
        font-size: 22px;
        padding-bottom: 10px;
        .song-icons span{
          display: block;
          text-align: center;
          font-size: $font-size-sm;
        }
      }
      .image-left{
        height: 0;
        width: 38%;
        padding-top: 38%;
        float: left;
        padding-left: 12px;
        position: relative;
        img{
          position: absolute;
          left: 0;
          top: 0;
          width: 100%;
          height: 100%;
          border-radius: 5px;
        }
      }
      .play-number{
        position: absolute;
        color: #ffffff;
        font-size: $font-size-lg;
        height: auto;
        width: auto;
        right: 10px;
        top: 0;
        .icon{
          vertical-align: middle;
          font-weight: bolder;
        }
        span{
          font-size: $font-size-smm;
          margin-left: -5px;
        }
      }
      img.sm-icon{
        float: left;
      }
      .introduce{
        float: left;
        width: 62%;
        padding-left: 15px;
        padding-right: 15px;
        height: 130px;
        position: relative;
        div{
          height: 30px;
          margin: 5px 0;
        }
        img{
          width: 30px;
          height: 30px;
          padding: 0;
          border-radius: 15px;
          vertical-align: middle;
        }
        span{
          font-size: $font-size-md;
          line-height: 30px;
          float: left;
          padding-left: 4px;
          width: calc(100% - 40px);
          text-align: left;
        }
        p{
          font-size: $font-size-smm;
          padding-right: 1px;
          text-align: left;
          margin-top: 5px;
          margin-bottom: 5px;
        }
        .title{
          font-size: $font-size-lg;
          line-height: 20px;
          margin-top: 0;
        }
        .description{
          line-height: 18px;
          margin: 0;
          padding-right: 15px;
          font-size: $font-size-sm;
        }
      }
    }
  }
</style>
