<template>
  <div id="home">
    <nav-bar class="home-nav"><div slot="center">购物街</div></nav-bar>
    <tab-control
        :titles="['流行', '新款', '精选']"
        @tabClick="tabClick" 
        ref="tabControl1" 
        class="tab-control"
        v-show="isTabFixed"/>
    <scroll class="content" 
            ref="scroll" 
            :probe-type="3" 
            @scroll="contentScroll"
            :pull-up-load="true"
            @pullingUp="loadMore">
            
      <home-swiper :banners="banners" @swiperImageLoad="swiperImageLoad"/>
      <recommend-view :recommends="recommends" />
      <feature-view />
<!-- class="tab-control" -->
      <tab-control
        
        :titles="['流行', '新款', '精选']"
        @tabClick="tabClick" 
        ref="tabControl2"/>
      <goods-list :goods="showGoods" />
    </scroll>
    <back-top @click.native="backClick" v-show="isShowBackTop"/>
  </div>
</template>

<script>
import HomeSwiper from "./childComps/HomeSwiper";
import RecommendView from "./childComps/RecommendView";
import FeatureView from "./childComps/FeatureView";

import NavBar from "components/common/navbar/NavBar";
import TabControl from "components/content/tabControl/TabControl";
import GoodsList from "components/content/goods/GoodsList";
import Scroll from "components/common/scroll/Scroll";
// const Scroll = ()=> import ("components/common/scroll/Scroll")
import BackTop from 'components/content/backTop/BackTop'

import { getHomeMultidata, getHomeGoods } from "network/home";
import { debounce } from "common/utils"
export default {
  name: "Home",
  components: {
    NavBar,
    HomeSwiper,
    FeatureView,

    TabControl,

    RecommendView,
    GoodsList,
    Scroll,
    BackTop,
  },
  data() {
    return {
      banners: [],
      recommends: [],
      goods: {
        pop: { page: 0, list: [] },
        new: { page: 0, list: [] },
        sell: { page: 0, list: [] },
      },
      currentType: "pop",
      isShowBackTop: false,
      tabOffsetTop: 0,
      isTabFixed: false,
      saveY: 0,
      itemImgListner: null
    };
  },
  computed: {
    showGoods() {
      return this.goods[this.currentType].list;
    },
  },
  created() {
    //1.请求多个数据
    this.getHomeMultidata();

    //2.请求商品数据
    this.getHomeGoods("pop");
    this.getHomeGoods("new");
    this.getHomeGoods("sell");

  },
  mounted() {
    //防抖1.事件加载完成的事件监听
    const refresh = debounce(this.$refs.scroll.refresh, 200)
    
    //对监听事件进行保存
    this.itemImgListner = () => {
      refresh()
    }
    this.$bus.$on('itemImageLoad', this.itemImgListner)
  },
  methods: {
    /*
     * 事件监听相关方法
     */
    //防抖动函数

    tabClick(index) {
      switch (index) {
        case 0:
          this.currentType = "pop";
          break;
        case 1:
          this.currentType = "new";
          break;
        case 2:
          this.currentType = "sell";
          break;
      }
      this.$refs.tabControl1.currentIndex = index;
      this.$refs.tabControl2.currentIndex = index;
    },
    backClick() {
      this.$refs.scroll.scrollTo(0, 0, 1000)
    },
    contentScroll(position) {
      //1.判断BackTop是否显示
      this.isShowBackTop = -(position.y) > 1000

      //2.决定tabControl是否吸顶（position：fixed）
      this.isTabFixed = (-position.y) > this.tabOffsetTop
    },
    //上拉刷新
    loadMore() {
      this.getHomeGoods(this.currentType)
    },
    swiperImageLoad() {
      //2.获取tabControl的offsetTop
      //所有组件都有一个属性$el：用于获取组件中的元素
      this.tabOffsetTop = this.$refs.tabControl2.$el.offsetTop
    },
    /*
     *网络请求相关方法
     */
    getHomeMultidata() {
      getHomeMultidata().then((res) => {
        this.banners = res.data.banner.list;
        this.recommends = res.data.recommend.list;
      });
    },
    //销毁
    destroyed() {
      
    },
    //进入
    activeted() {
      this.$refs.scroll.scrollTo(0, this.saveY, 0)
      //强制刷新
      this.$refs.scroll.refresh()
    },
    //离开
    deactiveted() {
      //1.保存Y值
      this.saveY = this.$refs.scroll.getScrollY()
      //2.取消全局事件监听
      this.$bus.$off('itemImgLoad', this.itemImgListner)
    },
    getHomeGoods(type) {
      const page = this.goods[type].page + 1
      getHomeGoods(type, page).then((res) => {
        this.goods[type].list.push(...res.data.list)
        this.goods[type].page += 1

        //完成上拉加载更多
        this.$refs.scroll.finishPullUp()
      });
    },
  },
};
</script>

<style scoped>
#home {
  /* padding-top: 44px; */
  height: 100vh; 
  position: relative;
}
.home-nav {
  color: #fff;
  background-color: var(--color-tint);
}
/* .tab-control { 
  position: sticky;
  top: 43px;
  z-index: 9;
 } */
.content {
  overflow: hidden;

  position: absolute; 
  top: 44px;
  bottom: 49px;
  left: 0px;
  right: 0px;
}
.tab-control {
  position: relative;
  z-index: 9;
}
/* .content {
  height: calc(100% - 44px);
  overflow: hidden;
} */
</style>
