---
title: "React コンポーネントの作成"
free: true
---

# 関数コンポーネントの作成

todo（タスク）を入力するためのフォームを持った、関数コンポーネントを作成するところから始めましょう。

関数コンポーネントは、React コンポーネントの一種であり、JavaScript 関数を使って UI コンポーネントを定義する方法を提供します。UI は JSX (JavaScript XML) を使用して記述され、その JSX を返すことで、React は UI をレンダリングします。

```jsx:例
// Hello コンポーネント
function Hello() {
  return <h1>Hello.</h1>;
}
```

![](https://storage.googleapis.com/zenn-user-upload/dbbc71ff9470-20230317.png =360x)
_Hello コンポーネント_

## 最初のコード

`src` ディレクトリ内にある `main.tsx` と `App.tsx` そして `index.css` の 3 つのファイルを、以下のコードで置き換えてください。

```jsx:src/main.tsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";

import { App } from "./App";
import "./index.css";

const root = createRoot(document.getElementById("root") as Element);

root.render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

```jsx:src/App.tsx
export const App = () => {
  return (
    <div>
      <form onSubmit={(e) => e.preventDefault()}>
        <input type="text" value="" onChange={(e) => e.preventDefault()} />
        <input
          type="submit"
          value="追加"
          onSubmit={(e) => e.preventDefault()}
        />
      </form>
    </div>
  );
};
```

```css:src/index.css
#root {
  margin: 0;
  padding: 0;
}

select {
  margin: 0.5em;
  padding: 0.5em;
}

input[type='text'] {
  width: 50vw;
  padding: 0.5em;
  margin: 0 0.5em;
}

input[type='submit'] {
  padding: 0.1em 0.5em;
}

ul {
  list-style: none;
  margin-left: 0.5em;
  padding-left: 0;
}

li {
  margin-bottom: 0.5em;
}
```

![](https://storage.googleapis.com/zenn-user-upload/e15516ca18f3-20230222.png =480x)
_App コンポーネント_

## main.tsx

### `createRoot` メソッドと `render` メソッド

上段の `main.tsx` では、[createRoot](https://ja.react.dev/reference/react-dom/client/createRoot) メソッドを使って、`index.html` 内の `<div id="root">` という要素を取得し、それを React ルートとしています。

```html:index.html
<body>
  <div id="root"></div>
  <!-- 省略 -->
</body>
```

```tsx:src/main.tsx
const root = createRoot(document.getElementById('root') as Element);
```

そして、その React ルートが持つ [render](https://ja.react.dev/reference/react-dom/client/createRoot) メソッドは、DOM 内部へ React 要素である **`<App />` コンポーネント**をレンダリングします。

```tsx:src/main.tsx
root.render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

:::message
**`<App />`** コンポーネントを囲っている **`<StrictMode>`** タグ（[Strict モード](https://ja.react.dev/reference/react/StrictMode)）は、目に見える UI を描画しません。これは、その内部にあるコンポーネントの検査をしてくれるものですが、現時点ではとくに気にする必要はないでしょう。
また、Strict モードでの検査は開発モードでのみ動きます。**本番ビルドには影響を与えません。**
:::

https://ja.react.dev/reference/react-dom/client

### `as Element`

`createRoot` の引数内では、取得した HTML 要素を **`as Element`** として[型アサーション](https://typescriptbook.jp/reference/values-types-variables/type-assertion-as)しています。型アサーションとは、型推論を上書きして別の型を割り当てる機能です。

```tsx
// Element 型としてアサーション
createRoot(document.getElementById("root") as Element);
```

https://typescriptbook.jp/reference/values-types-variables/type-assertion-as

もしも、この `as Element` が無いとどうなるでしょうか？

![](https://storage.googleapis.com/zenn-user-upload/1ce95b8a52b4-20230220.png)

> 型 'HTMLElement | null' の引数を型 'Element | DocumentFragment' のパラメーターに割り当てることはできません。

React アプリをマウントする HTML ファイルに `root` という _id_ を持った要素が存在しない _(= null)_ という事態は十分にあり得ます。
そのため、TypeScript の世界では `document.getElementById` から得られる値は、常に HTMLElement 型または Null 型 となります。

`createRoot` は、その引数に Element 型もしくは DocumentFragment 型の値しか受け取れないので、**_null_ となる可能性を持つ値**を渡すことは出来ません。そのため、ここでは Element 型へと型アサーションしているのです。

別解として、_null_ ではないことを保証する **Non-null アサーション演算子**: **`!`** を利用する方法もあります。

```tsx:Non-null アサーションの例
// "!" 演算子で null でないことを保証する
createRoot(document.getElementById('root')!);
```

:::message
`document.getElementById` が返す [HTMLElement 型](https://developer.mozilla.org/ja/docs/Web/API/HTMLElement)は、Element 型からプロパティを[継承](https://developer.mozilla.org/ja/docs/Web/JavaScript/Inheritance_and_the_prototype_chain)した型であるため、Non-null アサーションによって（Element 型の）引数として与えることが可能となります。
:::

https://developer.mozilla.org/ja/docs/Web/JavaScript/Inheritance_and_the_prototype_chain

## `App.tsx`

App コンポーネント内では、[JSX](https://ja.react.dev/learn/writing-markup-with-jsx) で作成した Todo 入力フォームを _return_ しています。

```jsx:src/App.tsx
export const App = () => {
  return (
    <div>
      <form onSubmit={(e) => e.preventDefault()}>
        {/* 省略 */}
      </form>
    </div>
  );
};
```

:::message
JSX 文が複数行に渡る場合、全体をカッコ **`()`** で囲んで _return_ する必要があります。カッコ **`()`** がないと _return_ 文は改行で終了とみなされるからです。

また、コンポーネントの名前は必ず**大文字**（[アッパー・キャメルケース](http://ja.wikipedia.org/w/index.php?curid=410559)）ではじまっている必要があります。これにより React は、通常の `<div>` などの HTML タグとの違いを判別できるようになります。
:::

https://ja.react.dev/learn/writing-markup-with-jsx

JSX (JavaScript XML) は、JavaScript 内で HTML に似た文法で UI を直感的に組み立てることを可能とします。
この時点での JSX に関する注意点は以下の 2 つです。

1. 必ず**単一の** JSX 要素（上の例では **`<div>~</div>`**）を _return_ する必要があります。
   以下のような例ではエラーとなってしまいます。

```jsx:例
// ❌ ダメな例
const Example = () => {
  // 単一の JSX 要素でないためエラー
  return (
    <p>最初の行</p>
    <p>次の行</p>
  )
}
```

JSX 要素を単一にラップするためだけに **`<div>`** 要素などを使いたくない場合は、代わりに [JSX フラグメント](https://ja.react.dev/reference/react/Fragment) **`<> ~ </>`** を使って JSX 文を囲ってください。

```jsx:例
// 🟢 OK!
const Example = () => {
  return (
    <>
      <p>最初の行</p>
      <p>次の行</p>
    </>
  )
}
```

2. JSX 内での `onsubmit` イベントは、**`onSubmit`** （[ロワー・キャメルケース](https://qiita.com/deerboy/items/f035b9044edf9a51aff7)）として記述する必要があります。
   これは、その他の **`onClick`** や **`onChange`** といったイベントリスナーでも同様です。

```jsx:例
<input type="text" value="" onChange={(e) => e.preventDefault()} />
```

## これから

以降、この関数コンポーネント **`App`** をベースとして **Todo アプリ**を作成していきます。

いまのところ `onSubmit` や `onChange` などのイベントリスナーでは、イベントが発生しても [preventDefault()](https://developer.mozilla.org/ja/docs/Web/API/Event/preventDefault) してしまっているので特に何も起きません。

https://developer.mozilla.org/ja/docs/Web/API/Event/preventDefault
