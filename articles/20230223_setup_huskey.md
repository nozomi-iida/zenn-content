---
title: "huskyを導入してコード品質を担保する"
emoji: "📘"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [react, husky, lint-staged, nextjs]
published: true
---

## husky とは？

husky は git のフックを利用してコード品質を担保するためのツールです。
git のフックとは、git のコマンドを実行する際に自動的に実行されるスクリプトのことです。
husky は git のフックを利用して、コミット前にテストを実行したり、コミットメッセージのフォーマットをチェックしたりすることができます。

## 導入方法

今回は変更したファイルに対してのコード整形を実行するフックと、コミットメッセージのフォーマットをチェックするフックを設定してみます。

### 1. husky をインストールする

```bash
npx husky-init && npm install
```

このコマンドを実行することで`.husky`のファイルが生成され、package.json に husky の設定が追加されます。

### 2. コミット前に eslint を実行するフックを設定する

```bash
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

npm run lint
```

この状態でコミットを実行すると、変更したファイルに対して eslint が実行されます。
これで husky の導入は完了です。
次は変更したファイルに対してのみコード整形を実行するフックを設定してみます。

## 変更したファイルに対してのみコード整形を実行するフックを設定する

変更したファイルに対してのみコード整形を実行するフックを設定するには、`lint-staged`を利用します。

### 1. lint-staged をインストールする

```bash
npm install lint-staged --save-dev
```

### 2. `.lintstagedrc`を作成する

```bash
touch .lintstagedrc
```

```json
{
  "**/*.{js,jsx,ts,tsx}": ["prettier --write", "eslint --fix"]
}
```

これで適当なファイルを変更して`npm run lins-staged`コマンドを実行すると、変更したファイルに対してのみコード整形が実行されます。

## lint-staged と husky を組み合わせる

lint-staged と husky を組み合わせることで、コミット前に変更したファイルに対してのみコード整形を実行することができます。

```bash:pre-commit
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

yarn lint-staged
```

これでコミット前に変更したファイルに対してのみコード整形が実行されます。

## コミットメッセージのフォーマットをチェックするフックを設定する

`commitlint`を使用してコミットメッセージのフォーマットをチェックするフックを設定してみます。

### 1. commitlint をインストールする

```bash
npm install @commitlint/cli @commitlint/config-conventional --save-dev
echo "module.exports = {extends: ['@commitlint/config-conventional']}" > commitlint.config.js
```

commitlint が正しくインストールされているか確認するためにコマンドを実行してみます。

```bash
echo 'foo: bar' | commitlint
⧗   input: foo: bar
✖   type must be one of [build, chore, ci, docs, feat, fix, perf, refactor, revert, style, test] [type-enum]

✖   found 1 problems, 0 warnings
ⓘ   Get help: https://github.com/conventional-changelog/commitlint/#what-is-commitlint
```

こんな感じのエラーが出れば正しくインストールされています。

### 2. コミットメッセージのフォーマットをチェックするフックを設定する

```bash
npm run husky add .husky/commit-msg 'yarn commitlint --edit "$1"'
```

これで変更したファイルに対してのコード整形を実行するフックと、コミットメッセージのフォーマットをチェックができるようになり、コード品質を担保することができます。

## 参考

@[card](https://typicode.github.io/husky/#/)
@[card](https://git-scm.com/book/ja/v2/Git-%E3%81%AE%E3%82%AB%E3%82%B9%E3%82%BF%E3%83%9E%E3%82%A4%E3%82%BA-Git-%E3%83%95%E3%83%83%E3%82%AF)
@[card](https://commitlint.js.org/#/)
