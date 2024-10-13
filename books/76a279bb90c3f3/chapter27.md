---
title: "ストレージのデータにも型安全性を担保する"
free: true
---

## ユーザー定義の型ガード関数を作成する

前々章の例では、`setTodos` するためにストレージのデータに対して、型アサーション **`as Todos`** を利用しています。しかし厳密に型安全を担保するのであれば、ストレージのデータもそれが **`Todo型`オブジェクトの配列である**ことを事前に確認すべきです。

そのためには、ユーザー定義の**型ガード関数**を用意する必要があります。

https://typescriptbook.jp/reference/functions/type-guard-functions

`src` フォルダ以下へ `lib` というフォルダを作成し、`isTodos.ts` を配置します。

```ts:src/lib/isTodo.ts
/* eslint-disable @typescript-eslint/no-explicit-any */
const isTodo = (arg: any): arg is Todo => {
  return (
    typeof arg === 'object' &&
    typeof arg.id === 'number' &&
    typeof arg.value === 'string' &&
    typeof arg.checked === 'boolean' &&
    typeof arg.removed === 'boolean'
  );
};

export const isTodos = (arg: any): arg is Todo[] => {
  return Array.isArray(arg) && arg.every(isTodo);
};
```

上段の `isTodo` 関数では、与えられた引数が `Todo型` のオブジェクトであるかどうかをチェックしています。
`:arg is Todo` の部分は [_Type predicate_](https://typescriptbook.jp/reference/functions/type-guard-functions) と呼ばれ、`boolean` を返す関数の戻り値を **`x is T`** と書くことで､ **`true` を返せば `x` が `T型`､ `false` を返せばそうでない**ことを表す機能です。

下段のエクポートされた `isTodos` 関数では、与えられた引数が、**配列** かつ **その各要素が `Todo型`オブジェクト** であるか否かをチェックしています。
`Array.every()` メソッドは、配列内のすべての要素が指定された関数で実装されたテストに合格するかどうかをテストし、論理値を返します。

https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/every

では、この **`isTodos.ts`** ファイルから型ガード関数をインポートして型チェックを適用しましょう。

```jsx:src/App.tsx
import { isTodos } from './lib/isTodos';

// 中略

  useEffect(() => {
    localforage
      .getItem('todo-20200101')
      .then((values) => isTodos(values) && setTodos(values));
  }, []);
```

ここまでで MUI 版 Todo アプリはいったん完成です。完成品のソースコードの全文は[こちら](https://github.com/sprout2000/todo)でも確認できます。

https://github.com/sprout2000/todo#readme

## 本章のソースコード全文

```ts:src/lib/isTodos.ts
/* eslint-disable @typescript-eslint/no-explicit-any */
const isTodo = (arg: any): arg is Todo => {
  return (
    typeof arg === 'object' &&
    typeof arg.id === 'number' &&
    typeof arg.value === 'string' &&
    typeof arg.checked === 'boolean' &&
    typeof arg.removed === 'boolean'
  );
};

export const isTodos = (arg: any): arg is Todo[] => {
  return Array.isArray(arg) && arg.every(isTodo);
};
```

:::details App.tsx

```tsx:src/App.tsx
import { useEffect, useState } from 'react';

import localforage from 'localforage';

import GlobalStyles from '@mui/material/GlobalStyles';
import { createTheme, ThemeProvider } from '@mui/material/styles';
import { indigo, pink } from '@mui/material/colors';

import { QR } from './QR';
import { ToolBar } from './ToolBar';
import { SideBar } from './SideBar';
import { TodoItem } from './TodoItem';
import { FormDialog } from './FormDialog';
import { AlertDialog } from './AlertDialog';
import { ActionButton } from './ActionButton';

import { isTodos } from './lib/isTodos';

const theme = createTheme({
  palette: {
    primary: {
      main: indigo[500],
      light: '#757de8',
      dark: '#002984',
    },
    secondary: {
      main: pink[500],
      light: '#ff6090',
      dark: '#b0003a',
    },
  },
});

export const App = () => {
  const [text, setText] = useState('');
  const [todos, setTodos] = useState<Todo[]>([]);
  const [filter, setFilter] = useState<Filter>('all');

  const [qrOpen, setQrOpen] = useState(false);
  const [alertOpen, setAlertOpen] = useState(false);
  const [dialogOpen, setDialogOpen] = useState(false);
  const [drawerOpen, setDrawerOpen] = useState(false);

  const handleToggleQR = () => {
    setQrOpen((qrOpen) => !qrOpen);
  };

  const handleToggleDrawer = () => {
    setDrawerOpen((drawerOpen) => !drawerOpen);
  };

  const handleToggleDialog = () => {
    setDialogOpen((dialogOpen) => !dialogOpen);
    setText('');
  };

  const handleToggleAlert = () => {
    setAlertOpen((alertOpen) => !alertOpen);
  };

  const handleChange = (
    e: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement>
  ) => {
    setText(e.target.value);
  };

  const handleSubmit = () => {
    if (!text) {
      setDialogOpen((dialogOpen) => !dialogOpen);
      return;
    }

    const newTodo: Todo = {
      value: text,
      id: new Date().getTime(),
      checked: false,
      removed: false,
    };

    setTodos((todos) => [newTodo, ...todos]);
    setText('');
    setDialogOpen((dialogOpen) => !dialogOpen);
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

  const handleSort = (filter: Filter) => {
    setFilter(filter);
  };

  useEffect(() => {
    localforage
      .getItem('todo-20200101')
      .then((values) => isTodos(values) && setTodos(values));
  }, []);

  useEffect(() => {
    localforage.setItem('todo-20200101', todos);
  }, [todos]);

  return (
    <ThemeProvider theme={theme}>
      <GlobalStyles styles={{ body: { margin: 0, padding: 0 } }} />
      <ToolBar filter={filter} onToggleDrawer={handleToggleDrawer} />
      <SideBar
        drawerOpen={drawerOpen}
        onSort={handleSort}
        onToggleQR={handleToggleQR}
        onToggleDrawer={handleToggleDrawer}
      />
      <QR open={qrOpen} onClose={handleToggleQR} />
      <FormDialog
        text={text}
        dialogOpen={dialogOpen}
        onChange={handleChange}
        onSubmit={handleSubmit}
        onToggleDialog={handleToggleDialog}
      />
      <AlertDialog
        alertOpen={alertOpen}
        onEmpty={handleEmpty}
        onToggleAlert={handleToggleAlert}
      />
      <TodoItem todos={todos} filter={filter} onTodo={handleTodo} />
      <ActionButton
        todos={todos}
        filter={filter}
        alertOpen={alertOpen}
        dialogOpen={dialogOpen}
        onToggleAlert={handleToggleAlert}
        onToggleDialog={handleToggleDialog}
      />
    </ThemeProvider>
  );
};
```

:::
