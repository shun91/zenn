---
title: "バックエンドエンジニアのためのVue.js/Nuxt.js入門"
emoji: "📝"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

JS のフレームワークの 1 つである Vue.js・Nuxt.js の基本的な内容を、バックエンドエンジニアの方向けに説明します。

:::message
5 年ぐらい前に書いたものを最新状況に合わせて加筆修正した記事となりますが、古い内容などが含まれていたらごめんなさい mm
:::

# Vue.js とは

- UI 構築のための JavaScript フレームワーク
- 標準的な HTML を拡張したテンプレート構文を使って、HTML の出力を宣言的に記述できる
- JavaScript の状態の変化を自動的に追跡し、変化が起きると効率的に DOM を更新してくれる
  - ユーザーの入力も追跡してくれる（双方向バインディング）

## 単一ファイルコンポーネント（SFC）

- 画面を開発していく時に、コンポーネント単位で区切って開発することが望ましい（理由は後述）。
- これを実現するために、Vue.js では SFC という仕組みがある。
  - 厳密には他の仕組みもありますが、ここでは割愛。
- SFC では、1 つのコンポーネントにかかわるテンプレート（HTML）、スクリプト（JS）、スタイル（CSS）を 1 つの `.vue` ファイルにまとめる。

```vue
<script>
export default {
  data() {
    return {
      count: 0,
    };
  },
};
</script>

<template>
  <button @click="count++">Count is: {{ count }}</button>
</template>

<style scoped>
button {
  font-weight: bold;
}
</style>
```

↑ https://ja.vuejs.org/guide/introduction.html#single-file-components より引用

- `<template> ~ </template>`: HTML を記述
- `<script> ~ </script>`: JS を記述
- `<style> ~ </style>`: CSS を記述
  - scoped: CSS が適用される範囲をこのコンポーネントに閉じることが可能

次のように、他のコンポーネントの取り込み（import）も簡単にできる。

```vue
<script>
import ButtonCounter from "./ButtonCounter.vue";

export default {
  components: {
    ButtonCounter,
  },
};
</script>

<template>
  <h1>Here is a child component!</h1>
  <ButtonCounter />
</template>
```

↑ https://ja.vuejs.org/guide/essentials/component-basics.html#using-a-component より引用

- 昔はファイルを分割するときは、通常 HTML・JS・CSS で分割することが多かった。
- 一方で、Vue.js ではコンポーネント単位でファイルを分割します。これは以下の理由による。
  - 「関心事項の分離」が「ファイルタイプの分離」と等しくない。
    - SFC なら「ボタン」という関心事項に含まれるソースコード（テンプレート、ロジック、スタイル）一式を 1 ファイルに記述できる。
    - 結果的にコンポーネントの一貫性と保守性が高くなる。

## Options API と Composition API

Vue.js では歴史的経緯から 2 つの API スタイルがある。

### Options API

`data`、`methods`、`mounted` といった数々のオプションからなる 1 つのオブジェクトを用いてコンポーネントのロジックを定義するスタイル。

```vue
<script>
export default {
  // data() で返すプロパティはリアクティブな状態になり、
  // `this` 経由でアクセスすることができます。
  data() {
    return {
      count: 0,
    };
  },

  // メソッドの中身は、状態を変化させ、更新をトリガーさせる関数です。
  // 各メソッドは、テンプレート内のイベントハンドラーにバインドすることができます。
  methods: {
    increment() {
      this.count++;
    },
  },

  // ライフサイクルフックは、コンポーネントのライフサイクルの
  // 特定のステージで呼び出されます。
  // 以下の関数は、コンポーネントが「マウント」されたときに呼び出されます。
  mounted() {
    console.log(`The initial count is ${this.count}.`);
  },
};
</script>

<template>
  <button @click="increment">Count is: {{ count }}</button>
</template>
```

### Composition API

インポートした各種 API 関数を使ってコンポーネントのロジックを定義するスタイル。

以下は、上記のサンプルコードを Composition API で書き換えたものとなる。

```vue
<script setup>
import { ref, onMounted } from "vue";

// リアクティブな状態
const count = ref(0);

// 状態を変更し、更新をトリガーする関数。
function increment() {
  count.value++;
}

// ライフサイクルフック
onMounted(() => {
  console.log(`The initial count is ${count.value}.`);
});
</script>

<template>
  <button @click="increment">Count is: {{ count }}</button>
</template>
```

詳細は以下を参照してください。（サンプルコードもこちらから引用しています）  
https://ja.vuejs.org/guide/introduction.html#api-styles

## 基本的な文法

公式チュートリアルが充実しているので、そちらをやってみるのが良いと思います。

[Tutorial | Vue.js](https://ja.vuejs.org/tutorial/#step-1)

## Vue Router

シングルページアプリケーションでは、クライアントサイドのルーターが必要になります。

- マルチページアプリケーション（MPA）
  - リンクを踏む
  - サーバーサイドでルーティングして**全データを取得**
  - 画面**遷移**
- シングルページアプリケーション（SPA）
  - リンクを踏む
  - クライアントサイドでルーティングして**必要なデータだけ API 呼び出し**
  - 画面**更新**

Vue.js 用のルーターが Vue Router です。使い方は至って簡単です。

```js
new VueRouter({
  // パスに対してコンポーネントを登録する。
  routes: [
    {
      path: "/",
      component: Home,
    },
    {
      path: "/posts/:id",
      component: Post,
    },
  ],
});
```

```html
<template>
  <div>
    <div>
      <!-- <router-link> は <a> になる。 -->
      <!-- クリックするとルーティングされる。 -->
      <router-link to="/">Home</router-link>
      <router-link to="/posts/1">Post 1</router-link>
      <router-link to="/posts/2">Post 2</router-link>
    </div>
    <!-- <router-view> にはルーティング結果の -->
    <!-- コンポーネントが表示される。 -->
    <router-view />
  </div>
</template>
```

なお、Nuxt.js では [pages ディレクトリ](https://nuxt.com/docs/getting-started/routing)に vue ファイルを配置するだけでルーティングが登録されます。

参考：[Vue Router](https://router.vuejs.org)

## Vuex

Vuex とは？

- Vue.js アプリケーションのための 状態管理パターン + ライブラリ。
  - 状態管理パターンとして Flux の影響を強く受けている（後述）
- 予測可能な方法によってのみ状態の変更を行うというルールを保証。
  - アプリケーションのあちこちで状態を変更できると、どこで状態変更がされたかが追いいづらくなる。デバッグが困難になる。
- アプリケーション内の全てのコンポーネントのための集中型のストアとして機能。
  - 複数のコンポーネント間で状態を共有することができる。
  - ただし、見方を変えると「グローバル変数」とも言えるので、何でもかんでも共有するのは好ましくない。

[Flux](https://qiita.com/knhr__/items/5fec7571dab80e2dcd92) とは？

- 単一方向にデータが流れる。
  - 状態の変更を追いやすくなる。
- Dispatcher: Action を受け取って Store に渡す
- Store: データを保持する場所。Action を受け取ってデータを変更する。変更を通知する
- Action: ユーザーからの操作を表わすオブジェクト
- View: Store のデータを表示する。React, Vue, Riot, ...

![](https://github.com/facebookarchive/flux/raw/main/examples/flux-concepts/flux-simple-f8-diagram-with-client-action-1300w.png)  
↑ https://github.com/facebookarchive/flux/tree/main/examples/flux-concepts より引用

Vuex では...

- Action: ユーザーからの操作を表わすオブジェクト。API を叩くなどのデータ処理を実行する。
- Mutations: 状態を変更できる唯一の手段。Action から commit されることで実行できる。
- State: データを保持する場所。変更を通知する。
- Vue Components: State のデータを表示する。Action を dispatch する。

![](https://vuex.vuejs.org/vuex.png)  
↑ https://vuex.vuejs.org/ja より引用

以下のように使います。  
簡単なカウンターを使って説明します。

```js
new Vuex.Store({
  // store で管理しているデータ。
  state: {
    counter: 0,
  },
  // dispatch で呼び出されるメソッド。
  actions: {
    increment({ commit }) {
      commit("INCREMENT");
    },
  },
  // state を変更するメソッド。
  mutations: {
    INCREMENT(state) {
      state.counter++;
    },
  },
});
```

```html
<template>
  <button @click="increment">{{ counter }}</button>
</template>
<script>
  export default {
    computed: {
      // store から counter を取り出す。
      // 取り出した値はテンプレート内で使える。
      counter() {
        return this.$store.state.counter;
      },
    },
    methods: {
      // ボタンがクリックされたら
      // store の increment を dispatch する。
      increment() {
        this.$store.dispatch("increment");
      },
    },
  };
</script>
```

実際には [モジュール](https://vuex.vuejs.org/ja/guide/modules.html) に分割して利用することが多いです。  
Nuxt.js では、[store ディレクトリ](https://v2.nuxt.com/ja/docs/directory-structure/store) を用いることで簡単に利用ができます。

また、Nuxt2 までは Vuex が組み込まれていましたが、[Nuxt3 では組み込まれなくなりました](https://nuxt.com/docs/getting-started/state-management)。状態管理ライブラリを使用する場合は Pinia が推奨され、別途[簡単な導入手順](https://nuxt.com/modules/pinia)を踏む必要があります。

参考：[Vuex](https://vuex.vuejs.org/ja/)

# Nuxt.js とは

- Vue.js アプリケーションを構築するためのフレームワーク
- 環境準備が大変、ディレクトリ構成迷う、というような課題を吸収してくれる
- Nuxt.js を使えば、Vue.js でアプリケーションを構築するときのデファクトスタンダードな構成が手に入る

Nuxt.js には以下の要素 ＋ それらを利用するための諸々の設定が含まれている。

| カテゴリ     | Nuxt2               | Nuxt3                                                                    |
| ------------ | ------------------- | ------------------------------------------------------------------------ |
| Vue          | Vue2                | Vue3                                                                     |
| ルーティング | Vue Router 3        | Vue Router 4                                                             |
| 状態管理     | Vuex                | なし（useState, Pinia などから選択）                                     |
| SSR          | vue-server-renderer | vue/server-renderer                                                      |
| メタタグ管理 | vue-meta            | [@unhead/vue](https://github.com/unjs/unhead)                            |
| バンドラー   | Webpack4 + Babel    | Vite or Webpack5                                                         |
| サーバー     | Express             | [nitro](https://github.com/unjs/nitro), [h3](https://github.com/unjs/h3) |
| テスト       | Jest                | Vitest                                                                   |

基本的には Vue.js の知識をそのまま利用できる。  
加えて、Nuxt 特有のディレクトリ構造や機能はあるので、それらは別途必要に応じて学ぶ必要がある。
