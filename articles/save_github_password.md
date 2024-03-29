---
title: "[ubuntuを使っている人へ]GitHubのパスワードを保存しておく方法"
emoji: "😁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [ubuntu]
published: true
---

ubuntuを使っていてGitHubを使おうとすると何度もユーザーネームとパスワードを求められてしまって面倒くさいなという経験をした人は多いはずです。
今回はubuntuでGitHubのパスワードを保存する方法を紹介したいと思います！
これでもうユーザーネームとパスワードを入力することはなくなります！！

ubuntuのバージョンは20.4です

# GitHubのユーザーネームとパスワードを保存する方法
[Libsecret](https://wiki.gnome.org/Projects/Libsecret)を使用します

1. `Libsecret`をインストールする
```markdown
sudo apt-get install libgnome-keyring-dev
```
2. `libsecret`をビルドする
````markdown
sudo make --directory=/usr/share/doc/GitHub/contrib/credential/libsecret
````
makeコマンドは**大きなソースファイルの中で再コンパイルする必要がある部分を自動的に抽出し、再コンパイルのための手続きを実行する**ためのコマンドです。
makeコマンドを実行するためには`Makefile`が必要になります

参考: 
https://linuc.org/study/knowledge/411/

3. GitHubのcredentialに`libsecret/GitHub-credential-libsecret`を登録する
credentialはITでは認証などに用いられるID、ユーザー名、暗証番号、パスワード、生体パターンなどの識別情報の意味を持ちます。
GitHubのcredential helperはユーザーネームやパスワードをキャッシュするために使用されます

参考: 
https://e-words.jp/w/%E3%82%AF%E3%83%AC%E3%83%87%E3%83%B3%E3%82%B7%E3%83%A3%E3%83%AB.html
````markdown
GitHub config --global credential.helper /usr/share/doc/GitHub/contrib/credential/libsecret/GitHub-credential-libsecret
````
`cat ~/.GitHubconfig`を見てみるとcredentialのhelperに`/usr/share/doc/GitHub/contrib/credential/libsecret/GitHub-credential-libsecret`が登録されていることを確認することができます。

4. `GitHub pull` or `GitHub push`コマンド入力して、ユーザーネームとパスワードを記入する
これで次回からはユーザーネームとパスワードは入力しなくて良いようになります！

ubuntuでGitHubを使う時には必須の設定だと思うので、まだ設定指定な人は簡単なので是非設定してみてください！
