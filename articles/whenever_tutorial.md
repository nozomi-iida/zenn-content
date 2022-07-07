---
title: "wheneverを使ってrailsで定期実行機能を実装する"
emoji: "🔥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [ruby, rails]
published: false
---
railsで平日の毎朝11時にメッセージを送ってくれるslack botを作る時に、 [whenever](https://github.com/javan/whenever)を使っている記事をよくみかけたので、今回はwheneverを使った定期実行機能の実装をまとめようと思います。

1. 設定ファイルを作る
設定ファイルは`bundle exec whenever`コマンドを実行すると`config/schedule.rb`ファイルが自動で生成されます

2. 定期処理の関数を実装する
とりあえずHelloというメッセージをslackで送信する関数を1分お気に実行するように実装します
```ruby:app/controllers/burndown_reports_controller.rb
def hello
  client = Slack::Web::Client.new
  client.chat_postMessage(
    channel: '#attendance_management',
    text: 'こんにちは'
  )
end
```

```ruby:config/schedule.rb
set :output, 'log/cron.log'
set :environment, :development
ENV.each { |k, v| env(k, v) }

every 1.minute do
  runner 'BurndownReportsController.hello'
end
```
`set :output, 'log/cron.log'`で`log/cron.log`というファイルにwheneverのログを出すようにします
`ENV.each { |k, v| env(k, v) }`で環境変数を
関数を作ったら`bundle exec whenever --update-crontab`コマンドを実行し、crontabに登録してあげます

2つのエラーが発生しました
1. contrabにwheneverで書いた処理を登録することができない
```markdown
bundler: failed to load command: whenever (/usr/local/bundle/bin/whenever)
/usr/local/bundle/gems/whenever-1.0.0/lib/whenever/command_line.rb:77:in `popen': No such file or directory - crontab (Errno::ENOENT)
```
エラー文からエラーの原因をしらべてみるとdockerの環境構築が原因でした
Dockerfileに`apt-get install -y cron`を追加することで解決しました！
参考: https://qiita.com/hiroki_404_/items/f4859c67be13ed74f258

2. contrabの実行時にエラーが発生する
```markdown
/usr/local/lib/ruby/3.1.0/bundler/definition.rb:481:in `materialize': Could not find bootsnap-1.12.0, dotenv-rails-2.7.6, graphql-2.0.11, graphql-client-0.18.0, puma-5.6.4, rails-6.1.5, slack-ruby-client-1.1.0, sqlite3-1.4.4, whenever-1.0.0, byebug-11.1.3, listen-3.7.1, rubocop-1.31.0, spring-4.0.0, msgpack-1.5.2, dotenv-2.7.6, railties-6.1.5, activesupport-6.1.5, nio4r-2.5.8, actioncable-6.1.5, actionmailbox-6.1.5, actionmailer-6.1.5, actionpack-6.1.5, actiontext-6.1.5, actionview-6.1.5, activejob-6.1.5, activemodel-6.1.5, activerecord-6.1.5, activestorage-6.1.5, sprockets-rails-3.4.2, faraday-2.3.0, faraday-mashify-0.1.0, faraday-multipart-1.0.4, gli-2.21.0, hashie-5.0.0, websocket-driver-0.7.5, chronic-0.10.2, rb-fsevent-0.11.1, rb-inotify-0.10.1, parallel-1.22.1, parser-3.1.2.0, rainbow-3.1.1, regexp_parser-2.5.0, rubocop-ast-1.18.0, ruby-progressbar-1.11.0, unicode-display_width-2.2.0, method_source-1.0.0, thor-1.2.1, concurrent-ruby-1.1.10, i18n-1.10.0, tzinfo-2.0.4, zeitwerk-2.5.4, mail-2.7.1, rails-dom-testing-2.0.3, rack-2.2.3, rack-test-1.1.0, rails-html-sanitizer-1.4.2, nokogiri-1.13.3-x86_64-linux, builder-3.2.4, erubi-1.10.0, globalid-1.0.0, marcel-1.0.2, mini_mime-1.1.2, sprockets-4.0.3, faraday-net_http-2.0.3, multipart-post-2.2.3, websocket-extensions-0.1.5, ffi-1.15.5, ast-2.4.2, loofah-2.15.0, crass-1.0.6 in any of the sources (Bundler::GemNotFound)
```


`crontab -l`コマンドでcrontabに書き込まれたかを確認することができます

schedule.rbファイルを更新したいときは`bundle exec whenever --update-crontab && service cron restart`のコマンドを実行しましょう
