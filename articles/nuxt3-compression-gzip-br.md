---
title: "Nuxt3でコンテンツやアセットを圧縮配信する（gzip/brotli）"
emoji: "🗜️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nuxt3", "nuxtjs", "performance", "圧縮", "gzip"]
published: true
---

Nuxt 3 アプリケーションで、gzip や brotli を使ってコンテンツやアセットを圧縮配信する方法を調べたので、まとめてみました。ここでは、3 つのケースに分けて、具体的な設定方法を説明します。

## 前提

通常、コンテンツやアセットの圧縮は CDN やリバースプロキシ（例：Cloudflare や NGINX など）で行うことが一般的です。しかし特定のケースでは、アプリケーション側での圧縮が必要になることがあります。ここでは、そういった場合に活用できる方法を紹介します。

## 動作デモ

実際に圧縮配信が動作するデモアプリも用意したので、動かしながらこの記事を読むことで理解が深まると思います。

https://shun91-nuxt-examples.vercel.app/compression
https://github.com/shun91/nuxt-examples

## 圧縮の方法：3 種類

何を圧縮するかによって方法が異なります。以下では、SSR レスポンスの圧縮、静的アセットの圧縮、API プロキシでの圧縮レスポンス処理に分けて説明します。

### SSR レスポンスの圧縮

SSR で生成された HTML などのレスポンスを gzip で圧縮する場合、Nitro プラグインを使用します。以下のプラグイン `compression.ts` は、`accept-encoding` ヘッダーに `gzip` が含まれていたらレスポンスを gzip で圧縮します。

```ts:server/plugins/compression.ts
export default defineNitroPlugin((nitro) => {
  nitro.hooks.hook('render:response', async (response, { event }) => {
    // accept-encodingにgzipが含まれている場合はtrue
    const acceptsEncoding = getRequestHeader(
      event,
      'accept-encoding',
    )?.includes('gzip');
    if (acceptsEncoding) {
      setResponseHeader(event, 'Content-Encoding', 'gzip');
      response.body = new Response(response.body).body?.pipeThrough(
        new CompressionStream('gzip'),
      );
    }
  });
});
```

このプラグインを追加した後、アプリケーションを起動してブラウザで確認すると、gzip 圧縮が適用されたレスポンスが返されることがわかります。

![ブラウザのキャプチャ](/images/nuxt3-compression-gzip-br/ssr-compression-result.png)

ちなみに、上記ソースコード内で使用している[`render:response`](https://nitro.unjs.io/guide/plugins#renderer-response)は、SSR でのページレンダリング時に呼び出される Nitro フックです。

### アセットの圧縮

次に、JavaScript や CSS などの静的アセットを圧縮する方法です。これを行うには、`nuxt.config.ts`に以下の設定を追加します。これにより、`nuxt build` コマンドによるビルド時にアセットが自動で gzip や brotli 形式で圧縮されます。

```ts:nuxt.config.ts
export default defineNuxtConfig({
  nitro: {
    compressPublicAssets: true,
  },
});
```

この設定を有効にすると、ビルド後に生成される `.output/public` フォルダ内のファイルが圧縮されます。以下は圧縮の有無を比較した結果です。

| compressPublicAssets: true                                                                 | compressPublicAssets: false                                                                 |
| ------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------- |
| ![compressPublicAssets: true](/images/nuxt3-compression-gzip-br/compressPublicAssets2.png) | ![compressPublicAssets: false](/images/nuxt3-compression-gzip-br/compressPublicAssets1.png) |

アプリケーションを起動してブラウザで確認すると、圧縮されたアセットが配信されていることがわかります。

![ブラウザのキャプチャ](/images/nuxt3-compression-gzip-br/compress-public-assets-result.png)

なお、開発モード（`nuxt dev`）ではこの圧縮設定は適用されないので、開発中に圧縮を確認するためにはビルド後に生成されたファイルをチェックしてください。

### API プロキシでの圧縮レスポンス処理

最後に、API プロキシを経由して圧縮レスポンスを保持する場合の処理です。API プロキシを使って外部 API にリクエストを行い、そのレスポンスがすでに圧縮されている場合、プロキシでレスポンスの圧縮が保持されるようにする設定が必要です。これを実現するために、`http-proxy-middleware` を使います。

以下は、プロキシの設定例です。Nitro のミドルウェアとして設定します。

```ts:server/middleware/proxy.ts
import { createProxyMiddleware } from 'http-proxy-middleware';

export default defineEventHandler(async (event) => {
  const proxyMiddleware = createProxyMiddleware({
    target: process.env.API_SERVER_URL, // プロキシ先のURL
    changeOrigin: true,
    pathFilter: ['/api'], // プロキシ対象のパス
    pathRewrite: { '^/api': '' }, // 任意：パスの書き換え
    on: {
      proxyReq: (proxyReq) => {
        // 任意：Request Header が溢れる場合は不要なcookieを削除する
        proxyReq.removeHeader('cookie');
        // 任意：認証情報を付与。ここではIdTokenだが必要なものを付与するとよい
        const idToken = getCookie(event, 'id_token');
        if (idToken) {
          proxyReq.setHeader('Authorization', `Bearer ${idToken}`);
        }
      },
    },
  });

  await new Promise((resolve, reject) => {
    proxyMiddleware(event.node.req, event.node.res, (err) => {
      if (err) reject(err);
      else resolve(true);
    });
  });
});
```

上記のミドルウェア設定を適用すると、API リクエストに対して圧縮が有効なレスポンスを保持できます。例えば、`process.env.API_SERVER_URL` に `https://dummyjson.com` を設定して http://localhost:3000/api/todos を開くと、次のように圧縮が適用されたレスポンスが返されることがわかります。

![ブラウザのキャプチャ](/images/nuxt3-compression-gzip-br/proxy-compression-result.png)

API プロキシの設定に関する詳細は、[http-proxy-middleware の公式リポジトリ](https://github.com/chimurai/http-proxy-middleware)を確認してください。

:::message
API プロキシの設定には、以下の記事のように `server/api/[...].ts` を実装する方法もありますが、この方法だとプロキシ時にレスポンスが解凍されることがあります。

[Nuxt3 で proxy して外部 API にアクセスするには - Qiita](https://qiita.com/hhirasawa/items/e4ec9b3c35464f081fe7)

原因は正確には特定できていませんが、h3（Nuxt の HTTP サーバー部分に用いられているライブラリ）がレスポンスの content-encoding ヘッダーを自動的に削除する実装となっているため、h3 内部でレスポンスを解凍してしまっている可能性があります。

https://github.com/unjs/h3/blob/2d941d3cfb1dddf543d48abe23d13488c88c7432/src/utils/proxy.ts#L86-L89
:::

## 圧縮の効果：70~90%のサイズ削減

以下の表のように、圧縮処理をすることで、テキストベースのデータ（HTML、CSS、JavaScript など）は 70%から 90%のサイズ削減が期待できます。

![alt text](/images/nuxt3-compression-gzip-br/effects-of-compression.png)
[テキストベースのアセットのエンコードと転送サイズを最適化する  |  Articles  |  web.dev](https://web.dev/articles/optimizing-content-efficiency-optimize-encoding-and-transfer?hl=ja)より引用

これにより、ネットワークを介して転送されるデータ量が削減され、結果としてページの表示速度の改善が期待できます。

## 圧縮の注意点：サーバーリソースの消費

アプリケーションのサーバー側での圧縮処理は CPU リソースを消費します。通常は無視できるほどの負荷で済むはずですが、トラフィックが多い場合やサーバーのリソースが限られている場合には注意が必要です。必要に応じて負荷検証を行っておくと安心です。

## まとめ

Nuxt 3 では、gzip や brotli によるコンテンツやアセットの圧縮を比較的簡単に導入できます。CDN やリバースプロキシでの圧縮ができない場合には、ぜひ試してみてください。

## 参考資料

- [CodeDredd/h3-compression： Adds compression to h3 requests (brotli, gzip, deflate)](https://github.com/CodeDredd/h3-compression)
- [Asset compression support with `nuxi generate` · Issue #14794 · nuxt/nuxt](https://github.com/nuxt/nuxt/issues/14794)
- [Nuxt3 で API proxy をする方法｜ SHOWROOM Blog](https://note.com/showroom_blog/n/n943b666c1574)
