---
title: "`go install`したパッケージをコマンドラインで使えるようにする仕組みを解説する"
emoji: "🦔"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

go のパッケージをインストールして、コマンドでインストールしたパッケージを使用するケースがあると思います。
ただ、なんで`go install`でインストールしたパッケージをコマンドで実行できるのかを理解している人は少ないのではないかと思います。
かくいう私も先日までその仕組みを理解できておらず、新しいパソコンで環境構築をした時に`go install`してもコマンドで実行することができずかなり悩まされました。
今回は`go install`したパッケージをコマンドで実行するまでの仕組みを解説できたらと思います。

# `go install`について

## `go install`と`go get`の違い

# golang のパスを確認する

# golang のパスを修正し、パスを通す