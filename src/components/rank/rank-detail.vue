<template>
  <div class="bookLists">
    <mt-header :title="title" class="book-header">
      <router-link to="/rank" slot="left">
        <mt-button icon="back">返回</mt-button>
      </router-link>
    </mt-header>
    <div class="select">
      <ul class="select-bar">
        <li v-for="(item,index) in largeTypes" :class="{'active':index===largeTypeIndex}"
            @click="setLargeType(item.type,index)">{{item.name}}
        </li>

      </ul>
    </div>
    <div class="load-more" ref="load">
      <mt-loadmore :bottom-method="loadBottom" ref="loadmore" :auto-fill="false"
                   :bottom-all-loaded="allLoaded">
        <div class="book-list">
          <book-list :book="book" v-for="(book,index) in bookLists" :key="index" :ranks="ranks"
                     :isLoadMore="isLoadMore"></book-list>
          <div class="loading-container" v-show="!bookLists.length">
            <div style="text-align: center;margin-top: 10px;">没有找到更多书籍</div>
          </div>
        </div>
      </mt-loadmore>
    </div>
  </div>
</template>
<script type="text/ecmascript-6">
  import {mapGetters} from 'vuex'
  import BookList from 'common/book-list'
  import {Indicator} from 'mint-ui'
  import API from  '../../api'
  const api = new API()
  export default{
    data(){
      return {
        largeTypeIndex: 0,
        title: '',
        largeTypes: [
          {
            name: '周榜',
            type: 'week'
          },
          {
            name: '月榜',
            type: 'month'
          },
          {
            name: '总榜',
            type: 'total'
          },
        ],
        bookLists: [],
        allLoaded: false,
        monthRank: '',
        totalRank: '',
        weekRank: '',
        currentLoadPage: 1,
        type: 'week',
        ranks: true,
        isLoadMore: false
      }
    },
    computed: {
      ...mapGetters([
        'rank'
      ]),
    },
    methods: {
      setLargeType(type, index){
        //点击后重置滚动距离
        this.$refs.load.scrollTop = 0;
        this.bookLists = [];
        this.currentLoadPage = 1;
        this.largeTypeIndex = index;
        this.type = type;
        this.allLoaded = false;
        this._getBookLists(type);
      },
      _getBookLists(url){
        Indicator.open('加载中');
        if (!this.rank.weekRank) {
          //在刷新页面的时候记录每个排行榜的id值
          api.get(`/rank/${this.$route.params.id}`, {}, (res) => {
            this.weekRank = res.ranking.id;
            this.monthRank = res.ranking.monthRank;
            this.totalRank = res.ranking.totalRank;
            this.title = res.ranking.title
          })
        } else {
          //从排行榜列表也点击进去排行榜详情记录id的值
          this.weekRank = this.rank.weekRank;
          this.monthRank = this.rank.monthRank;
          this.totalRank = this.rank.totalRank
        }
        if (!url) {
          //判断是否传参，如果没有传参那证明是用户在当前页面刷新，即把$route.params.id传给下面
          url = this.$route.params.id
        } else {
          //有传参那么证明是用户单击tab切换，直接把上面记录的值赋值给url
          url = this[url + 'Rank']
        }
        api.get(`/rank/${url}`, {}, (res) => {
          if (res.ok) {
            this.bookLists = this._unEscape(this.normalizeBooks(res.ranking.books.slice(0, 20)));//url转义
            Indicator.close();
          } else {
            this.bookLists = [];//奇葩，有些榜单没有数据，但是追书神器也把链接做出来了，所有当榜单没有数据的时候，清空当前的bookLists
            Indicator.close();
          }
        });

      },
      _unEscape(arr){
        for (let i = 0; i < arr.length; i++) {
          arr[i].cover = unescape(arr[i].cover);
          arr[i].cover = arr[i].cover.replace("/agent/", "")
        }
        return arr
      },
      loadBottom(){
        this.isLoadMore = true;
        Indicator.open('加载中');
        api.get(`/rank/${this[this.type + 'Rank']}`, {}, (res) => {
          if (this.bookLists.length === res.ranking.books.length) {
            this.allLoaded = true;
          }
          this.bookLists = this._unEscape(this.normalizeBooks(res.ranking.books.slice(0, this.currentLoadPage * 20 + 20)));
          this.currentLoadPage++;
          setTimeout(() => {
            Indicator.close();
            this.isLoadMore = false;
          }, 350)
        });

        this.$refs.loadmore.onBottomLoaded()
      }
    },
    components: {
      BookList,
    },
    beforeRouteEnter(to, from, next){
      next(vm => {
        vm.title = vm.$route.query.title;
        vm._getBookLists();
        vm.largeTypeIndex = 0
      })
    }
  }
</script>
<style scoped lang="stylus" rel="stylesheet/stylus">
  .bookLists
    .book-header
      height 100px
      line-height 100px
      font-size 25px
      background #409eff
      position absolute
      width 100%
      z-index 10
    .select {
      position absolute
      top 100px
      left 0
      background #fff
      z-index 10
      width 100%
    }
    .select-bar {
      display flex
      flex-direction row
      justify-content flex-start
      flex-wrap nowrap
      height 100px
      width 100%
      overflow-x auto
      overflow-y hidden
      border-bottom 1px solid #f2f2f2
    }
    .select-bar li {
      text-align center
      flex 1
      line-height 100px
      padding 0 20px
      font-size 25px
    }
    .book-list {
      width 100%
      background #fff
    }
    .load-more
      padding-top 200px
      overflow-y scroll
      height 100vh
      box-sizing border-box
    .active {
      color #409eff
      border-bottom 1px solid #409eff; /* no*/
    }
</style>
