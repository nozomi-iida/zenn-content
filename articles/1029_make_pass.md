---
title: "ハンズオンで「パスを通す」を理解する"
emoji: "🦔"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["linux"]
published: true
---

# コマンドを実行できる仕組み

普段開発をしていると「パスを通す」という言葉を聞いたことがあると思います。
パスを通すについてちゃんと理解していますか？
まずはパスを通すについて、そもそもなぜコマンドで色々な操作を実行できるのかの仕組みについて理解していきましょう。
例として、`htop`コマンドについてみていきます。
`htop`コマンドは現在の CPU とメモリの仕様状況について監視するためのコマンドです。
![](https://storage.googleapis.com/zenn-user-upload/e7bbe5d3bfcf-20221030.png)

## htop コマンドはどこから実行されているのか？

ここでは`which`コマンドを使用して、`htop`がどこから実行されているのかを確認してみます。
自分のパソコンで実行した結果が以下です(\*`which`コマンドの実行結果はインストール方法によって変わるので一致しない場合があります。)

```
$ which htop
/usr/bin/htop
```

コマンドと、実行結果が何を表しているのかを解説します。
`which`コマンドとは「指定されたコマンドの絶対パスを表示するコマンド」です。
言い方を変えるとそのコマンドを実行するためのファイルがどこに存在するかを確認することのできるコマンドです。
今回の例でいうと`/usr/bin/htop`が`htop`コマンドを実行した時に使用するファイルです。
`/usr/bin/htop`が本当にファイルを表示しているのかを確認するために`cat /usr/bin/htop`でファイルの中身を確認してみてください。
するととても読めるはずのないファイルを確認することができたと思います。
つまり、`htop`コマンドは`/usr/bin/htop`ファイルの内容を実行しているコマンドだったのです。
そうと分かると次は「なんで`htop`というコマンドは`/usr/bin/htop`というファイルを実行するのだろうか？」と疑問に思うのではないかと思います。
そこで初めて「パスを通す」というキーワードが出てきます。

## パスを通すとは？

まずは自分のパソコンに通っているパスを確認してみましょう

```
echo $PATH | sed -e 's/:/\n/g'
```

上記のコマンドを実行すると自分のパソコンに通っているパス一覧を確認することが出来ると思います。
このコマンドを実行したときに勘がいい人なら「さっきみた/usr/bin が含まれている」と気づいたかもしれません。
つまり、パスを通すとは **「パスにあるファイルを実行するときにいちいちファイルのパスをコマンドに入力するのではなく、実行したいファイル名だけをコマンドに入力するだけでそのファイルが実行される状態のこと」** をいうのです。
