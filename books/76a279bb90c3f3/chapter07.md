---
title: "イベントを処理する関数を作成する"
free: true
---

## コールバック関数の作成

それでは、`todos` ステートを更新、つまり新しいタスクの追加をしていきましょう。
イベントを処理する関数（＝イベントハンドラー）を作成し、それを実行することでステートを更新します。

![](https://storage.googleapis.com/zenn-user-upload/7d509bdf6f79-20230320.png =480x)

```jsx:src/App.tsx
  const [todos, setTodos] = useState<Todo[]>([]);

  // todos ステートを更新する関数
  const handleSubmit = () => {
    // 何も入力されていなかったらリターン
    if (!text) return;

    // 新しい Todo を作成
    // 明示的に型注釈を付けてオブジェクトの型を限定する
    const newTodo: Todo = {
      // text ステートの値を value プロパティへ
      value: text,
    };

    /**
     * 更新前の todos ステートを元に
     * スプレッド構文で展開した要素へ
     * newTodo を加えた新しい配列でステートを更新
     **/
    setTodos((todos) => [newTodo, ...todos]);
    // フォームへの入力をクリアする
    setText('');
  };
```

[スプレッド構文](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Spread_syntax) **`...todos`** は、元の `todos` 配列の「**すべての要素を列挙する**」ということを意味します。つまり、**`[]`** の先頭へ `newTodo` を追加した**新たな配列**を作成していることになります。

https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Spread_syntax

また、ここでは `setState` メソッドの引数へ関数を与えています。
じつは、`setState` メソッドの引数には、値だけでなく関数を渡すこともできます。そして、その関数は**更新前のステートを引数にとり、新しいステートを返す関数**となります。

```js:例
// 更新前のステートの値を元に新ステートを生成
setState((prev) => prev + 1);
```

このように**更新後のステートが更新前のステートの値に依存**している場合には、`setState` メソッドには**値ではなく関数を渡すべき**です。

ここでの詳説は避けますが、こうすることで **`setState` メソッドにステートから作った値を直接渡すことで生じる予期せぬ挙動**を防ぐことができます _（[（補遺）なぜステートから作った値を直接渡すべきでないのか？](https://zenn.dev/sprout2000/books/76a279bb90c3f3/viewer/appendix01)を参照）_。

```js:例
const [state, setState] = useState(0);

// ❌ No! ステートから作った値を直接渡す
setState(state + 1);

// 🟢 OK! 「更新前」 のステートの値を元に新ステートを生成
setState((state) => state + 1);
```

https://zenn.dev/sprout2000/books/76a279bb90c3f3/viewer/appendix01

## コールバック関数をイベントに割り当てる

上の関数を `onSubmit` イベント（イベントリスナー）へ紐付けましょう。
このようなイベントリスナーに渡す関数のことを**コールバック関数**といいます。

:::message
コールバック関数は、広義で言うと「他の関数に引数として渡す関数」のことですが、本書ではコンポーネント内で発火するイベントを処理する関数のことだと考えていれば十分です。
:::

**注意点:**

- イベントリスナーに渡す関数は、 **アロー関数の `() => hoge()`** もしくは **引数なしの `hoge` そのもの**です _（[（補遺）なぜコールバック関数をカッコ () 付きで渡すとダメなのか？](https://zenn.dev/sprout2000/books/76a279bb90c3f3/viewer/appendix02)を参照）_
- `hoge()` とのみ記述すると**即時実行**されてしまう、つまり**コンポーネントの再レンダリングをトリガー**してしまうため、React アプリケーションは**無限ループ**に陥ってしまいます
- あくまでも **`hoge`** 関数を実行するのは、ユーザーの操作があったときです

https://zenn.dev/sprout2000/books/76a279bb90c3f3/viewer/appendix02

なお、`<form>` タグの中でいったん `e.preventDefault()` しているのは **Enter キー打鍵でページそのものがリロードされてしまう**のを防ぐためです。

```jsx:src/App.tsx
  return (
    <div>
     {/* コールバックとして () => handleSubmit() を渡す */}
      <form
        onSubmit={(e) => {
          e.preventDefault();
          handleSubmit();
        }}
      >
        <input
          type="text"
          value={text}
          onChange={(e) => setText(e.target.value)}
        />
        {/* 上に同じ */}
        <input type="submit" value="追加" onSubmit={handleSubmit} />
     </form>
   </div>
```

`onSubmit` イベントが発火すると `handleSubmit` 関数が実行され、新しいタスクが追加されます。
フォームへ入力して submit（Enter キー打鍵）すれば、開発者ツールで **`todos ステート`** が更新されていることを確認できます。

![](https://storage.googleapis.com/zenn-user-upload/609c50a0042a-20230221.png)
_text ステートと todos ステートの様子_

## `text` ステート向けの関数も用意する

上の例で要領を得ましたので、**`text ステート`** についても JSX の中で直接 `setText` していた部分を関数 **`handleChange()`** として書き出しましょう。

関数として書き出すことで、のちにコンポーネントを分割する段階で _props_（後述）の受け渡しが容易になったことに気が付くでしょう。

```jsx:src/App.tsx
  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    setText(e.target.value);
  };
```

```jsx:src/App.tsx
<input type="text" value={text} onChange={(e) => handleChange(e)} />
```

## イベントの型を調べる

イベントの型がわからない時は、[VS Code](https://code.visualstudio.com/) であればイベント上でマウスカーソルをホバーさせるとポップアップが表示されます。

![](https://storage.googleapis.com/zenn-user-upload/da459fbbbc8a-20230205.png)

## この章のソースコード全文

:::details App.tsx

```tsx:App.tsx
import { useState } from 'react';

type Todo = {
  value: string;
};

export const App = () => {
  const [text, setText] = useState('');
  const [todos, setTodos] = useState<Todo[]>([]);

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    setText(e.target.value);
  };

  const handleSubmit = () => {
    if (!text) return;

    const newTodo: Todo = {
      value: text,
    };

    setTodos((todos) => [newTodo, ...todos]);
    setText('');
  };

  return (
    <div>
      <form
        onSubmit={(e) => {
          e.preventDefault();
          handleSubmit();
        }}
      >
        <input type="text" value={text} onChange={(e) => handleChange(e)} />
        <input type="submit" value="追加" onSubmit={handleSubmit} />
      </form>
    </div>
  );
};
```

:::
