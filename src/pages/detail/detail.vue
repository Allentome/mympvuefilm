<!--
作者:chenghao
功能:vue页
-->

<!-- html代码片段 -->
<template>

  <div class="detailContainer">
    <img class="detail_img"
         :src="isMusicPlay?detailObj.music.coverImgUrl:detailObj.detail_img"
         alt=""/>
    <img @tap="handleMusicPlay"
         class="music_img"
         :src="isMusicPlay?'/static/images/music/music-start.png':'/static/images/music/music-stop.png'"
         alt="">
    <div class="avatar_data">
      <img :src="detailObj.avatar"
           alt=""/>
      <span>{{detailObj.author}}</span>
      <span>发布于</span>
      <span>{{detailObj.date}}</span>
    </div>
    <p class="company">{{detailObj.title}}</p>
    <div class="collection_share_container">
      <div class="collection_share">
        <img @tap="handleCollection"
             :src="isCollected?'/static/images/icon/collection.png':'/static/images/icon/collection-anti.png'"
             alt=""/>
        <img @tap="handleShare"
             src="/static/images/icon/share-anti.png"
             alt=""/>
      </div>
      <div class="line"></div>
    </div>
    <button open-type="share">转发此文章</button>
    <p class="content">{{detailObj.detail_content}}</p>
  </div>

</template>

<!-- js脚本代码片段 -->
<script>
  import { mapState } from 'vuex'
  import isPlayObj from '../../datas/isPlay'

  export default {
    name : "detail" ,
    onShareAppMessage ( res ) {
      // console.log( res );

      // if ( res.from === 'button' ) {
      //   // 来自页面内转发按钮
      //   console.log( res.target )
      // }
      let url = '/pages/detail/main?index=' + this.index;
      console.log( url )

      return {
        title : '自定义转发标题' ,
        path : url ,
        imageUrl : '/static/images/tab/yuedu.png'
      }
    } ,
    //数据模型
    data () {
      return {
        index : '0' ,

        detailObj : {} ,

        isCollected : false , //标识该文章是否被收藏

        isMusicPlay : false  //标识音乐是否播放
      }
    } ,
    //方法
    methods : {

      handleShare () {
        let arr = [ "分享到朋友圈" , "分享到微博" , "分享给微信好友" ];

        wx.showActionSheet( {
          itemList : arr , //按钮的文字数组，数组长度最大为6个,
          itemColor : '#000000' , //按钮的文字颜色,
          success : res => {
            let txt = arr[ res.tapIndex ];

            console.log( txt )
          }
        } );
      } ,
      handleMusicPlay () {
        this.isMusicPlay = !this.isMusicPlay;
        let { dataUrl , title } = this.detailObj.music

        if ( this.isMusicPlay ) {
          wx.playBackgroundAudio( {
            dataUrl ,
            title
          } )
        }
        else {
          wx.pauseBackgroundAudio()
        }
      } ,
      handleCollection () {
        //修改 收藏 状态

        let title = this.isCollected ? '取消收藏' : '收藏成功'

        if ( !this.isCollected ) {
          wx.setStorageSync( this.CollectStorageName , this.CollectStorageName )
        }
        else {
          //取消 就清除了
          wx.removeStorageSync( this.CollectStorageName )
        }

        this.isCollected = !this.isCollected;

        wx.showToast( {
          title : title , //提示的内容,
          icon : 'success' , //图标,
          duration : 2000 , //延迟时间,
          mask : true , //显示透明蒙层，防止触摸穿透,
          success : res => {
            console.log( res )
          }
        } );

        // 之前代码 先屏蔽

        // this.isCollected = !this.isCollected;
        // let title = this.isCollected ? '收藏成功' : '取消收藏'

        // wx.showToast( {
        //   title : title , //提示的内容,
        //   icon : 'success' , //图标,
        //   duration : 2000 , //延迟时间,
        //   mask : true , //显示透明蒙层，防止触摸穿透,
        //   success : res => {
        //   }
        // } );

        // 之前代码 先屏蔽
        // //读取之前本地缓存的状态 查看是否收藏
        // let oldStorage = wx.getStorageSync( 'isCollected' );
        //
        // // oldStorage = {};
        // oldStorage[ this.index ] = this.isCollected
        //
        // //将本次设置的结果再缓存到本地
        // wx.setStorage( {
        //   key : 'isCollected' ,
        //   data : oldStorage
        // } );

      } ,

    } ,
    //计算属性
    computed : {
      ...mapState( [ 'listTmp' ] ) ,
      CollectStorageName () {
        return "MyCollect" + this.index;
      }
    } ,
    //生命周期(mounted)
    mounted () {
      this.detailObj = this.listTmp[ this.index ]
      console.log( this.detailObj )
    } ,
    beforeMount () {
      // console.log(this)
      // console.log(index)

      this.index = this.$mp.query.index

      //原有代码 屏蔽了
      //预处理工作: 本地是否收藏的缓存
      // let oldStorage = wx.getStorageSync( 'isCollected' );
      // if ( !oldStorage ) {
      //   wx.setStorage( {
      //     key : 'isCollected' ,
      //     data : {}
      //   } );
      // }
      // else {
      //   this.isCollected = ( oldStorage[ this.index ] ? true : false );
      // }

      const value = wx.getStorageSync( this.CollectStorageName )

      if ( value ) {
        // 取到值了
        this.isCollected = true;
      }
      else {
        //如果key的值不存在，就会运行这里
        this.isCollected = false;
      }

      //判读当前页面加载的时候 音乐是否在播放
      if ( isPlayObj.pageIndex === this.index && isPlayObj.isPlay ) {
        this.isMusicPlay = true
      }
      else {
        this.isMusicPlay = false
      }

      //监听音乐的播放和暂停
      wx.onBackgroundAudioPlay( () => {
        console.log( "music start" )
        this.isMusicPlay = true
        isPlayObj.pageIndex = this.index
        isPlayObj.isPlay = this.isMusicPlay
      } );

      wx.onBackgroundAudioPause( () => {
        console.log( "music stop" )
        this.isMusicPlay = false
        // isPlayObj.pageIndex = this.index
        isPlayObj.isPlay = this.isMusicPlay
      } )
    }
  }


</script>

<!-- 样式代码片段  scoped -->
<style src="./detail.css"
       scoped>
</style>
