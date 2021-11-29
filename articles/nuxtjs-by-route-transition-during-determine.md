---
title: "[Nuxt.js] route 遷移中を判定する"
emoji: "🐥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["vue", "vuerouter", "nuxtjs"]
published: true
---

## 概要

Nuxt.js を使っていて「route 遷移している最中かどうか」を判定したいことがあったので、プラグインを作ってみました。

## 実装

```js:~/plugins/routeChanging.js
export default ({ app }, inject) => {
  const state = { routeChanging: false };
  inject('routeChanging', () => state.routeChanging);

  // beforeEach と afterEach で遷移中の状態を切り替える
  app.router.beforeEach((to, from, next) => {
    state.routeChanging = true;
    next();
  });
  app.router.afterEach(() => (state.routeChanging = false));
};
```

Vue コンポーネントや Vuex ストアから以下のようにして使うことができます。

```vue:hoge.vue
<script>
export default {
  methods: {
    routeChanging() {
      // 遷移中なら true を返す
      return this.$routeChanging();
    }
  }
}
</script>
```

## 素の Vue.js で使いたい

もちろん、素の Vue.js でも Vue.prototype にメソッドを生やしてあげれば使えるはずです。

```js
Vue.prototype.$routeChanging = () => state.routeChanging;
```

## 使い道

route 遷移中って、computed の値なんかが目まぐるしく変化することがあります。
例えば、「computed の値が変更されたら自動的に API リクエストを発行する」処理があった場合、無駄なリクエストを投げてしまうかもしれません。
こんなときに、このプラグインがあれば、route 遷移中はリクエストをしないようにすることができます。

```vue:hoge.vue
<script>
export default {
  methods: {
    async fetch() {
      if (!this.$routeChanging()) {
        return axios.get('/path/to');
      }
    }
  }
}
</script>
```
