---
title: "rubocopを設定する"
emoji: "💨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [rails, rubocop]
published: true
---
Rubyの静的コード解析ツール（コーディングチェックツール）」の１つであるrubocopをセットアップし、Github Actionsでmasterブランチにマージされるたびにrubocopが走るまでの環境構築をしていきます

# [rubocop](https://docs.rubocop.org/rubocop/index.html)をセットアップする
1. rubocopのgemをインストールする
```ruby:Gemfile
gem 'rubocop', require: false
```

2. `rubocop -A`コマンドを実行する

3. rubocopの設定を編集する
`.rubocop.yml`ファイルを作成して、rubocopの設定を編集します
```ruby:.rubocop.yml
AllCops:
  NewCops: enable

Style/Documentation:
  Enabled: false
```
`AllCops`でRubyのバージョンを指定します
今回はclassのドキュメント(コメント)を必須にする設定をoffにしました

####  1行の警告を無効化したい場合
```ruby
# メソッド名が長すぎるというエラーを無効化
def handle_error_in_development(e) # rubocop:disable Naming/MethodParameterName
```

これだけでrubocopの設定は完了です

次にciを設定しましょう

# rubocopをGithub Actionsで事項する方法
ciには[reviewdog/action-rubocop](https://github.com/reviewdog/action-rubocop)を使用します
`reviewdog/action-rubocop`はrubocopの結果をpull requestでコメントしてくれる便利なGithub Actionです

[ドキュメント](https://github.com/reviewdog/action-rubocop#example-usage)を参考にGithub Actionsを設定します
```yaml:.github/workflows/rubocop.yml
name: reviewdog
on: [pull_request]
permissions:
  contents: read
  pull-requests: write
jobs:
  rubocop:
    name: runner /rubocop
    runs-on: ubuntu-latest
    steps:
      - name: Check out code
        uses: actions/checkout@v3
      - uses: ruby/setup-ruby@v1
        with:
          ruby-version: 3.1.2
      - name: rubocop
        uses: reviewdog/action-rubocop@v2
        with:
          rubocop_version: gemfile
          rubocop_extensions: rubocop-rails:gemfile
          github_token: ${{ secrets.github_token }}
          reporter: github-pr-review
```

注意するポイントは1つだけで、`github_token: ${{ secrets.github_token }}`の部分です
`secrets.github_token`はciを実行するとGithub側が自動で作ってくれるので、自分たちで特別secretsを設定する必要はありません

ここまで準備ができてからpull_requestを作成するとciが走って、rubocopに違反したコードがあると画像のようにコメントで伝えてくれます
https://github.com/nozomi-iida/zenhub_task_manager/pull/1
![](https://storage.googleapis.com/zenn-user-upload/10f39df4163d-20220627.png)

rubocopを使ってコードの保守性をあげていきましょう！
