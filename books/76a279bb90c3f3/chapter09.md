---
title: "Todo（タスク） の仕様を考える (その 2)"
free: true
---

## `Todo型`オブジェクトへプロパティを追加する

前章で見た通り、**`todos` ステート配列**をリストとして展開するためには、配列の各要素へその識別子を持たせる必要があります。

配列の各要素、つまり `Todo型`のタスクそれぞれに一意な _key_ を持たせる必要が生じたため、`Todo型`そのものを拡張しなければなりません。

ここでは、`id` プロパティとして一意な数字 (`number型`) を持たせることにします。
また、一意であるはずの識別子が書き換えられてはならないため、**readonly（読み取り専用）** のプロパティとします。

```diff jsx:src/App.tsx
  type Todo = {
    value: string;
+   readonly id: number;
  };
```

https://typescript-jp.gitbook.io/deep-dive/type-system/readonly

![](https://storage.googleapis.com/zenn-user-upload/f3083bf5a96c-20230320.png =360x)

`Todo型`オブジェクトには `id` プロパティの指定が必須となったため、`handleSubmit()` メソッドを更新しなければいけません。

```jsx:src/App.tsx
const handleSubmit = () => {
    e.preventDefault();
    if (!text) return;

    const newTodo: Todo = {
      value: text,
      /**
      * Todo型オブジェクトの型定義が更新されたため、
      * number型の id プロパティの存在が必須になった
      */
      id: new Date().getTime(),
    };

    setTodos((todos) => [newTodo, ...todos]);
    setText('');
  };
```

これでそれぞれのタスクが一意なプロパティを持つようになったので、これを _key_ プロパティへ適用しましょう。
`<li> ~ </li>` タグに **`key (=todo.id)`** を付加します。

```jsx:src/App.tsx
  <ul>
    {todos.map((todo) => {
      return <li key={todo.id}>{todo.value}</li>;
    })}
  </ul>
```

![](https://storage.googleapis.com/zenn-user-upload/010517ff28d9-20230221.png)
_各 Todo に **`id`** プロパティが追加された_

:::message
_key_ は特別なプロパティであり、React によって予約されているため、他の用途に使うことはできません。
:::

なお、以下のように _key_ へ配列のインデックスを利用することは[推奨されていません](https://ja.react.dev/learn/rendering-lists#rules-of-keys)。

```jsx:例
// ❌ No Good
list.map((item, index) => <li key={index}>{item}</li>)
```

その理由は前章で述べた `key` の重要性とほぼ同じですが、ここでは[公式ドキュメント](https://ja.react.dev/learn/rendering-lists#why-does-react-need-keys)からの引用を掲載しておきます。

> アイテムのインデックスを _key_ として使用したくなるかもしれません。実際、_key_ を指定しなかった場合、React はデフォルトでインデックスを使用します。しかし、アイテムが挿入されたり削除されたり、配列を並び替えたりすると、レンダーするアイテムの順序も変わります。インデックスをキーとして利用すると、微妙かつややこしいバグの原因となります。

配列のインデックスを _key_ に利用するのは、何らかの理由で各要素へ一意な識別子を与えることが困難な場合のあくまでも最終手段として捉えておくべきでしょう。

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
          return <li key={todo.id}>{todo.value}</li>;
        })}
      </ul>
    </div>
  );
};
```

:::
