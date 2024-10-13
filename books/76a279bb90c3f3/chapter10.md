---
title: "登録済みの todo を編集可能にする"
free: true
---

## 各 `todo` を入力フォーム化する

すでに登録済みの todo を編集可能にするにするため、`<li> ~ </li>` タグ内で展開される各項目の `todo.value` を `<input />` タグでラップします。

```jsx:src/App.tsx
  <ul>
    {todos.map((todo) => {
      return (
        <li key={todo.id}>
          <input
            type="text"
            value={todo.value}
            onChange={(e) => e.preventDefault()}
          />
        </li>
      );
    })}
  </ul>
```

ここでも `onChange` イベントではとりあえず `e.preventDefault()` しているので入力しても何の変化も起きません。

![](https://storage.googleapis.com/zenn-user-upload/3385e0cf909f-20230222.png =480x)

## 登録済み todo が編集された時のコールバック関数を作成する

編集されたタスクの内容を適用し、そのタスクを古いものから新しいものへ入れ替えた状態に `todos ステート` を更新しなければいけません。

ステートを更新する関数には以下の要件が求められます。

- どの `todo` が編集されたのか特定するため、その `todo` の `id` プロパティを引数として受け取る
- `e.target.value`（onChange イベントの結果）を書き換え後の `todo.value` の値とするために第 2 引数として受け取る
- 編集後の `todo` を含む `Todo型`の配列 で `todos ステート` を更新する

```jsx:src/App.tsx
  const handleEdit = (id: number, value: string) => {
    setTodos((todos) => {
      /**
       * 引数として渡された todo の id が一致する
       * 更新前の todos ステート内の todo の
       * value プロパティを引数 value (= e.target.value) に書き換える
       */
      const newTodos = todos.map((todo) => {
        if (todo.id === id) {
          todo.value = value;
        }
        return todo;
      });

      // todos ステートを更新
      return newTodos;
    });
  };
```

前述の通り、[Array.prototype.map()](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/map) は、与えられた関数を配列のすべての要素に対して呼び出し、その結果からなる**新しい配列を生成**する非破壊的メソッドです。

https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/map

上のコールバック関数を `<input onChange={} />` イベントに紐付けます。

```jsx:src/App.tsx
  <ul>
    {todos.map((todo) => {
      return (
        <li key={todo.id}>
          <input
            type="text"
            value={todo.value}
            onChange={(e) => handleEdit(todo.id, e.target.value)}
           />
        </li>
      );
    })}
  </ul>
```

![](https://storage.googleapis.com/zenn-user-upload/aacef9593c5f-20230222.png =480x)

## ステートのイミュータビリティは保たれているか？

では、上の処理を行うことによってもステートの**イミュータビリティ**（**immutability**, 不変性）は保たれているのでしょうか？

結論から言うと、この手法ではイミュータビリティを保つことはできません。

上のコードの `newTodos` 配列を作成した**後**かつ `todos ステート` を更新する**前**の `todos ステート` の値を確認してみましょう。

```jsx:例
  const handleEdit = (id: number, value: string) => {
    setTodos((todos) => {
      const newTodos = todos.map((todo) => {
        if (todo.id === id) {
          todo.value = value;
        }
        return todo;
      });

      // todos ステートが書き換えられていないかチェック
      console.log('=== Original todos ===');
      todos.map((todo) => {
        console.log(`id: ${todo.id}, value: ${todo.value}`);
      });
      // ここまで

      return newTodos;
    });
  };
```

結果は以下のようになります。

![](https://storage.googleapis.com/zenn-user-upload/4ebb919053548f0e8cb29390.png)

`return newTodos` が実行される**前に** `todos ステート` 配列が**直接ミューテート**されてしまっています。

[Array.prototype.map()](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/map) は、「新しい配列を生成する非破壊的メソッド」であるはずなのに何故なのでしょうか？
