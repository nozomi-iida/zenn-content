---
title: "Chakra UIのススメ"
emoji: "💨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

# [Chakra UI](https://chakra-ui.com/)とは？

Chakra UI とはシンプルでカスタマイズ性の高さが特徴的な UI ライブラリです。
Ant Design や、Material UI は初めからスタイルが当てられておりカスタマイズ性はあまり高くなく、スタイルの上書きなどがしにくいのに対し、Chakra UI は全てのコンポーネントがスタイルを上書きすることができます。
また、Chakra UI は全てのスタイルを props に渡すことで当てることができます。

# 基本的な使い方

まずはパッケージをインストールします

```
yarn add @chakra-ui/react @emotion/react @emotion/styled framer-motion
```

次に`index.tsx`または、`App.tsx`ファイルに`ChakraProvider`を追加します

```
import * as React from 'react'

// 1. import `ChakraProvider` component
import { ChakraProvider } from '@chakra-ui/react'

function App() {
  // 2. Wrap ChakraProvider at the root of your app
  return (
    <ChakraProvider>
      <TheRestOfYourApplication />
    </ChakraProvider>
  )
}
```

これだけで Chakra Component を使用することができるようになります

# テーマを変更する

次に Chakra のテーマの変更方法を見ていきましょう
テーマの初期値は以下のページでみることができます。
https://chakra-ui.com/docs/styled-system/theme

以下のテーマをカスタマイズすることができます

- カラーテーマ
- global スタイルの変更(body タグや、div タグのスタイル変更)
- コンポーネントのスタイルの変更 など

個人的に`red.500`という書き方は複数人で開発するときや、同じ色を複数回使い使うときに数字を覚えないといけないので好きではありません。
よく使用する色はここで追加しておきましょう

## CLI ツールを使用する

Chakra UI には CLI も用意されています。
CLI を使うことでテーマで作成した token の型を生成することができます
CLI を実行した後`node_modules/@chakra-ui/styled-system/dist/theming.types.d.ts`のファイルを確認することで自作した token が追加されていることが確認することができます。

```
// CLIを実行することでcustomColorが予測変換で出てくれるようになる
<Button color="customColor">Hoge</Button>
```

実際に使い方を見てみましょう

1. まずは cli のパッケージをインストールします

```
yarn add --dev @chakra-ui/cli
```

2. コマンドを実行します
   ｀``
   chakra-cli tokens path/to/theme.ts

```

これでカスタムtokenの型を作成することができます。

# ダークテーマを実装する

# レスポンシブ対応をする
# Chakra UIのTips
## `sx`を使用する
```
