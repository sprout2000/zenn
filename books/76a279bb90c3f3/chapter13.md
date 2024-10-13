---
title: "登録済みの todo を削除可能にする - Todo の仕様を考える (その 4)"
free: true
---

## `Todo型`オブジェクトの再拡張（その 2）

タスクの削除/未削除を示すフラグを `Todo型` に追加しましょう。
これも true _or_ false を表すプロパティなので型は **`Boolean型`** となります。

```jsx:src/App.tsx
type Todo = {
  value: string;
  readonly id: number;
  checked: boolean;
  removed: boolean;
};
```

前章の `checked` のときと同じく、 `handleSubmit()` メソッドを更新する必要があります。

```jsx:src/App.tsx
  if (!text) return;
  const newTodo: Todo = {
    value: text,
    id: new Date().getTime(),
    checked: false,
    removed: false, // <-- 追加
  };

  setTodos((todos) => [newTodo, ...todos]);
```

## 削除ボタンの追加

それぞれの項目の後ろへ削除ボタンを追加します。

```jsx:App.tsx
  return (
    <li key={todo.id}>
      <input
        type="checkbox"
        checked={todo.checked}
        onChange={() => handleCheck(todo.id, todo.checked)}
      />
      <input
        type="text"
        disabled={todo.checked}
        value={todo.value}
        onChange={(e) => handleEdit(todo.id, e.target.value)}
      />
      <button onClick={() => console.log('removed!')}>削除</button>
    </li>
  );
```

![](https://storage.googleapis.com/zenn-user-upload/b5df6ca41ca3-20230222.png =480x)

## 削除ボタンがクリックされたときのコールバック関数を作成する

これも前章の `handleChecked()` とまったく同じパターンです。
同様に `onClick` イベントへ紐付けします。

```jsx:src/App.tsx
  const handleRemove = (id: number, removed: boolean) => {
    setTodos((todos) => {
      const newTodos = todos.map((todo) => {
        if (todo.id === id) {
          return { ...todo, removed };
        }
        return todo;
      });

      return newTodos;
    });
  };
```

すでに削除済みかどうかを可視化するため、`todo.removed` の値によってボタンのラベルを入れ替えましょう。
また、前章同様に `removed` フラグの反転は呼び出し側で行うことに注意しましょう。

```jsx:src/App.tsx
    <button onClick={() => handleRemove(todo.id, !todo.removed)}>
      {todo.removed ? '復元' : '削除'}
    </button>
```

削除されたアイテムは改変できないようにするため、チェックボックスと編集用フォームも無効化します。

```diff jsx:src/App.tsx
  <input
    type="checkbox"
+   disabled={todo.removed}
    checked={todo.checked}
    onChange={() => handleCheck(todo.id, todo.checked)}
  />
  <input
    type="text"
+   disabled={todo.checked || todo.removed}
    value={todo.value}
    onChange={(e) => handleEdit(todo.id, e.target.value)}
  />
```

![](https://storage.googleapis.com/zenn-user-upload/177c30701019-20230222.png =480x)

## この章のソースコード全文

:::details App.tsx

```tsx:App.tsx
import { useState } from 'react';

type Todo = {
  value: string;
  readonly id: number;
  checked: boolean;
  removed: boolean;
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
      removed: false,
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

  const handleRemove = (id: number, removed: boolean) => {
    setTodos((todos) => {
      const newTodos = todos.map((todo) => {
        if (todo.id === id) {
          return { ...todo, removed };
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
                disabled={todo.removed}
                checked={todo.checked}
                onChange={() => handleCheck(todo.id, !todo.checked)}
              />
              <input
                type="text"
                disabled={todo.checked || todo.removed}
                value={todo.value}
                onChange={(e) => handleEdit(todo.id, e.target.value)}
              />
              <button onClick={() => handleRemove(todo.id, !todo.removed)}>
                {todo.removed ? '復元' : '削除'}
              </button>
            </li>
          );
        })}
      </ul>
    </div>
  );
};
```

:::
