---
title: "viteで型エラーとeslintのwarningを表示させる方法"
emoji: "👌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: true
---
今回はvite環境でも型エラーや、eslintを簡単に表示する方法が分かったので記事を書くことにしました。
型エラーとeslintのエラーに気づきやすくなり、開発がしやすくなると思います。

## 実装したリポジトリ
https://github.com/coadmap/react_vite_bp

## 型エラー, eslintのエラーを表示する方法
[eslint-plugin-checker](https://github.com/fi3ework/vite-plugin-checker)を使用することで型エラーを表示できるようになります
インストールをして、`vite.config.ts`ファイルに少し手を加えるだけでエラーを表示できるようになります。
`npm i vite-plugin-checker -D`

```typescript:vite.config.ts
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import checker from "vite-plugin-checker";
import tsconfigPaths from "vite-tsconfig-paths";
import svgr from "vite-plugin-svgr";

export default defineConfig({
  plugins: [
    react(),
    checker({
      typescript: true,
      eslint: {
        lintCommand: 'eslint "./src/**/*.{ts,tsx}"',
      },
    }),
  ],
});
```
これだけで表示することができます

試しに型エラーとeslintのwarningを出すと下記の画像のようにエラーが表示されます
エラーを表示したくないときは左したの**close**をクリックすると閉じることができます
![](https://storage.googleapis.com/zenn-user-upload/fea36ef0f485-20220707.png)

コマンドラインにもしっかり表示されます
```markdown
> vite

Port 3000 is in use, trying another one...

  vite v2.9.13 dev server running at:

  > Local: http://localhost:3001/
  > Network: use `--host` to expose

  ready in 463ms.


 ERROR(TypeScript)  Argument of type 'string' is not assignable to parameter of type 'SetStateAction<number>'.
 FILE  /home/nozomi/Desktop/react_vite_bp/src/App.tsx:9:14

     7 |   useEffect(() => {
     8 |     setCount(count)
  >  9 |     setCount("count")
       |              ^^^^^^^
    10 |   }, [])
    11 |
    12 |   return (

[TypeScript] Found 1 error. Watching for file changes.
 WARNING(ESLint)  React Hook useEffect has a missing dependency: 'count'. Either include it or remove the dependency array. You can also do a functional update 'setCount(c => ...)' if you only need 'count' in the 'setCount' call. (react-hooks/exhaustive-deps)
 FILE  /home/nozomi/Desktop/react_vite_bp/src/App.tsx:10:6

     8 |     setCount(count)
     9 |     setCount("count")
  > 10 |   }, [])
       |      ^^
    11 |
    12 |   return (
    13 |     <div className="App">

[ESLint] Found 0 error and 1 warning
```

自分はviteは食わず嫌いをしていたのですが、一度使ってみるとbuild時間の速さに感動しました。
型エラーとeslintのwarningを表示するとwebpackとほぼ変わらない開発が出来ると思うので、viteを使う人は[eslint-plugin-checker](https://github.com/fi3ework/vite-plugin-checker)の使用を検討してみてください
