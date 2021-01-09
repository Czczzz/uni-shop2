<template>
  <view>
    <view class="goods-list">
      <view v-for="(goods, i) in goodsList" :key="i"  @click="gotoDetail(goods)">
       <my-goods :goods="goods"></my-goods>
      </view>
    </view>
  </view>
</template>

<script>
  export default {
    data() {
      return {
        // 请求参数对象
        queryObj: {
          // 查询关键词
          query: '',
          // 商品分类Id
          cid: '',
          // 页码值
          pagenum: 1,
          // 每页显示多少条数据
          pagesize: 10
        },
        // 商品列表的数据
        goodsList:[],
        // 总数据
        total:0,
         // 默认的空图片
            defaultPic: 'https://img3.doubanio.com/f/movie/8dd0c794499fe925ae2ae89ee30cd225750457b4/pics/movie/celebrity-default-medium.png',
            // 是否真正请求数据
            isloading:false
      };
    },
    onLoad(options) {
      // 将页面参数转存到 this.queryObj 对象中
      this.queryObj.query = options.query || ''
      this.queryObj.cid = options.cid || '',
      this.getGoodsLoist()
    },
    methods:{
      async getGoodsLoist(cb){
         // 打开节流阀
        this.isloading=true
        const {data:res}= await uni.$http.get('/api/public/v1/goods/search',this.queryObj)
        // 关闭节流阀
        this.isloading=false
          // 只要数据请求完毕，就立即按需调用 cb 回调函数
          cb && cb()
        if(res.meta.status!==200) return uni.$showMsg()
        this.goodsList=[...this.goodsList, ...res.message.goods]
        this.total=res.message.total
      },
      // 点击跳转到商品详情页面
      gotoDetail(goods) {
        uni.navigateTo({
          url: '/subpkg/goods_detail/goods_detail?goods_id=' + goods.goods_id
        })
      }
  },
  onReachBottom(){
      // 判断是否还有下一页数据
      if (this.queryObj.pagenum * this.queryObj.pagesize >= this.total) return uni.$showMsg('数据加载完毕！')
      // 判断是否正在请求其它数据，如果是，则不发起额外的请求
      if (this.isloading) return
    // 让页码值自增 从而下拉获取第n页的数据
    this.queryObj.pagenum+=1
    // 重新获取列表数据
    this.getGoodsLoist()
  },
  // 下拉刷新的事件
  onPullDownRefresh() {
    // 1. 重置关键数据
    this.queryObj.pagenum = 1
    this.total = 0
    this.isloading = false
    this.goodsList = []
  
    // 2. 重新发起请求
    this.getGoodsLoist(() => uni.stopPullDownRefresh())
  }
  }
</script>

<style lang="scss">
.goods-item {
  display: flex;
  padding: 10px 5px;
  border-bottom: 1px solid #f0f0f0;

  .goods-item-left {
    margin-right: 5px;

    .goods-pic {
      width: 100px;
      height: 100px;
      display: block;
    }
  }

  .goods-item-right {
    display: flex;
    flex-direction: column;
    justify-content: space-between;

    .goods-name {
      font-size: 13px;
    }

    .goods-price {
      font-size: 16px;
      color: #c00000;
    }
  }
}
</style>
