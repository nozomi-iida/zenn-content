---
title: "Renovateでパッケージのバージョン更新を自動化する"
emoji: "💨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Renovate"]
published: true
---

# Renovat とは？

https://docs.renovatebot.com/

Renovate とは依存関係の更新を自動化できる複数のプラットフォームと言語に対応したツールのことです。
依存関係のバージョンに変更があると Renovate が自動でバージョンを更新した Pull Request を作ってくれる上に、Pull Request には Release note も説明欄に書いてあるのでバージョンの変更によって何が変わったのかが分かりやすいです
Renovat はパッケージファイルを自動で見つけます。
モノリポの場合でも複数のパッケージファイルを自動で見つけ、Renovate で管理することができます。

# Renovate を導入する

まずは Renovate を導入しましょう。
Renovate は無料で使用することができます。
以下のページから Renovate をインストールしましょう
https://github.com/apps/renovate

この時どのリポジトリに Renovate を導入するか選択できるのですが、「Only select repositories」で Renovate を入れたいパッケージを選択しましょう

インストールすると Renovat を入れたリポジトリに Pull Request があることを確認できることが確認できます。
ここで自動生成された Pull Requst のブランチから `renovate.json`ファイルで Renovate の設定を進めていきます。

# この記事で設定する Renovate

この記事では以下の設定をできるようにします。
プロジェクトは react のプロジェクトを想定しています。

- [x] 月曜日の朝に Renovate 動かす
- [x] 似たような Package はまとめて Pull Request を作ってもらう(ex: eslint, react)
- [x] patch アップデートの場合は自動で Pull Request をマージしてしまうようにする
- [x] Private な Package を監視する

# 時間を指定して更新する

### タイムゾーンを設定する

まずはタイムゾーンを日本に変更しましょう。
デフォルトだと UTC なので時間を設定しても時差でずれてしまいます。

```renovate.json
"timezone": "Asia/Tokyo",
```

### スケジュールを設定する

Renovate のスケジュールは[breejs/later](https://breejs.github.io/later/parsers.html#text)のシンタックスをサポートしています。

```renovate.json
"schedule": ["after 8am on monday", "before 10am on monday"],
```

Renovate の時間設定は正確にはできないので、幅を持たせます。
これで Renovate を動かすスケジュールを設定することができました。
今回は月曜の午前 8~10 時を設定しました。

# 似たような Package はまとめて Pull Request を作る

全てのパッケージのバージョンが上がるたびに Pull Request を作っていたら大量の Pull Requset が作られてしまい、Renovate の存在が鬱陶しくなってしまうと思います。
そこで似たようなパッケージはまとめて Pull Request を作るようにしましょう

```renovate.json
"packageRules": [
    {
      "groupName": "react",
      "matchPackageNames": ["@types/react", "@types/react-dom"],
      "matchPackagePrefixes": ["react"]
    },
    {
      "groupName": "eslint",
      "matchPackageNames": ["@types/eslint", "babel-eslint"],
      "matchPackagePrefixes": ["@typescript-eslint/", "eslint"]
    },
]
```

上記の設定で`react`に関するパッケージと`eslint`に関するパッケージはまとめて Pull Request が作られるように設定することができます。
設定したオプションをそれぞれ見ていきましょう

- `packageRules`: それぞれのパッケージに Renovate の設定ルールを適用することができます。
- `groupName`: グループの名前を設定しています。ここで設定した名前がブランチで使用されたりします。
- `matchPackageNames`: 名前が一致するパッケージをこのパッケージをこのグループに含むようにします。
- `matchPackagePrefixes`: 接頭辞(最初の文字)が一致するパッケージをこのグループに含むようにします。

これで似たような役割のパッケージがまとめられるので動作確認もしやすくなりました。

# patch バージョンの更新は自動でマージされるようにする

パッケージのバージョンにはメジャー、マイナー、パッチの 3 種類が一般的です。
一般的にパッチバージョンの更新はあまり大きな変更がないので Renovate の Pull Request を自動でマージするようにしたいです。
Renovate の automerging を利用して自動でマージするようにしましょう

```renovate.json
{
   "matchUpdateTypes": ["patch"],
   "automerge": true,
   "platformAutomerge": true,
}
```

上記の設定で automerging の機能を使用することができます。
設定したオプションを確認しましょう

- `matchUpdateTypes`: ここでバージョンのタイプを指定します。
- `automerge`: automerge はここで有効にします
- `platformAutomerge`: 有効にするとより早く Pull Request がマージされるようになります。

これで package のパッチバージョンは自動でマージされるようになります。

# private package を監視する

仕事では社内でしか公開されていない package もあると思います。
private な package は何も設定しないと Renovate はバージョンの確認ができず、Dashboard にもバージョンの確認ができませんという warning が表示されます。
今回は private な GitHub の npm リポジトリを使用していることを想定しています。

```renovate.json
{
  "hostRules": [
    {
      "matchHost": "https://npm.pkg.github.com/",
      "hostType": "npm",
      "encrypted": {
        "token": "<Encrypted PAT Token>"
      }
    }
  ],
  "npmrc": "@organizationName:registry=https://npm.pkg.github.com/"
},
```

- `hostRules`: この中で認証するための資格を設定する
- `matchHost`: 認証したいドメインネームやホストネームを設定します
- `hostType`: ルールやプラットフォームを指定するための方法
- `encrypted`: 認証情報を渡す。詳細は後述
- `npmrc`: `.npmrc`ファイルの文字列をコピーしたもの

`encrypted`の中で `token`を渡すのですが`encrypted`は以下のサイトでできます。
https://app.renovatebot.com/encrypt

token は`read: package`の権限を持つ token を GitHub で作成してください。

# Renovate を使っていて詰まった点

上記の設定でとりあえず完成だと思って楽しみに月曜日も待っていたところ 2 箇所上手く動かなくて詰まってしまったところがありました

## Pull Request が 2 つまでしか作られない

Renovate をインストールしたときに自動で`config:base`が設定されており、`config:base`の設定の中に`:prHourlyLimit2`の設定がされていて、1 時間に作成される Pull Request が 2 個に制限されているのが原因でした。
また、`:prConcurrentLimit10`によって同時に作られる Pull Request も 10 個に制限されているので注意が必要です。

上記の設定がされていたのが原因でした
他にもいくつか設定があるので初めて使う場合は注意してください。
個人的には`config:base`は削除して自分で全部描くのが他の人も分かりやすくて良いかなと思っています。

**config:base**の設定一覧
https://docs.renovatebot.com/presets-config/

## automerge が適用されない

次に躓いた問題は Pull Request が作られたは良いもののパッチバージョンの更新でも自動でマージされない問題でした。
これはテストがない場合に automerge が適用されない仕様が原因でした。
https://docs.renovatebot.com/faq/#renovates-default-behavior-for-majorminor-releases

> If you have no tests but still want Renovate to automerge, you need to add "ignoreTests": true to your configuration.

これは`ignoreTests`のオプションを true にすることでテストがなくても automerge が適用されるようになりました。

## 同じパッケージの Pull Request が 2 つ作られる

同じパッケージの Pull Request が 2 つ作られることがあったのですが、これはメジャーバージョンの更新とマイナーバージョンの更新で Renovate はそれぞれ別で Pull Request を作る仕様になっているのが原因でした。
ただ、これはこのままの挙動が良いのではということになり、この設定は特に変更しませんでした。

https://docs.renovatebot.com/faq/#renovates-default-behavior-for-majorminor-releases

# Renovate の便利に思った点

最後に Renovate を使用していて便利だなって思った点をいくつか紹介します

## Pull Request にマージしたらどんな挙動になるかが書いてある

Renovate が自動で生成した Pull Request の説明欄にマージしたらどんな Pull Request を作るのか、何時に動くのかみたいな説明が書いてあります。
設定をいじるたびにこの Pull Request の説明欄も自動で変わって正しく設定されているのかが分かりやすくて便利でした。

![](https://storage.googleapis.com/zenn-user-upload/3aafc058d070-20221107.png)
スケジュールや、group にまとまっていることが確認できます

## Renovate の Dashboard 機能

Renovate は Pull Request をマージすると Renovate によって自動で Issue が作成されます。
その Issue にはこれから開く予定の Pull Request が表示されます
ここでは他にも設定を修正した時にちゃんと設定が修正されているのかの確認をすることもできます。
また、チェックボックスをチェックすることでスケジュールよりも早く Pull Request を作成したりすることもできます。
こちらの Dashboard は設定によっては作らなくすることもできます

## 設定ファイルに間違えがあったら issue を作ってくれる

設定ファイルに間違えがあって、Renovate が動かなくなった時に issue を作って Renovate の設定に誤りがあることを教えてくれます。
issue の内容も分かりやすく修正がとても楽でした。

## 設定ファイルを更新したら一度 Renovate が動いてくれる

スケジュールを設定した場合でも Renovate の設定ファイルを更新すると一度 Renovate が動くので、正しく修正されているか確認するのに便利でした。

以上が Renovate の設定方法と Renovate を使用してみて便利に感じた点をまとめてみました。
今後の上記の設定をもとに reviwer を設定したり、マージするのに Approve が必須なプロジェクトでは[Renovate Approve 2](https://github.com/apps/renovate-approve-2)を使用したりより良い仕組みを作れるようにしていきたいです。
