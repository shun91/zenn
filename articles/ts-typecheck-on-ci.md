---
title: "Claude Code使ってるのにCIで型チェック忘れてないですか？"
emoji: "🚦"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["typescript", "ci", "frontend", "react", "vue"]
published: true
---

## はじめに

Claude Code などの AI コーディングツールを使っていると、コードの生成や編集は驚くほど効率的になります。しかし、**AI が生成したコードでも型エラーが含まれることがある**のが現実です。

さらに、最近のフレームワークの公式スキャフォールド（例：`npm create vite@latest`）では、**開発時のフィードバック速度を優先するため、ビルド時の型チェックはデフォルトで無効**になっていることがあります。これに気づかず、lint や build は通るのに**型エラーが PR に混入**してしまうケースをよく見かけます。

なので、**AI コーディング時代だからこそ、CI で型チェックを実施するジョブを忘れずに回しましょう**というのが、この記事で言いたいことです。

ここで品質を担保しないと、とんでもない勢いで負債がたまっていきます。  
以下のポストでは Linter について言及されていますが、型チェックにも同じことが当てはまると考えています。

https://x.com/suthio_/status/1936672816503685179

時間がない方は、下のスニペットをコピペして `npm run typecheck` を CI に追加してもらえると効果が大きいはずです。

## なぜ『忘れがち』になるのか

- Claude Code や GitHub Copilot は優秀ですが、生成されたコードが常に型安全とは限りません。
  - 型チェックのようなガードレールがなければ、基本的に何かしらの型エラーを含んだコードを生成すると思ったほうがよいです。
- 多くの公式テンプレートは lint と unit test のスクリプトは付いていても、**typecheck は別タスク**になっています。
- また、ビルド速度優先のために **build コマンドでは意図的に型チェックを無効化**していることがあります。
  - Vite の場合、[公式ドキュメントにもこのことが記載](https://ja.vite.dev/guide/features.html#%E3%83%88%E3%83%A9%E3%83%B3%E3%82%B9%E3%83%8F%E3%82%9A%E3%82%A4%E3%83%AB%E3%81%AE%E3%81%BF)されています。
- 実務では、AI が生成したコードも含めて、レビューの手前で CI が型エラーを止めてくれる方が、結果として**手戻りが減る**と感じています。

## 一般的な TypeScript プロジェクトでの型チェック

純粋な TypeScript や React（Vite/webpack）の場合、**ビルドせずに型検査だけ**を行うには `tsc --noEmit` がシンプルです。Vue(SFC)は `vue-tsc` を使います。

### TS / React（.tsx）

```bash
npm i -D typescript
```

```json
// package.json
{
  "scripts": {
    "typecheck": "tsc --noEmit"
  }
}
```

`--noEmit` は出力を生成せずに型チェックのみ行うオプションです。

### Vue 3

```bash
npm i -D typescript vue-tsc
```

```json
// package.json
{
  "scripts": {
    "typecheck": "vue-tsc --noEmit -p tsconfig.json"
  }
}
```

`.vue` の中の `<script setup lang="ts">` なども検査対象にしたいので、`tsc` ではなく `vue-tsc` を使います。

### ローカルでの実行

上記の設定をしたら、ローカルで以下のコマンドを実行することで型チェックができます。

```bash
npm run typecheck
```

## CI に組み込む

ここでは GitHub Actions を例に、CI に組み込む方法を説明します。どんな CI ツールであっても、基本は「依存を入れる → `npm run typecheck`」というシンプルな流れで、既存の test ジョブに**後段 step として足す**か、**独立ジョブ**にして並列化するのが運用しやすいと考えています。

以下は独立ジョブを追加する例です。

```yaml
# .github/workflows/typecheck.yml
name: typecheck
on:
  pull_request:
jobs:
  typecheck:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm
      - run: npm ci
      - run: npm run typecheck
```

## 気をつけることは？

### ビルドや dev 時に毎回型チェックを強制しない

型チェックは比較的重い処理なので、開発時のフィードバック速度を落としがちです。CI で確実に実行する方がコスパが良いと考えています。

### VSCode など IDE 上で型チェックするだけでは不十分

IDE の型チェックは心強いのですが、環境の統一性がない点で CI の代替にはなりにくいと考えています。開発者ごとに拡張や TypeScript/プラグインのバージョンが異なると結果がぶれます。CI は同一環境で実行されるため判断が安定します。  
IDE は早いフィードバック、CI は品質ゲート。役割は補完的だと思います。

## まとめ

- AI コーディングツールは便利ですが、**生成されたコードが必ずしも型安全とは限りません**。
- 近年のテンプレートは**型チェックをデフォルトで実行しない**ことがあります。
- **`npm run typecheck` を用意して CI に組み込む**だけで、AI が生成したコードも含めて実行時不具合の芽をかなり早い段階で摘めると思います。
- 一般的な TS/React なら `tsc --noEmit`、Vue(SFC) なら `vue-tsc --noEmit` を基本に据えると良さそうです。

## 参考資料

- Vite 公式（TypeScript ページ。型チェックはビルドと別で行う前提の記述）
  - https://ja.vite.dev/guide/features.html#%E3%83%88%E3%83%A9%E3%83%B3%E3%82%B9%E3%83%8F%E3%82%9A%E3%82%A4%E3%83%AB%E3%81%AE%E3%81%BF
- vue-tsc（Vue SFC 用の型チェック）
  - https://github.com/vuejs/language-tools/tree/master/packages/tsc
- Nuxt 公式: TypeScript と Type-checking の解説（Type-checking セクション）
  - https://nuxt.com/docs/guide/concepts/typescript#type-checking
