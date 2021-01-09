<template>
  <view>
    <view class="search-box">
      <!-- 使用 uni-ui 提供的搜索组件 -->
      <uni-search-bar @input="input" :radius="100" cancelButton="none"></uni-search-bar>
    </view>
    <!-- 搜索建议列表 -->
    <view class="sugg-list" v-if="searchResults.length!==0">
      <view class="sugg-item" v-for="(item,i) in searchResults" :key="i" @click="gotoDetail(item)">
        <view class="goods-name">{{item.goods_name}}</view>
        <uni-icons type="arrowright" size="16"></uni-icons>
      </view>
    </view>
    
    <!-- 搜索历史 -->
    <view class="history-box" v-if="searchResults.length===0">
      <!-- 标题区域 -->
      <view class="history-title">
        <text>搜索历史</text>
        <uni-icons type="trash" size="17" @click='cleanHistory'></uni-icons>
      </view>
      <!-- 列表区域 -->
      <view class="history-list">
        <uni-tag :text="item" v-for="(item, i) in  historys" :key="i" @click="gotoGoodsList(item)"></uni-tag>
      </view>
    </view>
  </view>
</template>

<script>
  export default {
    data() {
      return {
        // 延期器的timerID
        timer: null,
        // 搜素的关键词
        kw: '',
        // 搜索结果列表
        searchResults: [],
        // 搜索历史的数据
        historyList:[]
      };
    },
    onLoad() {
      this.historyList=JSON.parse(uni.getStorageSync('kw')||'[]')
    },
    computed: {
      historys() {
        // 注意：由于数组是引用类型，所以不要直接基于原数组调用 reverse 方法，以免修改原数组中元素的顺序
        // 而是应该新建一个内存无关的数组，再进行 reverse 反转
        return [...this.historyList].reverse()
      }
    },
    methods: {
      input(e) {
        // 每次进来先清除之前timer对应的延时器
        clearTimeout(this.timer)
        // 在重新启动一个延时器,并把timerID赋值给this.timer
        this.timer = setTimeout(() => {
          // 如果500毫秒内,没有触发新的输入事件,则为搜索关键词赋值
          this.kw = e.value
          // console.log(this.kw)
          // 根据关键词，查询搜索建议列表
          this.getsearchList()
        }, 500)
        // e.value为最新的搜素内容
        // console.log(e.value)
      },
      async getsearchList() {
        if (this.kw === '') {
          this.searchResults = []
          return
        }
        // 获取搜索结果数据
        const {
          data: res
        } = await uni.$http.get('/api/public/v1/goods/qsearch', {
          query: this.kw
        })
        if (res.meta.status !== 200) return uni.$showMsg()
        this.searchResults = res.message
         this.saveSearchHistory()
      },
      gotoDetail(item) {
        console.log(item.goods_id)
        uni.navigateTo({
          // 指定详情页面的 URL 地址，并传递 goods_id 参数
            url: '/subpkg/goods_detail/goods_detail?goods_id=' + item.goods_id
        })
      },
      // 保存关键词
      saveSearchHistory() {
          // 2.1 直接把搜索关键词 push 到 historyList 数组中
            // this.historyList.push(this.kw)
          // 解决关键词重复
          // 将Array数组转化为set对象
          const set = new Set(this.historyList)
          set.delete(this.kw)
          set.add(this.kw)
          this.historyList=Array.from(set)
           // 调用 uni.setStorageSync(key, value) 将搜索历史记录持久化存储到本地
            uni.setStorageSync('kw', JSON.stringify(this.historyList))
      },
      cleanHistory(){
        this.historyList=[]
        uni.getStorageSync('kw','[]')
      },
     // 点击关键词跳转到商品列表
     gotoGoodsList(kw){
       uni.navigateTo({
           url: '/subpkg/goods_list/goods_list?query=' + kw
       })
     } 
    }
  }
</script>

<style lang="scss">
  .search-box {
    position: sticky;
    top: 0;
    z-index: 999;
  }

  .sugg-list {
    padding: 0 5px;

    .sugg-item {
      font-size: 12px;
      padding: 13px 0;
      border-bottom: 1px solid #efefef;
      display: flex;
      align-items: center;
      justify-content: space-between;

      .goods-name {
        // 文字不允许换行（单行文本）
        white-space: nowrap;
        // 溢出部分隐藏
        overflow: hidden;
        // 文本溢出后，使用 ... 代替
        text-overflow: ellipsis;
        margin-right: 3px;
      }
    }
  }
  .history-box {
    padding: 0 5px;
  
    .history-title {
      display: flex;
      justify-content: space-between;
      align-items: center;
      height: 40px;
      font-size: 13px;
      border-bottom: 1px solid #efefef;
    }
  
    .history-list {
      display: flex;
      flex-wrap: wrap;
  
      .uni-tag {
        margin-top: 5px;
        margin-right: 5px;
      }
    }
  }
</style>
