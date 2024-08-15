---
title: "Nuxt3以降におけるコンポーネントのディレクトリ設計（v0.1）"
emoji: "📁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nuxtjs", "nuxt3", "vue", "frontend", "設計"]
published: false
---

:::message
タイトルに v0.1 とあるように、この記事は未完成な部分が多く含まれます。現時点での筆者の頭の中を Dump したメモ程度のものだと思って読んでもらえると嬉しいです。
:::

Nuxt 3 でのアプリケーション開発を効率的に進めるためには、明確なディレクトリ構成と設計方針が重要です。本記事では、Feature 型を意識しつつ、Nuxt のルールを尊重したディレクトリ構成の案を紹介します。また、コンポーネント設計における`container`と`presentational`の分離についても解説します。

## ディレクトリ構成

以下は、提案するディレクトリ構成の例です。
[Nuxt4 で採用予定の新しいディレクトリ構造](https://github.com/nuxt/nuxt/issues/26444)に対応しています。

```
.
├── app
│   ├── app.vue
│   ├── components
│   │   ├── app
│   │   │   ├── AppBreadcrumb.vue
│   │   │   ├── AppHeader.vue
│   │   │   └── AppSidemenu.vue
│   │   └── dashboard
│   │       └── DashboardButton.vue
│   ├── composables
│   ├── layouts
│   │   └── default.vue
│   ├── pages
│   │   ├── dashboard
│   │   │   └── index.vue
│   │   ├── user
│   │   │   └── index.vue
│   │   └── index.vue
│   └── plugins
├── public
├── server
├── nuxt.config.ts
├── package.json
├── tsconfig.json
└── yarn.lock
```

## 設計方針

### 1. Feature 型を意識しつつ、Nuxt のルールを尊重する

ディレクトリ構成のベースには、Feature 型を意識しつつ、Nuxt のルールを尊重しています。Feature 型とは、特定の機能やページごとに関連するファイルをまとめる設計アプローチです。このアプローチにより、関連するコードを一箇所に集まり、見通しがよくなるのでメンテナンスがしやすくなります。

参考
https://zenn.dev/mybest_dev/articles/c0570e67978673

### 2. コンポーネントの設計 - `container` と `presentational`

コンポーネントは大きく分けて`container`と`presentational`の 2 つに分けられます。

- **presentational コンポーネント**

  - 純粋関数コンポーネントとして設計され、表示にのみ責務を持ちます。
  - 外部 API やアプリケーション全体の状態には依存しません。

- **container コンポーネント**
  - `presentational`に依存しますが、逆は許されません。
  - 外部 API やアプリケーション全体の状態に依存しても問題ありません。

### 3. コンポーネントの配置 - `pages`, `app`, `common`で分ける

コンポーネントを整理するために、`pages`, `app`, `common`の 3 つに分類します。

- **pages**

  - 特定のページでしか使わないコンポーネントをまとめます。
  - 自動インポート時の名前を短くするため、`components`の直下にディレクトリを切ります。
    - 参考：[新規プロダクトの開発に Nuxt 3 を採用して良かったこと - ANDPAD Tech Blog](https://tech.andpad.co.jp/entry/2024/01/17/100000)
  - CRUD でページが分かれている場合は、1 つのディレクトリにまとめます。

- **app**

  - アプリケーション全体で 1 つだけ存在するコンポーネントを配置します（例: ヘッダー、サイドメニュー）。
  - 主に`layouts`で使用され、`App`をプレフィックスとして命名します。

- **common**
  - 複数のページで使用される共通コンポーネントをまとめます。
  - 基本的には`presentational`コンポーネントのみですが、必要に応じて`container`を作成しても構いません。
  - `Common`をプレフィックスとして命名します。

#### 補足: なぜ`pages`ディレクトリに関連するコンポーネントを集めないのか？

Nuxt の推奨する`components`ディレクトリを採用し、パス構造が複雑になるのを防ぐためです。コンポーネントの視認性を高め、メンテナンス性を向上させることが目的です。

## この記事で言及できていないが今後検討したいこと

- composables の設計
  - 基本的には`components`と同じ考え方で分けるのがいい気がしています。
- 状態管理
  - `provide/inject`を使うパターンがよいのではないかと考えていますが、Pinia についても検討する価値がありそうです。
  - ↓ の記事が参考になりそうです。

https://zenn.dev/koudaiishigame/articles/810ce2d0ee8ade
