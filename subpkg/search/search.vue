<template>
  <view class="search-box">
    <!-- 使用 uni-ui 提供的搜索组件 -->
    <uni-search-bar v-model="kw" :radius="100" clearButton="auto" cancelButton="none" :focus="true"></uni-search-bar>
    <!--  @clear="clearList" -->
    <!-- 搜索建议列表 -->
    <view class="sugg-list" v-if="searchResults.length !== 0">
      <view class="sugg-item" v-for="item in searchResults" :key="item.goods_id" @click="gotoDetail(item.goods_id)">
        <view class="goods-name">{{item.goods_name}}</view>
        <uni-icons type="arrowright" size="16"></uni-icons>
      </view>
    </view>
    <!-- 搜索历史 -->
    <view class="history-box" v-else>
      <!-- 标题区域 -->
      <view class="history-title">
        <text>搜索历史</text>
        <uni-icons type="trash" size="17" @click="cleanHistory"></uni-icons>
      </view>
      <!-- 列表区域 -->
      <view class="history-list">
        <uni-tag :text="item" v-for="(item, i) in historyList" :key="i" :inverted="true"
          @click="gotoGoodsList(item)"></uni-tag>
      </view>
    </view>
  </view>
</template>

<script>
  export default {
    data() {
      return {
        // 延时器的 timerId
        timer: null,
        // 搜索关键词
        kw: '',
        // 搜索结果列表
        searchResults: [],
        // 搜索关键词的历史记录
        historyList: []
      }
    },
    onLoad() {
      this.historyList = JSON.parse(uni.getStorageSync('kw') || '[]')
    },
    watch: {
      kw: {
        handler(newVal, oldVal) {
          clearTimeout(this.timer)
          this.timer = setTimeout(() => {
            if (newVal !== '') {
              this.getSearchList();
            }
            if (this.kw === '') {
              this.searchResults = []
            }
          }, 1000)
        }
      }
    },
    methods: {
      // 根据搜索关键词，搜索商品建议列表
      async getSearchList() {
        // 发起请求，获取搜索建议列表
        const {
          data: res
        } = await uni.$http.get('/api/public/v1/goods/qsearch', {
          query: this.kw
        })
        if (res.meta.status !== 200) return uni.$showMsg()
        this.searchResults = res?.message
        // 1. 查询到搜索建议之后，调用 saveSearchHistory() 方法保存搜索关键词
        if (this.searchResults.length > 0 && this.searchResults) this.saveSearchHistory()
      },
      // 2. 保存搜索关键词的方法
      saveSearchHistory() {
        // 2.1 直接把搜索关键词 push 到 historyList 数组中
        this.historyList.unshift(this.kw)
        this.historyList = [...new Set(this.historyList)]

        // 调用 uni.setStorageSync(key, value) 将搜索历史记录持久化存储到本地
        uni.setStorageSync('kw', JSON.stringify(this.historyList))
        console.log(this.historyList);
      },
      // 跳转商品详情页面
      gotoDetail(goods_id) {
        uni.navigateTo({
          // 指定详情页面的 URL 地址，并传递 goods_id 参数
          url: `/subpkg/goods_detail/goods_detail?goods_id=${goods_id}`
        })
      },
      // 清空搜索历史记录
      cleanHistory() {
        // 清空 data 中保存的搜索历史
        this.historyList = []
        // 清空本地存储中记录的搜索历史
        uni.setStorageSync('kw', '[]')
      },
      // 点击跳转到商品列表页面
      gotoGoodsList(kw) {
        uni.navigateTo({
          url: '/subpkg/goods_list/goods_list?query=' + kw
        })
      },
    },
  }
</script>

<style lang="less">
  .search-box {
    position: sticky;
    top: 0;
    z-index: 999;

    .uni-searchbar {
      /* #ifndef APP-NVUE */
      display: flex;
      /* #endif */
      flex-direction: row;
      position: relative;
      padding: 16rpx;
      /* 将默认的 #FFFFFF 改为 #C00000 */
      background-color: #c00000;
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
          border: none;
        }
      }
    }
  }
</style>