---
title: "ソースコードをコンポーネントごとに分割する"
free: true
---

# この章以降のサンプルリポジトリ

下（↓）のリポジトリをブラウザで開き、**`<> Code`** タブの [commits](https://github.com/sprout2000/todo/commits/main) をクリックすることで各章の進捗をチェックすることができます。

https://github.com/sprout2000/todo#readme

![](https://storage.googleapis.com/zenn-user-upload/ba35e6a87886-20230424.png =640x)

:::message
おまけとして Jest と Testing Library によるユニットテストを行ったブランチも付いています。ユニットテストについては「[（補遺）ユニットテストを導入するには？](https://zenn.dev/sprout2000/books/76a279bb90c3f3/viewer/appendix03)」も参照してください。

```sh
# jestブランチのクローン
% git clone https://github.com/sprout2000/todo.git -b jest

# 依存パッケージのインストールとテスト
% cd todo
% npm install
% npm test
```

:::

# UI 部品のコンポーネント化

React の持つコンポーネント志向という特長を活かし、保守しやすいように各 UI 部品をコンポーネントとして別ファイルへ分割します。

## 型エイリアスを型宣言ファイルとして書き出す

`App.tsx` 内で宣言されている `Todo` と `Filter` の 2 つの型エイリアスを、他のファイル（＝コンポーネント）からも参照できるようにするため、[型宣言ファイル](https://typescript-jp.gitbook.io/deep-dive/type-system/intro/d.ts)として書き出します。

https://typescript-jp.gitbook.io/deep-dive/type-system/intro/d.ts

`src` ディレクトリ内へ **`@types`** ディレクトリを作成し、その中へそれぞれ **`Todo.d.ts`**、**`Filter.d.ts`** として配置します。

基本的には、上記の型エイリアスへ **`declare`** キーワード（[アンビエント宣言](https://typescript-jp.gitbook.io/deep-dive/type-system/intro)）を加えて型として宣言するだけです。

```typescript:src/@types/Todo.d.ts
declare type Todo = {
  value: string;
  readonly id: number;
  checked: boolean;
  removed: boolean;
};
```

```typescript:src/@types/Filter.d.ts
declare type Filter = 'all' | 'checked' | 'unchecked' | 'removed';
```

:::message alert
**`src/App.tsx`** から上記 2 つの型エイリアスを削除しても、エラーとならないことを確認してください。
:::

## タスク入力フォームを FormDialog コンポーネントへ書き出す

### FormDialog コンポーネントの作成

chapter.18 で見た通り、タスク入力フォームは **FormDialog コンポーネント** として再構成する予定なので、これを独立したファイルへ書き出しておきます（のちに [MUI](https://mui.com/) を被せて大幅に書き換えますが...）。

:::message
React 向け UI フレームワーク **MUI** については、次章以降で解説します。
:::

**`<form> ~ </form>`** 部分を `FormDialog.tsx` ファイルへコピーし、**FormDialog コンポーネント** として[名前付きエクスポート](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/export)（後述）します。

https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/export

```tsx:src/FormDialog.tsx
// 名前付きエクスポート
export const FormDialog = () => (
  <form
    onSubmit={(e) => {
      e.preventDefault();
      handleSubmit();
    }}
  >
    <input type="text" value={text} onChange={(e) => handleChange(e)} />
    <input type="submit" value="追加" onSubmit={handleSubmit} />
  </form>
);
```

![](https://storage.googleapis.com/zenn-user-upload/8cdbfe9f5302-20230302.png)
_未定義エラー_

ただコピーしただけでは、上の画像のようにたくさんのエラーが表示されてしまいます。これは、元の `App` コンポーネント内で定義されていた変数や関数が、このコンポーネント内では未定義となっているからです。

そこで、このコンポーネントへ引数（以下、**`props`** と呼びます）を与えます。
親コンポーネントが持つ関数やステート変数を以下のような 3 つの _props_ として受け取ることにします。

| 親コンポーネント |  _props_   |
| :--------------- | :--------: |
| `handleSubmit()` | `onSubmit` |
| `text`           |  そのまま  |
| `handleChange()` | `onChange` |

:::message
React では、イベントを処理する関数定義（イベントハンドラー）には handle[Event] を、それを受け取る _props_ には on[Event] を使用するのが通例です。
:::

```diff tsx:src/FormDialog.tsx
+ export const FormDialog = (props) => (
    <form
      onSubmit={(e) => {
        e.preventDefault();
+       props.onSubmit();
      }}
    >
+     <input type="text" value={props.text} onChange={props.onChange} />
+     <input type="submit" value="追加" onSubmit={props.onSubmit} />
    </form>
  );
```

![](https://storage.googleapis.com/zenn-user-upload/d0d971c01ecc-20230302.png)

まだ、肝心の _props_ の型が未定義です。_props_ のための型エイリアスを作成しましょう。

```tsx:src/FormDialog.tsx
type Props = {
  text: string;
  onSubmit: () => void;
  onChange: (e: React.ChangeEvent<HTMLInputElement>) => void;
};
```

基本的には、親コンポーネント内でのこれらの型を書き写すだけです。この型エイリアスを _props_ の[型注釈](https://typescriptbook.jp/reference/values-types-variables/object/type-annotation-of-objects)として与えれば完成です。

```diff tsx:src/FormDialog.tsx
+ export const FormDialog = (props: Props) => (
  // ...
  )
```

https://typescriptbook.jp/reference/values-types-variables/object/type-annotation-of-objects

:::message
子コンポーネントへの _props_ の与え方には、上記の例以外に[オブジェクトの分割代入](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#%E3%82%AA%E3%83%96%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88%E3%81%AE%E5%88%86%E5%89%B2%E4%BB%A3%E5%85%A5)を利用する方法もポピュラーです。

```diff tsx:例
+ export const FormDialog = ({ text, onChange, onSubmit }: Props) => (
    <form
      onSubmit={(e) => {
        e.preventDefault();
+       onSubmit();
      }}
    >
+     <input type="text" value={text} onChange={onChange} />
+     <input type="submit" value="追加" onSubmit={onSubmit} />
    </form>
  );
```

この方法では、`props.` を省略できるため、タイプ量が減少するメリットがある一方で、それが親コンポーネント由来のものか、そのコンポーネント内で定義されたものか一瞥では判断出来なくなるというデメリットもあります。

:::

### 親コンポーネントへインポートする

一方、`App.tsx` では **FormDialog コンポーネント** を[名前付きインポート](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/import)します。

https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/import

```jsx:src/App.tsx
import { FormDialog } from './FormDialog';
```

名前付きでエクスポートされたコンポーネントや関数は、中カッコ **`{}`** 内へあらかじめ決められたその名前（ここでは **`FormDialog`**）で名前付きインポートすることができ、コードエディタによる補完も効きます。

逆に、もしもエクスポート側でデフォルトエクスポートがなされていた場合、**`{}`** 内へインポートすることはできない代わりに、インポート側で自由に名前を設定できます。

```js:例
// デフォルトエクスポート
export default FormDialog = () => {}
//        ↓
// インポート側で名前を付ける（変更できる）
import FormComponent from './FormDialog';
```

では、**FormDialog** から要求されている _props_ を親コンポーネント内で与えましょう。

```diff tsx:src/App.tsx
  // ...
        ) : (
          filter !== 'checked' && (
+          <FormDialog
+            text={text}
+            onChange={handleChange}
+            onSubmit={handleSubmit}
+          />
        )
  // ...
```

## ごみ箱ボタンの書き出し

また、「ごみ箱を空にする」機能は **ActionButton コンポーネント** に持たせることになったので、同様にエクスポート/インポートしておきます。

```tsx:ActionButton.tsx
type Props = {
  todos: Todo[];
  onEmpty: () => void;
};

export const ActionButton = (props: Props) => (
  <button
    onClick={props.onEmpty}
    disabled={props.todos.filter((todo) => todo.removed).length === 0}
  >
    ごみ箱を空にする
  </button>
);
```

```tsx:src/App.tsx
import { ActionButton } from './ActionButton';
```

なお、**`{filter === 'removed' ? () : (filter !== 'checked' && ())`** のロジックは、親コンポーネントからステートを渡すことで、子コンポーネント内でそれぞれ描き分けることになったので、**このロジックの部分はカット**します。

```diff tsx:src/App.tsx
      <div>
         <select>{/* 省略 */}</select>
+       <FormDialog
+         text={text}
+         onChange={handleChange}
+         onSubmit={handleSubmit}
+       />
        <ul>{/* 省略 */}</ul>
+       <ActionButton todos={todos} onEmpty={handleEmpty} />
      </div>
```

## セレクタの書き出し

`filter ステート` の変更は、**SideBar コンポーネント** で行うことになったので、**`<select> ~ </select>`** の部分も、とりあえず `SideBar.tsx` として書き出しておきます（これものちに全面的に書き換えます）。

```tsx:src/SideBar.tsx
type Props = {
  onFilter: (filter: Filter) => void;
};

export const SideBar = (props: Props) => (
  <select
    defaultValue="all"
    onChange={(e) => props.onFilter(e.target.value as Filter)}
  >
    <option value="all">すべてのタスク</option>
    <option value="checked">完了したタスク</option>
    <option value="unchecked">現在のタスク</option>
    <option value="removed">ごみ箱</option>
  </select>
);
```

```tsx:src/App.tsx
import { SideBar } from './SideBar';
```

```diff tsx:src/App.tsx
    return (
      <div>
+       <SideBar onFilter={handleFilter} />
        <FormDialog
          text={text}
          onChange={handleOnChange}
          onSubmit={handleOnSubmit}
          />
        <ul>{/* 省略 */}</ul>
        <ActionButton todos={todos} onEmpty={handleEmpty} />
      </div>
    );
```

## TodoItem コンポーネントの書き出し

同様の手順で **`<ul> ~ <ul>`** タグの中で展開されていた **`todos`** を **TodoItem コンポーネント** として書き出します。

```jsx:src/TodoItem.tsx
type Props = {
  todos: Todo[];
  filter: Filter;
  onTodo: <K extends keyof Todo, V extends Todo[K]>(
    id: number,
    key: K,
    value: V
  ) => void;
};

export const TodoItem = (props: Props) => {
  const filteredTodos = props.todos.filter((todo) => {
    switch (props.filter) {
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
    <ul>
      {filteredTodos.map((todo) => {
        return (
          <li key={todo.id}>
            <input
              type="checkbox"
              disabled={todo.removed}
              checked={todo.checked}
              onChange={() => props.onTodo(todo.id, 'checked', !todo.checked)}
            />
            <input
              type="text"
              disabled={todo.checked || todo.removed}
              value={todo.value}
              onChange={(e) => props.onTodo(todo.id, 'value', e.target.value)}
            />
            <button
              onClick={() => props.onTodo(todo.id, 'removed', !todo.removed)}
            >
              {todo.removed ? '復元' : '削除'}
            </button>
          </li>
        );
      })}
    </ul>
  );
};
```

同様に `App.tsx` へインポートし、親コンポーネントの `filteredTodos` 変数は削除します。

```tsx:src/App.tsx
import { TodoItem } from './TodoItem';
```

```diff tsx:src/App.tsx
    return (
      <div>
        <SideBar onFilter={handleFilter} />
        <FormDialog
          text={text}
          onChange={handleChange}
          onSubmit={handleSubmit}
        />
+      <TodoItem todos={todos} filter={filter} onTodo={handleTodo} />
       <ActionButton todos={todos} onEmpty={handleEmpty} />
      </div>
    );
```

## スタイルシートの削除

:::message alert
以下は必ず実施してください。
:::

今後の UI は CSS 部分も含めて MUI が担当することとなったため、`src/index.css` と、これを `src/main.tsx` でインポートしていた部分を削除してください。

```diff sh:zsh
  % tree src
  src
  ├── @types
  │   ├── Filter.d.ts
  │   └── Todo.d.ts
  ├── ActionButton.tsx
  ├── App.tsx
  ├── FormDialog.tsx
  ├── SideBar.tsx
  ├── TodoItem.tsx
- ├── index.css    # 削除
  ├── main.tsx
  └── vite-env.d.ts

  2 directories, 9 files
```

```diff tsx:main.tsx
  import { createRoot } from 'react-dom/client';

  import { App } from './App';
- import './index.css'; // <-- 削除

  const root = createRoot(document.getElementById('root') as Element);

  root.render(
    <StrictMode>
      <App />
    </StrictMode>
  );
```

## 分割完了

![](https://storage.googleapis.com/zenn-user-upload/df5fff7678fe-20230207.png)
_コンポーネントの分割_

Todo 入力フォームやごみ箱ボタンなどの一部の UI 部品は、各コンポーネントでの処理の分岐がまだ未実装なので常時表示されるようになったものの、コンポーネントが分割されてもアプリの基本的な機能には変化がないことが確認できるでしょう。

これで主なロジックの部分はコンポーネントとして独立させることが出来たので、残りの部分と MUI コンポーネントとして作成する部分を実装していきましょう。

## この章のソースコード全文

:::details src/main.tsx

```tsx:src/main.tsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';

import { App } from './App';

const root = createRoot(document.getElementById('root') as Element)

root.render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

:::

:::details src/@types/Todo.d.ts

```tsx:src/@types/Todo.d.ts
declare type Todo = {
  value: string;
  readonly id: number;
  checked: boolean;
  removed: boolean;
};
```

:::

:::details src/@types/Filter.d.ts

```tsx:src/@types/Filter.d.ts
declare type Filter = 'all' | 'checked' | 'unchecked' | 'removed';
```

:::

:::details src/FormDialog.tsx

```tsx:src/FormDialog.tsx
type Props = {
  text: string;
  onSubmit: () => void;
  onChange: (e: React.ChangeEvent<HTMLInputElement>) => void;
};

export const FormDialog = (props: Props) => {
  return (
    <form
      onSubmit={(e) => {
        e.preventDefault();
        props.onSubmit();
      }}
    >
      <input
        type="text"
        value={props.text}
        onChange={(e) => props.onChange(e)}
      />
      <input type="submit" value="追加" onSubmit={props.onSubmit} />
    </form>
  );
};
```

:::

:::details src/ActionButton.tsx

```tsx:src/ActionButton.tsx
type Props = {
  todos: Todo[];
  onEmpty: () => void;
};

export const ActionButton = (props: Props) => {
  return (
    <button
      onClick={props.onEmpty}
      disabled={props.todos.filter((todo) => todo.removed).length === 0}
    >
      ごみ箱を空にする
    </button>
  );
};
```

:::

:::details src/SideBar.tsx

```tsx:src/SideBar.tsx
type Props = {
  onFilter: (filter: Filter) => void;
};

export const SideBar = (props: Props) => {
  return (
    <select
      defaultValue="all"
      onChange={(e) => props.onFilter(e.target.value as Filter)}
    >
      <option value="all">すべてのタスク</option>
      <option value="checked">完了したタスク</option>
      <option value="unchecked">現在のタスク</option>
      <option value="removed">ごみ箱</option>
    </select>
  );
};
```

:::

:::details src/TodoItem.tsx

```tsx:src/TodoItem.tsx
type Props = {
  todos: Todo[];
  filter: Filter;
  onTodo: <K extends keyof Todo, V extends Todo[K]>(
    id: number,
    key: K,
    value: V
  ) => void;
};

export const TodoItem = (props: Props) => {
  const filteredTodos = props.todos.filter((todo) => {
    switch (props.filter) {
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
    <ul>
      {filteredTodos.map((todo) => {
        return (
          <li key={todo.id}>
            <input
              type="checkbox"
              disabled={todo.removed}
              checked={todo.checked}
              onChange={() => props.onTodo(todo.id, 'checked', !todo.checked)}
            />
            <input
              type="text"
              disabled={todo.checked || todo.removed}
              value={todo.value}
              onChange={(e) => props.onTodo(todo.id, 'value', e.target.value)}
            />
            <button
              onClick={() => props.onTodo(todo.id, 'removed', !todo.removed)}
            >
              {todo.removed ? '復元' : '削除'}
            </button>
          </li>
        );
      })}
    </ul>
  );
};
```

:::

:::details src/App.tsx

```tsx:src/App.tsx
import { useState } from 'react';

import { SideBar } from './SideBar';
import { TodoItem } from './TodoItem';
import { FormDialog } from './FormDialog';
import { ActionButton } from './ActionButton';

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

  const handleEmpty = () => {
    setTodos((todos) => todos.filter((todo) => !todo.removed));
  };

  const handleFilter = (filter: Filter) => {
    setFilter(filter);
  };

  return (
    <div>
      <SideBar onFilter={handleFilter} />
      <FormDialog text={text} onChange={handleChange} onSubmit={handleSubmit} />
      <TodoItem todos={todos} filter={filter} onTodo={handleTodo} />
      <ActionButton todos={todos} onEmpty={handleEmpty} />
    </div>
  );
};
```

:::
