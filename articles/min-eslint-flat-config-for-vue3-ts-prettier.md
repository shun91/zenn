---
title: "Vue3 + TypeScript + Prittier に対応した ESLint Flat Config の最小構成"
emoji: "🛠"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["eslint", "vue", "vue3", "typescript", "prettier"]
published: true
---

## TL; DR

```js:eslint.config.js
import globals from "globals";
import pluginJs from "@eslint/js";
import tseslint from "typescript-eslint";
import pluginVue from "eslint-plugin-vue";
import vueParser from "vue-eslint-parser";
import eslintConfigPrettier from "eslint-config-prettier";

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

### Pritter 対応

これは従来通り eslint-config-prettier を追加するだけです。

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

FlatConfig は JavaScript ファイルですが、以下のように JSDoc を使用して型チェックすることが可能です。

```diff js:eslint.config.js
+/** @type { import("eslint").Linter.FlatConfig[] } */
 export default [
   // 省略
 ]
```

ただし、プラグインによっては型エラーになるようなので、`@ts-expect-error` を記述しておく必要がありそうでした。
型チェックは効かなくなりますが、補完の恩恵は受けられるので、それだけでも十分価値がありそうです。

```diff js:eslint.config.js
 /** @type { import("eslint").Linter.FlatConfig[] } */
+// @ts-expect-error
 export default [
   // 省略
 ]
```

### Nuxt に対応したい

以下の記事が参考になると思います。

https://zenn.dev/comm_vue_nuxt/articles/setup-nuxt-eslint
