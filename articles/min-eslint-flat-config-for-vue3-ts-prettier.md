---
title: "Vue3 + TypeScript + Prettier に対応した ESLint Flat Config の最小構成"
emoji: "🛠"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["eslint", "vue", "vue3", "typescript", "prettier"]
published: true
---

タイトルの構成を Flat Config で実現しようと思ったら、若干ハマったので雑に書きました。

## TL; DR

別途必要なライブラリはインストールしてください。

```js:eslint.config.js
import globals from "globals";
import pluginJs from "@eslint/js";
import tseslint from "typescript-eslint";
import pluginVue from "eslint-plugin-vue";
import vueParser from "vue-eslint-parser";
import eslintConfigPrettier from "eslint-config-prettier";

export default [
  { languageOptions: { globals: { ...globals.browser, ...globals.node } } },
  pluginJs.configs.recommended,
  ...tseslint.configs.recommended,
  ...pluginVue.configs["flat/essential"],
  {
    files: ["*.vue", "**/*.vue"],
    languageOptions: {
      parser: vueParser,
      parserOptions: { parser: tseslint.parser, sourceType: "module" },
    },
  },
  eslintConfigPrettier,
];
```

## 解説

### 前提

yarn を使用しています。npm でも同様のことができるはずです。

```bash
$ yarn -v
1.22.22
```

### ベースとなる設定

以下のコマンドで最低限の設定ファイルを作成できるので、これを利用しています。

```bash
yarn add -D eslint
yarn eslint init
```

実行すると以下のようになります。  
`Which framework does your project use?` は `vue` を、`Does your project use TypeScript?` は `typescript` を選択します。他はプロジェクトに合わせて選択します。

```bash
$ yarn eslint --init
yarn run v1.22.22
$ eslint --init
You can also run this command directly using 'npm init @eslint/config@latest'.
✔ How would you like to use ESLint? · problems
✔ What type of modules does your project use? · esm
✔ Which framework does your project use? · vue
✔ Does your project use TypeScript? · typescript
✔ Where does your code run? · browser
The config that you've selected requires the following dependencies:

eslint@9.x, globals, @eslint/js, typescript-eslint, eslint-plugin-vue
✔ Would you like to install them now? · No / Yes
Successfully created /path/to/eslint.config.js file.
You will need to install the dependencies yourself.
✨  Done in 35.97s.
```

これだけで Vue + TypeScript に対応した eslint.config.js が作成されます。楽ちんですね。  
ちなみに中身はこんな感じです。

```js:eslint.config.js
import globals from "globals";
import pluginJs from "@eslint/js";
import tseslint from "typescript-eslint";
import pluginVue from "eslint-plugin-vue";

export default [
  { languageOptions: { globals: globals.browser } },
  pluginJs.configs.recommended,
  ...tseslint.configs.recommended,
  ...pluginVue.configs["flat/essential"],
];
```

### Vue ファイルの Parsing error 対応

このまま Vue ファイルに対して eslint を実行すると以下のようなエラーが出ます。

```
Parsing error: Unexpected token )
```

こちらの解決方法は以下に記載されています。

https://typescript-eslint.io/troubleshooting/#i-am-running-into-errors-when-parsing-typescript-in-my-vue-files

記載に従って、eslint.config.js を修正します。

```diff js:eslint.config.js
 import globals from "globals";
 import pluginJs from "@eslint/js";
 import tseslint from "typescript-eslint";
 import pluginVue from "eslint-plugin-vue";
+import vueParser from "vue-eslint-parser";

 export default [
   { languageOptions: { globals: globals.browser } },
   pluginJs.configs.recommended,
   ...tseslint.configs.recommended,
   ...pluginVue.configs["flat/essential"],
+  {
+    files: ["*.vue", "**/*.vue"],
+    languageOptions: {
+      parser: vueParser,
+      parserOptions: { parser: tseslint.parser, sourceType: "module" },
+    },
+  },
 ];
```

これで Parsing error が解消します。

### Prettier 対応

これは従来通り eslint-config-prettier を追加するだけです。

```bash
yarn add -D eslint-config-prettier
```

```diff js:eslint.config.js
 import pluginJs from "@eslint/js";
 import tseslint from "typescript-eslint";
 import pluginVue from "eslint-plugin-vue";
 import vueParser from "vue-eslint-parser";
+import eslintConfigPrettier from "eslint-config-prettier";

 export default [
   { languageOptions: { globals: globals.browser } },
   pluginJs.configs.recommended,
   ...tseslint.configs.recommended,
   ...pluginVue.configs["flat/essential"],
   {
     files: ["*.vue", "**/*.vue"],
     languageOptions: {
       parser: vueParser,
       parserOptions: { parser: tseslint.parser, sourceType: "module" },
     },
   },
+  eslintConfigPrettier,
 ];
```

## おまけ

### 特定のディレクトリなどを ignore したい

これまでは `eslint --ignore-path .gitignore` のようにしていたと思いますが、これができなくなりました。  
代わりに eslint.config.js 内で設定する必要があります。

```diff js:eslint.config.js
 export default [
+  { ignores: ["**/dist/**"] },
   { languageOptions: { globals: globals.browser } },
   pluginJs.configs.recommended,
   ...tseslint.configs.recommended,
   ...pluginVue.configs["flat/essential"],
   {
     files: ["*.vue", "**/*.vue"],
     languageOptions: {
       parser: vueParser,
       parserOptions: { parser: tseslint.parser, sourceType: "module" },
     },
   },
 ];
```

ちなみに、node_modules はデフォルトで ignore されていそうな挙動だったので含めていません。

### eslint-plugin-vue のルールセットを変更したい

以下のようにすれば OK です。

```diff js:eslint.config.js
-...pluginVue.configs["flat/essential"],
+...pluginVue.configs["flat/recommended"],
```

どのような指定ができるかは以下を参照してください。

https://eslint.vuejs.org/user-guide/#bundle-configurations-eslint-config-js

### eslint.config.js も型が効くようにしたい

FlatConfig は JavaScript ファイルですが、以下のように [typescript-eslint が提供する config というヘルパー関数](https://typescript-eslint.io/packages/typescript-eslint#config)を使用して型チェックすることが可能です。

```diff js:eslint.config.js
- export default [
+ export default tseslint.config(
   // 省略
- ]
+ )
```

ただし、プラグインによっては型エラーになるようなので、そのような場合は `@ts-expect-error` を記述しておくとよいです。
型チェックは効かなくなりますが補完の恩恵は受けられるので、それだけでも十分価値があります。

```diff js:eslint.config.js
 export default tseslint.config(
   // 省略

+  // @ts-expect-error エラーが出るため
   ...pluginVue.configs["flat/essential"],

   // 省略
 )
```

### Nuxt に対応したい

以下の記事が参考になると思います。
セットアップ方法が 3 つ紹介されていますが、個人的には「`@nuxt/eslint-config`のバージョンアップのみ行う」がよさそうに思っています（ちゃんと試せてはいません mm）

https://zenn.dev/comm_vue_nuxt/articles/setup-nuxt-eslint

## 追記：筆者の Linter に対するスタンス

**Linter そのものの運用保守を最低限で済ませたい**
Linter の設定や運用に時間をかけすぎるのは本質的ではないと考えています。開発の効率を上げるために導入している Linter が、逆に手間を増やす原因になることは避けたいところです。

**基本は公式が提供しているルールセットに従う**
Linter の設定は、公式のルールセットを可能な限り採用する方針です。独自のルールをカスタマイズしすぎると、その保守が複雑になり、コストが増大するからです。公式のルールセットは、多くの優秀な開発者たちによって設計されており、通常はそれで十分と考えています。プロジェクトの特性に合わせてルールを細かく調整するよりも、公式のルールを活用する方がコストに見合った成果が得られると感じています。

上記のスタンスなので、従来の eslintrc から Flat Config に移行する際も、これまで設定していたルールを厳密に再現することよりも、ルールが多少変わったとしても設定をシンプルに保つことを優先したい派です。
なので、これまで設定していた eslintrc は破棄して、新たに Flat Config を設定し直すのがいいと考えています。
ちなみに、これまで設定していたルールを厳密に再現したい場合は以下の記事がおすすめです。

https://zenn.dev/cybozu_frontend/articles/introduce-eslint-config-compat
