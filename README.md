## Personal Note

### fork from [RealWorld](https://github.com/gothinkster/realworld)


### 架構
- .vue檔，分成views相關的，和component相關的folder
- common 裡面包括 api服務，jwt服務，和global filter的定義
- api.service.js 就是簡單的 get, post, update, put這個物件一開始還有啟動
- 下面就是相關的API call 分成 TagsService, ArticlesService, CommentsService, FavoriteService, 分別處理相關的API呼叫
- 這樣可以讓每一個函數清楚好讀
- jwt服務也拉出來，主要就是local storage的存取


### Store
- store有定義type, 像是actions.type, mutations.type
- 模組化，這邊分成article, auth, home, profile, settings
- 統一由store/index.js 把模組包好，輸出一個Vuex實例


### 一個事件處理的例子
1. 到setting, 點擊 update setting 觸發事件-> invoke Setting.vue 裡面的method `updateSettings` ，裡面會 dispatch `UPDATE_USER` action
-> `auth.module.js` 裡面已經有定義好 這個action 要進行的操作，哪到更新的資料，透過api.service 的 put 請求，拿到response後，commit mutation `SET_AUTH`
-> 這個mutaion 就會 change state -> 最後用 `this.$router.push` 返回 `/home`


### 如何找ArtilceAction.vue的作用？
看看誰會import他就知道
我們在ArticleMeta.vue看到這段
```
import RwvArticleActions from "@/components/ArticleActions";
```
又在Article.vue看到這段
```
import RwvArticleMeta from "@/components/ArticleMeta";
```
另外又可以在@/router/index.js裡面看到這段
```
  {
      name: "article",
      path: "/articles/:slug",
      component: () => import("@/views/Article"),
      props: true
    },
```
因此可以判定，但你連到/articles/:slug(這裡是動態生成的)
你看到的是Article.vue，裡面包了ArticleMeta.vue這個組件（當然也可以包其他的）
然後ArticleMeta.vue這個組件裡面又包了ArticleActions.vue這個組件。


### Article.vue, ArticleMeta.vue and ArticleActions.vue
1. Article.vue 是你進入article畫面看到的畫面
2. ArticleMeta是負責user icon, user name, article created date的那個組件
3. ArticleActions有兩個button link, 如果你是作者，就是edit and delete, 如果不是作者，就是follow author 和 Favorite

## 一些.vue檔的說明

### app.vue
- UI的進入點，就header, 中間用router-view 下面用footer這樣而以。很簡單。

### components/ArticleList.vue
- 用v-if設計一個loading狀態的顯示, 類似loading中
- 再包一個v-if，設計一個loading結果後如果len為零的顯示, 類似沒有文章
- 然後裡面又包了一個VArticlePreview的組件
- 也利用了VPagination組件來實現Pagination
- data裡面的currentPage來動態調整上面的Pagination
- once mount，就會跑 fetchArticles, 而 fetchArticles 會dispatch FETCH_ARTICLES action with payload this.listConfig
- this.listConfig 在computed 裡面，基本會返回type and filter屬性, type從this裡面拿資料，filter裡面也會從this裡面拿很多屬性的資料
- 拿的屬性包括offset要設定多少，limit, author, tag, favorited
- computed裡面也要計算page數量，算法也沒那麼直覺就是。
- 也透過mapGetters拿到articlesCount, isLoading, articles
- 另外也watch滿多屬性的，如果這些屬性變動，都會 跑 `resetPagination`和`fetchArticles`，watch 的屬性 包括 type, author, tag, favorited
- 也watch currentPage, 如果變了，會從算listConfig.filters.offset和fetchArticles

### main.js
- 用到router, vuex
- inject global filter
- init Api service
- also use router.beforeEach to ensure we check auth for each page load, 這裡用promise.all


### Register.vue
- 相對單純，只有一個method和一個computed, computed會去拿auth那邊的error, 如果有的話。
- method就是用來submit註冊的mail, pass and username，成功的話會轉去home page
- 這邊頁面的連接都是用router link去做，看情況會加上參數
- 這邊有用一個v-if來條件判斷是否需要顯示error, 另外也用上global filter顯示第一個錯誤訊息就好。

### Settings.vue
- 一樣透過computred 然後用mapGetters拿到store那邊的currentUser
- 有一個form 來 updateSettings  and logout
- 這兩個method都會dispatch action，然後進行轉路徑


### views/Home.vue
- 裡面包了HomeGlobal.vue和HomeMyFeed.vue
- 這兩個分別是nav bar上你選不同情況下，會展示出不同條件的ArticleList組件（透過type prop）
- 也包括了homeTag





## Getting started

```bash
# install editorconfig globally
> npm install -g editorconfig
```

The stack is built using [vue-cli webpack](https://github.com/vuejs-templates/webpack) so to get started all you have to do is:

``` bash
# install dependencies
> yarn install
# serve with hot reload at localhost:8080
> yarn serve
```

Other commands available are:

``` bash
# build for production with minification
yarn run build

# run unit tests
yarn test
```
