<template>
  <view>
    <!-- 使用自定义的搜索组件 -->
    <view class="search-box">
      <my-search @click="gotoSearch"></my-search>
    </view>
    <!-- 轮播图区域 -->
    <swiper :indicator-dots="true" :autoplay="true" :interval="3000" :duration="1000" circular="true">
      <swiper-item v-for="(item,i) in swiperList" :key="i">
        <navigator class="swiper-item" :url="'/subpkg/goods_detail/goods_detail?goods_id='+item.goods_id">
          <image :src="item.image_src"></image>
        </navigator>
      </swiper-item>
    </swiper>

    <!-- 分类导航区域 -->
    <view class="nav-list">
      <view class="nav-item" v-for="(item, i) in navList" :key="i" @click="navSwitch(item)">
        <image :src="item.image_src" class="nav-img"></image>
      </view>
    </view>
    <!-- 首页楼层区域 -->
    <view class="floor-list">
      <view class="floor-item" v-for="(item,i) in floorList" :key="i">
        <image :src="item.floor_title.image_src" class="floor-img" ></image>
        <!-- 楼层图片区域 -->
         <view class="floor-img-box">
           <!-- 左侧盒子 -->
           <navigator class="left-img-box" :url="item.product_list[0].url">
             <image :src="item.product_list[0].image_src" mode="widthFix":style="{width: item.product_list[0].image_width + 'rpx'}" ></image>
           </navigator>
           <!-- 右侧盒子 -->
           <view class="right-img-box">
             <navigator class="right-img-item" v-for="(item2, i2) in item.product_list" :key="i2" v-if="i2 !== 0" :url="item2.url">
               <image :src="item2.image_src" mode="widthFix" :style="{width: item2.image_width + 'rpx'}"></image>
             </navigator>
           </view>
         </view>
      </view>
    
    </view>
  </view>
</template>

<script>
  export default {
    data() {
      return {
        // 轮播图的数据列表
        swiperList: [],
        // 分类选项数据列表
        navList: [],
        // 首页楼层数据列表
        floorList: []
      };
    },
    onLoad() {
      // 一加载就获取轮播图的数据
      this.getSwiperList(),
        // 一加载就获取分类列表的数据
        this.getnavList()
      this.getfloorList()
    },
    methods: {
      // 获取轮播图
      async getSwiperList() {
        const {
          data: res
        } = await uni.$http.get('/api/public/v1/home/swiperdata')
        // console.log(res)
        // 请求失败
        if (res.meta.status !== 200) return uni.$showMsg()
        // 请求成功
        this.swiperList = res.message
      },
      // 获取分类列表
      async getnavList() {
        const {
          data: res
        } = await uni.$http.get("/api/public/v1/home/catitems")
        // console.log(res)
        if (res.meta.status !== 200) return uni.$showMsg()
        this.navList = res.message
        // console.log(this.navList)
      },
      // 点击分类图标切换到分类tabber
      navSwitch(item) {
        if (item.name === "分类") {
          uni.switchTab({
            url: "/pages/cate/cate"
          })
        }
      },
      // 获取首页楼层数据
      async getfloorList() {
        const {
          data: res
        } = await uni.$http.get('/api/public/v1/home/floordata')
        console.log(res)
        if (res.meta.status !== 200) return uni.$showMsg()
        // 通过双层forEach循环,处理url地址
        res.message.forEach(floor=>{
          floor.product_list.forEach(prod=>{
           prod.url='/subpkg/goods_list/goods_list?'+prod.navigator_url.split('?')[1]
          })
          })
        this.floorList = res.message
      },
      // 点击搜索跳转到搜索页
      gotoSearch(){
        uni.navigateTo({
          url:'/subpkg/search/search'
        })
      }
      
    },
  }
</script>

<style lang="scss">
  swiper {
    height: 330rpx;

    .swiper-item,
    image {
      width: 100%;
      height: 100%;
    }
  }

  .nav-list {
    display: flex;
    justify-content: space-around;
    margin: 15px 0;

    .nav-img {
      width: 128rpx;
      height: 140rpx;
    }
  }
  .floor-img {
    width: 100%;
    height: 60rpx;
    display: flex;
  }
  
  .right-img-box {
    display: flex;
    flex-wrap: wrap;
    justify-content: space-around;
  }
  
  .floor-img-box {
    display: flex;
    padding-left: 10rpx;
  }
  .search-box{
    position: sticky;
    top: 0;
    z-index: 999;
    }
</style>
