<template>
  <div>
    <div class="book" ref="wrapper" @click="setRead">
      <ul class="books-con" v-infinite-scroll="loadMore" infinite-scroll-disabled="loading"
          infinite-scroll-distance="0" infinite-scroll-immediate-check="false">
        <li class="book-content" v-for="book in bookSource" ref="content">
          <h1 class="book-title">{{book.title}}</h1>
          <div class="book-inner" v-html="bookChaptersBody(book.content)"></div>

        </li>
        <div class="loadMore" style="text-align: center" v-show="moreLoading">
          <mt-spinner type="triple-bounce" color="#409eff"></mt-spinner>
        </div>

      </ul>
      <div class="change_source_btn" v-show="sourceBtn" @click.stop="showSource">
        <mt-button type="primary" class="source_btn">换源</mt-button>
      </div>
    </div>

    <div class="page-read-option" :class="{'show-panel':option}" @click="setRead">
      <div class="top-controll-bar">
        <mt-header :title="title" class="book-header">
          <div @click="$router.back(-1)" slot="left">
            <mt-button icon="back">返回</mt-button>
          </div>
          <mt-button slot="right" @click="showSource" class="right">换源</mt-button>
        </mt-header>
      </div>
      <div class="bottom-controll-bar">
        <mt-button type="default" @click="prevChapter">上一章</mt-button>
        <mt-button type="primary" @click.stop="setList">目录</mt-button>
        <mt-button type="default" @click="nextChapter">下一章</mt-button>
      </div>
    </div>
    <div class="chapter-list" v-show="listOption" @click.stop.pervent="setList">
      <div class="list-content">
        <div class="top-bar">
          <p>{{title}}</p>
        </div>
        <div class="chapter-con">
          <div ref="chapterList" class="chapterList">
            <div v-for="(list,index) in chapterList" @click="selectChapter(index)"
                 :class="{'current':bookCurrentPage===index}">{{list}}
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="changeSource" v-show="changerSource" @click.stop="showSource">
      <div class="source-content">
        <ul>
          <li v-for="(item,index) in  SourceList" class="source-item" @click.stop="_changeSource(item,index)">
            <span class="item-left">{{item.name}}</span>
            <p class="center">最后更新：{{item.lastChapter}}</p>
            <div class="select" :class="{'active':_bookSource()===index}">已选择</div>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>
<script type="text/ecmascript-6">
  import {Indicator} from 'mint-ui'
  import Book from '../assets/js/bookSource'
  import API from '../api'
  const api = new API();
  export default{
    data(){
      return {
        bookSource: [],
        bookCurrentPage: 0,
        bookSourceLinks: [],
        loading: false,
        title: '',
        option: false,
        chapterList: [],
        BookSourcesId: 0,
        listOption: false,
        moreLoading: false,
        changerSource: false,
        SourceList: [],
        sourceBtn: false
      }
    },
    computed: {},
    beforeCreate () {
      Indicator.open('加载中')
    },
    created(){
      this.getBookSources(this.$route.params.id)
    },
    methods: {
      //换源列表显示
      showSource(){
        this.changerSource = !this.changerSource;
        this.sourceBtn = false;
      },
      //换源
      _changeSource(item, index){
        this.changerSource = !this.changerSource;
        this.moreLoading = false
        this.bookSource = [];
        Indicator.open('加载中');
        this.bookSourceLinks = [];
        this.BookSourcesId = index;
        this.getBookChapters(item._id);
        if (this.getBookLocalStorage()[this.$route.params.id]) {
          this.setBookLocalStorage()
        }
      },
      //切换目录
      selectChapter(index){
        this.moreLoading = false;
        this.loading = true;
        Indicator.open('加载中...');
        this.bookSource = [];
        this.bookCurrentPage = index;
        this.getChapters(this.bookSourceLinks[index])
        if (this.getBookLocalStorage()[this.$route.params.id]) {
          this.setBookLocalStorage()
        }

      },
      setList(){
        this.listOption = !this.listOption;
        if (!this.option) {
          return
        }
        this.option = !this.option;
      },
      setRead(){
        this.option = !this.option
      },
      bookChaptersBody(books){
        let c = /[\u4E00-\u9FA5\uF900-\uFA2D]/g;
        let _book = JSON.stringify(books).replace(/^"|"$/g, '');
        this.loading = false;
        if (c.test(_book)) {
          return _book.replace(/\\n|\\t|\\r/g, '<br/>')
        } else {
          return 'vip章节，请购买或者换源阅读！' //目前没有换源功能
        }

      },
      //获取书籍源
      getBookSources(id){
        api.get(`/book-sources?view=summary&book=${id}`, {}, (res) => {
          this.SourceList = res;
          //试阅读和加入书籍阅读
          if (this._bookSource()) {
            this.getBookChapters(res[this._bookSource()]._id)
          } else {
            this.getBookChapters(res[this.BookSourcesId]._id)
          }

        })

      },
      //根据书籍源获取章节
      getBookChapters(id){
        let arr = [];
        api.get(`/book-chapters/${id}`, {}, (res) => {
          this.title = res.bookName;
          for (let i = 0; i < res.chapters.length; i++) {
            arr.push(res.chapters[i].title);
            this.bookSourceLinks.push(encodeURIComponent(res.chapters[i].link))
          }
          this.chapterList = arr;//当书有番外篇的时候，追书神奇api返回的内容是错乱的！
          this.getChapters(this.bookSourceLinks[this._chapterIndexCache()]);
        })
      },
      //获取和设置LocalStorage
      getBookLocalStorage(){
        let localShelf = JSON.parse(window.localStorage.getItem('book')) ? JSON.parse(window.localStorage.getItem('book')) : {};
        return localShelf
      },
      setBookLocalStorage(){
        let localShelf = JSON.parse(window.localStorage.getItem('book')) ? JSON.parse(window.localStorage.getItem('book')) : {};
        localShelf[this.$route.params.id].chapterIndexCache = this.bookCurrentPage;
        localShelf[this.$route.params.id].bookSource = this.BookSourcesId;
        window.localStorage.setItem('book', JSON.stringify(localShelf))
      },
      //将书源和章节缓存到LocalStorage
      _chapterIndexCache(){
        return this.getBookLocalStorage()[this.$route.params.id] ? this.getBookLocalStorage()[this.$route.params.id].chapterIndexCache : 0
      },
      _bookSource(){
        if (this.getBookLocalStorage()[this.$route.params.id]) {
          this.BookSourcesId = this.getBookLocalStorage()[this.$route.params.id].bookSource
        } else {
          this.BookSourcesId = 0
        }
        return this.getBookLocalStorage()[this.$route.params.id] ? this.getBookLocalStorage()[this.$route.params.id].bookSource : 0
      },
      //根据章节link获取详细内容
      getChapters(link){
        api.get(`/chapters/${link}`, {}, (res) => {
          if (res.ok) {
            Indicator.close();
            if (res.chapter.cpContent) {
              this.bookSource.push(this._nromalBook(res.chapter.title, res.chapter.cpContent))
            } else {
              this.bookSource.push(this._nromalBook(res.chapter.title, res.chapter.body))
            }
            if (this.getBookLocalStorage()[this.$route.params.id]) {
              this.bookCurrentPage = this._chapterIndexCache()
            }
          } else if (res.code === -1) {
            Indicator.close();
            this.bookSource.push({title: '找不到资源', content: '请切换书源'})
            this.sourceBtn = true
          }

        })
      },
      //构建book类
      _nromalBook(title, content){
        let book = new Book({
          title: title,
          content: content
        });
        return book
      },
      //无限加载
      loadMore() {
        this.loading = true;
        if (this.bookCurrentPage === this.bookSourceLinks.length) {
          return
        }
        this.moreLoading = true;
        this.bookCurrentPage++;
        setTimeout(() => {
          this.getChapters(this.bookSourceLinks[this.bookCurrentPage]);
          if (this.getBookLocalStorage()[this.$route.params.id]) {
            this.setBookLocalStorage();
          }
          this.loading = false;
          this.moreLoading = false
        }, 2500);

      },
      //上下章节切换
      nextChapter(){
        this.moreLoading = false;
        this.loading = true;
        Indicator.open('加载中...');
        this.bookSource = [];
        this.getChapters(this.bookSourceLinks[this.bookCurrentPage + 1]);
        this.bookCurrentPage++;
        if (this.getBookLocalStorage()[this.$route.params.id]) {
          this.setBookLocalStorage()
        }
      },
      prevChapter(){
        this.moreLoading = false;
        this.loading = true;
        Indicator.open('加载中...');
        this.bookSource = [];
        this.getChapters(this.bookSourceLinks[this.bookCurrentPage - 1])
        this.bookCurrentPage--;
        if (this.getBookLocalStorage()[this.$route.params.id]) {
          this.setBookLocalStorage()
        }
      }
    }
  }
</script>
<style scoped lang="stylus" rel="stylesheet/stylus">
  .book
    background #eee6dd
    color #5c5d58
    width 100%;
    height 100vh
    overflow-y scroll
    margin-top -1px
    position relative
    .books-con
      width 100%;
      .book-content
        padding 50px 40px 80px 40px
        .book-inner
          font-size: 18px; /* no*/
          text-align justify
          line-height 50px
        .book-title
          line-height 50px
          font-size 22px; /*no*/
          font-weight 400
          margin-bottom 25px
    .loadMore
      height 80px;
      line-height 80px
      position fixed
      bottom 0
      left 0
      width 100%;

  .page-read-option
    position fixed
    z-index 888
    left 0;
    top: 0;
    bottom: 0;
    width 100%;
    height 100%
    visibility: hidden;
    .book-header
      line-height 80px
      height 80px
      background-color rgba(50, 51, 52, .9)
    .bottom-controll-bar
      position absolute
      bottom 0
      left 0
      width 100%
      text-align center
      font-size 0
      .mint-button
        width 33.33333%
        line-height 80px
        height 80px
        background rgba(50, 51, 52, .9)
        color #fff
      .add_my
        position absolute
        bottom 200px
        right 0
        width 150px
        height 60px
        line-height 60px
        border-radius 50px 0 0 50px

  .show-panel
    visibility: visible;

  .chapter-list
    position fixed
    z-index 999
    left 0;
    top 0;
    bottom 0
    width 100%;
    height 100%
    background transparent
    .list-content
      width 80%;
      height 100%
      background #fff
      position absolute
      .top-bar
        position absolute
        width 100%
        height 120px
        line-height 120px
        font-size 25px
        font-weight 600
        padding-left 20px
        border-bottom 1px solid #ebebeb
        box-sizing border-box
        z-index 100
        background #fff
      .chapter-con
        position: absolute;
        top: 0;
        width: 100%;
        height 100vh
        margin-top 120px
        padding-left 20px
        box-sizing border-box
        overflow-y: scroll;
        .chapterList
          width 100%

          div
            line-height 80px
            height 80px
            font-size 23px
            color #999
            border-bottom 1px solid #ebebeb
            &.current
              color #409EFF

  .right
    width 100%
    height 80px
    text-align right

  .changeSource
    position fixed
    left 0
    top 0
    bottom 0
    right 0
    width 100%
    height 100%
    z-index 1000
    background-color rgba(50, 51, 52, 0.8)
    .source-content
      width 500px
      height 800px
      background #fff
      position absolute
      top 50%
      left 50%
      margin-left -250px;
      margin-top -400px
      border-radius 15px
      padding 20px
      box-sizing border-box
      overflow-y scroll
      .source-item
        line-height 50px
        font-size 25px
        position relative
        .center
          width 80%
          font-size 20px
          color #999
          overflow: hidden;
          text-overflow: ellipsis;
          white-space: nowrap
        .select
          position absolute
          right 0
          top 20px
          background #c9c9c9
          padding 0 10px
          border-radius 15px
          display none
          color #fff
          font-size 18px
        .active
          display block

  .change_source_btn
    width 200px
    height 80px
    text-align center
    line-height 80px
    position absolute
    left 50%;
    top 50%
    margin-top -80px;
    margin-left -100px
    z-index 1000
    .source_btn
      width 100%;
      height 100%;
      font-size 25px
</style>
