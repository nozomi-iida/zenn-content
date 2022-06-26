---
title: "RailsでzenhubAPIを使ってみる"
emoji: "👋"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [rails, graphql]
published: true
---

今回は仕事でタスク管理ツールとしてzenhubを使用しているのですが、zenhubが最近[GraphqlAPI](https://developers.zenhub.com/)を公開したので、zenhubのAPIをRailsで使いながら遊んでみようと思います

## API Keyを作成する
左下のユーザー名をクリックして、アカウントマネジメントページの移動し、`api`タブをクリックすると下記のページに行くのでそこからAPI keyを作成してください
   ![](https://storage.googleapis.com/zenn-user-upload/d3ab9babe925-20220626.png)

API Keyは外部に公開してはいけないので、API Keyを作成したら、`.env`ファイルに入力しましょう

## Zenhubのエンドポイントを定数で管理する
zenhubのエンドポイントは`https://api.zenhub.com/public/graphql`を定数で管理します
定数管理は[こちら](https://qiita.com/srockstyle/items/daed31a78c343e607822)の記事を参考にしました
```ruby:config/initializers/constants.rb
module Constants
  ZEN_HUB_URL = https://api.zenhub.com/public/graphql
end
```
## issueを取得するためのqueryを確認する
zenhubのgraphqlの使用は[Explorer](https://developers.zenhub.com/explorer)で確認することができます
以下のqueryでissueの情報を取得することができます
```graphql endpoint
query getIssueInfo($repositoryGhId: Int!, $issueNumber: Int!, $workspaceId: ID!) {
  issueByInfo(repositoryGhId: $repositoryGhId, issueNumber: $issueNumber) {
    id
    title
    body
  }
}
```
queryを書いてみるとissueを取得するにはrepositoryGhIdが必要なことが分かります。
````graphql endpoint
query getRecentlyViewedWorkspaces{
  recentlyViewedWorkspaces {
    nodes {
      id
      repositoriesConnection {
        nodes {
          id
　　　　　　ghId
        }
      }
    }
  }
}
````
上記のqueryでrepositoryGhIdを取得することができます
必要なqueryが分かったところで実際にRailsでqueryを取得してみましょう
## rubyでgraphqlのqueryを取得する
ここからが本題です
RubyでzenhubのAPIを使ってissueを取得するための実装をします

1. [graphql-client](https://github.com/github/graphql-client)のgemを追加します
```ruby:Gemfile
gem 'graphql-client'
```
2. graphql-clientの設定ファイルを実装する
```ruby:config/initializers/zenhub_api.rb
require "graphql/client"
require "graphql/client/http"

module ZenhubApi
  HTTP = GraphQL::Client::HTTP.new("https://api.zenhub.com/public/graphql") do
    def headers(context)
      { "Authorization": "Bearer " + ENV["ZENHUB_API_KEY"] }
    end
  end

  schema = GraphQL::Client.load_schema("db/schema.json")
  Client = GraphQL::Client.new(schema: schema, execute: HTTP)
end
```
基本は[document](https://github.com/github/graphql-client#configuration)通りに実装していますが、
注意点は`db/schema.json`ファイルの存在です
graphql-clientを使うには、スキーマファイルが必要なのですが、zenhubのdocumentにはschema.jsonファイルは見つけられません
そこで、自分でzenhub apiの`schema.json`ファイルを作る必要があります。
https://zenn.dev/nekoshita/articles/7c454e8e552c0d

上記の記事を参考に`schema.json`ファイルを作成しましょう

```markdown
npm install -g get-graphql-schema
get-graphql-schema -j -h 'Authorization=Bearer <先程作成したAPI KEY>'  https://api.zenhub.com/public/graphql > schema.json
```
こちらのコマンドを実行すると`schema.json`ファイルを作成することができます

3. routesを設定します
```ruby:routes.rb
resources :issues, only: [:show]
```

4. showメソッドを実装する
```ruby:application_controller.rb
class ApplicationController < ActionController::API
  include ZenhubApi
end
```
```ruby:app/controllers/v1/issues_controller.rb
class V1::IssuesController < ApplicationController
  def show
    workspace = ZenhubApi::Client.query(RecentlyViewedWorkspacesQuery)
    ghId = workspace.to_h["data"]["recentlyViewedWorkspaces"]["nodes"][0]["repositoriesConnection"]["nodes"][0]["ghId"]

    issueInfo = ZenhubApi::Client.query(IssueInfoQuery, variables: { repositoryGhId: ghId, issueNumber: 1659 })

    render json: issueInfo.to_h
  end

  private
  
  RecentlyViewedWorkspacesQuery = ZenhubApi::Client.parse <<~'GRAPHQL'
    query {
      recentlyViewedWorkspaces {
        nodes {
          id
          name
          repositoriesConnection {
            nodes {
              id
              ghId
              name
            }
          }
        }
      }
    }
  GRAPHQL

  IssueInfoQuery = ZenhubApi::Client.parse <<~'GRAPHQL'
    query($repositoryGhId: Int!, $issueNumber: Int!) {
      issueByInfo(repositoryGhId: $repositoryGhId, issueNumber: $issueNumber) {
        id
        title
        body
      }
    }
  GRAPHQL
end
```
これでissue情報を取得することができるはずです

今回はzenhubのAPIを使ってissue情報を取得するための方法を詳解しました。
`graphql-client`をRailsで使うことや、schema.jsonを作ることに対して馴染みがなかったのでつまりましたが、なんとかissue情報を取得することができました。
