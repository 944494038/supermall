<template>
  <div id="home" class="wrapper">
    <nav-bar class="home-nav">
			<div slot="center">购物街</div>
		</nav-bar>
    <tab-control class="tab-control" 
                  :titles="['流行','新款','精选']"
                  @tabClick="tabClick"
                  ref="tabControl1"
                   v-show="isTabFixed"
                  ></tab-control>
    <scroll class="content" 
            ref="scroll" 
			      :probe-type="3"
            :pull-up-load="true"
            @scroll="contentScroll"
            @pullingUp="loadMore">

      <home-swiper :banners="banners" @swiperImageLoad="swiperImageLoad"></home-swiper>
      <recommend-view :recommends="recommends"/>
      <feature-view></feature-view>

      <tab-control :titles="['流行','新款','精选']"
                   @tabClick="tabClick"
                   ref="tabControl2"
                   ></tab-control>
				
      <goods-list :goods="showGcods"></goods-list> 
    </scroll>

    <back-top @click.native="backClick" v-show="isShowBackTop"/>
  </div>
</template>

<script>
	import NavBar from 'components/common/navbar/NavBar'
	import TabControl from 'components/content/tabControl/TabControl'
	import GoodsList from 'components/content/goods/GoodsList'
	import Scroll from 'components/common/scroll/Scroll'

	import HomeSwiper from './childCornps/HomeSwiper'
	import RecommendView from './childCornps/RecommendView'
	import FeatureView from './childCornps/FeatureView'

	import {getHomeMultidata, getHomeGoods} from "network/home"
  import {debounce} from 'common/utils'
  import {itemListenerMixin, backTopMixin} from 'common/mixin'

	export default {
    name: 'home',
    components: {
      NavBar,
      HomeSwiper,
      RecommendView,
      FeatureView,
      TabControl,
      GoodsList,
      Scroll,
    },
    mixins: [itemListenerMixin, backTopMixin],
    data() {
      return{
        banners: [],
        recommends: [],
        goods: {
          'pop': {page: 0, list: []},
          'new': {page: 0, list: []},
          'sell': {page: 0, list: []},
          },
          currentType: 'pop',
          taboffsetTop: 0,
          isTabFixed: false,
          saveY: 0,
          // ItemImgListener: null
        }
    },
    computed:{
      showGcods(){
        return this.goods[this.currentType].list
      }
    },
    activated() {
      this.$refs.scroll.scrollTo(0, this.saveY, 0)
      this.$refs.scroll.refresh()
    },
    deactivated() {
      // 1.保存Y值
      this.saveY = this.$refs.scroll.getScrollY()

      // 2.取消全局事件的监听
      // this.$bus.$off('itemImgLoad', this.ItemImgListener)
    },
    created() {
      //1.请求多个数据
      this.getHomeMultidata()
       //this.getHomeMultidata().then(res => {
        // this.banners = res.data.banner.list;
        // this.recommends = res.data.recommend.list;
       // })

        // 2.请求商品数据
      this.getHomeGoods('pop') //.then(res => {
      this.getHomeGoods('new') 
      this.getHomeGoods('sell')
        // }
    },
    mounted() {
        // // 防止频繁刷新 1.图片加载完成的事件监听
        // const newRefresh = debounce(this.$refs.scroll.refresh, 50)
        
        // // 对监听的事件进行保存
        // this.ItemImgListener = () => {
        //     newRefresh()
        // }
        // this.$bus.$on('itemImageLoad', this.ItemImgListener)

    },
    methods: {
        /**
         * 事件监听相关的方法
         * 
         */
      tabClick(index){
        switch (index){
          case 0:
            this.currentType = 'pop'
              break
          case 1:
            this.currentType = 'new'
              break
          case 2:
            this.currentType = 'sell'
              break
        }
        this.$refs.tabControl1.currentIndex = index;
        this.$refs.tabControl2.currentIndex = index;
      },

      contentScroll(position){
        // 1.判断BackTop是否显示
        this.isShowBackTop = (-position.y) > 1000

        // 2.决定tabControl是否吸顶(position: fixed)
        this.isTabFixed = (-position.y) > this.taboffsetTop
      },

      loadMore() {
        this.getHomeGoods(this.currentType)
            
        this.$refs.scroll.refresh()
      },

      
      swiperImageLoad() {
        // 2.获取tabControl的offsetTop
        // 所有的组件都有一个属性$el,用于获取组件中的元素
       
        this.taboffsetTop = this.$refs.tabControl2.$el.offsetTop
      },

        /**
         * 网络请求相关的方法
         */
      getHomeMultidata(){
        getHomeMultidata().then(res => {
          this.banners = res.data.banner.list;
          this.recommends = res.data.recommend.list;
          })
      },
      getHomeGoods(type){
        const page = this.goods[type].page + 1
        getHomeGoods(type, page).then(res => {
          this.goods[type].list.push(...res.data.list)
          this.goods[type].page += 1  
                console.log(res);
                
        // 添加下一页数据
          this.$refs.scroll.finishPullUp()
        })
      }
    }
}
</script>

<style scoped>
  #home {
    padding-top: 44px;
    /* vh -> viewport height*/
    height: 100vh;
  }

  .home-nav {
    background-color: var(--color-tint);
    color: #ffffff;

    position: fixed;
    left: 0;
    right: 0;
    top: 0;
    z-index: 9;
  }

  .tab-control {
    position: relative; 
    z-index: 9;
  }

  .content {
    overflow: hidden;

    position: absolute;
    top: 44px;
    bottom: 49px;
  }

    /* .content{
        height: calc(100% - 93px);
        overflow: hidden;
        margin-top: 44px
    } */
</style>