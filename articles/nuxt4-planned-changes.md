---
title: "Nuxt4 で予定されている変更点まとめ"
emoji: "🗂"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

# ⚠ 注意

- この記事は作成中です
- 2024/04 上旬時点での情報です
- 内容には誤りを含む可能性があります。参照したソースを記載しているので、合わせて確認してください

# ソース

https://nuxt.com/blog/looking-forward-2024

https://github.com/nuxt/nuxt/issues?q=is:issue+label:4.x
https://github.com/orgs/nuxt/projects/8/views/4

# 【Breaking Change】Data fetching composables から params の削除

[Data fetching composables](https://nuxt.com/docs/getting-started/data-fetching) ( [`useFetch`](https://nuxt.com/docs/api/composables/use-fetch), [`$fetch`](https://nuxt.com/docs/api/utils/dollarfetch) ) にクエリパラメータを渡すには `query` と `params` という 2 つのプロパティが使えます。このうち、`params` が廃止されるようです。

```ts:NG
const { data } = await useFetch(
  "https://jsonplaceholder.typicode.com/comments",
  { params: { postId: 1 } } // params は使えなくなる
);
```

```ts:OK
const { data } = await useFetch(
  "https://jsonplaceholder.typicode.com/comments",
  { query: { postId: 1 } } // query は引き続き使える
);
```

https://github.com/nuxt/nuxt/issues/25208

# 【Breaking Change】Data fetching composables の返り値から pending の削除

[Data fetching composables](https://nuxt.com/docs/getting-started/data-fetching) ( [`useFetch`](https://nuxt.com/docs/api/composables/use-fetch), [`useAsyncData`](https://nuxt.com/docs/api/composables/use-async-data), [`$fetch`](https://nuxt.com/docs/api/utils/dollarfetch) ) は、データが fetch 中かどうかを表すものを 2 つ提供しています。

- 返り値の `pending`
- 返り値の `status` (`status.value === "pending"`)

このうち、「返り値の `pending`」が廃止されるようです。

```ts:NG
// pending は返されなくなる
const { data, pending } = await useFetch("https://jsonplaceholder.typicode.com/comments");
```

```ts:OK
// status は引き続き使える
const { data, status } = await useFetch("https://jsonplaceholder.typicode.com/comments");
const pending = status.value === "pending";
```

https://github.com/nuxt/nuxt/issues/25225

# 【Breaking Change】composables, plugins, middleware の自動登録の一部廃止

composables, plugins, middleware はそれぞれのディレクトリの以下の場所にファイルを配置するだけで、 Nuxt の起動時に自動で読み込まれます。

- ディレクトリの最上位にあるファイル
- サブディレクトリ内の `index.ts`

このうち、「サブディレクトリ内の `index.ts`」の自動登録が廃止されるようです。

https://github.com/nuxt/nuxt/issues/25331

# `compileTemplate` 関数の廃止

[Nuxt Kit](https://nuxt.com/docs/guide/going-further/kit)の内部？で使用されている `compileTemplate` という関数が廃止されるようです。

https://github.com/nuxt/nuxt/issues/25332

# レガシーな `layout.ts` の廃止

Nuxt 内部で使用されている `layout.ts` というファイルが削除されるようです。

https://github.com/nuxt/nuxt/issues/25333

# `window.__NUXT__` の廃止

Hydration のためなどに Nuxt 内部で用いられている`window.__NUXT__`が廃止されるようです。
もし、`window.__NUXT__`相当のデータにアクセスする必要がある場合は、代わりに [`window.useNuxtApp`](https://github.com/nuxt/nuxt/pull/21636) を使用できます。

https://github.com/nuxt/nuxt/issues/25336

# `templateUtils` の廃止

Nuxt 内部で使用されている `templateUtils` が廃止されるようです。

https://github.com/nuxt/nuxt/issues/25338

# 【破壊的変更】`builder:watch`フックの仕様変更

[`builder:watch`](https://nuxt.com/docs/api/advanced/hooks#nuxt-hooks-build-time) フックで Emit される path が絶対パスに変更されるようです。

https://github.com/nuxt/nuxt/issues/25339

# Data fetching composables における `dedupe` の仕様変更

[Data fetching composables](https://nuxt.com/docs/getting-started/data-fetching) ( [`useFetch`](https://nuxt.com/docs/api/composables/use-fetch), [`useAsyncData`](https://nuxt.com/docs/api/composables/use-async-data) ) のオプションとして指定できる `dedupe` には `cancel` と `defer` という文字列の他に boolean ( `true` or `false` ) も指定できます。  
このうち、boolean での指定を廃止するようです。

ちなみに、[公式ドキュメント](https://nuxt.com/docs/api/composables/use-async-data#params)上では既に boolean の記述は削除されているようです。

https://github.com/nuxt/nuxt/issues/25344

# 【Breaking Change】Plugin の読み込みをデフォルトで並列化

Nuxt3 では、Plugin の読み込みはデフォルトでは直列に行われます。[`parallel` というオプションに `true` を渡す](https://nuxt.com/docs/guide/directory-structure/plugins#parallel-plugins)ことで、これを並列化することができます。

Nuxt4 では、デフォルトで並列に読み込まれるようにするようです。

ただし、読み込み順が決まっている plugin では予期せぬ動作を引き起こす可能性もあるため、この変更が取り込まれるかは以下の issue で議論が行われています。

https://github.com/nuxt/nuxt/issues/25350

# Nuxt4 へのマイグレーションガイドとツールの提供

Nuxt4 へのマイグレーションを省力化するためのガイドやツールの提供が計画されているようです。

https://github.com/nuxt/nuxt/issues/25713

# 【Breaking Change】`tsconfig.json` に `noUncheckedIndexedAccess` の追加

型安全性を向上させるため、Nuxt が出力する `tsconfig.server.json` と `tsconfig.json` に `noUncheckedIndexedAccess` を 追加するようです。

https://github.com/nuxt/nuxt/issues/25789

# 【Breaking Change】`clearNuxtData`の関数名と仕様の変更

[`clearNuxtData`](https://nuxt.com/docs/api/utils/clear-nuxt-data) を呼び出すと、`data` の値が `undefined` になります。  
これを、`default` プロパティがセットされている場合は、その値になるように変更されるようです。

合わせて、関数名が実態に合うように `clearNuxtData` から `resetNuxtData` に変更されるようです。

また、[`clear`](https://nuxt.com/docs/getting-started/data-fetching#clear)関数を呼び出した際の挙動や、関数名も同様に変更されるようです。

```ts
const { data } = await useFetch(
  "https://jsonplaceholder.typicode.com/comments",
  { default: () => [], key: "comments" }
);
clearNuxtData("comments");
console.log(data); // Nuxt3 では undefined が、Nuxt4 では [] が出力される
```

https://github.com/nuxt/nuxt/issues/26269

#

https://github.com/nuxt/nuxt/issues/26295

---

# 【Breaking Change】ディレクトリ構造の改善

新しいディレクトリ構造が提案されています。現時点で有力な案は以下のようなものです。

```
.output/
.nuxt/
app/
  assets/
  components/
  composables/
  layouts/
  middleware/
  pages/
  plugins/
  utils/
  main.vue
  router.options.ts
content/
layers/
modules/
node_modules/
public/
server/
  api/
  middleware/
  plugins/
  routes/
  utils/
nuxt.config.ts
```

- app ディレクトリが追加され、既存のディレクトリの一部がこの配下に移動しています。
- layers ディレクトリが追加されています。[こちらの機能](https://nuxt.com/docs/getting-started/layers)に関するディレクトリのようです。

現在は以下の issue で熱い議論？が繰り広げられているようです。（ディレクトリ名など）

https://github.com/nuxt/nuxt/issues/26444
