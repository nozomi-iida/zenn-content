---
title: "protocol bufferに入門した"
emoji: "💭"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["protocolbuffers"]
published: true
---

10 月から自分が入った案件でスキーマ言語に Protocol Buffers を利用していて自分も Protocol Buffers を学んだので、今回は入門者向けに Protocol Buffers とはなにか、Protocol Buffers の使い方に関して自分が学んだことをアウトプットしてみようと思い記事を書くことにしました。
protocol buffers のかなり初歩的な記事となります。

# Protocol Buffers とは？

Protocol Buffers とは Google の[公式ドキュメント](https://developers.google.com/protocol-buffers)から引用すると

> Protocol buffers are Google's language-neutral, platform-neutral, extensible mechanism for serializing structured data

日本語に訳すと「Google が作成したデータをシリアライズするためのメカニズムのこと」です
ただ、これだけ見てもなんのこっちゃか分からないと思います。
自分が一番しっくりした回答としては「Protocol Buffers はスキーマ言語である」です。
スキーマ言語とは有名なところでいうと OpenAPI や、Graphql スキーマなんかが有名だとおもうのですが、Protocol Buffers の役割もこれらに近いと考えていいと思います。
要するに「フロントエンドとバックエンドが別れて開発する場合のデータ構造を共有するためのもの」という説明が分かりやすいのではないかと思います。

# Protocol Buffers を書いてみる

概要が分かったところで早速 Protocol Buffers を実際に書いていきましょう

```
syntax = "proto3";

option go_package = "./gen";

package helloWorld;

service Greeter {
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

message HelloRequest {
  string name = 1;
}

message HelloReply {
  string message = 1;
}
```

究極にシンプルなスキーマを書きました。
初めて見た人でもなんとなく意味は分かるんじゃないでしょうか？
このシンプルさは Protocol Buffers の利点だと思います。
ファイルの中で使用しているそれぞれの値をみていきましょう
(option に関しては必ず必要というわけではないので、コードの自動生成のタイミングで説明します。)

### syntax

ここで protoc のバージョンを指定します

### package

protoc ファイルのパッケージ名を追加します
パッケージ名を追加することで他のフィールドでも message の型を使うことができるようになります

```
message Test {
  helloWorld.HelloResponse hello = 1;
}
```

### message

リクエスト、レスポンスなどのフィールドの名前と型を定義します。
特に難しいところはないと思うのですが、フィールドの最後にある` = 1`はなんだ？と思ったのではないでしょうか？
これはフィールドナンバーで一意(ユニーク)な値を設定します。
この数字はフィールドがバイナリーに変換された時にフィールドを特定するために使用するらしいのですが、詳しく分かっていません。。。
[こちら](https://engineering.mercari.com/blog/entry/20210921-ca19c9f371/)のサイトが詳しく書いてあると思いました。
フィールド番号に関して覚えておかないといけないことは「フィールド番号は一意な値を設定すること」です。

### service

rpc システムで定義したメッセージを使用したい場合はこのサービスに定義していきます。
こちらは特に解説がなくても分かるのではないかなと思います。

# Protocol Buffers でコードを自動生成してみる

.protoc ファイルの基本を理解できたところで実際にコードを自動で生成してみましょう
今回は Go のコードを生成してみます

## 事前準備

1. `protoc`コマンドを使用できるようにする

.protoc ファイルからコードを生成するには protoc コマンドを使用できるようにする必要があります
Mac を使ってる方なら`brew`を使用してインストールすることができます。

```
$ brew install protobuf
```

他の PC を使っている方は[こちら](https://github.com/protocolbuffers/protobuf/releases)から最新のバージョンを使うようにしましょう

2. go のプラグインを使用できるようにする
   `.proto`ファイルを go に変換するためのプラグインをインストールします

```
$ go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
$ go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
```

これで準備完了です。
準備が完了したら以下のコマンドをターミナルで実行してみてください

```
protoc --go_out=proto --go_opt=paths=source_relative \
    --go-grpc_out=proto --go-grpc_opt=paths=source_relative \
    .
```

事前準備が正しくできていればこれでうまくコードを自動生成できたと思います。
option の意味ですが、go でコードの自動生成を行う場合それぞれの`.proto`ファイルに`go_package`というオプションを使用してコード生成したファイルをどこに置くのかを書く必要があります。

## protoc コマンドのオプションがそれぞれどんな意味を持っていたのか確認してみましょう

アンダーバーの直前`go_out`でいうと`go`の部分が使用するプラグインの名前を書いていて、今回は`go`と`go-grpc`という２つのプラグインを使用しています。
アンダーバーの直後がオプションを表していて、`out`は生成したコードをどこに置くか、`opt`はプラグインのオプションを指定しています。
オプションの`paths=source_relative`は`out`での指定が相対パスであることを明示しています。

これで最低限の protocol buffers を使えるようになります。
次はここで生成したコードを使用して grpc を作るのですが、今回はここまでとします。
