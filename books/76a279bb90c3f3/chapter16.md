---
title: "TypeScript のジェネリクスを使ってよく似た関数を一つにまとめる"
free: true
---

# よく似た関数

第 11 章～第 13 章で作成したコールバック関数は、どれもロジックが同じで見た目もよく似通ったものでした。

```js:handleEdit
  const handleEdit = (id: number, value: string) => {
    // ...
    if (todo.id === id) return { ...todo, value };
    // ...
  };
```

```js:handleCheck
  const handleCheck = (id: number, checked: boolean) => {
    // ...
    if (todo.id === id) return { ...todo, checked };
    // ...
  };
```

```js:handleRemove
  const handleRemove = (id: number, removed: boolean) => {
    // ...
    if (todo.id === id) return { ...todo, removed };
    // ...
};
```

これらの冗長な記述を一つの関数にまとめる手段として、TypeScript の[ジェネリクス](https://typescriptbook.jp/reference/generics)という機能が利用できます。

https://typescriptbook.jp/reference/generics

## ジェネリクス

`any型`を使えば上の 3 つの関数で同一のコードを使い回すことができますが、それではせっかくの型安全性が損なわれてしまいます。

ジェネリクスを使うことでこの問題を解決できます。ジェネリクスとは、一言で言えば「**型も変数のように扱う**」機能です。

例として、以下の 2 つの関数を見てみましょう。

```ts
const getId = (arg: number): number => arg;
const getName = (arg: string): string => arg;

getId(13); // --> 13
getName("Taro"); // --> 'Taro'
```

まったく同じロジック（与えられた引数をそのまま返しているだけ）であるにも関わらず、引数と戻り値の**型が異なる**ためだけに 2 つの関数が必要となっています。

これを「型も変数 **`(T)`** のように扱う」ように書き換えると以下のようになります。

```ts
const getGeneric = <T>(arg: T): T => arg;
```

**`<>`** 内に引数の型（「型引数」と言います）**`T型`** を設定し、戻り値の[型注釈](https://typescriptbook.jp/reference/values-types-variables/type-annotation)にも同じく **`T型`** を与えています。
こうすれば、どの型でも同じコードが使えるようになります。

```js
getGeneric(13); // --> 13
getGeneric("Taro"); // --> 'Taro'
```

## 3 つの関数をジェネリクスでリファクタリングする

上の 3 つの関数の呼び出し側を見てみます。

```diff jsx
  return (
    <li key={todo.id}>
      <input
        type="checkbox"
        disabled={todo.removed}
        checked={todo.checked}
+       onChange={() => handleCheck(todo.id, !todo.checked)}
      />
      <input
        type="text"
        disabled={todo.checked || todo.removed}
        value={todo.value}
+       onChange={(e) => handleEdit(todo.id, e.target.value)}
      />
+     <button onClick={() => handleRemove(todo.id, !todo.removed)}>
        {todo.removed ? '復元' : '削除'}
      </button>
    </li>
);
```

いずれも `todo.id` プロパティ（`number型`）を第 1 引数に、書き換えるプロパティの値を第 2 引数にとっています。
このパターンを「型も変数のように扱う」関数 (=`handleTodo`) に書き換えることを考えます。

```ts
const handleTodo = <K: '書き換えたいプロパティ', V: '新しい値'>(
  id: number,
  key: K,
  value: V
  ) => {
  /** ... */
};
```

### `Todo型`オブジェクトの書き換え対象プロパティ

TypeScript では [**`extends`**](https://typescriptbook.jp/reference/generics/type-parameter-constraint) キーワードを用いることで、型引数 **`K`, `V`** を特定の型に限定することができます。

https://typescriptbook.jp/reference/generics/type-parameter-constraint

オブジェクトのプロパティは、[**`keyof`**](https://typescriptbook.jp/reference/type-reuse/keyof-type-operator) 演算子で取得します。

```js
K extends keyof Todo
// => 'id', 'value', 'checked', 'removed' のいずれか
```

https://typescriptbook.jp/reference/type-reuse/keyof-type-operator

### 新しい値

新しい値の型変数名は **`V型`** とし、[ブラケット記法](https://jsprimer.net/basic/object/#property-access)を利用して、対象となるプロパティへ **`V型`** の新しい値を代入します。

```js
V extends Todo[K]
// => todo.id, todo.value, todo.checked, todo.removed のいずれかの値
```

### まとめ

では、**`handleTodo`** 関数を完成させましょう。

```js:handleTodo
  const handleTodo = <K extends keyof Todo, V extends Todo[K]>(
    id: number,
    key: K,
    value: V
  ) => {
    setTodos((todos) => {
      const newTodos = todos.map((todo) => {
        if (todo.id === id) {
          return { ...todo, [key]: value };
        } else {
          return todo;
        }
      });

      return newTodos;
    });
  };
```

**`return { ...todo, [key]: value }`** の部分では、まず書き換え対象でないプロパティをいままで通りスプレッド構文で元の todo 要素からコピー・展開し、書き換え対象のプロパティを[計算プロパティ名](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Object_initializer#%E8%A8%88%E7%AE%97%E3%83%97%E3%83%AD%E3%83%91%E3%83%86%E3%82%A3%E5%90%8D)を使って特定し、引数の値 `value` を代入しています。

計算プロパティ名では、大カッコ **`[]`** の中へ式を記述することができ、それが計算されてプロパティ名として使用されます。

```js
[key]:  // --> K型 == id, value, checked, removed のいずれか
```

では、この関数を使って呼び出し側もアップデートしましょう。

```diff jsx
  return (
    <li key={todo.id}>
      <input
        type="checkbox"
        disabled={todo.removed}
        checked={todo.checked}
+       onChange={() => handleTodo(todo.id, 'checked', !todo.checked)}
      />
      <input
        type="text"
        disabled={todo.checked || todo.removed}
        value={todo.value}
+       onChange={(e) => handleTodo(todo.id, 'value', e.target.value)}
      />
      <button
+       onClick={() => handleTodo(todo.id, 'removed', !todo.removed)}
      >
        {todo.removed ? '復元' : '削除'}
      </button>
    </li>
);
```

これで重複していた `handleEdit`、`handleCheck`、そして `handleRemove` の 3 つの関数を一つにまとめることができました。

ここまでで React による Todo アプリはいったん完成です。
次章以降では、UI フレームワークの導入による見栄えの調整や、PWA 化によるインストール可能なアプリのデプロイメントを進めていきます。

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

  const handleTodo = <K extends keyof Todo, V extends Todo[K]>(
    id: number,
    key: K,
    value: V
  ) => {
    setTodos((todos) => {
      const newTodos = todos.map((todo) => {
        if (todo.id === id) {
          return { ...todo, [key]: value };
        } else {
          return todo;
        }
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
                onChange={() => handleTodo(todo.id, 'checked', !todo.checked)}
              />
              <input
                type="text"
                disabled={todo.checked || todo.removed}
                value={todo.value}
                onChange={(e) => handleTodo(todo.id, 'value', e.target.value)}
              />
              <button
                onClick={() => handleTodo(todo.id, 'removed', !todo.removed)}
              >
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
