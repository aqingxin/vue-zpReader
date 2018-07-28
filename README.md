# vue-zpReader
一个vue+vuex+axios+mint-ui实现的小说阅读器

> 技术栈:vue2.0+vuex+axios+mint-ui

> API:https://github.com/xiadd/zhuishushenqi （感谢）

## 部署
先git大佬的api接口
```bash
git https://github.com/xiadd/zhuishushenqi.git
npm instal
npm run dev
```
然后git本仓库
``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

```
## 实现功能

- [x] 小说书架

- [x] 分类查询

- [x] 书单查询

- [x] 排行榜

- [x] 搜索

- [x] 小说详情

- [x] 小说换源

- [x] 阅读历史记录

## 截图
![cat.png](https://github.com/ZpsssLll/vue-zpReader/blob/master/screenshot/cat.png)
![mybooks.png](https://github.com/ZpsssLll/vue-zpReader/blob/master/screenshot/mybooks.png)
![rank.png](https://github.com/ZpsssLll/vue-zpReader/blob/master/screenshot/rank.png)
![booklist.png](https://github.com/ZpsssLll/vue-zpReader/blob/master/screenshot/booklist.png)
![reader.png](https://github.com/ZpsssLll/vue-zpReader/blob/master/screenshot/reader.png)
![bookDetail.png](https://github.com/ZpsssLll/vue-zpReader/blob/master/screenshot/bookDetail.png)
![cat-list.png](https://github.com/ZpsssLll/vue-zpReader/blob/master/screenshot/cat-list.png)
![index.png](https://github.com/ZpsssLll/vue-zpReader/blob/master/screenshot/index.png)

#### 问题
- [x] mint-ui ```infinite-scroll``` 滚动的问题，无限滚动会加载多次！
