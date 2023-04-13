---
title: "semantic releaseを使ってパッケージを手軽に公開する方法"
emoji: "😎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [javascript]
published: true
---

先日仕事でパッケージを作っていて、semantic-release を知って感動したので記事にすることにしました。

まず、semantic-release の良いところは

- master ブランチに push したら自動でパッケージのバージョンが上がって、パッケージが更新されること
- コミットメッセージから自動で Release 　 Note を作成してくれること
  だと思います。
  また、バージョン毎のタグ付けだったり、コミットのリンクもつけて Release Note を作ってくれてすごく良く自動化されていると思います。
  ただ、semantic-release を使うにあたって、ドキュメントがイマイチ読みづらく実際に使ってみないと良さが分からないなと思ったので、この記事を呼んだら semantic-release の良さを分かってくれるような記事にしたいです。

## セットアップ

では、早速プロジェクトを作って見ましょう
今回は究極にシンプルな cli を作ります
今回は`sayHello`とコマンドを叩いたら`Hello!`と返ってくる cli を作ります
https://qiita.com/toshi-toma/items/ea76b8894e7771d47e10
こちらの記事を参考に作りました

1. ファイルを作成する

```markdown
mkdir semantic-release-tutorial
npm init -y
mkdir lib
touch lib/cli.js
```

2. 実行ファイルの作成

```javascript:./lib/index.js
module.exports = () => {
  console.log("Hello!")
}
```

3. コマンドとファイルのマッピング

```json:package
{
  // ...
  "bin": {
    "sayHello": "bin/cli.js"
  },
  // ...
}
```

コマンドとファイルのマッピングは bin フィールドでします

```markdown
mkdir bin
touch bin/cli.js
```

```javascript:bin
#!/usr/bin/env node

require("../lib/index")();
```

`#! /usr/bin/env node`は node の環境を作るためにあります。
ためしに command line で`/usr/bin/env node`と実行してみてください
node が起動するはずです
詳細は[こちら](https://wa3.i-3-i.info/word14585.html)を見てみてください

4. `npm link`を使って手元で試せるようにします
   `npm link`を使うとグローバルなパッケージがインストールされる場所にシンボリックリンクが作られます

[//]: # "TODO: シンボリックリンクに関して書く"

これでコマンドラインで`sayHello`とうつと`Hello!`と返ってくるはずです

## semantic-release を使用する

最低限の package を作ったつもりなのですが、思ったよりも長くなってしまったかもしれません
ここからがこの記事の本題です！

1. semantic-release を install する

```markdown
npm i -D semantic-release
```

## ci の設定をする

https://github.com/semantic-release/semantic-release/blob/master/docs/recipes/ci-configurations/github-actions.md
上記のコードを参考に setup をしていきます

1. 環境変数を設定する
   `NPM_TOKEN`と`GITHUB_TOKEN`が必要です。
   `GITHUB_TOKEN`は ci の過程で自動で生成されるので問題ないのですが、`NPM_TOKEN`は[こちら](https://docs.npmjs.com/creating-and-viewing-access-tokens)を参考に自分で用意する必要があります
   :::message
   NPM_TOKEN の権限あ`Read and write`にする必要があります
   :::
   `NPM_TOKEN`を作成したら下記の画像のように token を保存しましょう
   ![](https://storage.googleapis.com/zenn-user-upload/d4373de0148b-20220530.png)

2. github actions を作成する

```yaml:./.github/workflows/release.yml
name: Release
on:
  push:
    branches:
      - master
permissions:
  contents: write
jobs:
  release:
    name: Release
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3
        with:
          fetch-depth: 0
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: "lts/*"
      - uses: pnpm/action-setup@v2
        with:
          version: 7
      - name: Install dependencies
        run: pnpm install
      - name: Release
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
        run: npx semantic-release
```

上記のコードをそのまま使えます
ci を作ったら github に push して、ci を動かしてみましょう
この時 commit メッセージは以下と同じにしてください

```markdown
git add .
git commit -m 'chore: setup semantic release
git push
```

ci の結果を見ると全てパスをしています。
ただ、肝心な package が公開されていないです。

```markdown
[3:16:39 PM] [semantic-release] › ✔ Completed step "verifyConditions" of plugin "@semantic-release/npm"
[3:16:39 PM] [semantic-release] › ℹ Start step "verifyConditions" of plugin "@semantic-release/github"
[3:16:39 PM] [semantic-release] [@semantic-release/github] › ℹ Verify GitHub authentication (https://api.github.com)
[3:16:39 PM] [semantic-release] › ✔ Completed step "verifyConditions" of plugin "@semantic-release/github"
[3:16:39 PM] [semantic-release] › ℹ Found git tag v1.0.0 associated with version 1.0.0 on branch master
[3:16:39 PM] [semantic-release] › ℹ Found 1 commits since last release
[3:16:39 PM] [semantic-release] › ℹ Start step "analyzeCommits" of plugin "@semantic-release/commit-analyzer"
[3:16:39 PM] [semantic-release] [@semantic-release/commit-analyzer] › ℹ Analyzing commit: chore: package version to 1
[3:16:39 PM] [semantic-release] [@semantic-release/commit-analyzer] › ℹ The commit should not trigger a release
[3:16:39 PM] [semantic-release] [@semantic-release/commit-analyzer] › ℹ Analysis of 1 commits complete: no release
[3:16:39 PM] [semantic-release] › ✔ Completed step "analyzeCommits" of plugin "@semantic-release/commit-analyzer"
[3:16:39 PM] [semantic-release] › ℹ There are no relevant changes, so no new version is released.
```

github actions のログをみてみると`There are no relevant changes, so no new version is released.`と書いてあり、パッケージが公開されてそうにありません

https://github.com/semantic-release/semantic-release/issues/192
調べてみると`commit analyzer`はコミットメッセージに`feat`又は`fix`がないとバージョンをあげてくれないそうです。
試しに`feat`をつけてからコミットをしてみます

```markdown
git commit --allow-empty -m 'feat: release'
git push
```

無事リリースノートとパッケージが公開されました！
https://github.com/nozomi-iida/semantic-release-tutorial/releases/tag/v1.1.0
https://www.npmjs.com/package/semantic-release-tutorial

npm にパッケージを公開した場合自分のアイコンをクリックして、**packages**をクリックすると確認することができます
![](https://storage.googleapis.com/zenn-user-upload/2cd2bc5b5b95-20220531.png)

## CHANGELOG を作成する

パッケージを公開するという目的は達成したのですが、最後に Changelog を作ってみましょう
Changelog とはバージョンごとにどのような変更があったのかをまとめるテキストのことを指します。
これも`semantic-release`を使うと自動化することができます

1. `@semantic-release/changelog`というプラグインをインストールする
   [@semantic-release/changelog](https://github.com/semantic-release/changelog)と[@semantic-release/git](https://github.com/semantic-release/git)という公式プラグインをインストールします
   @semantic-release/git は changelog で作成したファイルをコミットするためにいれます

```markdown
npm install @semantic-release/changelog @semantic-release/git -D
```

2. 設定ファイルを作成し、設定を書く

```markdown
touch .releaserc.yml
```

`.releaserc.yml`というファイルを作成し、設定を書きます

```yaml:.releaserc
{
  "plugins": [
    "@semantic-release/commit-analyzer",
    "@semantic-release/release-notes-generator",
    [
      "@semantic-release/changelog",
      {
        "changelogFile": "docs/CHANGELOG.md"
      }
    ],
    "@semantic-release/npm",
    [
      "@semantic-release/git",
      {
        "assets": ["docs/CHANGELOG.md"]
      }
    ]
  ]
}
```

https://github.com/semantic-release/changelog#usage

ほとんど公式通りにすれば問題ないです。
注意点としてはプラグインの順番で、`@semantic-release/changelog`を記述した後に`@semantic-release/npm`と`@semantic-release/git`を記述してください
これで再度 github に push されると CHANGELOG が自動で出来上がっています

https://github.com/nozomi-iida/semantic-release-tutorial/blob/master/docs/CHANGELOG.md

いかがだったでしょうか？
手動だと tag を作って、リリースノートを作って、changelog を作ってとかなり手間になり、適当になってしまうと思います。
しかし、`semantic-release`を使うことでそれらの作業を全て自動化することができました！
みなさんも package を作成するときは是非使ってみてください！
