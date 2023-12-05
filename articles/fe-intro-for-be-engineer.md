---
title: "バックエンドエンジニアのためのモダンフロントエンド入門"
emoji: "😸"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["frontend", "javascript", "typescript"]
published: true
---

バックエンドエンジニアの方がフロントエンド開発に触れる際に必要となる最新の技術要素にフォーカスし、それぞれの要素がなぜ必要なのかを解説します。

:::message
5 年ぐらい前に書いたものを最新状況に合わせて加筆修正した記事となりますが、古い内容などが含まれていたらごめんなさい mm
:::

# ECMAScript（エクマスクリプト）

- ECMAScript とは JavaScript の新しい標準（仕様）のことで、最新バージョンは ES2023。
- 学ぶのにオススメのサイト → [JavaScript Primer](https://jsprimer.net/)
- 仕様を調べるのにオススメのサイト → [MDN](https://developer.mozilla.org/ja/docs/Web)
- この標準に従い、各ブラウザは実装される。ただし対応状況はブラウザによる... → トランスパイラが必要

# トランスパイラ・モジュールバンドラー

- トランスパイラ
  - モダンな記法で記述された JS を各ブラウザで動作する記述に変換してくれるもの。
- モジュールバンドラー
  - 複数のファイル（モジュール）をまとめるためのもの。依存関係の解決も担う。
  - なぜ必要なのか？
    - 依存関係解決
      - HTML と JS はファイルを分けて書くべき。
      - このとき モジュールバンドラー がないと、HTML から JS を呼び出す（ `<script>` を書く）時に順番を意識しなければならなくなる
    - パフォーマンス向上
      - HTTP/1.1 のオーバーヘッドは大きいため、ファイルをまとめてリクエスト数を減らすことで高速化（最近は HTTP/2 も出てきたが...）
    - その他必要な処理をまとめてやってくれる
      - トランスパイル、TypeScript の コンパイル、JS の圧縮、...
      - 1 つ 1 つコマンドで実行するのは大変
    - 後述のフレームワークの利用にはそもそもモジュールバンドラーが必須
      - 難しいことをしないのであれば、あらかじめ設定してくれたテンプレが提供されている。

## Vite

- Vite は、近代的なフロントエンド開発環境におけるビルドツールとして、従来の [Babel](https://babeljs.io/) や [Webpack](https://webpack.js.org/) に代わる存在。
- 特に開発環境の起動・更新が爆速なことが特徴。（キーワード：ES モジュール、Hot Module Replacement（HMR））
- Nuxt3 においては Vite が標準のビルドツールとして採用されており、効率的なモジュールバンドリングとトランスパイルを実現。
  - Nuxt3 においては[`nuxt.config.ts`](https://nuxt.com/docs/getting-started/configuration#with-vite)で Vite の設定を変更できる

![](https://ja.vitejs.dev/logo-with-shadow.png)  
↑[公式サイト](https://ja.vitejs.dev/)より拝借

# TypeScript

- [TypeScript (TS) ](https://www.typescriptlang.org/)とは、JavaScript (JS) に「静的型付け」と「クラスベースオブジェクト指向」を加えたもの。
- TS は JS のスーパーセットであり、既存の JS コードは全て TS として実行可能。
- ここで気軽に試せる → [TypeScript - Playground](https://www.typescriptlang.org/play/index.html)
- 通常はコマンドラインから利用する。

```sh
# インストール
npm install -g typescript    # or `yarn global add typescript`

# infile.ts を infile.js に変換
tsc infile.ts
```

- なぜ TS が必要なのか
  - JS には型がない
    - エディタでの補完が効きにくい
    - 関数の入出力の期待値がわからない（特に人の書いたコード）
  - クラスの機能が不完全
    - インターフェースがなく、仕様と実装を分離できない
    - private, protected がなく、カプセル化できない（private は ES 標準仕様に一応あるにはある）

→ 大規模な開発になるほど JS だとツラい

# フレームワーク（Angular, React, Vue）

- モダンな FE 環境では主に以下の 3 つのいずれかのフレームワークが採用されているケースが多い
  - [Angular](https://angular.jp/)
  - [React](https://ja.react.dev/)
  - [Vue](https://ja.vuejs.org/)
- [jQuery](https://jquery.com/) だとだめなのか？
  - 複雑な状態に合わせて適切な画面を描画する必要が出てきた
    - jQuery だと DOM が状態を持つことになり、メンテし辛いコードになる
    - フレームワークを使えば DOM と状態/ロジックを分離できる（キーワード：宣言的 UI）
  - DOM を直接書き換えるのは高コスト
    - フレームワークでは仮想 DOM が採用されている（最近脱仮想 DOM の動きもある...）
  - jQuery を使わずとも標準 API が充実してきた
    - Promise, addEventListener, querySelectorAll, fetch, ...
