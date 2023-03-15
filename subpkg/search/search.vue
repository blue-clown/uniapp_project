<template>
  <view >
    <view class="search-box">
      <!-- 使用uni-ui提供的搜索组件 -->
    <uni-search-bar @input="input" :radius="100" cancelButton="none"></uni-search-bar>
    </view>
    <!-- 搜索建议列表 -->
    <!-- 按需展示搜索建议和搜索历史 -->
    <view class="sugg-list" v-if="searchResults.length!==0">
      <view class="sugg-item" v-for="(item,i) in searchResults" :key="i" @click="gotoDetail(item.goods_id)">
        <view class="goods_name">
          {{item.goods_name}}
        </view>
        <uni-icons type="arrowright" size="16"></uni-icons>
      </view>
    </view>
    <!-- 搜索历史 -->
    <view class="history-box" v-else>
      <!-- 标题区域 -->
      <view class="history-title">
        <text>搜索历史</text>
        <uni-icons type="trash" size="17"></uni-icons>
      </view>
      <!-- 列表区域 -->
      <view class="history-list">
        <uni-tag :text="item" v-for="(item, i) in historys" :key="i" @click="gotoGoodsList(item)"></uni-tag>
      </view>
    </view>
  </view>
</template>

<script>
  export default {
    data() {
      return {
        // 延时器的timerId
        timer:null,
        // 搜索关键词
        kw:'',
        // 搜索结果列表
        searchResults:[],
        // 搜索关键词的历史记录
        historyList:['a','apple'],
      };
    },
    computed:{
      // 解决关键字前后顺序的问题
      historys(){
        // 注意：由于数组是引用类型，所以不要直接基于原数组调用reverse方法，以免修改原数组中元素的顺序
        // 而是应该新建一个内存无关的数组，再进行reverse反转
        return [...this.historyList].reverse()
      }
    },
    onLoad(){
      this.historyList = JSON.parse(uni.getStorageSync('kw') || '[]')
    },
    methods:{
      input(value){
        // 清除timer对应的延时器
        clearTimeout(this.timer);
        // 重新启动一个延时器，并把timerId赋值给this.timer
        this.timer = setTimeout(() => {
          // 如果500毫秒内，没有触发新的输入事件，则为搜索关键字赋值
          this.kw = value;
          // 根据关键字，查询搜索建议列表
          this.getSearchList();
        },500)
      },
      // 获取搜索建议列表
      async getSearchList(){
        // 判断关键词是否为空
        if(this.kw === ''){
          this.searchResults = [];
          return
        }
        // 发起请求，获取搜索建议列表
        const { data: res } = await uni.$http.get('/api/public/v1/goods/qsearch', { query: this.kw })
        console.log(res)
          if (res.meta.status !== 200) return uni.$showMsg()
          this.searchResults = res.message
          // 1.查询到搜索建议之后，调用saveSearchHistory方法保存搜索关键词
          this.saveSearchHistory()
      },
      // 2.保存搜索关键词的方法
      saveSearchHistory(){
        // 2.1 直接把搜索关键词push到historyList数组中（有问题）
        // this.historyList.push(this.kw)
        
        // 解决关键词重复的问题
        // 1.将Array数组转化为Set对象
        const set = new Set(this.historyList)
        // 2.调用Set对象的delete方法，移除对应的元素
        set.delete(this.kw)
        // 3.调用Set对象的add方法，向Set中添加元素
        set.add(this.kw)
        // 4.将Set对象转化为Array数组
        this.historyList = Array.from(set)
        // 调用uni.setStorageSync(key,value)将搜索历史记录持久化存储到本地
        uni.setStorageSync('kw',JSON.stringify(this.historyList))
      },
      // 点击跳转到商品详情页面
      gotoDetail(goods_id){
        uni.navigateTo({
          // 指定详情页面的 URL 地址，并传递 goods_id 参数
              url: '/subpkg/goods_detail/goods_detail?goods_id=' + goods_id

        })
      },
      // 点击跳转到商品列表页面
      gotoGoodsList(kw){
        uni.navigateTo({
          url: '/subpkg/goods_list/goods_list?query=' + kw
        })
      }
    }
  }
</script>

<style lang="scss">
// 实现搜索框的吸顶效果
.search-box{
  position: sticky;
  top:0;
  z-index:999;
}
// 美化搜索建议列表
.sugg-list{
  padding: 0 5px;
  .sugg-item{
    font-size:12px;
    padding: 13px 0;
    border-bottom: 1px solid #efefef;
    display: flex;
    align-items: center;
    justify-content: space-around;
  }
  .goods-name{
    // 文字不允许换行（单行文本）
    white-space:nowrap;
    // 溢出部分隐藏
    overflow: hidden;
    // 文本溢出后，使用...代替
    text-overflow: ellipsis;
    margin-right:3px;
  }
}
//美化搜索历史区域样式
.history-box{
  padding: 0 5px;
  .history-title{
    display: flex;
    // justify-content: space-around;
    align-items: center;
    height: 40px;
    font-size: 13px;
    border-bottom: 1px solid #efefef;
  }
  .history-list{
    display: flex;
    flex-wrap: wrap;
    .uni-tag{
      margin-top: 5px;
      margin-right: 5px;
    }
  }
}
</style>
