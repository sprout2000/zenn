---
title: "開発環境の準備"
free: true
---

## Vite.js で React プロジェクトを作成する

フロントエンドツール [Vite.js](https://ja.vitejs.dev/)（以下、**Vite**）を利用して React + TypeScript のプロジェクトを作成します。

https://ja.vitejs.dev/

ターミナルで以下のコマンドを実行します。

```sh:zsh
npm create vite
```

:::message
`npm` コマンドは Node.js に同梱されています。
また、この画面で以下のように聞かれた場合は、**`y`** とタイプするかそのまま _Enter_ を打鍵してください。

```sh
Need to install the following packages:
  create-vite@5.0.0
Ok to proceed? (y)
```

:::

1. そのまま Enter キーを押します。

![](https://storage.googleapis.com/zenn-user-upload/231c26deb146-20230222.png =480x)
_プロジェクトに名前をつける_

2. `React` を選択します。

![](https://storage.googleapis.com/zenn-user-upload/4cf1a26942e8-20230222.png =540x)
_フレームワークの選択_

3. `TypeScript` を選択します。

![](https://storage.googleapis.com/zenn-user-upload/d86cf8414cbc-20230222.png =540x)
_JavsScript または TypeScript の選択_

```sh:zsh
% npm create vite

✔ Project name: … vite-project
✔ Select a framework: › react
✔ Select a variant: › react-ts

Scaffolding project in /Users/foo/vite-project...

Done. Now run:

  cd vite-project
  npm install
  npm run dev

```

4. **Vite** の指示に従い、さっそくこのプロジェクトを起動してみましょう。

```sh:zsh
% cd vite-project
% npm install
% npm run dev
```

![](https://storage.googleapis.com/zenn-user-upload/c47e90ddbdea-20231123.png =540x)
_Vite.js ローカルサーバーが起動_

この画面で **`o`** + **`Enter`** をタイプするか、ブラウザで [http://localhost:5173](http://localhost:5173) にアクセスすると React アプリが表示されます。

![](https://storage.googleapis.com/zenn-user-upload/768c094064d1-20230222.png =540x)

![](https://storage.googleapis.com/zenn-user-upload/8e38056dc70e-20231123.png =540x)
_**`h`** + **`Enter`** をタイプしてヘルプを表示_

:::message
この開発用ローカルサーバーを停止するには、ターミナルで **`q`** + **`Enter`** を打鍵してください。

```sh:zsh

  VITE v5.0.2  ready in 156 ms

  ➜  Local:   http://localhost:5173/
  ➜  Network: use --host to expose
  ➜  press h + enter to show help

q   # <-- "q" を入力して Enter を打鍵

% 🔲
```

:::

## React デベロッパーツールの準備

[Google Chrome](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi)、[Mozilla Firefox](https://addons.mozilla.org/en-US/firefox/addon/react-devtools/)、または [Microsoft Edge](https://microsoftedge.microsoft.com/addons/detail/react-developer-tools/gpphkfbcpidddadnkolkpfckpihlkkil) には **React Developer Tools** という拡張機能がそれぞれ用意されています。

https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi

これは、React コンポーネントの状態をブラウザの開発者ツール画面に表示してくれる拡張機能です。必ずインストールしておきましょう。

開発者ツールを表示するには、キーボードから **`Ctrl + Shift + I`** を打鍵します（※）。**`Components`** タブからコンポーネント名（ここでは **`App`**）を選択すると React コンポーネントの状態を確認できます。

![](https://storage.googleapis.com/zenn-user-upload/12823d30afb8-20230221.png =480x)

:::message
macOS 版 Chrome では `Command + Option + I` を打鍵します。
:::

## ホットリロード機能の確認

VS Code でプロジェクトフォルダを開き、`src/App.tsx` を編集してみましょう。

```diff jsx:src/App.tsx
  function App() {
    const [count, setCount] = useState(0)

    return (
      <div className="App">
        <div>
          <a href="https://vitejs.dev" target="_blank">
            <img src="/vite.svg" className="logo" alt="Vite logo" />
          </a>
          <a href="https://reactjs.org" target="_blank">
            <img src={reactLogo} className="logo react" alt="React logo" />
          </a>
        </div>
-       <h1>Vite + React</h1>
+       <h1>Todo App</h1>
        <div className="card">
          <button onClick={() => setCount((count) => count + 1)}>
            count is {count}
          </button>
          <p>
            Edit <code>src/App.tsx</code> and save to test HMR
          </p>

  // ~ snip ~
```

ファイルを保存するとブラウザ画面へ変更が自動的に反映されています。

![](https://storage.googleapis.com/zenn-user-upload/4bb0e650e8b6-20220714.png =480x)
_中央のメッセージが 'Todo App' に変わっている_

:::message alert
Vite の開発時における非バンドルという仕組み上、ホットリロードが上手く機能してくれないことが稀にあります。そういう時はブラウザのリロードボタンをクリックしてページの再読み込みを試してください。
:::

これで準備が整いました。次章からさっそく React アプリを作成していきましょう。
