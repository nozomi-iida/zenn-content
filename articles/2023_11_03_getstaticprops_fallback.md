---
title: "`getStaticPaths`で設定できる3種類のfallbackについて"
emoji: "🦁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Next.js"]
published: true
---

[Functions: getStaticPaths](https://nextjs.org/docs/pages/api-reference/functions/get-static-paths)

## fallback: `false`

`getStaticPaths` によって返されなかったパスは 404 を返す

少ない数のパスしかない場合や、データが頻繁に更新されないものに関しては有効

もし path を追加する必要があれば、 `next build` を実行する必要がある

## fallback: `true`

- build 時に生成されなかったページは 404 を返すのではなく、代わりに最初のリクエスト時に fallback バージョンのページを表示する
- Google クローラーに対しては fallback バージョンではなく、 `fallback: 'blocking'` と同じ挙動をする
- `next/link` や、 `next/router` 経由で `fallback: true` のページに遷移した場合、そのページは fallback バージョンではなく、 `fallback: 'blocking'` の挙動をしたページを表示する
- 背景では、Next.js は静的な html ファイルと json(レンダリングするのに必要なデータ)を生成し、 `getStaticProps` も走る
- 上記の処理が完了すると、json を受け取り、自動的に json を使用してページをレンダリングする
- 同時に、Next.js ではそのパスをプリレンダリングページのリストに入れ、次回のリクエストからは静的ページとして扱われる

### fallback バージョンのページ

[Functions: getStaticPaths](https://nextjs.org/docs/pages/api-reference/functions/get-static-paths#fallback-pages)

ページの props は空になる

router の `router.isFallback` を使用して、 `getStaticProps` の処理が終わるまでページを表示しないようにする工夫が必要

### fallback: `true` が便利な場合

静的ページが大量にあり、pre-rener しようとすると build にとても長い時間がかかる

代わりに小さなページのサブセットを静的に生成し、残りを `fallback: true` することができる

## fallback: `'blocking'`

`getStaticPaths` で作成されていない新しいパスは HTML ファイルが作成されるのを待つ

つまり、SSR と同じである。

そして、これも一度生成されたらキャッシュされる

- SSR が完了し、ページが表示されても loading/fallback の状態変化はない

## まとめ

`fallback: true` の場合と、 `fallback: 'bloking'` の場合が若干似ている気がするが、 `fallback: true` の場合は HTML と JSON を返し、 `blocking` の場合は HTML しか返さないという違いがある

SSR を使用して静的ページを作りたい場合は `fallback: true` を設定した方が良さそうに感じた

どちらかというと、 `fallback: 'bloking'` は `fallback: true` の場合に Google クローラーや、`next/link` や、 `next/router` で `fallback: true` ページに遷移した場合に使われるものという感じだと思う。

特に、`fallback: true` を使用することによってビルド時間を短縮できるのは大きな魅力に思った。
