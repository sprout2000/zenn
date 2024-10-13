---
title: "GitHub Pages へデプロイする"
free: true
---

# アプリをデプロイ（公開）する

最後にこの Todo アプリを [GitHub Pages](https://docs.github.com/ja/pages/getting-started-with-github-pages/about-github-pages) へデプロイ（公開）してみましょう。

https://docs.github.com/ja/pages

:::message
この章では、すでに GitHub アカウントを取得済みで、かつ GitHub へなんらかのリポジトリを作成済みであることを前提としています。
このあたりの設定手順については、巻末の[「（補遺）プログラミング環境の準備」](https://zenn.dev/sprout2000/books/76a279bb90c3f3/viewer/appendix04)を参照してください。
:::

https://zenn.dev/sprout2000/books/76a279bb90c3f3/viewer/appendix04

## GitHub Pages とは？

GitHub Pages とは、[GitHub](https://github.co.jp/) 社が提供する静的サイトのホスティングサービスです。GitHub アカウントがあれば、非常にお手軽に静的サイトが公開できます。

https://docs.github.com/ja/pages/getting-started-with-github-pages

この他にも、[Netlify](https://www.netlify.com/) や [Heroku](https://jp.heroku.com/home) といった静的 Web サイトホスティングサービスがあります。

https://www.netlify.com/

https://jp.heroku.com/home

## デプロイする

[GitHub Actions](https://docs.github.com/ja/actions) を用いて自動デプロイする方法もありますが、入門書である本書ではもっとも簡単な方法で GitHub Pages へデプロイします。

GitHub Actions を使ってのテストやデプロイの自動化に興味のある人は、以下のドキュメント等を参考にしてください。

https://docs.github.com/ja/actions

https://github.com/peaceiris/actions-gh-pages

### 1. `vite.config.ts` をアップデートする

Vite ではリソースの読み込み先に絶対パスの **`/` (= ルートディレクトリ)** が使用されるため、[GitHub Pages](https://docs.github.com/ja/pages/getting-started-with-github-pages/about-github-pages) のようなルートディレクトリ以外の場所にデプロイされるサービスではそのままでは読み込みに失敗してしまいます。

[`base`](https://vitejs.dev/config/#base) を設定しましょう。

```diff ts:vite.config.ts
  // https://vitejs.dev/config/
  export default defineConfig({
+   base: './',
    plugins: [
      react(),
```

また、マニフェストへ **`id`** プロパティ（デプロイ場所）を設定することも推奨されているので、これも `vite.config.ts` へ指定します。

```ts:vite.config.ts
export default defineConfig({
  base: './',
  plugins: [
    VitePWA({
      manifest: {
        // 省略
        id: '/todo/', // 通常は GitHub リポジトリ名と同じ
      }
    })
  ]
})
```

### 2. `gh-pages` によるデプロイ

GitHub Pages へのデプロイを手軽に実現できるツール、[gh-pages](https://www.npmjs.com/package/gh-pages) を利用します。

https://www.npmjs.com/package/gh-pages

gh-pages によるデプロイでは、**`gh-pages -d {デプロイするディレクトリ}`** というコマンドを発行するだけで GitHub リポジトリの `gh-pages` というブランチへ静的ページを自動的にデプロイしてくれます。

本書の例では、Vite の出力先である `dist` ディレクトリを指定します。

```sh
npx gh-pages -d dist
```

:::message
`gh-pages` をインストールする必要がある旨が表示される場合は、そのまま `Enter` キーを押すか、**`y`** をタイプしてください。

```sh
Need to install the following packages:
  gh-pages@5.0.0
Ok to proceed? (y)
```

:::

![](https://storage.googleapis.com/zenn-user-upload/3ce3dcd5e9e1-20220601.png)
_gh-pages によるデプロイ作業は、リポジトリの `Actions` タブから確認できます。_

デプロイが完了したら、デスクトップのブラウザやスマートフォンから [https://あなたのユーザ名.github.io/todo/]() を開きましょう。

:::message alert
デプロイ結果の反映には 5 分間ほどかかる場合があります。ページが表示されないときは、少し待ってページの強制再読込みを試して下さい。
:::

![](https://storage.googleapis.com/zenn-user-upload/d10c3788b474249209c86cc7.png =360x)
