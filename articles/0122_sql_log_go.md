---
title: "Golangでsqlのログを表示する方法"
emoji: "😊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

Rails を使っていると最初から sql のログが流れるようになってえいるのですが、Go で `database/sql`を使用する場合は自分でログを表示できるように実装しなければなりません。
今回は [SQLDB-Logger](https://github.com/simukti/sqldb-logger)という go のパッケージを使用して、sql のログを表示する方法をまとめてみました。

## interface を実装する
