---
title: "Headless CMSを使って自作ブログを作ってみた(1)~strapiを使ってみる~"
emoji: "👌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

エンジニアたるもの「自作ブログを作ってみたい!」ということで今回は open-source の Headless CMS で一番 Github 上でも Star の多い Strapi というサービスを使用してみようと思います。

# Strapi のとは？

https://docs.strapi.io/developer-docs/latest/getting-started/introduction.html

strapi とは、open-source の Headless CMS です。
ちなみに、Headless CMS とはブログのコンテンツ管理と、コンテンツを表示するフロントエンドを分けることをいいます。
対して従来の CMS の定番でもある Wordpress はブログのコンテンツ管理と、ブログを表示するフロントエンドが一体になっています。
strapi は記事を追加・編集・削除する管理者画面と、記事を取得するための API などを提供してくれています。

## Strapi の使用料金

https://strapi.io/pricing-self-hosted

strapi の利用料金は基本的には無料で使用でき、会社で使用し細かいロール設定が必要な場合や、カスタマーサポートを受けたい場合はプランに加入する必要がありそうですが基本は無料で使えると考えていいと思います。

# Starapi を使ってみる

```
npx create-strapi-app@latest my-project --quickstart
```

このコマンドを実行するだけで strapi の環境を作ることができます。
