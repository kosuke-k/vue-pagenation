## Vue Routerを使ったページネーション機能を作成

Vue.js + Vue Routerで、ブラウザバックに対応したページネーションを作成しました。

### 要件
- Vue.js + Vue Routerだけで実装
- ブラウザの「戻る」「進む」でページ遷移可能にする
- ページ送りボタンは、PCのみ表示（最大5件）、SPではセレクトボックスにする
- ページ送りボタンは、1ページ目と最後のページでは無効にする
- データは、Jsonファイルから読み込む

[Demo](https://kohskeleton.github.io/vue-pagenation/)


Vue.jsでのページネーションは、探すと色々サンプル見つかりますが、ブラウザバックに対応したサンプルは意外と少なく、スマホでの見せ方も検討する必要があるため、Vue Routerの勉強も兼ねて作ってみました。サンプルということで、Vue.jsの処理はHTMLに埋め込んでいます。

単純にページネーションさせるだけなら、[Vue.js Examples](https://vuejsexamples.com/tag/pagination/)のサンプルとか参考にしてみるといいかと。


CDNを利用するので、公式サイトに合わせて、[jsdelivr](https://cdn.jsdelivr.net/npm/vue/)でVue.jsを、vue-router.jsとaxios.jsは[unpkg](https://unpkg.com/)で読み込みます。<br />
(cdnjsは同期に少し時間がかかるため、最新版ではない可能性があるよう）

### axiosを使ってAjax通信でJsonを取得

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

### View
ルートテンプレート内に親コンポーネント`<news-container>`を登録。<br />
テンプレート`<news-container>`では、下記の子コンポーネントを読み込みます。

- 記事の表示用：`<article-list>`
- PC用ページネーション：`<pagination>`
- SP用セレクトボックス：`<sp-select>`

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
