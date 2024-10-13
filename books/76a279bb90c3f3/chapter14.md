---
title: "タスクをフィルタリングする機能を追加する"
free: true
---

# フィルタリング機能の追加

このままでは、完了済みアイテムや削除済みアイテムもいっしょにそのまま表示されてしまうので、タスクをフィルタリングする機能を追加します。

## フィルタリングするセレクタを作成

ここでも `onChange` イベントへはとりあえずダミーを与えておきます。

```jsx:src/App.tsx
    <div>
      <select defaultValue="all" onChange={(e) => e.preventDefault()}>
        <option value="all">すべてのタスク</option>
        <option value="checked">完了したタスク</option>
        <option value="unchecked">現在のタスク</option>
        <option value="removed">ごみ箱</option>
      </select>
      <form
        onSubmit={(e) => {
          e.preventDefault();
          handleSubmit();
        }}
      >
    {/* 省略 */}
    </div>
```

![](https://storage.googleapis.com/zenn-user-upload/3f637f78d080-20230222.png =480x)

## 現在のフィルターを格納する `filter` ステートを追加する

フィルターの状態をあらわす `Filter型` を新設し、 その種別は４種類とします。

```jsx:src/App.tsx
type Filter = 'all' | 'checked' | 'unchecked' | 'removed';
```

| フィルター  | タスクの種別                               |
| :---------: | :----------------------------------------- |
|    `all`    | すべてのタスク（削除済みのタスクをのぞく） |
|  `checked`  | 完了したタスク                             |
| `unchecked` | 現在の（未完了の）タスク                   |
|  `removed`  | ごみ箱（削除済みのタスク）                 |

前項の `<option />` タグの値を **`Filter型`のステート** として保持しましょう。

```jsx:src/App.tsx
export const App = () => {
  const [text, setText] = useState('');
  const [todos, setTodos] = useState<Todo[]>([]);
  // 追加
  const [filter, setFilter] = useState<Filter>('all');
```

## コールバック関数の作成とイベントへの紐付け

上のセレクタの値が変化 (`onChange` イベントの発火)すると `filter ステート` を更新する関数を作成します。
`Filter型`の変数を引数に取り、ステートを更新するだけの関数ですが、[chap.07](https://zenn.dev/sprout2000/books/76a279bb90c3f3/viewer/chapter07) での `handleChange` 関数と同様に、のちのコンポーネント分割の際に _props_ として利用しやすくなります。

```tsx:src/App.tsx
const handleSort = (filter: Filter) => {
  setFilter(filter);
};
```

上の関数を `onChange` イベントへ紐づけます。

```jsx:src/App.tsx
      // e.target.value: string を Filter型にアサーションする
      <select
        defaultValue="all"
        onChange={(e) => handleSort(e.target.value as Filter)}
      >
        <option value="all">すべてのタスク</option>
        <option value="checked">完了したタスク</option>
        <option value="unchecked">現在のタスク</option>
        <option value="removed">ごみ箱</option>
      </select>
```

`Filter` を単なる `string型` にすれば、上記コードのような型アサーションは不要ですが、次項の **switch 文で型によるエディタの補完を享受する**ため、あえて `Filter型` を適用しています。

## フィルタリング後の `Todo型`の配列をリスト表示する

フィルタリング後のリストを格納する変数を作成しましょう。

- `<ul> ~ </ul>` タグの中で展開されている **`todos ステート`** をタグへ渡す前に加工する
- 現在の **`filter ステート`** に応じて `Todo型`配列の要素をフィルタリングする
- [Array.prototype.filter()](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/filter) メソッドは、配列の各要素の中から条件に合致した要素を抽出して**新しい配列を生成**する非破壊的メソッド

```js:例
const items = [0, 1, 2, 3, 4, 5, 6];
items.filter((item) => item % 2 === 0); // --> [0, 2, 4, 6]
```

- `Todo型`オブジェクト内のプロパティを編集するわけではないので、**イミュータビリティ**には影響がない

https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/filter

```jsx:src/App.tsx
  const filteredTodos = todos.filter((todo) => {
    // filter ステートの値に応じて異なる内容の配列を返す
    switch (filter) {
      case 'all':
        // 削除されていないもの
        return !todo.removed;
      case 'checked':
        // 完了済 **かつ** 削除されていないもの
        return todo.checked && !todo.removed;
      case 'unchecked':
        // 未完了 **かつ** 削除されていないもの
        return !todo.checked && !todo.removed;
      case 'removed':
        // 削除済みのもの
        return todo.removed;
      default:
        return todo;
    }
  });
```

`todos ステート` を展開する `<ul> ~ </ul>` タグにフィルタリング済みのリストを渡すように書き換えます。

```diff jsx:src/App.tsx
        <ul>
-         {todos.map((todo) => {
+         {filteredTodos.map((todo) => {
            return (
              <li key={todo.id}>
                <input
                  type="checkbox"
                  disabled={todo.removed}
```

「ごみ箱」や「完了したタスク」が表示されている時は、あらたなタスクを追加できないように Todo 入力フォームは無効化しましょう。

```diff tsx:src/App.tsx
    <form
      onSubmit={(e) => {
        e.preventDefault();
        handleSubmit();
      }}
    >
      <input
        type="text"
        value={text}
+       disabled={filter === 'checked' || filter === 'removed'}
        onChange={(e) => handleChange(e)}
      />
      <input
        type="submit"
        value="追加"
+       disabled={filter === 'checked' || filter === 'removed'}
        onSubmit={handleSubmit}
      />
    </form>
```

![](https://storage.googleapis.com/zenn-user-upload/55697b5c1efe-20230222.png =480x)
![](https://storage.googleapis.com/zenn-user-upload/f6e1995e22ef-20230222.png =480x)

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

  const handleSort = (filter: Filter) => {
    setFilter(filter);
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
        onChange={(e) => handleSort(e.target.value as Filter)}
      >
        <option value="all">すべてのタスク</option>
        <option value="checked">完了したタスク</option>
        <option value="unchecked">現在のタスク</option>
        <option value="removed">ごみ箱</option>
      </select>
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
