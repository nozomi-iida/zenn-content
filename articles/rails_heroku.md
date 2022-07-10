---
title: "DockerとRailsで作ったアプリをheokuにgithub actionsでデプロイする方法"
emoji: "🤖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [rails, heroku]
published: true
---
DockerとRailsで作ったアプリをherokuで公開するまでの方法をメモしました

まずはcliでデプロイして、その後github actionsで自動化していきます

## cliでデプロイをする
````markdown
heroku auth:login
heroku container:login
heroku config:set 環境変数 
heroku container:push web(webサーバーを指定している)
heroku addons:create heroku-postgresql:hobby-dev
heroku run bundle exec rake db:migrate
heroku container:release web
heroku open
````

正しくデプロイされているかは`heroku logs --tail`で確認することができます

もし上記のやり方でうまくいかず、logをみても原因がわからなかったら一回gitを使ってherokuにデプロイしてみるのが良いと思います
デプロイできない原因がcli側にあるのか、コード側にあるのかが分かります。
参考:
https://qiita.com/kazukimatsumoto/items/a0daa7281a3948701c39

## github actionsでデプロイする
cliでのデプロイがうまく行ったら次はgithub actionsを使って`master`ブランチにpushしたら自動でherokuにデプロイされるようにしていきます
下記のgithub actionを使うとめっちゃ簡単に設定することができます
https://github.com/AkhileshNS/heroku-deploy#deploy-with-docker

```yaml:deploy.yml
name: deploy to heroku

on:
  push:
    branches:
      - master

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Check out code
        uses: actions/checkout@v3

      - name: Deploy
        uses: akhileshns/heroku-deploy@v3.12.12
        with:
          heroku_api_key: ${{secrets.HEROKU_API_KEY}}
          heroku_app_name: 'heorkuのアプリ名'
          heroku_email: 'メールアドレス'
          usedocker: true
```
これだけでOKです
