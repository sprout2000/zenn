---
title: "Todo アプリを PWA 化する"
free: true
---

# プログレッシブ・ウェブアプリの作成

では、いよいよこの Todo アプリをプログレッシブ・ウェブアプリ（以下、**PWA**）にしましょう。

https://developer.mozilla.org/ja/docs/Web/Progressive_web_apps

## PWA の必須条件

ブラウザに PWA として認めてもらうには以下の条件を満たしている必要があります。

- HTTPS で提供されているウェブアプリであること
- 正しくフィールドが入力された **`manifest.json`** が HTML の `head` 内からリンクされていること
- ホームスクリーン上で適切に表示されるサイズのアイコンを持っていること
- サイトへ Service Worker が登録されていること

これらの条件を 1 つずつ見ていきます。

### HTTPS で提供されているウェブアプリ

https://developer.mozilla.org/ja/docs/Glossary/https

Web サイトを HTTPS で提供するもっとも容易な方法は、静的サイトのホスティングサービスを利用することです。静的サイトのホスティングサービスには、[GitHub Pages](https://docs.github.com/ja/pages/getting-started-with-github-pages/about-github-pages) や [Netlify](https://www.netlify.com/)、 [Heroku](https://jp.heroku.com/home) などが存在しています。

本書では、このうち GitHub Pages の利用方法をのちの章で解説します。

### `manifest.json` が HTML の head 内からリンクされている

https://developer.mozilla.org/ja/docs/Web/Manifest

`manifest.json` は [JSON](https://developer.mozilla.org/ja/docs/Glossary/JSON) 形式のファイルで、[ウェブアプリマニフェスト](https://developer.mozilla.org/ja/docs/Web/Manifest)と呼ばれているものです。

ウェブアプリマニフェストへは、アプリへそのアイコンやネイティブアプリ風の見た目を与えるために必要な情報を JSON 形式で記述します。

```json:manifest.json の例
{
  "name": "Todo App (PWA)",
  "theme_color": "#000000",
  "background_color": "#ffffff",
  "icons": [
    {
      "sizes": "192x192",
      "src": "icon-192x192.png",
      "type": "image/png"
    }
  ]
}
```

本書では、**Vite** の[プラグイン](https://ja.vitejs.dev/plugins/)が提供してくれる機能を利用することにより、この `manifest.json` が自動生成されるよう次項で設定します。

https://ja.vitejs.dev/plugins/

### ホームスクリーン上で適切に表示されるサイズのアイコン

ファビコンの他に、最低でも以下の 2 つのサイズの画像を用意することが必要です。

- 192 x 192 ピクセルのアイコン
- 512 x 512 ピクセルのアイコン

[Favicon Generator](https://realfavicongenerator.net/) のような Web サービスや [pwa-asset-generator](https://github.com/onderceylan/pwa-asset-generator) のような NPM ライブラリを利用して必要な画像を一括生成すると良いでしょう。

https://realfavicongenerator.net/

https://www.npmjs.com/package/pwa-asset-generator

また、マスカブルアイコンの生成には以下のような Web サービスを利用しましょう。

https://maskable.app/editor

マスカブルアイコンは、PWA をホーム画面へ追加する際に、文字化けせずに画像が全体を埋め尽くすことを保証するものです。 いまのところは PWA の必須条件ではないものの、推奨事項のひとつとされています。

https://web.dev/maskable-icon/

これらのアイコンは、前項で見たとおり **`manifest.json` 内からそのファイルパスを指定する**ことで有効となります。

:::message
本書では以下の 3 つのアイコンを用意しました。
画像を右クリック →「名前を付けて画像を保存...」でダウンロードできます。
:::

![](https://storage.googleapis.com/zenn-user-upload/1b9f1618bb85-20230207.png =64x)
_アイコン 192x192 ピクセル_

![](https://storage.googleapis.com/zenn-user-upload/44a6329c15bf-20230207.png =86x)
_アイコン 512x512 ピクセル_

![](https://storage.googleapis.com/zenn-user-upload/a527d50937f8-20230207.png =86x)
_マスカブルアイコン 512x512 ピクセル_

[Vite.js](https://ja.vitejs.dev/config/#publicdir) では、上のようなアセットファイルはプロジェクト直下の `public` フォルダへ配置しておけばビルド時に自動的に出力フォルダへコピーされます。

https://ja.vitejs.dev/config/#publicdir

```sh
% tree public
public
├── icon-192x192.png
├── icon-512x512-mask.png
├── icon-512x512.png
└── vite.svg

0 directories, 4 files
```

### Service Worker が登録されていること

https://developers.google.com/web/fundamentals/primers/service-workers?hl=ja

Service Worker はブラウザによって Web ページとは別にバックグラウンドで実行されるスクリプトです。これによりプッシュ通知やバックグラウンド同期、アプリのオフライン利用などの機能をユーザーに提供します。

本書では、上記 `manifest.json` と同様に **Vite** のプラグインによって自動生成される方法を採用します。

## Vite での PWA 化

### プラグインのインストール

**Vite** には、[vite-plugin-pwa](https://vite-plugin-pwa.netlify.app/) という便利な PWA 化プラグインが存在しています。まずはこれをプロジェクトへ追加しましょう。

https://www.npmjs.com/package/vite-plugin-pwa

```sh:zsh
npm i -D vite-plugin-pwa
```

:::message
`npm i -D` は、`npm install --save-dev` の短縮形です。
:::

### プラグインの有効化

このプラグインを有効化するため、**Vite** の設定ファイル `vite.config.ts` を編集します。

```typescript:vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

// VitePWA をインポート
import { VitePWA } from 'vite-plugin-pwa';

export default defineConfig({
  // plugins 配列に追加
  plugins: [react(), VitePWA()],
});
```

:::message alert
サーバーサイドでの PWA の更新をアプリの起動時に即時に適用（キャッシュを更新）させたい場合は、**`registerType: 'autoUpdate'`** オプションを追加してください。

```typescript
export default defineConfig({
  plugins: [react(), VitePWA({ registerType: "autoUpdate" })],
});
```

:::

### `manifest.json` を生成する

また、前項で見た `manifest.json` を生成するための `manifest` オプションも設定します。

```typescript:vite.config.ts
export default defineConfig({
  plugins: [
    react(),
    VitePWA({
      manifest: {
        // ユーザーに通常表示されるアプリ名
        name: 'Todo App (PWA)',
        // name を表示するのに十分なスペースがない場合に表示されるアプリ名
        short_name: 'Todo',
        // アプリの詳細な説明
        description: 'Todo プログレッシブ・ウェブアプリ',
        /**
         * アプリの開始 URL:
         * 通常はサーブするディレクトリそのもの
         */
        start_url: '.',
        /**
         * 表示モード:
         * fullscreen: フルスクリーン
         * standalone: 単独のアプリのようになる
         * minimal-ui: 最小限のブラウザ UI は残る
         * browser: 通常のブラウザ
         */
        display: "standalone",
        /**
         * アプリの向き:
         * portrait: 縦向き
         * landscape: 横向き
         * any: 向きを強制しない
         */
        orientation: "portrait",
        // 既定のテーマカラー
        theme_color: '#3f51b2',
        // スタイルシートが読み込まれる前に表示するアプリページの背景色
        background_color: "#efeff4",
        /**
         * favicon やアプリアイコンの配列:
         * 最低でも 192x192ビクセルと512x512ビクセルの 2 つのアプリアイコンが必要
         */
        icons: [
          {
            src: 'icon-192x192.png',
            sizes: '192x192',
            type: 'image/png',
          },
          {
            src: 'icon-512x512.png',
            sizes: '512x512',
            type: 'image/png',
          },
          {
            src: 'icon-512x512-mask.png',
            sizes: '512x512',
            type: 'image/png',
            // 用途をマスカブルアイコンとする
            purpose: 'maskable',
          },
        ],
      },
    }),
  ],
});
```

### サービスワーカーを登録する

このプロジェクトのエントリファイル (= `src/man.tsx`) へサービスワーカーを登録するメソッドをインポートします。

```diff jsx:src/main.tsx
+ import { registerSW } from 'virtual:pwa-register';

  ReactDOM.createRoot(document.getElementById('root')!).render(
    <React.StrictMode>
      <App />
    </React.StrictMode>
  );
```

ただし、これだけだと型宣言なしのエラーになってしまうため、`tsconfig.json` に型定義を追記します。

![](https://storage.googleapis.com/zenn-user-upload/d197527b9feca0df1003ca19.png)

```json:tsconfig.json
{
  "compilerOptions": {
    // ~ snip ~
  },
  // "node_modules/vite-plugin-pwa/client.d.ts" を追加
  "include": ["./src", "node_modules/vite-plugin-pwa/client.d.ts"]
}
```

エントリファイルでサービスワーカーを登録します。

```diff jsx:src/main.tsx
  import { registerSW } from 'virtual:pwa-register';

  ReactDOM.createRoot(document.getElementById('root')!).render(
    <React.StrictMode>
      <App />
    </React.StrictMode>
  );

  // サービスワーカーの登録
+ registerSW();
```

### 本番ビルドと出力ファイルなどの確認

では、さっそくこの Todo アプリを PWA としてビルドしてみましょう。

```sh:zsh
% npm run build

> tsc && vite build
vite v2.6.7 building for production...
✓ 32 modules transformed.
dist/assets/favicon.17e50649.svg   1.49 KiB
dist/index.html                    0.51 KiB
dist/manifest.webmanifest          0.41 KiB
dist/assets/index.ff720c7c.js      1.66 KiB / gzip: 0.89 KiB
dist/assets/vendor.af5bf2b7.js     134.64 KiB / gzip: 43.72 KiB

PWA v0.11.3
mode      generateSW
precache  4 entries (136.80 KiB)
files generated
  dist/sw.js
  dist/workbox-afb9f189.js
✨  Done in 9.35s.
```

`dist` ディレクトリ以下へ出力されたファイルを確認します。

```sh:zsh
% tree dist
dist
├── assets
│   ├── index.233b4b52.js
│   └── workbox-window.prod.es5.d2780aeb.js
├── icon-192x192.png
├── icon-512x512-mask.png
├── icon-512x512.png
├── index.html
├── manifest.webmanifest
├── sw.js
├── vite.svg
└── workbox-7369c0e1.js

1 directory, 10 files
```

`public` フォルダへ配置しておいたアイコンも `dist` へコピーされています。

## より iOS に適合させる（オプション）

実は、この `dist` ディレクトリに出力された `index.html` では、iOS をはじめとした Apple デバイスに十全には対応できません。

これらにより適合させるには、ルートディレクトリの `index.html` のヘッダへ以下の 4 行を付け加え、**再ビルド**する必要があります。また、180x180 ピクセルの `apple-touch-icon` を `public` フォルダへ配置しておく必要もあります。

![](https://storage.googleapis.com/zenn-user-upload/062a8ebffdca-20230514.png =180x)
_apple-touch-icon.png_

```html:index.html
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-title" content="TODO" />
<meta name="apple-mobile-web-app-status-bar-style" content="default">

<link rel="apple-touch-icon" href="apple-touch-icon.png" />
```

**apple-mobile-web-app-status-bar-style** の値は、以下の 3 つから選択します。

|        値         | 効果                                                                              |
| :---------------: | :-------------------------------------------------------------------------------- |
|      default      | 黒いテキストと記号が付いた白いステータスバー                                      |
|       black       | 黒いステータスバー                                                                |
| black-translucent | 白いテキストと記号になり、ステータスバーは Web アプリの本体要素と同じ背景色になる |

## 完成

これで Todo アプリの PWA が完成しました。
**Vite** には、ビルド後のアプリをプレビューするスクリプトも用意されています。これを利用して動作確認を行いましょう。

```sh:zsh
% npm run preview
```

![](https://storage.googleapis.com/zenn-user-upload/cb31bc588379-20230222.png)

次章では、このアプリを [GitHub Pages](https://docs.github.com/ja/pages/getting-started-with-github-pages/about-github-pages) へとデプロイします。

## 本章のソースコード全文

:::details tsconfig.json

```json:tsconfig.json
{
  "compilerOptions": {
    "target": "ESNext",
    "useDefineForClassFields": true,
    "lib": ["DOM", "DOM.Iterable", "ESNext"],
    "allowJs": false,
    "skipLibCheck": true,
    "esModuleInterop": false,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "module": "ESNext",
    "moduleResolution": "Node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx"
  },
  "include": ["src", "node_modules/vite-plugin-pwa/client.d.ts"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

:::

:::details main.tsx

```tsx:src/main.tsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import { registerSW } from 'virtual:pwa-register';

import { App } from './App';

createRoot(document.getElementById('root') as Element).render(
  <StrictMode>
    <App />
  </StrictMode>
);

registerSW();
```

:::

:::details vite.config.ts

```ts:vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { VitePWA } from 'vite-plugin-pwa';

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [
    react(),
    VitePWA({
      manifest: {
        name: 'Todo App (PWA)',
        short_name: 'Todo',
        description: 'Todo プログレッシブ・ウェブアプリ',
        start_url: '.',
        display: 'standalone',
        orientation: 'portrait',
        theme_color: '#3f51b2',
        background_color: '#efeff4',
        icons: [
          {
            src: 'icon-192x192.png',
            sizes: '192x192',
            type: 'image/png',
          },
          {
            src: 'icon-512x512.png',
            sizes: '512x512',
            type: 'image/png',
          },
          {
            src: 'icon-512x512-mask.png',
            sizes: '512x512',
            type: 'image/png',
            purpose: 'maskable',
          },
        ],
      },
    }),
  ],
});
```

:::

:::details index.html

```html:index.html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/vite.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Todo</title>
    <meta name="theme-color" content="#3F51B2" />
    <meta name="description" content="Todo App PWA" />
    <meta name="apple-mobile-web-app-capable" content="yes" />
    <meta name="apple-mobile-web-app-title" content="TODO" />
    <meta name="apple-mobile-web-app-status-bar-style" content="default" />
    <link rel="apple-touch-icon" href="/apple-touch-icon.png" />
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

:::
