---
title: "Nuxt4アップグレードガイドの内容を整理してみた"
emoji: "🚀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nuxtjs", "nuxt4", "vue", "typescript", "migration"]
published: false
---

## はじめに

Nuxt3 から Nuxt4 へのアップグレードは、公式の充実したガイドが用意されているため、基本的には記載されている手順に従って作業すれば問題ありません。

[Upgrade Guide · Get Started with Nuxt v4](https://nuxt.com/docs/4.x/getting-started/upgrade)

しかし、公式ガイドは分量が多く、項目も多岐にわたるため、全体像を把握するのに時間がかかります。この記事では、実際のアップグレード作業を通じて得た知見をもとに、**公式ガイドのどこを重点的に読めばよいかを整理**しました。

最終的には公式ガイドを参照して作業することになりますが、事前にこの記事を読むことで全体の作業を効率化できるはずです。

## アップグレードの全体的な流れ

Nuxt4 アップグレードは、大きく 3 つのステップに分かれます：

| ステップ            | 内容                            | 目的                                 |
| ------------------- | ------------------------------- | ------------------------------------ |
| 1. バージョンアップ | Nuxt4 へのアップデート          | 最新バージョンの導入                 |
| 2. 互換性設定の追加 | v3 の挙動を維持する設定を追加   | **まず動かす**ことを優先             |
| 3. 段階的な移行     | 互換性設定を 1 つずつ外していく | 安全な移行とビッグバンリリースの回避 |

### Step 1: Nuxt4 へのバージョンアップ

[公式ガイドのコマンド](https://nuxt.com/docs/4.x/getting-started/upgrade#latest-release)を実行して Nuxt4 にアップデートします。

### Step 2: 互換性設定の追加による動作確認

この段階では、**既存コードを大量に書き換える前に、まず動かすこと**が重要です。

- 後述の「v3 の挙動を保持する設定」をすべて追加
- 動作がおかしい部分があれば修正（特に互換性設定がない項目）
- 型エラーが発生する可能性があるので対応
- この時点で一度 PR レビュー＆マージできると理想的

### Step 3: 段階的な移行

互換性設定を**1 つずつ外して**動作確認とコード修正を行います。

- [codemod](https://nuxt.com/docs/4.x/getting-started/upgrade#migrating-using-codemods)も活用
- **影響範囲と原因を特定しやすくなる**
- **こまめに PR マージができる**ためビッグバンリリースを避けられる
- 全ての設定を外す必要はないが、将来的には移行が必要
- **フラグを残し続けると新機能が利用できず、負債化する可能性が高い**

:::message
**うまくいかない場合の Tips**

もし Step 2 の時点で動作させるのが困難な場合は、**Nuxt3.18.x を経由する**のも選択肢の一つです。

- Nuxt4 の機能の一部がバックポートされている
- `compatibilityVersion: 4` を設定することで、Nuxt3 の状態で Nuxt4 の動作を再現できる
- [参考: Opting in to Nuxt 4](https://nuxt.com/docs/3.x/getting-started/upgrade#opting-in-to-nuxt-4)

:::

## 項目別の影響範囲と対応方法

実際のアップグレード作業を通じて感じた影響度をまとめました。  
特に影響を受けやすいのは **「New Directory Structure」** と **「Data Fetch（useFetch/useAsyncData）関連項目」** です。

| 項目名                                                                                                                                                                                                               | Impact Level | 影響を受けるケース（代表例）                                                                                 | codemod                               | v3 の挙動を保持する設定                                                                                                    |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | ------------------------------------------------------------------------------------------------------------ | ------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| [New Directory Structure](https://nuxt.com/docs/4.x/getting-started/upgrade#new-directory-structure)                                                                                                                 | Significant  | ほぼ全プロダクト                                                                                             | `nuxt/4/file-structure`               | `export default defineNuxtConfig({ srcDir: '.', dir: { app: 'app' } })`                                                    |
| [Singleton Data Fetching Layer](https://nuxt.com/docs/4.x/getting-started/upgrade#singleton-data-fetching-layer)                                                                                                     | Moderate     | 同じキーで `useAsyncData`/`useFetch` を複数回呼んでいる                                                      | –                                     | `export default defineNuxtConfig({ experimental: { granularCachedData: false, purgeCachedData: false } })`                 |
| [Corrected Module Loading Order in Layers](https://nuxt.com/docs/4.x/getting-started/upgrade#corrected-module-loading-order-in-layers)                                                                               | Minimal      | [Nuxt layers](https://nuxt.com/docs/4.x/guide/going-further/layers) を使用してレイヤーの module 実行順に依存 | –                                     | なし（フックで対応）                                                                                                       |
| [Deduplication of Route Metadata](https://nuxt.com/docs/4.x/getting-started/upgrade#deduplication-of-route-metadata)                                                                                                 | Minimal      | `route.meta.name` などを直接参照                                                                             | –                                     | なし                                                                                                                       |
| [Normalized Component Names](https://nuxt.com/docs/4.x/getting-started/upgrade#normalized-component-names)                                                                                                           | Moderate     | @vue/test-utils の findComponent を使用、`<KeepAlive>`を使用                                                 | –                                     | `export default defineNuxtConfig({ experimental: { normalizeComponentNames: false } })`                                    |
| [Unhead v2](https://nuxt.com/docs/4.x/getting-started/upgrade#unhead-v2)                                                                                                                                             | Minimal      | useHead で`vmid`, `hid`, `children`, `body`等の削除済 API を使用                                             | –                                     | `export default defineNuxtConfig({ unhead: { legacy: true } })`                                                            |
| [New DOM Location for SPA Loading Screen](https://nuxt.com/docs/4.x/getting-started/upgrade#new-dom-location-for-spa-loading-screen)                                                                                 | Minimal      | CSS または`document.queryElement`で SPA loading template をターゲット                                        | –                                     | `export default defineNuxtConfig({ experimental: { spaLoadingTemplateLocation: 'within' } })`                              |
| [Parsed error.data](https://nuxt.com/docs/4.x/getting-started/upgrade#parsed-errordata)                                                                                                                              | Minimal      | `error.data` を手動で `JSON.parse`                                                                           | –                                     | なし                                                                                                                       |
| [More Granular Inline Styles](https://nuxt.com/docs/4.x/getting-started/upgrade#more-granular-inline-styles)                                                                                                         | Moderate     | グローバル CSS を使用（`nuxt.config.ts`の`css`プロパティなど）                                               | –                                     | `export default defineNuxtConfig({ features: { inlineStyles: true } })`                                                    |
| [Scan Page Meta After Resolution](https://nuxt.com/docs/4.x/getting-started/upgrade#scan-page-meta-after-resolution)                                                                                                 | Minimal      | `pages:extend` フックを使用                                                                                  | –                                     | `export default defineNuxtConfig({ experimental: { scanPageMeta: true } })`                                                |
| [Shared Prerender Data](https://nuxt.com/docs/4.x/getting-started/upgrade#shared-prerender-data)                                                                                                                     | Medium       | 複数ページで同一データを fetch している                                                                      | –                                     | `export default defineNuxtConfig({ experimental: { sharedPrerenderData: false } })`                                        |
| [Default data and error values in useAsyncData and useFetch](https://nuxt.com/docs/4.x/getting-started/upgrade#default-data-and-error-values-in-useasyncdata-and-usefetch)                                           | Minimal      | `useAsyncData`/`useFetch`の`data.value`または`error.value`が`null`かチェック                                 | `nuxt/4/default-data-error-value`     | `export default defineNuxtConfig({ experimental: { defaults: { useAsyncData: { value: 'null', errorValue: 'null' } } } })` |
| [Removal of deprecated boolean values for dedupe option](https://nuxt.com/docs/4.x/getting-started/upgrade#removal-of-deprecated-boolean-values-for-dedupe-option-when-calling-refresh-in-useasyncdata-and-usefetch) | Minimal      | `refresh({ dedupe: true/false })`を使用                                                                      | `nuxt/4/deprecated-dedupe-value`      | なし                                                                                                                       |
| [Respect defaults when clearing data](https://nuxt.com/docs/4.x/getting-started/upgrade#respect-defaults-when-clearing-data-in-useasyncdata-and-usefetch)                                                            | Minimal      | `clear`/`clearNuxtData`を使用                                                                                | –                                     | なし                                                                                                                       |
| [Alignment of pending value in useAsyncData and useFetch](https://nuxt.com/docs/4.x/getting-started/upgrade#alignment-of-pending-value-in-useasyncdata-and-usefetch)                                                 | Medium       | `useAsyncData`/`useFetch`の`pending`を使用                                                                   | –                                     | `export default defineNuxtConfig({ experimental: { pendingWhenIdle: true } })`                                             |
| [Key Change Behavior in useAsyncData and useFetch](https://nuxt.com/docs/4.x/getting-started/upgrade#key-change-behavior-in-useasyncdata-and-usefetch)                                                               | Medium       | `useAsyncData`/`useFetch`で`immediate:false`を指定                                                           | –                                     | `export default defineNuxtConfig({ experimental: { alwaysRunFetchOnKeyChange: true } })`                                   |
| [Shallow Data Reactivity in useAsyncData and useFetch](https://nuxt.com/docs/4.x/getting-started/upgrade#shallow-data-reactivity-in-useasyncdata-and-usefetch)                                                       | Minimal      | `data.value`の深い reactivity に依存                                                                         | `nuxt/4/shallow-function-reactivity`  | `export default defineNuxtConfig({ experimental: { defaults: { useAsyncData: { deep: true } } } })`                        |
| [Absolute Watch Paths](https://nuxt.com/docs/4.x/getting-started/upgrade#absolute-watch-paths-in-builderwatch)                                                                                                       | Minimal      | `nuxt.hook('builder:watch')`でカスタム module が相対パスを期待                                               | `nuxt/4/absolute-watch-path`          | なし                                                                                                                       |
| [Removal of window.\_\_NUXT\_\_ object](https://nuxt.com/docs/4.x/getting-started/upgrade#removal-of-window__nuxt__-object)                                                                                          | Minimal      | クライアントコードで`window.__NUXT__`を参照                                                                  | –                                     | なし                                                                                                                       |
| [Directory index scanning](https://nuxt.com/docs/4.x/getting-started/upgrade#directory-index-scanning)                                                                                                               | Medium       | middleware ディレクトリに子ディレクトリが存在                                                                | –                                     | なし                                                                                                                       |
| [Template Compilation Changes](https://nuxt.com/docs/4.x/getting-started/upgrade#template-compilation-changes)                                                                                                       | Minimal      | ファイルシステム上の`.ejs`テンプレートを直接扱うコード生成                                                   | `nuxt/4/template-compilation-changes` | なし                                                                                                                       |
| [Default TypeScript Config Changes](https://nuxt.com/docs/4.x/getting-started/upgrade#default-typescript-configuration-changes)                                                                                      | Minimal      | TS の`noUncheckedIndexedAccess`に引っかかるコード                                                            | –                                     | `export default defineNuxtConfig({ typescript: { tsConfig: { compilerOptions: { noUncheckedIndexedAccess: false } } } })`  |
| [TypeScript Config Splitting](https://nuxt.com/docs/4.x/getting-started/upgrade#typescript-configuration-splitting)                                                                                                  | Minimal      | なし（明示的に設定変更しなければ有効化されない）                                                             | –                                     | なし                                                                                                                       |
| [Removal of Experimental Features](https://nuxt.com/docs/4.x/getting-started/upgrade#removal-of-experimental-features)                                                                                               | Minimal      | 記載のフラグを使用                                                                                           | –                                     | なし                                                                                                                       |
| [Removal of Top-Level generate Config](https://nuxt.com/docs/4.x/getting-started/upgrade#removal-of-top-level-generate-configuration)                                                                                | Minimal      | `nuxt.config.ts`の`generate`オプションを使用                                                                 | –                                     | なし                                                                                                                       |

## 実体験に基づく感想

小〜中規模なプロダクトであれば、**以前の Nuxt2→3 のアップグレードとは打って変わって、非常にスムーズにアップグレードできるケースが多い**と感じています。

実際に Vue ファイル数が 60 程度の小規模プロダクトで作業した際は、0.5 人日程度で完了することができました。

Nuxt4 のアップグレードガイドは非常に丁寧に作られており、段階的に移行できる仕組みが整っています。このガイドを参考に、ぜひスムーズなアップグレードを実現してください。

## まとめ

- **公式ガイドは充実しているが、全体像把握に時間がかかる**
- **「まず動かす」→「段階的移行」の 2 段階アプローチが効果的**
- **New Directory Structure と Data Fetch 関連が影響を受けやすい**
- **小〜中規模プロダクトなら比較的短時間でアップグレード可能**

時間がない方は、まずは「New Directory Structure」と「Data Fetch 関連」の項目だけでも事前に確認しておくと、作業時間の短縮につながると思います。
