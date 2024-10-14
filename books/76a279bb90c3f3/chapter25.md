---
title: "ブラウザのストレージへデータを保存する ~ useEffect フックの利用"
free: true
---

# ブラウザ内ストレージへのデータの保存

ブラウザのストレージ ([IndexedDB](https://developer.mozilla.org/ja/docs/Web/API/IndexedDB_API/Using_IndexedDB)) を利用し、タスクの現在の状態を保存することでページの再読み込み後も Todo リストが消えないようにします。

https://developer.mozilla.org/ja/docs/Web/API/IndexedDB_API/Using_IndexedDB

## LocalForage のインストール

LocalForage は、ローカルストレージ風のシンプルな API で非同期ストレージ（IndexedDB または WebSQL）を利用可能とするライブラリですが、今回はこのツールを使って IndexedDB にデータを保存します。

```sh:zsh
npm install localforage
```

https://www.npmjs.com/package/localforage

## LocalForage の使い方

LocalForage の使い方には、引数にコールバック関数を渡す方法と Promise を使う方法の 2 つがあります。本書では Promise による方法を採用します。

https://jsprimer.net/basic/async/

```javascript:構文
// データを取得
localforage
  .getItem('データベースのキー名')
  // value: 保存されている値
  .then((value) => console.log(JSON.stringify(values)));

// データ (=value) を保存
localforage.setItem('データベースのキー名', value);
```

LocalForage では、保存する値として文字列や数値だけでなく、配列やオブジェクトも扱えます。
つまりこの Todo アプリでは、 **`todos` ステート**（`todo型`オブジェクトの配列）を渡すことになります。

## useEffect フックで React コンポーネントへ組み込む

React Hooks では、関数コンポーネント内で**副作用** (サイドエフェクト) を実行するための **useEffect** というフックが用意されています。

副作用とは、関数コンポーネントの出力（＝レンダリング）に関係ない処理のことです。 つまり、 **useEffect** を用いることでレンダリングと副作用を切り離すことが可能になります。

典型的には、**React が DOM を更新した後で何らかの追加のコードを実行したい**という場合にこの **useEffect** フックを使います。

https://ja.react.dev/reference/react/useEffect

## useEffect フックの構文

```jsx:構文
useEffect(() => console.log('TODO!'), []);
```

### 第 1 引数のコールバック関数

React コンポーネントがマウント（描画）またはアップデート（更新・再描画）された**あと**に実行したい何らかの処理を指定します。

:::message
React コンポーネントのライフサイクルについては、次のコラムで概説します。
:::

### 第 2 引数の配列

**useEffect** 内で参照している外部の変数や関数を配列内に列挙します（**「依存配列」** と呼ばれます）。
**useEffect**フックでは、この**依存配列内のいずれかの要素が作成・更新されたとき**に第 1 引数の処理を実行します。

空の配列とするとコンポーネントが**マウントされたときのみ**に第 1 引数の処理を実行します。また、この配列そのものを省略すると**常に**副作用が実行されます。

```jsx:例
useEffect(() => {
  console.log(`コンポーネントがマウントされたよ！`);
}, []);

useEffect(() => {
  console.log(`filter ステートが更新されたよ！: ${filter}`);
}, [filter]);
```

![](https://storage.googleapis.com/zenn-user-upload/fd76c421fc15-20230321.png)

### 依存配列の重要性

依存配列への要素の指定には注意が必要です。

なぜなら、これへ適切な要素が指定されていなかったり、この配列そのものを、そうすべきでないときに省略したりすると、副作用が意図した通りに実行されなかったり、React コンポーネントが無限ループに陥るなどのバグを生じさせてしまうからです（※）。

しかし、現在の **Vite** では最初から [eslint-plugin-react-hooks](https://www.npmjs.com/package/eslint-plugin-react-hooks) が設定済みであるため、**ESLint** が依存配列の指定漏れを警告してくれるので、これを活用しましょう。

![](https://storage.googleapis.com/zenn-user-upload/ed39d2bc23c0-20230215.png)
_VS Code で依存配列内の黄色の下線にホバー_

> useEffect フックが依存する 'count' が指定されていません。
> これを依存配列に含めるか、依存配列そのものを削除してください。

:::message
React コンポーネントが無限ループに陥ってしまう典型例が、useEffect フック内でステートを更新する必要がある場合です。

```jsx
useEffect(() => {
  setFoo("boo");
});
```

**「`setState` メソッドの実行はコンポーネントの再レンダリングをトリガーする」** ことを思い出してください。
_「レンダー~副作用~レンダー~(以下ループ)」_ の事態を起こさないためにも、依存配列の指定には十分に注意する必要があります。
:::

## useEffect フックの実装

では、`App.tsx` へサイドエフェクト処理を追加してみましょう。

```jsx:src/App.tsx
// localforage をインポート
import localforage from 'localforage';

// useEffect フックをインポート
import { useEffect, useState  } from 'react';
```

なお、**useEffect** フックはコンポーネントの**内側**で定義しなければいけません。典型的には _return_ 文の直前に配置することになります。

```jsx:src/App.tsx
  /**
   * キー名 'todo-20200101' のデータを取得
   * 第 2 引数の配列が空なのでコンポーネントのマウント時のみに実行される
  */
  useEffect(() => {
    localforage
      .getItem('todo-20200101')
      .then((values) => setTodos(values as Todo[]));
  }, []);

  /**
   * todos ステートが更新されたら、その値を保存
  */
  useEffect(() => {
    localforage.setItem('todo-20200101', todos);
  }, [todos]);
```

ブラウザの IndexedDB の状態は、開発者ツールの **Application** -> **Storage** -> **IndexedDB** -> **localforage** からも確認できます。

![](https://storage.googleapis.com/zenn-user-upload/dd2fd08649f6-20211122.png)
_アプリケーション -> ストレージ -> IndexedDB -> localforage_

## 本章のソースコード全文

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

  const handleFilter = (filter: Filter) => {
    setFilter(filter);
  };

  useEffect(() => {
    localforage
      .getItem('todo-20200101')
      .then((values) => setTodos(values as Todo[]));
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
        onSort={handleFilter}
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
