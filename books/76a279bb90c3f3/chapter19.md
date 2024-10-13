---
title: "UI フレームワーク Material UI を導入する"
free: true
---

# UI フレームワーク MUI

本書では、数多ある React 向け UI フレームワークの中でも一番人気と言われている [Material UI](https://mui.com/)（以下、**MUI**）を UI フレームワークとして採用します。

https://qiita.com/yudwig/items/8d8c29467ceb76a91be9

https://mui.com/

公式の[インストールガイド](https://mui.com/getting-started/installation/)にしたがって NPM パッケージをインストールしていきます。

https://mui.com/getting-started/installation/

## 本体とスタイルエンジン

```sh:zsh
npm install @mui/material @emotion/react @emotion/styled
```

スタイルエンジンには、**material-ui v4.x** までの [styled-components](https://github.com/styled-components/styled-components) も利用可能です。その場合には、[公式のガイド](https://mui.com/guides/styled-engine/)を参照してください。

https://mui.com/guides/styled-engine/

## Roboto フォント

プロジェクトフォルダ直下の `index.html` へ以下の行を追加して [Roboto フォント](https://fonts.google.com/specimen/Roboto)を CDN 経由で読み込みます。

```html:index.html
<link
  rel="stylesheet"
  href="https://fonts.googleapis.com/css?family=Roboto:300,400,500,700&display=swap"
/>
```

## アイコンフォント

同じく `index.html` へ以下の行を追加して、[Google Material フォントアイコン](https://fonts.google.com/icons?icon.set=Material+Icons)も CDN 経由で読み込みます。

```html:index.html
<link
  rel="stylesheet"
  href="https://fonts.googleapis.com/icon?family=Material+Icons"
/>
```

使い方は、`@mui/material/Icon` をインポートしてコンポーネント内で `<Icon> ~ </Icon>` タグ内にフォントアイコン名を指定するだけです。

```jsx:例
import Icon from "@mui/material/Icon";

<Icon>menu</Icon>;
```

使用するフォントアイコンの名前は[こちら](https://fonts.google.com/icons?icon.set=Material+Icons)の一覧から検索できます。

https://fonts.google.com/icons?icon.set=Material+Icons

## この章のソースコード全文

```html:index.html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/vite.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Todo</title>
    <link
      rel="stylesheet"
      href="https://fonts.googleapis.com/css?family=Roboto:300,400,500,700&display=swap"
    />
    <link
      rel="stylesheet"
      href="https://fonts.googleapis.com/icon?family=Material+Icons"
    />
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```
