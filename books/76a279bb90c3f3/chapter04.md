---
title: "フォームに入力された文字列を状態 (=state) として保持する"
free: true
---

# useState フックの導入

フォームに入力された文字列を **状態 (=ステート)** として保持するため、`useState` フックを導入します。
React がコンポーネントのステートに応じて DOM をリアクティブに更新する様子をチェックしましょう。

## useState フックの構文

React では、コンポーネントに状態を「記憶」させるための `useState` という特別な関数が用意されています。

```jsx:構文
const [text, setText] = React.useState('hello');
```

- **`useState`:**
  - 引数となるのは**ステートの初期値** (= `'hello'`) です
  - 現在のステート `text` と、それを更新するための関数 `setText` とをペアにした配列を返します
- **`text`**: 現在のステートの値が格納された変数です
- **`setText`**: ステートを更新するメソッドです
  - `'set' + ステート`（[ロワー・キャメルケース](https://qiita.com/deerboy/items/f035b9044edf9a51aff7)）とするのが通例です
  - `setText('bye')` のようにしてステートを更新します

https://ja.react.dev/reference/react/useState

:::message
これ以降、本書では `setText('bye')` のようなステートを更新する関数のことを総称して **`setState` メソッド**と呼びます。
:::

## text ステートを作成

App コンポーネントへ `text` ステートの値を「記憶」させ、その値に応じてリアクティブに DOM を書き換えましょう。

```jsx:src/App.tsx
// React から useState フックをインポート
import { useState } from 'react';

export const App = () => {
  // 初期値: 空文字列 ''
  const [text, setText] = useState('');

  return (
    <div>
      <form onSubmit={(e) => e.preventDefault()}>
        <input
          type="text"
          // text ステートが持っている入力中テキストの値を value として表示
          value={text}
          // onChange イベント（＝入力テキストの変化）を text ステートに反映する
          onChange={(e) => setText(e.target.value)}
        />
        <input type="submit" />  {/* ← 省略 */}
      </form>

      {/* ↓ DOM のリアクティブな反応を見るためのサンプル */}
      <p>{text}</p>
      {/* ↑ あとで削除 */}
    </div>
  );
};
```

:::message
JSX 文の中で **JavaScript 変数の値**を展開するには、中カッコ **`{}`** で囲んであげる必要があります。上のコードの例では、以下のような部分がこれに該当します。

```jsx
<input value={text} />
```

```jsx
<p>{text}</p>
```

もしもクオート（**`''`** や **`""`**）で囲ったり、そのまま **`text`** と書いてしまうと、通常の文字列 **`"text"`** として評価されてしまうため用をなしません。

:::

![](https://storage.googleapis.com/zenn-user-upload/ca8abbfb2db8-20230221.gif =480x)
_DOM がリアクティブに変化_

![](https://storage.googleapis.com/zenn-user-upload/b096a28b8a44-20230221.png =480x)
_開発者ツールの Components タブに State が表示されている_

ステートの値が更新されると、React はそのステートを持つコンポーネントとその子コンポーネント（※）を再レンダリングします。

:::message
本書の前半部分では、まだ子コンポーネントは存在しません。
:::

言い換えると、**`setState` メソッドの実行は、コンポーネントの再レンダリングをトリガー**します。

![](https://storage.googleapis.com/zenn-user-upload/80e0a660970f-20230221.gif =540x)
_再レンダリングされる部分のハイライト_

## ステート更新の原則

React では、原則として **`setState` メソッドを利用しないでステートの値を書き換えてはいけません**。

```ts:例
const [jotai, setJotai] = useState('いまの状態');

// ❌ やってはいけない
jotai = 'あたらしい状態';

// 🟢 OK!
setJotai('あたらしい状態');
```

その理由は、上で述べた通りコンポーネントの再レンダリングが正しくトリガーされないからですが、他にも**ステートのイミュータビリティ**が保てなくなるからという事情もあります。
ステートのイミュータビリティについては、のちの章で解説します。
