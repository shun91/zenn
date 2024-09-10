---
title: "Promise.allで非同期処理を高速化して表示速度改善"
emoji: "⏳"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["javascript", "promise", "performance", "typescript", "非同期処理"]
published: false
---

## 概要

JavaScript で非同期処理を行う際、API リクエストやデータベース操作などの処理を順番に実行すると、処理が終わるまで無駄な待ち時間が発生します。しかし、複数の非同期処理が互いに依存しない場合は、`Promise.all`を使って並列に実行することができます。これにより、全体の処理速度が向上し、ユーザー体験を大きく改善できます。

## どうやるの？

例えば、以下のように `await` を使って順番に API リクエストを実行している場合、各リクエストが完了するまで次のリクエストが待機してしまいます。

```ts
await fetchApi1(); // 1秒
await fetchApi2(); // 1秒
await fetchApi3(); // 1秒
```

この場合、合計で 3 秒かかります。しかし、これらの API リクエストが互いに依存していない、つまり 1 つ目の API リクエストの結果を 2 つ目で使う必要がない場合、`Promise.all`を使えばすべてを並列に実行することが可能です。

```ts
await Promise.all([fetchApi1(), fetchApi2(), fetchApi3()]);
```

このコードでは、`fetchApi1`、`fetchApi2`、`fetchApi3`がすべて並列で実行されるため、合計で 1 秒しかかかりません。結果として、全体の処理が非常に効率的になります。

`Promise.all`についてさらに詳しく知りたい方は、公式のドキュメントを確認してみてください 👇

https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Promise/all

## どれくらい効果があるの？

以下のシナリオを考えてみます。複数の API リクエストを行い、その応答を使ってページにデータを表示するような場合、各リクエストの処理に 1 秒かかるとします。

- **直列実行の場合**  
  各リクエストが順番に実行されるため、合計で **3 秒** かかります。

- **並列実行の場合**  
  すべてのリクエストが並列に実行されるため、合計で **1 秒** で完了します。

実際に動作するデモも用意してみました 👇

https://www.typescriptlang.org/ja/play/?#code/MYewdgzgLgBAZgUysAFgQQA4EsCMMC8MAFAJQEB8AUDPEqkQEQpRQYQBcA9JwCYCuAWwEBPAFYRwAOlADOfCAgBOEAPwAbLAKxR8ABgYlJUFAjBEiisvnIxFk8eFIkA3JUqhIsRMnTYATATEVlQ03vRMLGxcvIIiDmDSILIYINCqGlo6+obGpuaWFLb2EmYkLm4e0LQ+mFgAzIGkFNTV4cysHNz8QmIlibIAjnwgUAjpmtp6BkYmZhbBRfFOrpQAhhDCYMDwfFtQWOAwCopYq2oAYnQoTQDeLZWwimN4hKsA7qvarb64pK40D1sYwCrw+XzCPz8f3u4CqTwgDVBny8V1qdWhNCeUD4ijAMAA2vCcAAaIEQPyk+F1AC6rgAvm51pttnBdsB9ocMKtFGc1AgLldbjDPASiZTgeKEdTAu9kTAAAqKJJYBSSXlEfEtUKo7A4UjErXfWpQkgGmjamrYdGmlrU8qYpA4vGE56Silkmn0xkbLY7PYHPECT6lGB3AGw2DQblQAAqmgQAEkwABlJRYMaBDBKOAgRRBrYISRgEBvDEwWVfY6nAU+MuAhAADwQwDjAkTKbTGcIWcUObzqwLRZLTQAtEcoNHW+3UycxitwyKo4pY-Gk-Lubz+Zns7n88BC8XS-by2DYFyeWo+TX6Mf602W6uwOuL1ft73dwP90OjzAx0uV22a4bpe-LzjAlQgHykhqCAADmRA3DAjbNlOSYzumECkshD6AU+wGvnS5QMpQQZYKUrhAA

## 気をつけることは？

`Promise.all`を使う際に注意すべき点はいくつかあります。

- **依存関係がない場合のみ有効**  
  各非同期処理が独立して動作する場合にのみ、並列実行が有効です。例えば、1 つ目の API リクエストの結果を 2 つ目のリクエストで使用するような場合は、直列での実行が必要です。この場合、`Promise.all`ではなく、通常の `await` を使って順番に処理する必要があります。

- **瞬間的なリソース消費の増加**  
  並列実行により、ネットワーク帯域やメモリ、CPU の使用量が一時的に増加する可能性があります。特にリクエスト数が多い場合、サーバーやブラウザがその負荷に耐えられるかを考慮する必要があります。

- **一部の処理が失敗した場合**  
  `Promise.all`は、すべての処理が成功しない限り、全体が失敗と見なされます。つまり、1 つでも処理が `reject` されると、全体がエラーとなります。もし、いくつかの非同期処理が失敗しても残りの処理を続行したい場合は、`Promise.allSettled` を使うのが適しています。これは、すべての Promise が解決または拒否された後に結果を返すため、部分的な成功を処理できます。

Promise.allSettled について詳しくは以下のリンクを確認してください 👇

https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Promise/allSettled

## 参考

https://qiita.com/nuko-suke/items/50ba4e35289e98d95753

https://zenn.dev/tesla/articles/0a3375f4fe9e89

https://zenn.dev/sengosha/articles/498398d82fb056
