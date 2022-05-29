---
title: "goでslack認証を実装する方法"
emoji: "🐙"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["golang"]
published: true
---

golangでslackの認証機能を作成したかったのですが、実装している記事を見つけることができなかったので記事として公開することにしました
webフレームワークは[gin](https://gin-gonic.com/)を使用しています
フロントはReactで実装しました(と言ってもボタンをおいただけです)
# 実装を始める前に
goとReactの最低限の環境を作っておきましょう
https://gin-gonic.com/docs/quickstart/
https://create-react-app.dev/docs/adding-typescript/
# slack側の用意
https://qiita.com/mksava/items/b0031894375616077fc4#slack-%E5%81%B4%E3%81%AE%E7%94%A8%E6%84%8F

### Slackにログインする
連携したいSlackのチームにログインをしておいてください
https://slack.com/intl/ja-jp/

### 連携用の鍵を作成する
以下にアクセスして Create Nes App を押下し、認証用の鍵を作成します
https://api.slack.com/apps

### リダイレクト先URLを設定する
1. Your Apps から先ほど作った App の名前のリンクをクリックします
2. サイドバーの**OAuth & Permissions**をクリックします
ここではあとで実装するAPIのURLを入力します
※localhostのURLは登録できないので注意してください。自分は[ngrok](https://ngrok.com/)を使用して、URLを設定したのですが、開発のたびにリダイレクトURLを設定しないと行けないのがめんどくさいです。。

### Slack認証ページに遷移するためのURLをコピーする
1. サイドバーの**Manage Distribution**をクリックします
2. **Embeddable Slack Button**をコピーしてReactディレクトリに適当に貼り付けてしまいましょう
試しにこのボタンををクリックするとおなじみの認証ページに遷移するはずです
![](https://storage.googleapis.com/zenn-user-upload/a3b33fed48b4-20220529.png)
**allow**をクリックするとコールバックURLへ遷移するはずです

# goでの実装
### 認証機能の実装
### [slack-go](https://github.com/slack-go/slack)というパッケージをインストールします
```markdown
$ go get -u github.com/slack-go/slack
```

### リダイレクトURLを受け取るためのエンドポイントを実装する
次にSlack認証ページで認証した後にリダイレクトURLが返って来た時の処理実装します
```go: AuthController
func (ac AuthController) SlackAuth(c *gin.Context) {
	res, _ := slack.GetOAuthV2Response(http.DefaultClient, os.Getenv("SLACK_CLIENT_ID"), os.Getenv("SLACK_SECRET_KEY"), c.Query("code"), os.Getenv("SLACK_REDIRECT_URI"))
	slackApi := slack.New(res.AccessToken)
	userInfo, _ := slackApi.GetUserInfo(res.AuthedUser.ID)
	fmt.Println(userInfo)
}
```
これでslack認証は完了です
今回はログインしたユーザーの情報を取得するまでを実装しました
実装は全く難しくありません
1. slack apiの**Basic Information**タブのclient_id, secret_keyをコピーして環境変数で管理します
2. codeに関してはリダイレクトされるときにqueryとしてcodeがついてくるのでそれを使用します 
codeはhttps://api.slack.com/legacy/oauthによると一時的な認証コードのことを指すそうで、OAuthResponseの時に必須のパラメーターになります
3. 最後に前に設定したリダイレクトURLを設定すれば[GetOAuthV2Response](https://pkg.go.dev/github.com/slack-go/slack#GetOAuthV2Response)を使用することができます 
4. `GetOAuthV2Response`は返り値で認証したaccessToken, UserのIDを返してくれるので、それを[GetUserInfo](https://pkg.go.dev/github.com/slack-go/slack#Client.GetUserInfo)を使用してアカウント情報を取得することができます
※ここで取得するaccessTokenは有効期限がありません

以上で認証機能の実装は終わりです。
実装自体は難しくないのですが、初めてSlack認証をする人で、OAuthが初めての人には流れを知るまでが一番難しいところかなと思いました
こまで来たらあとはデータベースに取得した情報を保存したり、フロントエンドにRedirectしたりする処理を追加すると良いと思います

#### 参考記事
https://slack.dev/java-slack-sdk/guides/ja/sign-in-with-slack
https://qiita.com/mksava/items/b0031894375616077fc4
https://api.slack.com/legacy/oauth

[//]: # (## エラーメモ)

[//]: # (GetOAuthV2Responseをcallするとinvalid_stateが出てしまい解決策が分からない)

[//]: # (`Value passed for code was invalid.`って書いてあるけど、正しいcodeって何？)

[//]: # (### 試したこと)

[//]: # (`Signing Secret`,`Verification Token`ではなかった)

[//]: # (https://qiita.com/Kontam/items/79fe0c3d338a78132c85を参考にredirect_uriを修正してみたが変わらず)

[//]: # ()
[//]: # (### ワンチャン)

[//]: # (https://github.com/deepbaksu/timebot/blob/bd33c6c654ab481083837b74696ad2a4732e5b9c/api/slack/oauth.go)

[//]: # (このコードをみるとcodeとはqueryで取得できるものらしい)

[//]: # (ただ、今の自分の実装ではcodeはqueryから取得できる可能生は0)

[//]: # (ってことは実装の仕方が間違えてると睨んだ方が良い)

[//]: # (おそらくだけど、GetOAuthV2Responseはredirectされた時に使うメソッドなのかもしれない)

[//]: # ()
[//]: # (完全理解した)

[//]: # (https://app.slack.com/app-settings/T02SDTEDV9R/A02SDSG31V0/distribute)

[//]: # (ここに乗っているURLでログインして、)

[//]: # (リダイレクトされた時にメソッドを使うのが正解だ)

