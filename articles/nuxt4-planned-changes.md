---
title: "Nuxt4 で予定されている変更点まとめ（2024/04現在）"
emoji: "⚠️"
type: "tech"
topics: ["nuxtjs", "nuxt3", "vue", "frontend"]
published: false
---

先日、Nuxt の公式から以下のようなブログポストがありました。

https://nuxt.com/blog/looking-forward-2024

この中で、**Nuxt4 が 2024/06 にリリース予定**であるとの言及がありました。  
また、合わせて変更予定の内容が以下から確認できるようになっていたので、簡単にではありますがまとめてみました。

https://github.com/nuxt/nuxt/issues?q=is:issue+label:4.x
https://github.com/orgs/nuxt/projects/8/views/4

:::message

- この記事の内容は 2024/04 上旬時点での情報です
- 内容が最新ではなかったり、誤りを含む可能性があります。参照したソースを記載しているので、合わせて確認してください
  :::

# 利用者目線での注目すべき変更点

※ 利用者とは、Nuxt を利用してアプリケーションを開発している人を指します。プラグイン開発者などは含まれません。

- [ディレクトリ構造の改善](#ディレクトリ構造の改善)が特に大きな変更で、大抵の Nuxt プロジェクトで影響を受けるはず。
- その他は小さな変更が中心。以下は利用者に関係がありそうな変更点であり、利用していたら影響を受けるはず。
  - [Data fetching composables から `params` の削除](#data-fetching-composables-から-params-の削除)
  - [Data fetching composables の返り値から `pending` の削除](#data-fetching-composables-の返り値から-pending-の削除)
  - [composables, plugins, middleware の自動登録の一部廃止](#composables%2C-plugins%2C-middleware-の自動登録の一部廃止)
  - [Data fetching composables における `dedupe` の仕様変更](#data-fetching-composables-における-dedupe-の仕様変更)
  - [`tsconfig.json` に `noUncheckedIndexedAccess` の追加](#tsconfig.json-に-nouncheckedindexedaccess-の追加)
  - [`clearNuxtData` の関数名と仕様の変更](#clearnuxtdata-の関数名と仕様の変更)
  - [Data fetching composables の `error` のデフォルト値変更](#data-fetching-composables-の-error-のデフォルト値変更)
  - [Data fetching composables の `data` を shallow ref object に変更](#data-fetching-composables-の-data-を-shallow-ref-object-に変更)
  - [`process.server`, `process.client` の非推奨化](#process.server%2C-process.client-の非推奨化)

一番大きな変更は、何と言っても [ディレクトリ構造の改善](#ディレクトリ構造の改善) です。大抵の Nuxt プロジェクトは影響を受けるものと思われます。

その他は小さな変更が中心ですが、利用していたら影響を受けそうなものは以下です。

# 変更点一覧

## Data fetching composables から `params` の削除

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

## Data fetching composables の返り値から `pending` の削除

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

## composables, plugins, middleware の自動登録の一部廃止

composables, plugins, middleware はそれぞれのディレクトリの以下の場所にファイルを配置するだけで、 Nuxt の起動時に自動で読み込まれます。

- ディレクトリの最上位にあるファイル
- サブディレクトリ内の `index.ts`

このうち、「サブディレクトリ内の `index.ts`」の自動登録が廃止されるようです。

https://github.com/nuxt/nuxt/issues/25331

## `compileTemplate` 関数の廃止

[Nuxt Kit](https://nuxt.com/docs/guide/going-further/kit)の内部？で使用されている `compileTemplate` という関数が廃止されるようです。

https://github.com/nuxt/nuxt/issues/25332

## レガシーな `layout.ts` の廃止

Nuxt 内部で使用されている `layout.ts` というファイルが削除されるようです。

https://github.com/nuxt/nuxt/issues/25333

## `window.__NUXT__` の廃止

Hydration のためなどに Nuxt 内部で用いられている`window.__NUXT__`が廃止されるようです。
もし、`window.__NUXT__`相当のデータにアクセスする必要がある場合は、代わりに [`window.useNuxtApp`](https://github.com/nuxt/nuxt/pull/21636) を使用できます。

https://github.com/nuxt/nuxt/issues/25336

## `templateUtils` の廃止

Nuxt 内部で使用されている `templateUtils` が廃止されるようです。

https://github.com/nuxt/nuxt/issues/25338

## `builder:watch` フックの仕様変更

[`builder:watch`](https://nuxt.com/docs/api/advanced/hooks#nuxt-hooks-build-time) フックで Emit される path が絶対パスに変更されるようです。

https://github.com/nuxt/nuxt/issues/25339

## Data fetching composables における `dedupe` の仕様変更

[Data fetching composables](https://nuxt.com/docs/getting-started/data-fetching) ( [`useFetch`](https://nuxt.com/docs/api/composables/use-fetch), [`useAsyncData`](https://nuxt.com/docs/api/composables/use-async-data) ) のオプションとして指定できる `dedupe` には `cancel` と `defer` という文字列の他に boolean ( `true` or `false` ) も指定できます。  
このうち、boolean での指定を廃止するようです。

ちなみに、[公式ドキュメント](https://nuxt.com/docs/api/composables/use-async-data#params)上では既に boolean の記述は削除されているようです。

https://github.com/nuxt/nuxt/issues/25344

## Plugin の読み込みをデフォルトで並列化

Nuxt3 では、Plugin の読み込みはデフォルトでは直列に行われます。[`parallel` というオプションに `true` を渡す](https://nuxt.com/docs/guide/directory-structure/plugins#parallel-plugins)ことで、これを並列化することができます。

Nuxt4 では、デフォルトで並列に読み込まれるようにするようです。

ただし、読み込み順が決まっている plugin では予期せぬ動作を引き起こす可能性もあるため、この変更が取り込まれるかは以下の issue で議論が行われています。

https://github.com/nuxt/nuxt/issues/25350

## Nuxt4 へのマイグレーションガイドとツールの提供

Nuxt4 へのマイグレーションを省力化するためのガイドやツールの提供が計画されているようです。

https://github.com/nuxt/nuxt/issues/25713

## `tsconfig.json` に `noUncheckedIndexedAccess` の追加

型安全性を向上させるため、Nuxt が出力する `tsconfig.server.json` と `tsconfig.json` に `noUncheckedIndexedAccess` を 追加するようです。

https://github.com/nuxt/nuxt/issues/25789

## `clearNuxtData` の関数名と仕様の変更

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

## Data fetching composables の `error` のデフォルト値変更

[Data fetching composables](https://nuxt.com/docs/getting-started/data-fetching) ( [`useFetch`](https://nuxt.com/docs/api/composables/use-fetch), [`useAsyncData`](https://nuxt.com/docs/api/composables/use-async-data) ) の返り値である `data` と `error` はそれぞれデフォルト値が以下のように異なっています。

- `data`: `undefined`（または `default` プロパティで指定した値）
- `error`: `null`

一貫性を保つために、これを `undefined` に統一するようです。

https://github.com/nuxt/nuxt/issues/26295

## Data fetching composables の `data` を shallow ref object に変更

[Data fetching composables](https://nuxt.com/docs/getting-started/data-fetching) ( [`useFetch`](https://nuxt.com/docs/api/composables/use-fetch), [`useAsyncData`](https://nuxt.com/docs/api/composables/use-async-data) ) の返り値である `data` はデフォルトで deep ref object です。これは `deep` プロパティのデフォルト値が `true` であるためです。  
このデフォルト値が `false` (shallow ref object) に変更されるようです。これにより、パフォーマンスの向上が見込まれています。

https://github.com/nuxt/nuxt/issues/26443

## ディレクトリ構造の改善

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

現在は以下の issue で熱い？議論が繰り広げられているようです。（ディレクトリ名など）

https://github.com/nuxt/nuxt/issues/26444

## `NuxtLink` の軽量化

Nuxt におけるリンクコンポーネントである`NuxtLink`は、多くのユースケースをカバーするように多機能化された影響で、サイズが肥大化しています。これを軽量化することが提案されています。

https://github.com/nuxt/nuxt/issues/26445

## `process.server`, `process.client` の非推奨化

サーバー or クライアントサイドでのみ実行したい処理がある場合などに使用される `process.server`, `process.client` が非推奨となるようです。  
移行先は [`import.meta.server`, `import.meta.client`](https://nuxt.com/docs/api/advanced/import-meta#runtime-app-properties) となります。

また、Nuxt5 で `process.server`, `process.client` は完全に廃止される計画もあるようです。

https://github.com/nuxt/nuxt/issues/26474
