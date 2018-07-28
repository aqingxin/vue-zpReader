<template>
  <div class="booklist-item">
    <mt-header title="书单详情" class="book-header">
      <router-link to="/bookLists" slot="left">
        <mt-button icon="back">返回</mt-button>
      </router-link>
    </mt-header>
    <div class="author">
      <div class="author-top">
        <img :src="img+author.avatar" alt="">
        <div class="top-right">
          <div class="name">{{author.nickname}}</div>
          <div class="lastTime">{{upDate <= 0 ? '刚刚更新' : `${upDate}小时前更新`}}
          </div>
        </div>
      </div>
      <div class="author-des">
        <h1>{{title}}</h1>
        <div class="desc">{{desc}}</div>
      </div>
    </div>
    <div class="item">
      <ul>
        <li @click="goBooks(item.book._id)" v-for="item in booklist">
          <div class="book-info">
            <div class="img">
              <img :src="imgEscpe(item.book.cover)"
                   alt="">
            </div>
            <div class="book-right">
              <h1>{{item.book.title}}</h1>
              <div class="book-data">
                <span>{{item.book.author}}</span> | <span>{{item.book.cat}}</span> | <span
                class="c-rq">{{latelyFollower(item.book.latelyFollower)}}</span>人气
              </div>
            </div>
          </div>
        </li>
      </ul>
    </div>
  </div>
</template>
<script type="text/ecmascript-6">
  import API from '../api'
  const api = new API()
  export default{
    data(){
      return {
        booklist: [],
        title: '',
        desc: '',
        author: {},
        img: 'http://statics.zhuishushenqi.com',
        upDate: ''
      }
    },
    computed: {},
    created(){
      this.getBooksList(this.$route.params.id)
      console.log(this.author)
    },
    methods: {
      latelyFollower (lately) {
        return (lately / 10000).toFixed(1)
      },
      imgEscpe(url){
        return unescape(url.replace("/agent/", ""));

      },
      goBooks(id){
        this.$router.push({
          path: `/book/${id}`
        });
      },
      getBooksList(id){
        api.get(`/booklists/${id}`, {}, (res) => {
          this.upDate = this._time(res.bookList.updated);
          this.author = res.bookList.author;
          this.desc = res.bookList.desc;
          this.title = res.bookList.title;
          this.booklist = res.bookList.books
        })
      },
      _unEscape(arr){
        for (let i = 0; i < arr.length; i++) {
          arr[i].cover = unescape(arr[i].cover);
          arr[i].cover = arr[i].cover.replace("/agent/", "")
        }
        return arr
      },
      _time(time){
        let currentTime = Math.round(new Date() / 1000);
        this.upDate = time.replace("T", ' ').replace("Z", ' ');
        this.upDate = Date.parse(new Date(this.upDate)) / 1000;
        return Math.floor((currentTime - this.upDate) % 86400 / 3600)
      },
    }
  }
</script>
<style scoped lang="stylus" rel="stylesheet/stylus">
  .booklist-item
    position fixed
    left 0
    top 0
    right 0
    bottom 0
    width 100%
    z-index 100
    background #fff
    overflow-y scroll
    max-width: 750px; /*no*/
    margin: 0 auto;
    .book-header
      height 100px
      line-height 100px
      font-size 25px
      background #409eff
      width 100%
    .author
      padding 30px
      position relative
      box-sizing border-box
      border-bottom 1px solid #ebebeb; /*no*/
      .author-top
        height 80px
        img
          width 70px
          height 70px
          position absolute
          left 30px
          top 30px
        .top-right
          margin-left 80px
          .name
            margin-bottom 10px

      .author-des
        h1
          line-height 80px;
          height 80px
          font-size 25px
        .desc
          line-height 40px
          color #999
    .item
      .book-info
        padding 30px
        display flex
        border-bottom 1px solid #ebebeb
        box-sizing border-box
        .img
          height 140px
          width 100px
          margin-right 10px
          overflow hidden
          background url('../assets/images/default.jpg') no-repeat
          background-position 50%
          background-size 100%
          img
            max-width 100%
        .book-right
          flex 1
          h1
            line-height 80px
            height 80px
            font-size 25px
          .c-rq
            color red
</style>
