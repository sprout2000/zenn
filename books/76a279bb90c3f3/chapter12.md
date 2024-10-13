---
title: "タスクの完了/未完了を操作できるようにする - Todo の仕様を考える (その 3)"
free: true
---

## `Todo型`オブジェクトの再拡張

タスクの完了/未完了を示すフラグを `Todo型` に追加しましょう。
完了/未完了 (= true _or_ false) を表すので型は **`Boolean型`** となります。

```jsx:src/App.tsx
type Todo = {
  value: string;
  readonly id: number;
  // 完了/未完了を示すプロパティ
  checked: boolean;
};
```

`Todo型`オブジェクトには **`checked`** プロパティが必須となったため、 `handleSubmit()` メソッドを更新します。

```jsx:src/App.tsx
    if (!text) return;

    const newTodo: Todo = {
      value: text,
      id: new Date().getTime(),
      // 初期値（todo 作成時）は false
      checked: false,
    };

    setTodos((todos) => [newTodo, ...todos]);
```

それぞれの項目の前へ、完了/未完了を操作をするためのチェックボックスを置きます。

```jsx:App.tsx
      <ul>
        {todos.map((todo) => {
          return (
            <li key={todo.id}>
              <input
                type="checkbox"
                checked={todo.checked}
                onChange={() => console.log('checked!')}
              />
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

![](https://storage.googleapis.com/zenn-user-upload/6f493aea57f1-20230222.png =480x)

## チェックボックスがチェックされたときのコールバック関数を作成する

前章の `handleEdit()` 関数とパターンは同じです。

どの `todo` がチェックされたのか特定するための `id` と `checked` プロパティの値を引数として受け取り、その `todo型` オブジェクトの `checked` プロパティを更新します。

```jsx:src/App.tsx
  const handleCheck = (id: number, checked: boolean) => {
    setTodos((todos) => {
      const newTodos = todos.map((todo) => {
        if (todo.id === id) {
          return { ...todo, checked };
        }
        return todo;
      });

      return newTodos;
    });
  };
```

同様にチェックボックスのイベントへ紐付けますが、**呼び出し側で `checked` プロパティの値を反転させる**必要があることに注意してください。
呼び出し側で反転しておくことで、のちに[「第 16 章 TypeScript のジェネリクスを使ってよく似た関数を一つにまとめる」](https://zenn.dev/sprout2000/books/76a279bb90c3f3/viewer/chapter16)でのイベント処理関数のリファクタリングが楽になることに気づくでしょう。

```jsx:src/App.tsx
 return (
   <li key={todo.id}>
    <input
      type="checkbox"
      checked={todo.checked}
      // 呼び出し側で checked フラグを反転させる
      onChange={() => handleCheck(todo.id, !todo.checked)}
    />
    <input
      type="text"
      value={todo.value}
      onChange={(e) => handleEdit(todo.id, e.target.value)}
    />
   </li>
 );
```

![](https://storage.googleapis.com/zenn-user-upload/85168913aade-20230222.png =480x)

このままではチェック済みのタスクも編集できてしまうので、チェック済みの項目は編集用フォームを無効にします。

```diff jsx:src/App.tsx
  <input
    type="text"
+   disabled={todo.checked}
    value={todo.value}
    onChange={(e) => handleEdit(todo.id, e.target.value)}
  />
```

![](https://storage.googleapis.com/zenn-user-upload/2066b99ce150-20230222.png =480x)

## この章のソースコード全文

:::details App.tsx

```tsx:App.tsx
import { useState } from 'react';

type Todo = {
  value: string;
  readonly id: number;
  checked: boolean;
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
      checked: false,
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

  const handleCheck = (id: number, checked: boolean) => {
    setTodos((todos) => {
      const newTodos = todos.map((todo) => {
        if (todo.id === id) {
          return { ...todo, checked };
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
                type="checkbox"
                checked={todo.checked}
                onChange={() => handleCheck(todo.id, !todo.checked)}
              />
              <input
                type="text"
                disabled={todo.checked}
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
