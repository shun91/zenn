---
title: "Nuxt3以降におけるコンポーネントのディレクトリ設計（v0.1）"
emoji: "📁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nuxtjs", "nuxt3", "vue", "frontend", "設計"]
published: true
---

:::message
タイトルに v0.1 とあるように、この記事は未完成な部分が多く含まれます。現時点での筆者の頭の中を Dump したメモ程度のものだと思って読んでもらえると嬉しいです。
:::

Nuxt 3 でのアプリケーション開発を効率的に進めるためには、明確なディレクトリ構成と設計方針が重要です。本記事では、Feature 型を意識しつつ、Nuxt のルールを尊重したディレクトリ構成の案を紹介します。また、コンポーネント設計における`container`と`presentational`の分離についても解説します。

## ディレクトリ構成

以下は、提案するディレクトリ構成の例です。[Nuxt4 で採用予定の新しいディレクトリ構造](https://github.com/nuxt/nuxt/issues/26444)に対応しています。

```
.
├── app
│   ├── app.vue
│   ├── components
│   │   ├── app
│   │   │   ├── AppBreadcrumb.vue
│   │   │   ├── AppHeader.vue
│   │   │   └── AppSidemenu.vue
│   │   ├── common
│   │   │   └── CommonButton.vue
│   │   ├── dashboard
│   │   │   └── DashboardButton.vue
│   │   └── todo
│   │       ├── TodoCreateContainer.vue
│   │       ├── TodoDetailContainer.vue
│   │       ├── TodoForm.vue
│   │       └── TodoListContainer.vue
│   ├── composables
│   │   └── models
│   │       └── useTodo.ts
│   ├── layouts
│   │   └── default.vue
│   ├── pages
│   │   ├── dashboard
│   │   │   └── index.vue
│   │   ├── todos
│   │   │   ├── [id].vue
│   │   │   ├── create.vue
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

ディレクトリ構成のベースには、Feature 型を意識しつつ、Nuxt のルールを尊重しています。Feature 型とは、特定の機能やページごとに関連するファイルをまとめる設計アプローチです。このアプローチにより、関連するコードを一箇所に集約し、見通しを良くすることでメンテナンスがしやすくなります。

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

### 3. コンポーネントの配置

コンポーネントを整理するために、以下の 4 つに分類します。

- **pages**

  - ページ本体となるコンポーネントをまとめます。
  - Nuxt の機能により、ディレクトリ構造がそのままパス構造となります。
  - 原則として`container`コンポーネントを呼び出す以外のことはしません。
  - ディレクトリ名は原則として複数形を用います（REST の慣習に従う）。

- **components/{PageName}**

  - 特定のページでしか使わないコンポーネントをまとめます。
  - 自動インポート時の名前を短くするため、`components`の直下にディレクトリを切ります。
    - 参考: [新規プロダクトの開発に Nuxt3 を採用して良かったこと - ANDPAD Tech Blog](https://tech.andpad.co.jp/entry/2024/01/17/100000)
  - ディレクトリ名は原則として単数形を用います。
  - CRUD でページが分かれている場合は、1 つのディレクトリにまとめます。

- **components/app**

  - アプリケーション全体で 1 つだけ存在するコンポーネントを配置します（例: ヘッダー、サイドメニュー）。
  - 主に`layouts`で使用され、`App`をプレフィックスとして命名します。

- **components/common**
  - 複数のページで使用される共通コンポーネントをまとめます。
  - 基本的には`presentational`コンポーネントのみですが、必要に応じて`container`を作成しても構いません。
  - `Common`をプレフィックスとして命名します。

#### 補足: なぜ`pages`ディレクトリに関連するコンポーネントを集めないのか？

Nuxt の推奨する`components`ディレクトリを採用し、パス構造が複雑になるのを防ぐためです。これにより、コンポーネントの視認性が高まり、メンテナンス性が向上します。

また、筆者はフロントエンドにあまり精通していないメンバーがいる環境下で開発することが多いため、Nuxt の標準から外れた設計をできるだけ避けるべきだと考えています。公式ドキュメントに記載されていないディレクトリ構成は、学習コストを高める要因となるため、明らかなメリットがある場合を除いて、Zero config を意識しています。

## 今後検討すべき事項

- 複雑な画面の実装に耐えうる設計か

  - 作成や編集のフォーム
  - モーダル
  - 複雑なクエリ問い合わせのある画面
  - 複数のページで使用される共通コンポーネントだが、外部 API に依存するもの（例: 検索サジェスト）

- composables の設計をどうするか

  - 基本的には`components`と同じ考え方で分けるのが良さそうです。
  - 外部 API を呼び出す処理についても専用のディレクトリを用意し、composables として実装するのが適していると考えています（いわゆるリポジトリ層に相当）。
    - 前述したディレクトリ構成における `models` がこれに該当します

- 状態管理をどうするか
  - `provide/inject`を使うパターンが良いと考えていますが、Pinia の導入も検討する価値がありそうです。
  - ↓ の記事が参考になりそうです。

https://zenn.dev/koudaiishigame/articles/810ce2d0ee8ade

最後に、冒頭でも述べたように、この設計はまだまだ未完成です。ぜひ、「ここはこうすると開発しやすくなります」「私は現状こういう設計をしています」といった気づきがあれば、コメントで教えていただけると嬉しいです！
