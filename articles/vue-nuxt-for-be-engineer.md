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
- これを実現するために、Vue.js では単一ファイルコンポーネント（Single File Component; SFC）という仕組みがある。
- SFC では、1 つのコンポーネントにかかわるテンプレート（HTML）、スクリプト（JS）、スタイル（CSS）を 1 つの `.vue` ファイルにまとめる。

```vue:ButtonCounter.vue
<template>
  <button @click="count++">Count is: {{ count }}</button>
</template>

<script lang="ts">
import { defineComponent } from "vue";

export default defineComponent({
  data: () => ({
    count: 0,
  }),
});
</script>

<style scoped>
button {
  font-weight: bold;
}
</style>
```

↑ https://ja.vuejs.org/guide/introduction.html#single-file-components より引用（一部改変）

- `<template> ~ </template>`: HTML を記述
- `<script> ~ </script>`: JS を記述
  - `lang="ts"`: JS ではなく TS で記述するための宣言
- `<style> ~ </style>`: CSS を記述
  - `scoped`: CSS が適用される範囲をこのコンポーネントに閉じることが可能

次のように、他のコンポーネントの取り込み（import）も簡単にできる。

```vue:ButtonCounterWrapper.vue
<template>
  <h1>Here is a child component!</h1>
  <ButtonCounter />
</template>

<script lang="ts">
import { defineComponent } from "vue";
import ButtonCounter from "./ButtonCounter.vue";

export default defineComponent({
  components: {
    ButtonCounter,
  },
});
</script>
```

↑ https://ja.vuejs.org/guide/essentials/component-basics.html#using-a-component より引用（一部改変）

- 昔はファイルを分割するときは、通常 HTML・JS・CSS で分割することが多かった。
- 一方で、Vue.js ではコンポーネント単位でファイルを分割する。これは以下の理由による。
  - 「関心事項の分離」が「ファイルタイプの分離」と等しくない。
    - SFC なら「ボタン」という関心事項に含まれるソースコード（テンプレート、ロジック、スタイル）一式を 1 ファイルに記述できる。
    - 結果的にコンポーネントの一貫性と保守性が高くなる。

## Options API と Composition API

Vue.js では歴史的経緯から 2 つの API スタイルがある。

### Options API

`data`、`methods`、`mounted` といった数々のオプションからなる 1 つのオブジェクトを用いてコンポーネントのロジックを定義するスタイル。

```vue:ButtonCounter.vue（Options API）
<template>
  <button @click="increment">Count is: {{ count }}</button>
</template>

<script lang="ts">
import { defineComponent } from "vue";

export default defineComponent({
  // data() で返すプロパティはリアクティブな状態になり、
  // `this` 経由でアクセスすることができます。
  data: () => ({
    count: 0,
  }),

  // ライフサイクルフックは、コンポーネントのライフサイクルの
  // 特定のステージで呼び出されます。
  // 以下の関数は、コンポーネントが「マウント」されたときに呼び出されます。
  mounted() {
    console.log(`The initial count is ${this.count}.`);
  },

  // メソッドの中身は、状態を変化させ、更新をトリガーさせる関数です。
  // 各メソッドは、テンプレート内のイベントハンドラーにバインドすることができます。
  methods: {
    increment() {
      this.count++;
    },
  },
});
</script>
```

### Composition API

インポートした各種 API 関数を使ってコンポーネントのロジックを定義するスタイル。

以下は、上記のサンプルコードを Composition API で書き換えたものとなる。

```vue:ButtonCounter.vue（Composition API）
<template>
  <button @click="increment">Count is: {{ count }}</button>
</template>

<script setup lang="ts">
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
```

詳細は以下を参照してください。（サンプルコードもこちらから引用しています）  
https://ja.vuejs.org/guide/introduction.html#api-styles

## 基本的な文法

公式チュートリアルが充実しているので、そちらをやってみるのが良いと思います。

https://ja.vuejs.org/tutorial

## Vue Router

シングルページアプリケーションでは、クライアントサイドのルーターが必要になります。

- マルチページアプリケーション（MPA）の場合
  - リンクを踏む
  - サーバーサイドでルーティングして**全データを取得**
  - 画面**遷移**
- シングルページアプリケーション（SPA）の場合
  - リンクを踏む
  - クライアントサイドでルーティングして**必要なデータだけ API 呼び出し**
  - 画面**更新**

Vue.js 用のルーターが Vue Router です。使い方は以下の公式ガイドの通りで、至って簡単です。

https://router.vuejs.org/guide/#JavaScript

なお、Nuxt.js では pages ディレクトリに vue ファイルを配置するだけでルーティングが登録されます。

https://nuxt.com/docs/getting-started/routing

## Vuex

Vuex とは？

- Vue.js アプリケーションのための 状態管理パターン + ライブラリ。
  - 状態管理パターンとして Flux の影響を強く受けている（後述）
- 予測可能な方法によってのみ状態の変更を行うというルールを保証。
  - アプリケーションのあちこちで状態を変更できると、どこで状態変更がされたかが追いいづらくなる。デバッグが困難になる。
- アプリケーション内の全てのコンポーネントのための集中型のストアとして機能。
  - 複数のコンポーネント間で状態を共有することができる。
  - ただし、見方を変えると「グローバル変数」とも言えるので、何でもかんでも共有するのは好ましくない。

https://vuex.vuejs.org/ja

[Flux](https://qiita.com/knhr__/items/5fec7571dab80e2dcd92) とは？

- 単一方向にデータが流れる。
  - 状態の変更を追いやすくなる。
- Action: ユーザーからの操作を表わすオブジェクト
- Dispatcher: Action を受け取って Store に渡す
- Store: データを保持する場所。Action を受け取ってデータを変更する。変更を通知する
- View: Store のデータを表示する

![](https://github.com/facebookarchive/flux/raw/main/examples/flux-concepts/flux-simple-f8-diagram-with-client-action-1300w.png)  
↑ https://github.com/facebookarchive/flux/tree/main/examples/flux-concepts より引用

Vuex では...

- Action: ユーザーからの操作を表わすオブジェクト。API を叩くなどのデータ処理を実行する。
- Mutations: 状態を変更できる唯一の手段。Action から commit されることで実行できる。
- State: データを保持する場所。変更を通知する。
- Vue Components: State のデータを表示する。Action を dispatch する。

![](https://vuex.vuejs.org/vuex.png)  
↑ https://vuex.vuejs.org/ja より引用

概念だけだと分かりにくいので、コード例を示します。  
簡単なカウンターを使って説明します。

```ts:store.ts
import { createStore } from 'vuex';

createStore({
  // store で管理しているデータ。
  state: () => ({
    counter: 0,
  }),
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

```vue:ButtonCounter.vue
<template>
  <button @click="increment">{{ counter }}</button>
</template>

<script>
import { defineComponent } from "vue";

export default defineComponent({
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
});
</script>
```

実際には [モジュール](https://vuex.vuejs.org/ja/guide/modules.html) に分割して利用することが多いです。  
Nuxt2 では、store ディレクトリを用いることで簡単に利用ができます。

https://v2.nuxt.com/ja/docs/directory-structure/store

また、Nuxt2 までは Vuex が組み込まれていましたが、[Nuxt3 では組み込まれなくなりました](https://nuxt.com/docs/getting-started/state-management)。状態管理ライブラリを使用する場合は Pinia が推奨され、別途[簡単な導入手順](https://nuxt.com/modules/pinia)を踏む必要があります。

https://nuxt.com/docs/getting-started/state-management

# Nuxt.js とは

- Vue.js アプリケーションを構築するためのフレームワーク
- 環境準備が大変、ディレクトリ構成迷う、というような課題を吸収してくれる
- Nuxt.js を使えば、Vue.js でアプリケーションを構築するときのデファクトスタンダードな構成が手に入る

Nuxt.js には以下の要素 ＋ それらを利用するための諸々の設定が含まれている。

| カテゴリ     | Nuxt2                                                                    | Nuxt3                                                                                                                            |
| ------------ | ------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------- |
| Vue          | [Vue2](https://jp.vuejs.org/)                                            | [Vue3](https://ja.vuejs.org/)                                                                                                    |
| ルーティング | [Vue Router 3](https://v3.router.vuejs.org/ja)                           | [Vue Router 4](https://router.vuejs.org/)                                                                                        |
| 状態管理     | [Vuex](https://vuex.vuejs.org/ja/)                                       | なし（[useState](https://nuxt.com/docs/getting-started/state-management), [Pinia](https://nuxt.com/modules/pinia) などから選択） |
| SSR          | [vue-server-renderer](https://www.npmjs.com/package/vue-server-renderer) | [vue/server-renderer](https://www.npmjs.com/package/@vue/server-renderer)                                                        |
| メタタグ管理 | [vue-meta](https://vue-meta.nuxtjs.org/)                                 | [@unhead/vue](https://github.com/unjs/unhead)                                                                                    |
| バンドラー   | [Webpack4](https://v4.webpack.js.org/) + [Babel](https://babeljs.io/)    | [Vite](https://ja.vitejs.dev/) or [Webpack5](https://webpack.js.org/)                                                            |
| サーバー     | [Express](https://expressjs.com/ja/)                                     | [nitro](https://github.com/unjs/nitro), [h3](https://github.com/unjs/h3)                                                         |
| テスト       | [Jest](https://jestjs.io/ja/)                                            | [Vitest](https://vitest.dev/)                                                                                                    |

Nuxt.js による開発では、基本的には Vue.js の知識をそのまま利用できる。  
加えて、Nuxt 特有のディレクトリ構造や機能はあるので、それらは別途必要に応じて学ぶ必要がある。

https://nuxt.com/docs/getting-started
