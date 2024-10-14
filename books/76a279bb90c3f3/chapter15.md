---
title: "ごみ箱を空にする機能を追加する"
free: true
---

# ごみ箱を空にする機能を追加する

フィルターで「ごみ箱」の Todo リストを表示しているときには、削除済みタスクを完全に消去できるよう機能追加しましょう。

## 「ゴミ箱を空にする」ボタンの作成

フィルターが「ごみ箱」の場合は「ゴミ箱を空にする」ボタンを表示し、それ以外のときは従前の Todo 入力フォームを表示するよう改修します。

また、フィルターが「完了済みのタスク」であるときに無効化した Todo 入力フォームを表示する意味が無くなったのでこれも非表示にしましょう。

```jsx:src/App.tsx
      <option value="removed">ごみ箱</option>
    </select>
    {/* フィルターが `removed` のときは「ごみ箱を空にする」ボタンを表示 */}
    {filter === 'removed' ? (
      <button onClick={() => console.log('remove all')}>
        ごみ箱を空にする
      </button>
    ) : (
      // フィルターが `checked` でなければ Todo 入力フォームを表示
      filter !== 'checked' && (
        <form
          onSubmit={(e) => {
            e.preventDefault();
            handleSubmit();
          }}
        >
          <input
            type="text"
            value={text}
            disabled={filter === 'checked' || filter === 'removed'}
            onChange={(e) => handleChange(e)}
          />
          <input
            type="submit"
            value="追加"
            disabled={filter === 'checked' || filter === 'removed'}
            onSubmit={handleSubmit}
          />
        </form>
      )
    )}
    <ul>
      {/* 省略 */}
```

こうなると Todo 入力フォームが描画される場合には **`filter === 'removed'` や `filter === 'checked'` という状態が発生し得ない**ので、Todo 入力フォームからこれらを削除しなければいけません。

![](https://storage.googleapis.com/zenn-user-upload/eb6afc68c196-20220329.png =480x)
_VS Code 上でのエラー表示_

```jsx:src/App.tsx
    <form
      onSubmit={(e) => {
        e.preventDefault();
        handleSubmit();
      }}
    >
      <input type="text" value={text} onChange={(e) => handleChange(e)} />
      <input type="submit" value="追加" onSubmit={handleSubmit} />
    </form>
```

![](https://storage.googleapis.com/zenn-user-upload/19626e9dfbc2-20230222.png =480x)

## 「ゴミ箱を空にする」関数の作成と紐付け

`todos ステート` 配列から `removed` フラグが立っている要素を取り除くのみなので、これまでと同様のパターンで処理すれば良いでしょう。

```jsx:src/App.tsx
  const handleEmpty = () => {
    // シャローコピーで事足りる
    setTodos((todos) => todos.filter((todo) => !todo.removed));
  };
```

```jsx:src/App.tsx
  {filter === 'removed' ? (
    // クリックイベントに handleEmpty() を渡す
    <button onClick={handleEmpty}>ゴミ箱を空にする</button>
  ) : (
```

また、ゴミ箱が空の場合、つまり `removed` フラグが立っているタスクが `todos ステート` 配列に存在しない場合にはボタンを無効化します。

```jsx:src/App.tsx
  <button
    onClick={handleEmpty}
    disabled={todos.filter((todo) => todo.removed).length === 0}
  >
    ゴミ箱を空にする
  </button>
```

![](https://storage.googleapis.com/zenn-user-upload/8c073c9f0148-20230222.png =480x)

ここまでで[第 1 章](https://zenn.dev/sprout2000/books/76a279bb90c3f3/viewer/chapter01)に掲げたサンプルと同等の機能をもった Todo アプリが完成しました。
次章では TypeScript ならではの機能を活かし、冗漫なコードの取り纏めに着手します。

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

type Filter = 'all' | 'checked' | 'unchecked' | 'removed';

export const App = () => {
  const [text, setText] = useState('');
  const [todos, setTodos] = useState<Todo[]>([]);
  const [filter, setFilter] = useState<Filter>('all');

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

  const handleFilter = (filter: Filter) => {
    setFilter(filter);
  };

  const handleEmpty = () => {
    setTodos((todos) => todos.filter((todo) => !todo.removed));
  };

  const filteredTodos = todos.filter((todo) => {
    switch (filter) {
      case 'all':
        return !todo.removed;
      case 'checked':
        return todo.checked && !todo.removed;
      case 'unchecked':
        return !todo.checked && !todo.removed;
      case 'removed':
        return todo.removed;
      default:
        return todo;
    }
  });

  return (
    <div>
      <select
        defaultValue="all"
        onChange={(e) => handleFilter(e.target.value as Filter)}
      >
        <option value="all">すべてのタスク</option>
        <option value="checked">完了したタスク</option>
        <option value="unchecked">現在のタスク</option>
        <option value="removed">ごみ箱</option>
      </select>
      {filter === 'removed' ? (
        <button
          onClick={handleEmpty}
          disabled={todos.filter((todo) => todo.removed).length === 0}
        >
          ごみ箱を空にする
        </button>
      ) : (
        filter !== 'checked' && (
          <form
            onSubmit={(e) => {
              e.preventDefault();
              handleSubmit();
            }}
          >
            <input type="text" value={text} onChange={(e) => handleChange(e)} />
            <input type="submit" value="追加" onSubmit={handleSubmit} />
          </form>
        )
      )}
      <ul>
        {filteredTodos.map((todo) => {
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
