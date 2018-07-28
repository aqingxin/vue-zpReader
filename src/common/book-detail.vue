<template>
  <div class="books-detail">
    <mt-header :title="title" class="book-header">
      <div slot="left" @click="$router.back(-1)">
        <mt-button icon="back">返回</mt-button>
      </div>
    </mt-header>
    <div class="book-detail">
      <div class="cover">
        <img :src="_unEscape(book.cover)" alt="">
      </div>
      <div class="desc">
        <h1>{{book.title}}</h1>
        <p class="info">
          <span>{{book.author}}</span> | <span>{{book.minorCate}}</span> | <span>{{latelyCount(book.wordCount)}}</span>
        </p>
        <p class="update">{{upDate <= 0 ? '刚刚更新' : `${upDate}小时前更新`}}</p>
      </div>
    </div>
    <div class="read-link">
      <div class="sj" @click="setBookLocalStorage()">{{flag ? '不追了' : '追更新'}}</div>
      <div class="read" @click="goReadBooks()">开始阅读</div>
    </div>
    <div class="reader-data">
      <p class="item">
        <span class="key">追人气</span>
        <span>{{latelyFollower(book.latelyFollower)}}</span>
      </p>
      <p class="item">
        <span class="key">读者存留率</span>
        <span>{{book.retentionRatio}}%</span>
      </p>
      <p class="item">
        <span class="key">日更字数/天</span>
        <span>{{book.serializeWordCount}}</span>
      </p>
    </div>
    <div class="introduction">
      {{book.longIntro}}
    </div>
    <div class="zj">
      {{upDate}}小时前更新-{{book.lastChapter}}
    </div>
    <div class="comment">
      <p class="comment-title">热门评论</p>
      <ul>
        <li v-for="item in comments" class="comment-list">
          <div class="inner">
            <div class="cover">
              <img :src="imgUrl+item.author.avatar" alt="">
            </div>
            <div class="right">
              <p class="name">{{item.author.nickname}}</p>
              <p class="title">{{item.title}}</p>
              <p class="content">{{item.content}}</p>
            </div>
          </div>
        </li>
      </ul>
    </div>
  </div>
</template>
<script type="text/ecmascript-6">
  import {mapGetters} from 'vuex'
  import API from '../api'
  const api = new API();
  export default{
    data(){
      return {
        title: '书籍详情',
        book: {},
        upDate: '',
        comments: [],
        sUrl: 'http://statics.zhuishushenqi.com',
        flag: false
      }
    },
    computed: {
      imgUrl(){
        return this.sUrl
      },
      ...mapGetters([
        'bookId'
      ])
    },
    created(){
    },
    methods: {
      latelyFollower (lately) {
        if (lately > 10000) {
          return (lately / 10000).toFixed(1)
        } else {
          return lately
        }

      },
      latelyCount(count){
        if (count > 10000) {
          return (count / 10000).toFixed() + '万字'
        } else {
          return count + '字'
        }
      },
      getBook(id){
        api.get(`/book-info/${id}`, {}, (res) => {
          this.book = res;
          this.upDate = this._time(this.book.updated);
          this.isFlag()
        });
      },
      _unEscape(str){
        str = unescape(str).replace("/agent/", "")
        return str
      },
      _time(time){
        let currentTime = Math.round(new Date() / 1000);
        time = time.replace("T", ' ').replace("Z", ' ');
        time = Date.parse(new Date(time)) / 1000;
        return Math.floor((currentTime - time) % 86400 / 3600)
      },
      getComments(id){
        api.get(`/book/reviews?book=${id}&sort=updated&start=0&limit=10`, {}, (res) => {
          this.comments = res.reviews
        })
      },
      isFlag () {
        // 返回本地是否缓存（加入书架）
        let localShelf = JSON.parse(window.localStorage.getItem('book'))
        this.flag = !!(localShelf && localShelf[this.book._id])
      },
      setBookLocalStorage(){
        let localShelf = JSON.parse(window.localStorage.getItem('book')) ? JSON.parse(window.localStorage.getItem('book')) : {};
        if (this.flag) {
          delete localShelf[this.book._id];
          window.localStorage.setItem('book', JSON.stringify(localShelf))
          this.flag = !this.flag
        } else {
          localShelf[this.book._id] = {
            cover: this.book.cover,
            title: this.book.title,
            id: this.book._id,
            chapterIndexCache: null,
            bookSource: 0
          };
          window.localStorage.setItem('book', JSON.stringify(localShelf))
          this.flag = !this.flag
        }
      },
      goReadBooks(){
//        this.setBookLocalStorage();
        this.$router.push({
          path: `/readBook/${this.$route.params.id}`
        })
      }
    },
    beforeRouteEnter(to, from, next){
      /*获取大类中的小类*/
      next(vm => {
        vm.getBook(vm.$route.params.id);
        vm.getComments(vm.$route.params.id)
      })
    }
  }
</script>
<style scoped lang="stylus" rel="stylesheet/stylus">
  .books-detail
    position fixed
    z-index 100
    top 0
    left 0
    right 0
    bottom 0
    background #fff
    width 100%
    overflow-y scroll
    max-width 750px; /* no*/
    margin 0 auto
    .book-header
      height 100px
      line-height 100px
      font-size 25px
      background #409eff

    .book-detail
      display flex
      padding 30px
      .cover
        flex 0 0 150px
        height 200px
        margin-right 10px
        overflow hidden
        background url('../assets/images/default.jpg') no-repeat
        background-position 50%
        background-size 100%
        img
          max-width 100%
      .desc
        h1
          height 50px
          font-size 28px
        p
          height 40px
          color #999

    .read-link
      display flex
      padding 0 30px 30px 30px
      font-size 30px
      div
        flex 1
        text-align center
        line-height 80px
        height 80px
      .sj
        border 1px solid #409eff
        color #409eff
        box-sizing border-box
        margin-right 10px
      .read
        background #409eff
        color #fff
        margin-left 10px

    .reader-data
      padding 30px 0
      border-top 1px solid #ebebeb
      border-bottom 1px solid #ebebeb
      box-sizing border-box
      font-size 0
      .item
        display inline-block
        width 33%
        text-align center
        vertical-align middle
        span
          font-size 16px;
        /* no*/
        .key
          display block
          font-size 12px; /* no*/
          color #999
          margin-bottom 20px

    .introduction
      width 100%
      padding 30px
      box-sizing border-box
      font-size 13px
      color #999
      line-height 40px
      overflow hidden
      text-align justify
      border-bottom 1px solid #ebebeb

    .zj
      padding 30px
      box-sizing border-box
      text-align right
      color #999
      border-bottom 1px solid #ebebeb

    .comment
      padding-left 30px
      .comment-title
        height 60px
        line-height 60px
      .comment-list
        padding 30px 0
        border-bottom 1px solid #ebebeb
        .inner
          position relative
          .cover
            position absolute
            width 70px
            height 70px
            background url('../assets/images/default.jpg') no-repeat
            background-position: 50%;
            background-size: 100%;
            border-radius 50%
            img
              width 70px
              height 70px
              border-radius 50%
          .right
            padding 0 30px 0 80px
            .name
              font-size 20px
              color #999
              height 30px
            .title
              font-size 20px
              font-weight 600
              height 30px
            .content
              height 1.06667rem
              overflow hidden
              line-height .53333rem
              text-align justify
              color #999
</style>
