---
title: "配列ステートの操作には要注意 (その 2)"
free: true
---

## シャロー（浅い）コピー

第 7 章では[スプレッド構文](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Spread_syntax)によって保たれた**イミュータビリティ**が、前章の [Array.prototype.map()](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/map) メソッドでは保てませんでした。

それは、これまでの `Array.map()` やスプレッド構文によるコピーが**シャローコピー**（薄いコピー）と呼ばれるものだからです。

シャローコピーでは、**1 段階目の要素のみ**（ここでは `Todos` ステート配列の各オブジェクト）がコピーされます（メモリ内で別領域が確保される）。

![](https://storage.googleapis.com/zenn-user-upload/d2832118b03d-20230409.png =480x)

しかし、そのオブジェクト内で入れ子になった **2 段階目以降**のプロパティ (= `value`, `id` ) は、**コピー元配列のそれを変わらず参照**しています（メモリ内の同じ領域を参照している）。これを変更すると**コピー元配列のプロパティも変更**してしまいます。

![](https://storage.googleapis.com/zenn-user-upload/cd4b8f76b177e699591bc8f3.png =640x)

第 7 章では、もう一段上の階層、つまり **1 段階目であるオブジェクトそのものの追加**であったためにシャローコピーによる操作で十分だったのです。

![](https://storage.googleapis.com/zenn-user-upload/2a1d7759c382-20230517.png =640x)

https://developer.mozilla.org/ja/docs/Glossary/Shallow_copy

## 入れ子になったプロパティを書き替える

元の値をイミュータブルに保ちつつ、オブジェクト内で入れ子になったプロパティを書き換えるには、**そのオブジェクトの階層までたどって複製**しなければなりません。
具体的には、配列の要素（Todo 型オブジェクト）の階層においてスプレッド構文でコピー・展開したうえで、入れ子のプロパティの値を更新します。

![](https://storage.googleapis.com/zenn-user-upload/d55c0887a4b5-20230408.png =640x)

```jsx:src/App.tsx
  const handleEdit = (id: number, value: string) => {
    setTodos((todos) => {
      const newTodos = todos.map((todo) => {
        if (todo.id === id) {
          /**
           * この階層でオブジェクト todo をコピー・展開し、
           * その中で value プロパティを引数で上書きする
           */
          return { ...todo, value: value };
        }
        return todo;
      });

      // todos ステート配列をチェック（あとでコメントアウト）
      console.log('=== Original todos ===');
      todos.map((todo) => {
        console.log(`id: ${todo.id}, value: ${todo.value}`);
      });
      // ここまで

      return newTodos;
    });
  };
```

**`return { ...todo, value: value };`** の部分では、元の todo 要素をスプレッド構文でコピー・展開し、書き換え対象である **`value`** を引数で上書きしたものを**新しい要素として複製**しています。

```typescript:例
      /**
       * 参考：同じことをスプレッド構文なしで書いた場合
       */
      const newTodos = todos.map((todo) => {
        if (todo.id === id) {
          // todo オブジェクトをコピー
          const copyObj = Object.assign({}, todo);
          // コピーの value プロパティを引数で更新
          copyObj.value = value;
          // コピーを返す
          return copyObj;
        }
        return todo;
      });
```

![](https://storage.googleapis.com/zenn-user-upload/51b31c8a00c60fd1b9c6cfbc.png =360x)

![](https://storage.googleapis.com/zenn-user-upload/a500bc89846abf15e55d6c9d.png =360x)

![](https://storage.googleapis.com/zenn-user-upload/e6ff0ef8e023282483f03ed6.png =480x)

:::message
上のコード中の `Array.map()` 内では、[オブジェクトの分割代入](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)による省略記法を用いることもできます。

```ts
const newTodos = todos.map((todo) => {
  if (todo.id === id) {
    return { ...todo, value };
    /**
     * 以下と同義:
     * return { ...todo, value: value }
     */
  }
  return todo;
});
```

オブジェクトの分割代入では、プロパティ名とその値をもつ変数名（上の例では引数 `value`）とが同名の場合、上のように省略して記述することが可能となります。
:::

## この章のソースコード全文

:::details App.tsx

```tsx:App.tsx
import { useState } from 'react';

type Todo = {
  value: string;
  readonly id: number;
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
      id: new Date().getTime(),
    };

    setTodos((todos) => [newTodo, ...todos]);
    setText('');
  };

  const handleEdit = (id: number, value: string) => {
    setTodos((todos) => {
      const newTodos = todos.map((todo) => {
        if (todo.id === id) {
          return { ...todo, value };
        }
        return todo;
      });

      return newTodos;
    });
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
    </div>
  );
};
```

:::
