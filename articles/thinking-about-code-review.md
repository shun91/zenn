---
title: "「コードレビューたまりがち問題」を解決するには"
emoji: "📝"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["コードレビュー", "レビュー"]
published: true
---

チーム開発あるあるの 1 つに「コードレビューたまりがち問題」があると思います。  
この問題を解決する方法についての自分の見解を書いてみます。

ここで言うコードレビューとは、「Pull Request（以下 PR と書きます）を使ったコードレビュー」のことです。

時間がない方は目次とまとめだけ読んでもらえたらと思います。

# どうしてコードレビューはたまるのか？

これには大きく 2 つ理由があると考えています。

- コードレビューを後回しにしてしまう
- レビュアーが特定のメンバーに偏ってしまう

以下では、それぞれについて具体的に書いていきます。

# コードレビューを後回しにしてしまう

コードレビューの依頼が来ているにも関わらず、自分の開発タスクを優先して進めたら、当然コードレビューがたまってしまいます。

では、どうしてそのようなことが起きてしまうのでしょうか？

## コンテキストスイッチが大変だから

個人的にはこれが一番大きいと考えています。自分の開発タスクを中断してコードレビューを始めるには、

- 自分の開発タスクをどこまでやったかを記録して、
- レビュー対象の PR を開いて、
- どんなコンテキストなのかを思い出したり、把握したりして、
- コードを読み始める

というような頭の切り替えが必要です。コードレビューを終えて、自分の開発タスクに戻るときはこの逆のことをしないといけません。

これ、ハッキリ言ってかなり大変です。  
特にレビュー対象の PR が以下のような状況だと切り替えにかかる時間は非常に長くなります。

- PR がデカい
  - 変更行数が数百行あったり、複数のコンテキストが 1 つの PR に混ざっているとか
- PR の説明を読んでも、レビューしてほしい観点・目的・意図などがよく分からない
  - コードを隅々まで読まないと意図が見えてこないとか
- PR の内容と、自分が取り組んでいる開発タスクとの関連が浅い
  - とある機能開発をしている最中に規模の大きめのリファクタリングのレビュー依頼をされたとか

結果、自分の開発を優先して、コードレビューが後回しになってしまいます。

では、どうすればコンテキストスイッチの辛さを軽減し、レビューを後回しにせずに済むようになるでしょうか。

### 対策：コンテキストの異なるタスクの並列度を下げる

コンテキストの異なる開発タスクが並行して走っていると、自分が取り組んでいる開発タスクと関連が浅いタスクについてのコードレビューをする機会は増えます。  
であれば、できるだけコンテキストの同じタスク（≒ 同じ機能の開発に関するタスク）にリソースを集中してしまうのです。

コンテキストの同じタスクにリソースを集中させた状態のことを、「フロー効率が高い」と言ったりします。  
フロー効率については以下の記事が参考になります。フロー効率を高めるプラクティスを知りたい場合は「フロー効率性を向上させるプラクティスや事例」のところを読んでみてください。

https://i2key.hateblo.jp/entry/2017/05/15/082655

あともう一つ、コンテキストの同じタスクにリソースを集中させるには、複数人で開発ができる状態になっている必要があります。  
規模の大きい機能の開発は、独立した細かいいくつかのタスクに分割することで、複数人での開発がしやすくなります。  
分割の仕方は例えば以下の記事が参考になると思います。

https://www.ryuzee.com/contents/blog/14554

### 対策：レビュアーは、コンテキストスイッチが大変ならさっさとそれを伝える

後回しにする前にレビュイーに伝えましょう。  
その際、理由も忘れずに伝えます。例えば以下のようなものです。

- 変更行数がデカすぎるので分割してほしいです
- 色んな修正が混ざっちゃってるので分けてほしいです
- 説明を読んでもどんな観点でレビューしてほしいのか自分には分からなかったので、教えてほしいです
- このタスクにはあまり関わってなくて背景知識に乏しいので、もう少し説明をお願いしたいです

正直、伝えにくいとは思いますが、自分が大変に感じるということは多分他の人がレビュアーになっても同じように感じるはずです。そして、レビュイー自身もレビュワーの依頼に応えることで、自分が書いたコードに対する理解がより深まることもあります。

お互いのためだと思って、敬意を持って伝えましょう。  
伝え方については以下も参考になると思います。

https://fujiharuka.github.io/google-eng-practices-ja/ja/review/reviewer/comments.html

### 対策：朝イチは必ずコードレビューする時間にする

朝、業務を開始するタイミングというのはコンテキストスイッチが発生しません。（あるいは必ず発生すると言ってもいいかもしれないですが）  
このタイミングで必ずコードレビューをするのです。  
また、レビュイーはコメントに即レスできる状態にしておくとベストです。（レビューがある場合はそちら優先でいいと思います）  
30〜60 分あれば最低でも 1 人 1 つは PR のレビューを完了させられるはずです。

自分 1 人だと「今日はちょっと開発から...」と甘い誘惑に負けてしまいやすいので、以下の事例のようにチームの取り組みにしてしまうのもいいと思います。

https://blog.mmmcorp.co.jp/blog/2019/05/24/mokumoku-pr/

ただし、強制されたものは長続きしないと思います。  
なので、合わせて「コードレビューに対する認識」を変えていき、いずれは強制されなくてもコードレビューが後回しにされなくなる必要があると考えます。この点については後述します。

### 対策：コード以外のレビューは別でやる

コードレビューでは、あくまでコードのレビューをしましょう。  
「そんなの当然じゃないか」と思うかもしれませんが、設計のレビューや仕様のレビューも兼ねられている PR を何度も見たことがあります。

レビュアーが仕様すら把握できていない状態というのは、スイッチするコンテキストがそもそも存在しない状態です。コードレビューを始めるには、コンテキストスイッチとは比にならないくらいの時間がかかります。

仕様レビューなら仕様書のレビューを、  
設計レビューなら設計書のレビューを、  
コード以外のレビューは適切な他の手段で行いましょう。

### 対策：対面レビューを行う

コードレビューは非同期コミュニケーションです。  
不明点が多くて質問をたくさんしたい場合や、コードレベルの詳細な設計を議論する場合など、コメントのやり取りが何十回も続くとしたら、その度にコンテキストスイッチが発生してしまいます。

このようなケースが予想される場合は対面レビューが有効です。  
相手からのレスポンスが即返ってくるので、頻繁なコンテキストスイッチの発生を防げます。

レビューとは少し違いますが、ペアプロ・モブプロも同様の効果があるのでおすすめです。

ただし、対面レビューの内容はサマリでもいいので後で PR のコメントに残すようにしましょう。

### 対策：レビュイーは、レビュアーに事前に予告する

コンテキストスイッチが発生するタイミングがあらかじめわかるだけでも負担はかなり減るはずです。  
特に、レビューに時間がかかりそうな PR を投げるときは、できるだけ事前に伝えるようにしましょう。

スクラムを取り入れている場合、スプリントプランニングでおおよそどの程度のコードレビュー依頼が来そうか把握できるとよさそうです。

### 対策：可能な限り自動化する

レビューすべき観点が増えるほどコンテキストスイッチのコストは大きくなります。  
自動でチェックできる観点は自動化しましょう。  
代表的なのは、Lint・フォーマット・テストなどです。

このあたりはググればいくらでもいい記事が出てくるので詳細は割愛します。

### 対策：レビュイーは、レビューしやすい PR を作る

レビューしやすい PR は、コンテキストスイッチのコストが低いです。  
↓ を読んでレビューしやすい PR を作りましょう。

https://fujiharuka.github.io/google-eng-practices-ja/ja/review/developer/

## 知見の浅いコードをレビューしないといけないから

知見が浅いのでコードを読むハードルが高く、結果としてコードレビューを後回しにしてしまうパターンです。
チームへ新規加入したメンバーに起きがちだと思います。

このケースで後回しを防ぐには、コードレビューに対する認識を変えることが重要だと考えます。

### 対策：コードレビューは学習機会と捉える

コードレビューと言うと、書かれたコードが正しく動作するかとか、設計思想に合っているかとかを確認する行為だと思われがちです。  
もしそうだとすると、確かに知見が浅いとコードレビューのハードルは高くなります。

しかし、実際にはそれだけではありません。  
新しい知識を獲得する学習の機会でもあると考えます。

「知見が浅いから、人の書いたコードを読んで勉強しよう」という視点でコードを読めばいいのです。

人の書いたコードから学ぶことはとても多いです。  
実際、自分も新卒 1・2 年目ぐらいのときは、先輩が書いたコードを読んでは、わからないことを質問しまくっていました。  
先輩は自分が知らない便利な関数などを普通に使っているので、それを知れることはとても勉強になりました。

また、質問ではなく「XX という部分が自分の知識では理解できなかったので、参考になるドキュメントを教えてほしいです」と伝えてもいいと思います。  
実は単に可読性の低いコードだっただけで、質問されることでレビュイーもそれに気づいて、コードを修正することもあります。これは立派にコードレビューの目的を果たせていますね。

こんな感じでコードレビューを続けているうちに、知見の浅さというのは解消されていくと思います。

## コードレビューの優先度が低いから

これもよくあるケースだと思います。  
エンジニアはコードを書いていたいと思う人が多く、それ以外の作業の優先度を低くしがちです。

「自分がやらなくてもきっと他の誰かが拾ってくれるだろう」と、自分の中でのコードレビューの優先度を低く見てしまうのです。

これも、コードレビューに対する認識を変えることが重要だと考えます。

### 対策：レビュー優先度の認識を変える

以下のように考えるのが妥当だと思います。

```
レビューの優先度 >= レビュー対象の開発タスクの優先度
```

なぜかというと、「レビューまで含めて開発タスクだから」です。  
レビューが完了して PR がマージされるまで、開発タスクは完了とみなせないはずです。みなしていたら認識を変えましょう。

一方、そう思ってもなかなかできないものなので、始めのうちはコードレビューが強制される仕組みと組み合わせるのも有効だと考えます。

- 朝イチは必ずコードレビューする時間にする（前述の通り）
- レビュアーを 1 人だけ指定する（自分がレビューしなければならない状況を作り出す）

# レビュアーが特定のメンバーに偏ってしまう

これもよくあることだと思います。  
どうしてそのようなことが起きてしまうのでしょうか？

## リードエンジニアにレビュー依頼が集中するから

前述しましたが、コードレビューの目的の一部には以下のようなものがあると思います。

- 書かれたコードが正しく動作するかを確認する
- 書かれたコードが設計思想に合っているかを確認する
- 書かれたコードが他の範囲に悪影響を与えていないかを確認する

このような目的を達成するには、そのコードベースへの知見が深い人にコードを見てもらう必要があると考えがちです。  
その結果、いわゆるリードエンジニアのようなポジションの人にコードレビューの依頼が偏ってしまいます。

ですが、実際にはそんなことはないと考えます。

### 対策：第三者の目線が入るだけで意味があると考える

第三者がコードを確認することで、コードを書いた本人が見落としていたことに気づけたりします。  
これはリードエンジニアが確認しないと達成できないということはなく、チームのメンバーであれば大体誰でも問題ないです。

もちろん、リードエンジニアによるレビューがとても有効なのは間違いありませんが、リソースは有限です。

第三者の目線で確認すること自体が、コードレビューの目的でもあるのです。

この点については、以下の記事がとても参考になりました。

https://techblog.karupas.org/entry/2021/11/07/212429

### 対策：リードエンジニアのレビューが必要な基準を決める

前述したように、第三者の目線で確認すること自体がコードレビューの目的です。  
とはいえそれだけだと、どんなケースではリードエンジニア以外にコードレビューをお願いするのか・しないのかが曖昧です。

基本的には、誰がレビューしてもいいと思います。そして、レビューしてみて不安なところがでてきたら、そこだけリードエンジニアにレビューをお願いすればいいと思います。  
これだけでも、リードエンジニアへのレビュー依頼の集中はかなり減らせるはずです。

また、セキュリティなど、いわゆるクリティカルな部分だけは始めからリードエンジニアにレビューしてもらうというようなルールを決めておくのも有効だと思います。

ただこれも、分かってはいても気づいたらリードエンジニアがレビューを拾ってしまうケースはよくあります。  
始めのうちは、リードエンジニア以外がコードレビューを強制される仕組みを取り入れてもいいと思います。  
例えば、「リードエンジニアは最初にレビューしない」などです。

# まとめ：コードレビューに対する認識を変える必要がある

ここまで様々な対策をあげてきましたが、その中で度々言及したことがあります。

それは、「コードレビューに対する認識を変える」ということです。

「コードレビューたまりがち問題」を根本的に解決するには、これが一番重要だと考えています。  
仕組み作りや自動化も一定の効果はありますが、その根本にある認識を変えていかなければ、それらの効果は一時的なものとなり、元通りになってしまうと思います。

この記事中でも度々引用させていただきましたが、↓ はマジで読んだほうがいいです。  
しっかり読み込んで、自分なりに解釈をしてみることを強くおすすめします。コードレビューに対する認識がどこかしら変化するはずです。

https://fujiharuka.github.io/google-eng-practices-ja/

以下の記事のように、まずは現状の認識をチームで把握してみるのもいいと思います。

https://tech.recruit-mp.co.jp/project-management/post-21684/

認識を変えることとセットで仕組み化や自動化を取り入れることで、少しずつ問題解決に近づいていくと思います。

# さいごに

「コードレビューたまりがち問題」の解決方法ついて、過去の経験などを踏まえた自分の見解を思いつくままに書いてみました。  
1 つでも気づきを得てもらえてたら嬉しいです。

最後までお読みいただきありがとうございました！
