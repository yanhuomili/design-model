<template>
  <div class="v-just-video-detail-ctn bg-black detail-container" >
    <!-- 头部 -->

    <!-- <div class="header-wrapper">
      <div :class="['whiteLine',isIphoneX?'whiteLineX':'']"></div>
      <div class="header">
        <van-icon name="arrow-left" size="20px" class="back" @click="backToList" />
        <div class="logo-name">
          <img :src="logo?logo:default_profile" class="logo" />
          <p class="singleLine" style="line-height:43px;width: 150px;}">{{ shopName }}</p>
        </div>
        <div class="focus-share">
          <van-tag
            round
            type="primary"
            :class="{
            focus: isFocus ? false : true,
            focused: isFocus ? true : false
          }"
            @click="focusCustomer"
          >{{ focusText }}</van-tag>
          <div class="share" @click.stop="shareHandle(videoInfo[0])">
            <i class="iconfont iconfenxiang"></i>
          </div>
        </div>
      </div>
    </div> -->

    <!-- 视频轮播 -->
    <van-swipe
      ref="swiperCtn"
      class="my-swipe"
      style="height:100%"
      vertical
      :show-indicators="false"
      id="detail-vent-swipe"
      :loop="false"
      @change="onChange"
      :duration="300"
      :initial-swipe="initialSwipeIndex"
    >
      <van-swipe-item v-for="(item, i) in allVideos" :key="item.id">
        <!-- 视频容器 -->
        <!-- <div class="prism-player" :id="'test' + i"></div> -->
        <player-item
          :source="item.videoInfo.playUrl"
          :id="item.id"
          :state="getPlayerState(i)"
          :height="'100%'"
          :cover="i===0?item.videoInfo.coverPicUrl:''"
        ></player-item>

        <div class="header-wrapper">
          <div :class="['whiteLine',isIphoneX?'whiteLineX':'']"></div>
          <div class="header">
            <van-icon name="arrow-left" size="20px" class="back" @click="backToList" />
            <div class="logo-name">
              <img :src="item.contentOwnership.ownerLogo || default_profile" class="logo" />
              <p class="singleLine" style="line-height:43px;width: 150px;}">{{ item.contentOwnership.ownerName }}</p>
            </div>
            <div class="focus-share">
              <van-tag
                round
                type="primary"
                :class="{
                focus: !item.followSelf,
                focused: item.followSelf
              }"
                @click="focusCustomer(item, index)"
              >{{ isFocus ? '已关注': '关注' }}</van-tag>
              <div class="share" @click.stop="shareHandle(item)">
                <i class="iconfont iconfenxiang"></i>
              </div>
            </div>
          </div>
        </div>

        <!-- 展示的信息 -->
        <div class="video-info" :class="{'video-info-unfold': showMoreWords}" @click="showMoreWords && onToggleMoreWords()">
           <div class="video-info-goods" v-if="item.contentProductList && item.contentProductList.length > 0">
            <!-- 购物袋按钮和商品数量 -->
            <!-- <div class="bag" @click="toggleAllProduct(item)" >
              <img src="@/assets/imgs/bag.png" alt />
              <div class="amount">{{ item.contentProductList.length }}</div>
            </div> -->

            <!-- 商品展示(未点击时候) -->
            <!-- <article class="connectProduct" v-if="item.goodsArr && item.goodsArr.length">
              <connectProduct :item="item.goodsArr[0]"></connectProduct>
            </article> -->
            <div class="video-info-goods">
              <!-- 购物袋按钮和商品数量 -->
              <div class="video-info-goods-bag" @click="toggleAllProduct(item)">
                <img src="@/assets/imgs/bag.png" alt />
                <div class="amount">{{ item.contentProductList.length }}</div>
              </div>
              <!-- 商品展示(未点击时候) -->
              <div class="video-info-goods-item" v-if="item.goodsArr">
                <connectProduct :item="item.goodsArr[swiperVideoGoodsIndex]"></connectProduct>
              </div>
            </div>
          </div>

          <div class="video-info-content">
            <div class="video-info-content-title">{{item.contentTitle}}</div>
            <div class="video-info-content-desc">
              <span class="vicd-text" :class="{ 'vicd-nowrap': !showMoreWords }">{{item.descr}}</span>
              <span class="vicd-unfold" v-show="!showMoreWords && item.descr.length > 16" @click.stop="onToggleMoreWords">展开</span>
            </div>
            <div class="video-info-content-more vicd-unfold" v-if="showMoreWords" >
              收起
            </div>
          </div>
        </div>

        <!-- 点赞收藏评论条 -->
        <comment-bar
          class="video-deail-ccomment-bar"
          :contentId="item.id"
          :contentTitle="item.contentTitle"
          :likeCount="item.likeCount"
          :likeSelf="item.likeSelf"
          :collectionSelf="item.collectionSelf"
          :collectionCount="item.collectionCount"
          :commentCount="item.commentCount"
          v-if="showCommentBar"
          @commentIsClick="getShowListState($event)"
          @vInfoId="getVinfoId($event)"
           :oneVid="item.id"
        ></comment-bar>

      </van-swipe-item>
    </van-swipe>

    <!-- 商品展示（全部展开） -->
    <div class="product-list" v-if="showPorductList" ref="showAll" style="overflow:scroll" @click="showPorductList && toggleAllProduct()">
      <ul>
        <li v-for="(item, index) in pList" :key="index">
          <connectProduct :item="item"></connectProduct>
        </li>
      </ul>
    </div>

    <div class="comment_bar_test">
      <!-- 评论列表 -->
      <transition
        enter-active-class="animated slideInUp"
        leave-active-class="animated slideOutDown"
      >
        <div v-if="showCommentList">
          <common-list
            :contentId="contentId"
            @showBottomOperator="getOperatorVal(arguments)"
            @contentId="getContentId($event)"
          ></common-list>
        </div>
      </transition>

      <!-- 底部复制删除取消操作栏 -->
      <!-- <operator
        v-if="show"
        @cancleToComment="getCancelVal"
        @isClickCopy="getCopyStatus"
        @isDel="getDelStatus"
      ></operator> -->
      <!-- 遮罩层(点击底部复制栏专用) -->
      <!-- <van-overlay :show="show" @click="show = false" /> -->

      <!-- 弹出层 -->
      <!-- <van-popup v-model="showPopup">
        <p class="popup_title">{{ stretchTitle }}</p>
        <p class="popup_content">{{ stretchContent }}</p>
        <p @click="closePopup()">收起</p>
      </van-popup> -->
    </div>
  </div>
</template>
<script>
import JSBridge from '@/api/primeval/jsbridge'
import apiIndex from '@/api/index'
import bus from '@/utils/bus.js'
import playerItem from '@/views/pages/video/PlayerItem'
// import axios from 'axios'
// import ajax from '@/api/primeval/common/ajax.js'
import CommentBar from '@/components/comment/CommentBar.vue'
import CommonList from '@/components/comment/CommonList.vue'
// import Operator from '@/components/comment/Operator.vue'
import connectProduct from '@/components/product/connectProduct'
// import { __REQ_UTIL__ } from '@/api/eshopHelper'
import * as api from '@/api/custom'
// import { Toast } from 'vant'
import { getAuthorityInfo } from '@/api/primeval/common/auth'
import { getProductById } from '@/api/primeval/api/common/video-api'
let vm
export default {
  name: 'JustVideoDetail',
  components: {
    CommentBar,
    CommonList,
    // Operator,
    connectProduct,
    playerItem
    // [Tag.name]: Tag,
  },
  data () {
    // 请求类型常量
    const REQUEST_TYPE_APP_NAME = 'APP'
    // const REQUEST_TYPE_COOP_NAME = 'COOP'
    // var sk
    // var ss
    // if (localStorage.getItem('sessionKey') != null) {
    //   sk = localStorage.getItem('sessionKey')
    // }
    // if (localStorage.getItem('sessionSecret') != null) {
    //   ss = localStorage.getItem('sessionSecret')
    // }
    let authInfo = getAuthorityInfo() ? JSON.parse(getAuthorityInfo()) : null

    return {
      // REQUEST_TYPE_APP_NAME,
      // REQUEST_TYPE_COOP_NAME,
      sk: authInfo ? authInfo.sessionKey : null,
      ss: authInfo ? authInfo.sessionSecret : null,
      configForm: {
        user: {
          sessionKey: authInfo ? authInfo.sessionKey : null,
          sessionSecret: authInfo ? authInfo.sessionSecret : null
        },
        requestType: {
          value: REQUEST_TYPE_APP_NAME
        },
        method: 'utopa.eshop.product.getProduct',
        appUrl: this.$httpBaseUrl,
        request: {
          clientType: 1,
          productId: '',
          storeId: '',
          sysId: 1,
          type: 0
        }
      },
      state: '',
      default_profile: require('@/assets/imgs/default_profile.jpg'),
      transmitId: '', // list页面传过来的当前点击的视频id
      playIndex: 0, // 当前播放的index
      nosound: true,
      nosoundSrc: require('@/assets/imgs/nosound.png'),
      showConnectProduct: false, // 是否显示productList
      isFocus: false, // 是否关注
      focusText: '关注', // 按钮文字
      contentTitle: '', // 下面收缩内容的title
      descr: '', // 下面收缩内容的描述
      relatedProductInfo: {},
      uid: '', // 写死的
      commentCount: 0, // 评论数
      contentProductList: [], // 商品列表【电商接口返回】
      pList: [], // 最终渲染的商品列表
      collectionCount: 0, // 收藏数
      collectionSelf: false, // 是否收藏
      likeCount: 0, // 所有点赞数
      likeSelf: false, // 是否点赞
      showShareBlock: false, // 是否显示分享面板
      playerSave: [],
      allVideos: [], // 这个才是视频列表
      height: window.screenHeight,
      id: 'test0',
      videoList: [],
      stretch_ctn: true, // 可收缩文字容器是否显示
      showPorductList: false, // 是否展开商品列表
      showPopup: false, // 是否弹出
      showMoreWords: false, // 是否展示更多文字
      singleLine: true,
      doubleLine: true,
      stretchTitle: '', // 折叠标题
      stretchContent: '', // 折叠内容
      logo: '',
      showCommentList: false, // 是否显示评论列表
      // id: '', // 视频容器的id
      contentId: '',
      showCommentBar: true, // 是否展示底部点赞收藏评论
      show: false, // 是否显示遮罩层
      deleteSuccessResult: [], // 删除成功后，返回的新评论列表
      connectProductList: [],
      startIndex: 0,
      endIndex: 9,
      changeList: [],
      afterChangeInfo: {},
      videoInfo: [], // 现在videoInfo是数组
      // _vinfoId: '',
      oneVid: '',
      shopName: '',
      continueToGetVideo: true,
      currentIndex: 0,
      initialSwipeIndex: 0, // 由于swipe在children变化时会重置父级swipe，所以在添加视频到列表时，swipe初始值也设为当前下标

      // 新数据
      swiperVideoGoodsIndex: 0 // 对视频中关联商品的轮播播放
      // goodsArrMap:{}, // 商品
    }
  },
  methods: {
    // 获取播放状态
    getPlayerState (idx) {
      if (this.currentIndex === idx) return 'playing'
      if (Math.abs(this.currentIndex - idx) < 3) return 'ready'
      return 'dispose'
    },
    // 获取视频列表
    getListContentJustVideo () {
      api
        .getListContentJustVideo(this.startIndex, this.endIndex)
        .then(res => {
          let videoArr = []
          var currentIndex = 0
          this.videoList = res.result.records
          if (this.videoList.length != 0) {
            this.videoList.forEach((ele, index) => {
              // console.log(ele, "这是ele");
              if (ele.id === this.vid) {
                currentIndex = index
              }
              videoArr.push(ele)
            })
          }

          this.allVideos = videoArr
          // 更换顺序
          var clickItem = this.allVideos[currentIndex]
          this.allVideos.splice(currentIndex, 1)
          this.allVideos.unshift(clickItem)

          // if (this.allVideos.length === 1) {
          //   this.allVideos = [...this.allVideos, ...this.allVideos];
          // }
          // this.createPlayer();
          // console.log(this.videoList, "videoList");
        })
        .catch(err => {
          console.log(err)
        })
    },
    getOthers () {
      api.getListContentJustVideo(this.startIndex, this.endIndex).then(res => {
        this.allVideos = res.result.records
      })
    },
    getVinfoId (data) {
      console.log(data, '什么》》')
      this._vinfoId = data
      this.oneVid = this._vinfoId
    },
    playSound () {
      console.log(this.playerSave, 'this.playerSavethis.playerSave')
      console.log(this.playIndex)

      this.nosound = !this.nosound
      if (this.nosound) {
        this.nosoundSrc = require('@/assets/imgs/sound.png')
        console.log(this.playerSave[vm.playIndex])
        this.playerSave[this.playIndex].setVolume(0)
      } else {
        this.nosoundSrc = require('@/assets/imgs/nosound.png')
        this.playerSave[this.playIndex].setVolume(1)
      }
    },

    // 拼装请求参数
    initRequest () {
      return new Promise(resolve => {
        let config = {
          sessionKey: this.sk,
          sessionSecret: this.ss,
          settings: {
            type: this.configForm.requestType.value,
            withAccessToken: false
          },
          url: this.configForm.appUrl,
          method: this.configForm.method,
          args: this.configForm.request
        }
        resolve(config)
      })
    },
    // 请求商户数据
    getList (productId, storeId) {
      // return new Promise((resolve, reject) => {
      this.configForm.request['productId'] = productId
      this.configForm.request['storeId'] = storeId
      return getProductById({ productId, storeId })
      // .then(res => {
      //   if (res.code === 0) {
      //     resolve(res.data)
      //   }
      // }).catch(err => {
      //   console.log(err.msg)
      // })
      // this.initRequest().then(config => {
      //   __REQ_UTIL__.request(
      //     config,
      //     data => {
      //       if (data.code === 0) {
      //         resolve(data.data)
      //         // this.showConnectProduct = true;
      //         // return data.data;
      //       }
      //     },
      //     data => {
      //       if (data.code !== 0) {
      //         console.log(data.msg)
      //         // this.showConnectProduct = false; // 请求失败就不展示商品列表了
      //       }
      //     }
      //   )
      // })
      // })
    },
    // 分享
    shareHandle (item) {
      // console.log(item);
      let domain = window.location.host
      let param = {
        shareTitle: item.contentTitle,
        subTitle: item.descr,
        shareIcon: item.coverPicUrl || this.nullImg,
        shareUrl: `${domain}/m/vscustapp/#/justVideoDetailShare?contentId=${item.id}`
      }
      console.log(param.shareUrl)
      JSBridge.Common.GTBridge_Common_Base_showShare(param, res => {
        console.log(res)
      })
    },

    // 获取关注状态
    geFollowStatus (storeId) {
      console.log(this.videoInfo, '这有没有')
      api
        .isFollow(storeId)
        .then(res => {
          // console.log(res);
          if (res.code === 0) {
            if (res.result) {
              // 关注了
              this.isFocus = true
              this.focusText = '已关注'
            } else {
              this.isFocus = false
              this.focusText = '关注'
            }
          } else {
            console.log(res.message)
          }
        })
        .catch(err => {
          console.log(err)
        })
    },
    // 关注和取消关注
    focusCustomer (item) {
      const data = {
        targetId: item.contentOwnership.ownerId,
        targetType: item.contentOwnership.ownerType
      }
      if (item.followSelf) {
        // 已关注了，调用取消关注接口
        api
          .followDelete(data)
          .then(res => {
            // console.log(res);
            if (res.code === 0) {
              if (res.result) {
                item.followSelf = false
                this.allVideos.splice(index, 1, { ...item })
                // this.focusText = "关注";
              }
            } else {
              console.log(res.message)
            }
          })
          .catch(err => {
            console.log(err)
          })
      } else {
        // 还没关注，调用关注接口
        api
          .followAdd(data)
          .then(res => {
            // console.log(res);
            if (res.code === 0) {
              if (res.result) {
                item.followSelf = true
                this.allVideos.splice(index, 1, { ...item })
                // this.focusText = "已关注";
              }
            } else {
              console.log(res.message)
            }
          })
          .catch(err => {
            console.log(err)
          })
      }
    },

    // 根据商品id获取商品详情
    getProductDetailById () {
      var productId = ''
      var storeId = ''
      var productArr = []
    },

    loadMoreVideos () {
      this.startIndex = this.startIndex + 10
      this.endIndex = this.endIndex + 10
      if (this.continueToGetVideo) {
        api
          .getListContentJustVideo(this.startIndex, this.endIndex)
          .then(res => {
            if (res.result.records.length != 0) {
              var newVideo = res.result.records
              // this.allVideos = [...this.allVideos,...newVideo];
              // this.allVideos =  this.allVideos.concat(newVideo)
              this.allVideos.push(...newVideo)
              this.initialSwipeIndex = this.currentIndex

              if (newVideo.length < 10) {
                this.continueToGetVideo = false
              }
            }
          })
          .catch(err => {
            console.log(err)
          })
      }
    },

    // 切换轮播
    onChange (index) {
      this.currentIndex = index
      if (index > this.endIndex - 3) { // 在刷到倒数第三个的时候去拉新的视频
        this.loadMoreVideos()
      }

      const currentVideo = this.allVideos[index]
      // if (currentVideo && currentVideo.contentOwnership) {
      //   const ownership = currentVideo.contentOwnership
      //   this.shopName = ownership.ownerName
      //   this.logo = ownership.ownerLogo
      //   this.geFollowStatus(ownership.ownerId)
      // }
      // 去获取商品列表数据
      if (
        currentVideo &&
        Array.isArray(currentVideo.contentProductList) &&
        currentVideo.contentProductList.length &&
        !currentVideo.goodsArr
      ) {
        this.setContentProductList(currentVideo.contentProductList, currentVideo.contentOwnership.ownerId)
          .then((goodsArr) => {
            this.allVideos.splice(index, 1, { ...currentVideo, goodsArr })
            this.startSwiperVideoGoods(goodsArr.length)
          })
      } else {
        this.contentProductList = []
        this.pList = []
      }

      // if (currentIndex && (currentVideo.contentProductList)) {

      // }
      // this.allVideos.forEach((ele, i) => {
      //   if (index == i) {
      //     this.shopName = this.allVideos[index].contentOwnership.ownerName;
      //     this.logo = this.allVideos[index].contentOwnership.ownerLogo;
      //     console.log(this.allVideos[index].contentOwnership.ownerLogo, "logo");

      //     this.likeCount = this.allVideos[index].likeCount;
      //     this.likeSelf = this.allVideos[index].likeSelf;
      //      this.collectionCount = this.allVideos[index].collectionCount;
      //     this.collectionSelf = this.allVideos[index].collectionSelf;
      //     this.commentCount = this.allVideos[index].commentCount;
      //     this.oneVid = this.allVideos[index].id;
      //     this.contentId = this.allVideos[index].id;
      //     console.log(this.oneVid, "切换时，获取到id了么");
      //   }
      // });
      // 获取当前的player对象
      // api.getListContentJustVideo(this.startIndex, this.endIndex).then(res => {
      //   console.log(res, '切换后的请求')
      //   if (res.code === 0) {
      //     if (res.result.records.length != 0) {
      //       res.result.records.forEach((ele, i) => {
      //         if (index === i) {
      //           console.log(ele, '当前视频')
      //         // this.afterChangeInfo = ele.
      //         }
      //       })
      //     }
      //   } else {
      //     console.log(res.message)
      //   }
      // })
    },
    // 展开或者收起商品列表
    toggleAllProduct (item = {}) {
      this.showPorductList = !this.showPorductList
      if (this.showPorductList && Array.isArray(item.goodsArr)) this.pList = item.goodsArr
      else {
        this.pList = []
      }
    },
    // 点击展开文字
    showWords () {
      this.showPopup = true
    },
    // 点击收起文字
    closePopup () {
      this.showPopup = false
    },
    // 返回列表
    backToList () {
      // 判断是否app环境
      const isApp = !!window.WebViewJavascriptBridge
      if (isApp) {
        JSBridge.Common.GTBridge_Common_Base_back({}, res => {})
      } else {
        this.$router.go(-1)
      }
    },
    getShowListState (data) {
      console.log('~~~~~~~~~~~~~~~~~~~~~')

      console.log(data)
      this.showCommentList = data
      console.log(this.showCommentList, '是否显示评论列表')
    },
    // 获取CommonList传过来的contentId
    getContentId (data) {
      this.contentId = data
    },
    // 是否显示底部弹出层
    getOperatorVal (data) {
      this.show = data[0]
      this.id = data[1]
      this.contentId = data[2]
      // this.show = data;
    },
    // 是否点击了取消
    getCancelVal (data) {
      this.show = !data
    },
    // 是否点击了复制
    getCopyStatus (data) {
      this.show = !data
    },
    // 是否点击了删除
    getDelStatus (data) {
      this.show = !data
      this.deleteComment()
    },
    // 删除评论
    deleteComment () {
      var data = {
        id: this.id,
        contentId: this.contentId
      }

      api
        .deleteComment(data)
        .then(res => {
          // console.log(res);
          if (res.code === 0) {
            this.deleteSuccessResult = res.result
            location.reload() // 删除完成，刷新评论列表
          }
        })
        .catch(err => {
          console.log(err)
        })
    },
    createPlayer () {
    },
    // 设置当前视频的商品列表
    setContentProductList (list = [], storeId) {
      // this.contentProductList = list;
      console.log('setContentProductList-out')
      return Promise.all(list.map(async (ele) => {
        const productId = ele.productId
        console.log(1231232, productId, storeId)
        if (productId && storeId) {
          let res = await this.getList(productId, storeId)
          if (res.code === 0) {
            return res.data
          }
          // return this.getList('2830030', '273551')
        }
        return []
      })).then(resArr => resArr.map(item => ({
        name: item.name,
        price: item.price,
        businessLogoUrl: item.businessLogoUrl,
        id: item.id
      })))
      // .then((resArr) => {
      //   console.log({resArr})
      //   // this.pList.push(...resArr);
      //   // this.showConnectProduct = true;
      //   // this.contentProductList
      // })
      // const storeId = this.videoInfo[0].contentOwnership.ownerId;
      // if(this.contentProductList.length!==0){
      // this.contentProductList.forEach(ele => {
      //   const productId = ele.productId;
      //   console.log(
      //     productId,
      //     "productIdproductIdproductIdproductIdproductIdproductId"
      //   );
      //   console.log(
      //     storeId,
      //     "storeIdstoreIdstoreIdstoreIdstoreIdstoreId"
      //   );
      //   console.log({productId, storeId})
      //   if (productId != undefined && storeId != undefined) {
      //     this.getList(productId, storeId).then(res => {
      //       this.pList.push(res);
      //     });
      //   }
      // });
    },
    onToggleMoreWords () {
      this.showMoreWords = !this.showMoreWords
    },
    // 清除视频关联的商品轮播
    disposeSwiperGoods () {
      if (this.swiperVideoGoodsInterval) clearInterval(this.swiperVideoGoodsInterval)
    },
    // 开始关联商品的轮播，默认3秒切换
    startSwiperVideoGoods (length = 0) {
      // 进来就清除一遍
      this.disposeSwiperGoods()

      this.swiperVideoGoodsIndex = 0

      if (length === 0) {
        return () => {}
      }

      this.swiperVideoGoodsInterval = setInterval(() => {
        let next = this.swiperVideoGoodsIndex + 1
        if (next >= length) {
          next = 0
        }
        this.swiperVideoGoodsIndex = next
      }, 3000)

      return this.disposeSwiperGoods
    }
  },

  created () {
    vm = this
    // var sk = ''
    // var ss = ''
    // var sessionInfoFromList = localStorage.getItem('sessionInfoFromList')

    // if (sessionInfoFromList != null) {
    //   sessionInfoFromList = JSON.parse(sessionInfoFromList)
    //   this.sk = sessionInfoFromList.sessionKey
    //   this.ss = sessionInfoFromList.sessionSecret
    // }
    // 获取
    this.$request
      .get('/vsapp/app/content/info/queryPageByIdsNoAuth', {
        contentIds: [this.vid]
      })
      .then(res => {
        console.log(res, '有么')
        if (res.code === 0) {
          var productId = ''
          var storeId = ''
          var productArr = []
          this.videoInfo = res.result
          this.shopName = this.videoInfo[0].contentOwnership.ownerName
          this.contentId = this.vid
          this.contentTitle = this.videoInfo[0].contentTitle
          this.likeCount = this.videoInfo[0].likeCount
          this.likeSelf = this.videoInfo[0].likeSelf
          this.collectionSelf = this.videoInfo[0].collectionSelf
          this.collectionCount = this.videoInfo[0].collectionCount
          this.contentProductList = this.videoInfo[0].contentProductList
          this.commentCount = this.videoInfo[0].commentCount
          this.descr = this.videoInfo[0].descr

          if (this.videoInfo[0].contentProductList != null) {
            this.contentProductList = this.videoInfo[0].contentProductList
            // if(this.contentProductList.length!==0){
            this.contentProductList.forEach(ele => {
              productId = ele.productId
              storeId = this.videoInfo[0].contentOwnership.ownerId
              console.log(
                productId,
                'productIdproductIdproductIdproductIdproductIdproductId'
              )
              console.log(
                storeId,
                'storeIdstoreIdstoreIdstoreIdstoreIdstoreId'
              )
              console.log({ productId, storeId })
              if (productId != undefined && storeId != undefined) {
                this.getList(productId, storeId).then(res => {
                  productArr.push(res)
                  this.pList = productArr
                })
              }
            })
            // }
          }

          this.geFollowStatus(this.videoInfo[0].contentOwnership.ownerId)
        }
      })

    this.getListContentJustVideo()
    this.getProductDetailById()
    // 获取关联商品【电商接口】
    // this.getList().then(res => {
    //   console.log(res, "获取到么");
    // });

    bus.$on('transmitId', val => {
      this.transmitId = val
      console.log(this.transmitId, '还有没有这个id')
    })
    // 获取视频列表
    // api
    //   .getVideoList()
    //   .then(res => {
    //     let videoArr = [];
    //     var currentIndex = 0;
    //     this.videoList = res.result.records;
    //     if (this.videoList.length != 0) {
    //       this.videoList.forEach((ele, index) => {
    //         console.log(ele, "这是ele");
    //         if (ele.id === this.vid) {
    //           currentIndex = index;
    //         }
    //         videoArr.push(ele);
    //       });
    //     }

    //     this.allVideos = videoArr;
    //     // 更换顺序
    //     var clickItem = this.allVideos[currentIndex];
    //     this.allVideos.splice(currentIndex, 1);
    //     this.allVideos.unshift(clickItem);

    //     if (this.allVideos.length === 1) {
    //       this.allVideos = [...this.allVideos, ...this.allVideos];
    //     }
    //     this.createPlayer();
    //     // console.log(this.videoList, "videoList");
    //   })
    //   .catch(err => {
    //     console.log(err);
    //   });

    // this.geFollowStatus();
  },
  beforeDestroy () {
    this.disposeSwiperGoods()
  },
  computed: {
    isIphoneX () {
      return apiIndex.isIphoneX()
    },
    vid () {
      return this.$route.query.id // 地址栏上传递过来的id
    }
  }
}
</script>
<style lang="css" scoped>

.bg-black{
  background-color: black;
}
.nosound {
  position: absolute;
  bottom: 60px;
  right: 39px;
  width: 30px;
}

.nosound img {
  width: 100%;
}
.comment_bar_test {
  width: 100%;
  background-color: #ffffff;
  position: fixed;
  bottom: 0;
  z-index: 999;
}

.connectProduct {
  width: 60%;
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-top: 20px;
}

.product {
  margin-bottom: 6px;
}

article {
  position: absolute;
  top: 60%;
  left: 50%;
  transform: translate(-50%, 10px);
}
</style>
<style>
.comment_amount {
  background-color: #fff !important;
}

.money {
  display: inline-block;
}

.icongouwuche {
  float: right;
  display: inline-block;
  color: #ff2f7b;
}

.product {
  width: 100% !important;
}

.con {
  width: 100%;
}

.title {
  width: 100% !important;
}
</style>
<style lang="less" scoped>

.header-wrapper {
  color: #fff;
  width: 100%;
  position: absolute;
  top: 0px;
  z-index: 1000;
  // background-color: rgba(0, 0, 0, 0.3);
  .whiteLine {
    background: none;
  }
  .header {
    display: flex;
    align-items: center;
    height: 30px;
    padding: 0 10px 10px 10px;

    .back {
      width: 30px;
      text-align: center;
    }

    .logo-name {
      flex: 1;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 15px;
      color: #fff;
      margin-left: -90px;

      .logo {
        width: 30px;
        height: 30px;
        border-radius: 15px;
        margin-right: 6px;
        box-sizing: border-box;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
        margin-left: 65px;
      }
    }

    .focus-share {
      // width: 85px;
      display: flex;
      align-items: center;

      .focus {
        width: 50px;
        height: 18px;
        border-radius: 100px;
        border: 1px solid #fff;
        display: flex;
        align-items: center;
        justify-content: center;
        font-size: 12px;
        color: #fff;
        background: none;
        margin-right: 11px;
      }
    }
  }
}

.focus-share .iconfenxiang {
  color: #fff !important;
}

.stretch_ctn {
  color: #fff;
}

.stretchTitle {
  position: absolute;
  top: 80%;
  left: 20%;
  width: 200px !important;
  text-align: left;
  min-height: 20px;
}

.stretchContent {
  width: 200px;
  position: absolute;
  top: 83%;
  text-align: left;
  left: 20%;
}

.showMore {
  display: inline-block;
  position: absolute;
  width: 2.66667rem;
  height: 0.8rem;
  top: 85%;
  right: 0;
  text-align: left;
}

.popup_title,
.popup_content {
  width: 80%;
  text-align: left;
  margin: 0 auto;
}

.bag {
  width: 0.64rem;
  height: 0.88rem;
  position: absolute;
  top: 70%;
  left: 10%;
}

.bag img {
  width: 100%;
}

.amount {
  width: 0.48rem;
  height: 0.48rem;
  background-color: #fff;
  color: #ff2f7b;
  border-radius: 50%;
  text-align: center;
  line-height: 0.48rem;
  position: relative;
  top: -44px;
  right: -14px;
}

.focused {
  width: 1.33333rem;
  height: 0.48rem;
  border-radius: 2.66667rem;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 0.32rem;
  color: #fff;
  background-image: linear-gradient(to right, #ff2f3d, #ff00e8);
  margin-right: 0.29333rem;
}
.video-deail-ccomment-bar{
  border:none;
  position: absolute;
  bottom: 0;
  left:0;
  right: 0;
  z-index: 99999;
  background: rgba(255,255,255,0);
  color: #fff;
  height: 52px;
  //display: flex;
  //align-items: center;
  //justify-content: space-between;
}
.bar_content{
  display: flex;
  align-items: center;

}
</style>
<style lang="scss" scoped>

::v-deep .van-popup {
  background: transparent;
  color: #fff;
  width: 80%;
}
.share {
  margin-left: 4px;

  i {
    font-size: 24px;
  }
}

.video-info{
  width: 100%;
  position: absolute;
  left: 0;
  bottom: 53px;
  color: #fff;
  padding: 10px;
  font-size: 14PX;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  justify-content: flex-end;

  * {box-sizing: border-box;}
  transition: height 0.5s;

  &-unfold{
    top: 0;
    bottom: 0;
    background-color: rgba(0,0,0,.6);
    z-index: 9999999;
    padding-bottom: 53px;
    height: 100vh;
  }

  &-goods{
    display: flex;
    justify-content: flex-start;
    align-items: flex-end;
    margin-bottom: 4px;

    &-bag{
      height: 33px;
      width: 24px;
      position: relative;
      margin-right: 17px;
      img{
        height: 100%;
        width: 100%;
      }
    }

    &-item{
      .product{
        margin-bottom:0;
      }
    }
  }

  &-content{
    &-desc{
      display: flex;
      align-items:center;
      // width: 260px;

      .vicd-text{
        display: inline-block;
        max-width: none;
      }

    }
    .vicd-nowrap{
      overflow: hidden;
      text-overflow:ellipsis;
      white-space: nowrap;
      max-width: 225px;
    }

    &-more{
      display: flex;
      justify-content: flex-end;
    }
    .vicd-unfold{
        font-weight: bold;
      }
  }

}

.product-list{
  position: fixed;
  left: 0;
  bottom: 0;
  top: 0;
  height: 100vh;
  background-color: rgba(0,0,0,.5);
  display: flex;
  flex-direction: column;
  justify-content: flex-end;
  width: 100%;

  ul{
    padding: 20px 27px 60px;

    li{
      margin-bottom: 6px;
    }
  }
}
</style>
