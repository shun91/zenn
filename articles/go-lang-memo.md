---
title: "Go言語を触ってみたのでメモ"
emoji: "📝"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Go", "メモ"]
published: true
---

普段はフロントエンドを開発している筆者が Go 言語を触ってみたので気になったところとか残します。

# インストール

インストールは brew で一発。

https://zenn.dev/y16ra/articles/251c3770365689

# チュートリアル

チュートリアルにはこれが良さそう。公式が出してるものの日本語版らしい。

https://go-tour-jp.appspot.com/welcome/1

以下、やってみて気になったところのメモ。

## `:=` はどういう意味があるんだ・・？

Short variable declarations というものらしい。

https://go-tour-jp.appspot.com/basics/10

## `uint` 型？ `complex64` 型？

`uint` は非負の整数を表す型らしい。

`complex64` は複素数を表す型らしい。

## If with a short statement

`if`文において条件式の前にも式を記述できる。
より簡潔に、そして変数のスコープを狭めて可読性を上げるための記法なのかなという印象。

```go
func pow(x, n, lim float64) float64 {
	if v := math.Pow(x, n); v < lim {
		return v
	}
	return lim
}
```

## Defer

> `defer` へ渡した関数の実行を、呼び出し元の関数の終わり(`return`する)まで遅延させるものです。
>
> `defer` へ渡した関数の引数は、すぐに評価されますが、その関数自体は呼び出し元の関数が`return`するまで実行されません。

どういうケースで使うんだろうと思って調べてみたところ、これがわかりやすかった。  
条件分岐などがあってもリソースの開放処理を忘れないように、先に開放処理を書くことができる。

https://tech.yappli.io/entry/understanding-defer-in-go

## ポインタ

ポインタを介することでミュータブルに変数を扱えるようになると理解した。（通常はイミュータブル）

メモリを効率良く使いたいようなケースで使うのが良さそうだけど、基本的には可読性下がるので使うべきではなさそう。

# さいごに

一旦ここまで。  
やる気があったら「Methods and interfaces」「Concurrency」をやる。

# Appendix

https://qiita.com/tenntenn/items/0e33a4959250d1a55045

https://zenn.dev/y16ra/articles/251c3770365689

https://go-tour-jp.appspot.com/welcome/1
