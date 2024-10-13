---
title: "（補遺）なぜステートから作った値を直接渡すべきでないのか？"
free: true
---

# 予期せぬ挙動とは？

第 7 章では次の 2 つの注意点を挙げました。

- **更新後のステートが更新前のステートの値に依存している場合**には、`setState メソッド` には**値ではなく関数**を渡すべきである
- これにより、**`setState` メソッドにステートから作った値を直接渡すことで生じる予期せぬ挙動**を防ぐことができる

この「予期せぬ挙動」とは、一体どのようなものでしょうか？

## よくあるカウンターの例

次のようなボタンがクリックされるたびにカウントが `1` ずつ増えていくコンポーネントを考えてみましょう。

```tsx
function App() {
  const [count, setCount] = useState(0);

  const countUp = () => {
    setCount(count + 1); // ステートから作った値を直接渡している
  };

  return (
    <div>
      <h1>{count}</h1>
      <button onClick={countUp}>Count Up</button>
    </div>
  );
}
```

![](https://storage.googleapis.com/zenn-user-upload/17147284747d-20230323.gif)

ここでカウントを `1` ずつではなく、`2` ずつ増やしたいと考えて、`coutUp` 関数を二度実行する `doubleCountUp` 関数を作成し、これをイベントリスナーに渡した場合はどうなるでしょうか？

```tsx
function App() {
  const [count, setCount] = useState(0);

  const countUp = () => {
    setCount(count + 1);
  };

  const doubleCountUp = () => {
    // setState する関数を二度実行
    countUp();
    countUp();
  };

  return (
    <div>
      <h1>{count}</h1>
      {/* onClick イベントに上の関数を渡す */}
      <button onClick={doubleCountUp}>Count Up</button>
    </div>
  );
}
```

カウントは変わらず `1` ずつしか増えません。なぜでしょうか？

![](https://storage.googleapis.com/zenn-user-upload/17147284747d-20230323.gif)
_何も変わっていない..._

## イベントハンドラー内でのステートの値

なぜなら、React では**レンダリングごとにステート変数の値は固定されている**からです。イベントハンドラー内で何度 `setState メソッド`を呼び出しても、対象となるステート変数 (`count`) は **`0`** のまま変わりません。

```js
setCount(0 + 1); // 両方とも `1` を渡しているだけになる
setCount(0 + 1); // <-- `1 + 1` とはならない
```

そして、React は**イベントハンドラの中のコードがすべて実行されるのを待ったうえ**で、状態を更新、つまりコンポーネントを再レンダリングします。これにより React は無駄なレンダリングを省いてアプリケーションの実行を速めます。

つまり、あくまでも `count` ステート変数は、**一連の処理が完了した後**のステートの値を保持しています。そして、イベントハンドラー内部での計算途中に `count` ステートの値が変化することはありません。

## ステートの関数更新

しかし、更新後のステートが更新前のステートの値に依存している場合、その更新方法を関数として指示すれば（関数更新といいます）、ステートの値は期待値通りとなります。
つまり、`setState メソッド` に**書き替える値をそのまま渡すのでなく、ステートの値をどう更新すべきかを指示する更新関数** を渡すのです。

```jsx:例
setState((prev) => prev + 1);
```

これにより、React はこれらの更新関数をキュー（※）に入れ、イベントハンドラーのコードがすべて処理された**あと**で呼び出して処理します。そして、つぎのレンダリング時には最終的に更新された状態が返ります。

```js
// キューに入っている更新関数を **あとで** 処理
setCount((0) => 0 + 1);  // --> 1
setCount((1) => 1 + 1);  // --> 2
```

:::message
ここでは、関数実行のための順番待ちリストのようなものと考えてください。
:::

例文コードの `setState メソッド` の引数へ、ステートから作った値ではなく、更新関数を渡すようにアップデートしてみましょう。

```diff jsx
  function App() {
    const [count, setCount] = useState(0);

    const countUp = () => {
      // 引数に更新方法を指示する関数を与える
+     setCount((count) => count + 1);
    };

    const doubleCountUp = () => {
      // setState する関数を二度実行
      countUp();
      countUp();
    };

    return (
      <div>
        <h1>{count}</h1>
        <button onClick={doubleCountUp}>Count Up</button>
      </div>
    );
  }
```

![](https://storage.googleapis.com/zenn-user-upload/c79031cd7b7b-20230323.gif)
_カウントが `2` ずつ増えている_

## 参考記事

https://ja.react.dev/learn/queueing-a-series-of-state-updates
