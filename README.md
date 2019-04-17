## Vue Routerを使ったページネーション機能を作成

Vue.js + Vue Routerで、ブラウザバックに対応したページネーションを作成しました。

### 仕様
- Vue.js + Vue Routerだけで実装
- ブラウザの「戻る」「進む」でページ遷移可能にする
- PCではページ番号を最大5件表示、SPではセレクトボックスでページ番号を選択
- ページ送りボタンは、1ページ目と最後のページでは無効にする

Vue.jsでのページネーションは、探すと色々サンプル見つかりますが、ブラウザバックに対応したサンプルは意外と少ないのと、スマホでの見せ方も考えると、自分で作成した方がよいかと思い、Vue Routerの勉強も兼ねて作ってみました。

単純にページネーションさせるだけなら、[Vue.js Examples](https://vuejsexamples.com/tag/pagination/)のサンプルとか参考にしてみるといいかと。


CDNを利用するので、公式サイトに合わせて、[jsdelivr](https://cdn.jsdelivr.net/npm/vue/)でVue.jsを、vue-router.jsとaxios.jsは[unpkg](https://unpkg.com/)で読み込みます。
(cdnjsは同期に少し時間がかかるため、最新版ではない可能性があるよう）

### axiosを使ってAjax通信でJsonを取得
`
axiosとは
Node.jsで動くAjaxリクエストを送るためのPromiseベースのHTTPクライアント。
[npmの公式ページより](https://www.npmjs.com/package/axios)
`

Vueのドキュメントで推奨されているAjax通信のライブラリで、Jsonを取得する際によく利用されるみたい。

取得したJsonデータを、データオブジェクトで定義した配列[items]に格納し、items.lengthを表示件数（1ページ分）で割って切り上げた数値を最大のページ番号（max）に設定します。

```rb
methods: {
  getJson: function(){
    axios.get('json/data.json')
    .then(function (response) {
      var res_data = response.data;
      list.items = res_data;
      list.max = Math.ceil(list.items.length / list.showCount);
    }).catch(function (error) {
      console.log(error);
    });
  }
}
```

### 全体的なレイアウト
ルートテンプレート内に親コンポーネント<news-container>を登録。
テンプレート<news-container>では、下記の子コンポーネントを読み込みます。

- 記事の表示用：<article-list>
- PC用ページネーション：<pagination>
- SP用セレクトボックス：<sp-select>

```rb
<div id="app">
  <news-container></news-container>
</div>

<template id="news-container">
  <div class="newsList">
    <div class="newsList__list">
      <article-list :items="showItems"></article-list>
    </div>
    <div class="newsList__pager">
      <pagination :current-num="page" :max-num="max"></pagination>
    </div>
    <div class="newsList__select">
      <sp-select :current-num="page" :max-num="max"></sp-select>
    </div>
  </div>
</template>
```


Vue インスタンスに router プロパティを渡してルータを初期化し、templateに表示するための要素をIDで指定します。
```rb
var list = new Vue({
  router: router,
  template: '#news-container',
  data: function() {
    return {
      page: 1,
      max: 5,
      showCount: 12,
      items: [],
    }
  }
}).$mount('#app');
```

#### 子コンポーネント

```rb
<template id="article-list">
  <div class="articleList">
    <transition-group class="articleList__body" name="fade" tag="ul">
      <li class="articleList__item" v-for="(item, index) in items" v-bind:key="item.id">
        <article class="articleCard">
          <a class="articleCard__inner" href="#" :href="item.url">
            <p class="articleCard__img"><img :src="item.img" :alt="item.title"></p>
            <div class="articleCard__text">
              <h2 class="articleCard__ttl">{{item.title}}</h2>
            </div>
          </a>
        </article>
      </li>
    </transition-group>
  </div>
</template>

<script>
  Vue.component('article-list', {
    template: '#article-list',
    props: {
      items: {
        default: function() {
          return []
        },
        required: true
      }
    }
  });
</script>
```

```rb
<template id="pagination">
  <nav class="pagination">
    <ul class="pagination__body">
      <li class="pagination__item">
        <router-link class="pagination__btn pagination__btn--prev" :to="{ path: '', query: {page: (parseInt(currentNum) - 1).toString()}}" v-if="currentNum > 1">前へ</router-link>
        <span class="pagination__btn pagination__btn--prev pagination__btn--disable" v-else>前へ</span>
      </li>
      <li class="pagination__item" v-for="(item, index) in pagerData" :key="item">
        <router-link class="pagination__btn" :to="{ path: '', query: {page: (item).toString()}}" :class="{'pagination__btn--active': (item === currentNum)}">{{item}}</router-link>
      </li>
      <li class="pagination__item">
        <router-link class="pagination__btn pagination__btn--next" :to="{ path: '', query: {page: (parseInt(currentNum) + 1).toString()}}" v-if="currentNum < maxNum">次へ</router-link>
        <span class="pagination__btn pagination__btn--next pagination__btn--disable" v-else>次へ</span>
      </li>
    </ul>
  </nav>
</template>

<script>
  Vue.component('pagination', {
    template: '#pagination',
    props: {
      currentNum: {
        type: Number,
        default: 1,
        required: true
      },
      maxNum: {
        type: Number,
        default: 1,
        required: true
      },
    },
    computed: {
      pagerData: function() {
        if(this.maxNum <= 5) {
          var arr = [];
          for(var i = 1; i <= this.maxNum; i++) {
            arr.push(i);
          }
          return arr;
        }
        else if(this.currentNum <= 3) {
          return [1,2,3,4,5];
        }
        else if(this.currentNum >= this.maxNum - 2) {
          return [this.maxNum-4,this.maxNum-3,this.maxNum-2,this.maxNum-1,this.maxNum];
        }
        else {
          return [this.currentNum-2,this.currentNum-1,this.currentNum,this.currentNum+1,this.currentNum+2];
        }
      }
    }
  });
</script>
```

```rb
<template id="sp-select">
  <div class="spPager">
    <ul class="spPager__body">
      <li class="spPager__item spPager__item--btn">
        <router-link class="spPager__btn spPager__btn--prev" :to="{ path: '', query: {page: (parseInt(currentNum) - 1).toString()}}" v-if="currentNum > 1">前へ</router-link>
        <span class="spPager__btn spPager__btn--prev spPager__btn--disable" v-else>前へ</span>
      </li>
      <li class="spPager__item spPager__item--select">
        <select class="spPager__select" v-model="selectValue" @change="selectChange()">
          <option v-for="option in pages" :value="parseInt(option.val)" :key="option.val">{{option.text}}</option>
        </select>
        <p class="spPager__selectlabel"> {{currentNum}} / {{maxNum}}</p>
      </li>
      <li class="spPager__item spPager__item--btn">
        <router-link class="spPager__btn spPager__btn--next" :to="{ path: '', query: {page: (parseInt(currentNum) + 1).toString()}}" v-if="currentNum < maxNum">次へ</router-link>
        <span class="spPager__btn spPager__btn--next spPager__btn--disable" v-else>次へ</span>
      </li>
    </ul>
  </div>
</template>
<script>
  Vue.component('sp-select', {
    template: '#sp-select',
    data: function() {
      return {
        selectValue: 1
      }
    },
    props: {
      currentNum: {
        type: Number,
        default: 1,
        required: true
      },
      maxNum: {
        type: Number,
        default: 1,
        required: true
      },
    },
    computed: {
      pages: function() {
        var obj = [];
        for(var i = 0; i < this.maxNum; i++) {
          obj[i] = {
            'val': i+1,
            'text': i+1+'ページ'
          }
        }
        return obj;
      }
    },
    methods: {
      selectChange: function() {
        this.$router.push({ path: '', query: {page: this.selectValue}});
      }
    }
  });
</script>
```