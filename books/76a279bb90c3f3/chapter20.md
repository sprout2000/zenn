---
title: "ToolBar コンポーネントの作成"
free: true
---

# MUI コンポーネントを作成する

では、MUI を用いて **ToolBar コンポーネント**を作成していきましょう。

## モジュールのインポートと AppBar の作成

https://mui.com/components/app-bar/

https://mui.com/api/app-bar/

MUI でのいわゆる App Bar の基本構成は以下のような構造となっています。

```tsx
// ↓ AppBar モジュール
<AppBar position="static">
  {/* ↓ Toolbar モジュール */}
  <Toolbar>{/* バーの中に表示するモジュールを並べる */}</Toolbar>
</AppBar>
```

`ToolBar.tsx` を作成し、必要なモジュールをインポートします。

```jsx:src/ToolBar.tsx
import Box from '@mui/material/Box';
import Icon from '@mui/material/Icon';
import AppBar from '@mui/material/AppBar';
import Toolbar from '@mui/material/Toolbar';
import Typography from '@mui/material/Typography';
import IconButton from '@mui/material/IconButton';

export const ToolBar = () => (
  <Box sx={{ flexGrow: 1 }}>
    <AppBar position="static">
      <Toolbar>
        <IconButton
          aria-label="menu-button"
          size="large"
          edge="start"
          color="inherit"
          sx={{ mr: 2 }}
        >
          <Icon>menu</Icon>
        </IconButton>
        <Typography>TODO</Typography>
      </Toolbar>
    </AppBar>
  </Box>
);
```

**Box コンポーネント** は、CSS ユーティリティの機能を付与するラッパーコンポーネントです。
**`sx`** プロパティを通してすべてのシステムプロパティ（`width`, `bgColor` など）を利用できます。
また、MUI コンポーネントの既定スタイルのオーバーライドもできます。

https://mui.com/components/box/

## MUI モジュールのインポート方法

**MUI v5** では、ビルド時には自動的に [Tree Shaking](https://developer.mozilla.org/ja/docs/Glossary/Tree_shaking) が有効となるため、以下のようにバンドルサイズを気にすることなく[名前付きインポート](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/import)を安全に行うことができます。

```typescript:例
import { Button, TextField } from '@mui/material';
```

https://developer.mozilla.org/ja/docs/Glossary/Tree_shaking

https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/import

また、以下のようにパスインポートを利用することで、不要なモジュールの読み込みを避けて開発時のビルド速度を短縮することも可能です。

```typescript:例
// 🚀 速い
import Button from '@mui/material/Button';
import TextField from '@mui/material/TextField';
```

https://mui.com/guides/minimizing-bundle-size/

## GlobalStyles の利用

では、上のコンポーネントを `App.tsx` へインポートしてみましょう。

```diff jsx:src/App.tsx
+ import { ToolBar } from './ToolBar';
  import { TodoItem } from './TodoItem';
  import { FormDialog } from './FormDialog';
```

```diff jsx:src/App.tsx
    return (
      <div>
+       <ToolBar />
        <SideBar onFilter={handleFilter} />
        <FormDialog text={text} onChange={handleChange} onSubmit={handleSubmit} />
        <TodoItem todos={todos} filter={filter} onTodo={handleTodo} />
        <ActionButton todos={todos} onEmpty={handleEmpty} />
      </div>
    );
```

![](https://storage.googleapis.com/zenn-user-upload/52614c0ac9d7-20230222.png =480x)
_ブラウザデフォルトの margin, padding が当たっている_

上辺と左右に隙間が出来てしまいました。これは、グローバルスタイル（html, body など）がまだ当たっていないからです。 つまり、**MUI v5** では自動的には [CSS リセット](https://coliss.com/articles/build-websites/operation/css/css-reset-for-modern-browser.html)されません。

https://coliss.com/articles/build-websites/operation/css/css-reset-for-modern-browser.html

親コンポーネントへ **GlobalStyles** モジュールをインポートしましょう。

https://mui.com/api/global-styles/#main-content

```jsx:src/App.tsx
import GlobalStyles from '@mui/material/GlobalStyles';
```

グローバルスタイルを適用します。

```diff jsx:src/App.tsx
    return (
      <div>
+       <GlobalStyles styles={{ body: { margin: 0, padding: 0 } }} />
        <ToolBar />
```

![](https://storage.googleapis.com/zenn-user-upload/9e03b614fdfd-20230222.png =480x)
_ぴったりハマった_

## `filter ステート` を props として渡す

[chap.18](https://zenn.dev/sprout2000/books/76a279bb90c3f3/viewer/chapter18) で見た通り、ツールバーには **filter ステート** の状態が表示されなければいけません。
**ToolBar コンポーネント** へ `props` を渡します。

```diff jsx:src/App.tsx
  return (
    <div>
      <GlobalStyles styles={{ body: { margin: 0, padding: 0 } }} />
-     <ToolBar />
+     <ToolBar filter={filter} />
```

```diff jsx:src/ToolBar.tsx
+ type Props = {
+   filter: Filter;
+ };

+ export const ToolBar = (props: Props) => (
    <Box sx={{ flexGrow: 1 }}>

    {/* 省略 */}

          </IconButton>
-         <Typography>TODO</Typography>
+         <Typography>{props.filter}</Typography>
```

![](https://storage.googleapis.com/zenn-user-upload/a6f2c68eb60c-20230222.png =480x)

ただし、これでは英語のままの **filter ステート** の値が表示されるだけなので、これを書き換える関数をコンポーネント**外**へ加えます。
コンポーネントが受け取る _props_ などに依存しない関数は、コンポーネント外へ定義しておくことでコンポーネントの再レンダリングごとに再生成されることを防げます。

```jsx:src/ToolBar.tsx
const translator = (arg: Filter) => {
  switch (arg) {
    case 'all':
      return 'すべてのタスク';
    case 'unchecked':
      return '現在のタスク';
    case 'checked':
      return '完了したタスク';
    case 'removed':
      return 'ごみ箱';
    default:
      return 'TODO';
  }
};

export const ToolBar = (props: Props) => (
  {/* 省略 */}
        <Typography>{translator(props.filter)}</Typography>
      </Toolbar>
    </AppBar>
  </Box>
);
```

![](https://storage.googleapis.com/zenn-user-upload/d8a13385994b-20230222.png =480x)

## MUI のテーマ機能を利用する

**MUI v5** では、ツールバーのデフォルトの色が少し明るくなりました。テーマ機能を利用して、これを **v4** のような indigo へ変更します。

https://mui.com/customization/theming/#heading-theme-provider

テーマは React コンポーネントの**外側**で作成してください。

```jsx:src/App.tsx
// スタイルエンジンのモジュールとカラーをインポート
import { createTheme, ThemeProvider } from '@mui/material/styles';
import { indigo, pink } from '@mui/material/colors';

// テーマを作成
const theme = createTheme({
  palette: {
    // プライマリーカラー
    primary: {
      main: indigo[500],
      light: '#757de8',
      dark: '#002984',
    },
    // ついでにセカンダリーカラーも v4 に戻す
    secondary: {
      main: pink[500],
      light: '#ff6090',
      dark: '#b0003a',
    },
  },
});
```

JSX 全体を `<div>` の代わりに `<ThemeProvider>` タグでラップします。

```jsx:src/App.tsx
  return (
    <ThemeProvider theme={theme}>
      <GlobalStyles styles={{ body: { margin: 0, padding: 0 } }} />
      <ToolBar filter={filter} />
      {/* 略 */}
    </ThemeProvider>
  );
```

![](https://storage.googleapis.com/zenn-user-upload/63b91f45ea3a-20230222.png =480x)

カラーパレットの設定には以下のリンクが有用でしょう。

https://material.io/resources/color/#!/?view.left=0&view.right=0

## この章のソースコード全文

:::details ToolBar.tsx

```tsx:src/ToolBar.tsx
import Box from '@mui/material/Box';
import Icon from '@mui/material/Icon';
import AppBar from '@mui/material/AppBar';
import Toolbar from '@mui/material/Toolbar';
import Typography from '@mui/material/Typography';
import IconButton from '@mui/material/IconButton';

type Props = {
  filter: Filter;
};

const translator = (arg: Filter) => {
  switch (arg) {
    case 'all':
      return 'すべてのタスク';
    case 'unchecked':
      return '現在のタスク';
    case 'checked':
      return '完了したタスク';
    case 'removed':
      return 'ごみ箱';
    default:
      return 'TODO';
  }
};

export const ToolBar = (props: Props) => (
  <Box sx={{ flexGrow: 1 }}>
    <AppBar position="static">
      <Toolbar>
        <IconButton
          aria-label="menu-button"
          size="large"
          edge="start"
          color="inherit"
          sx={{ mr: 2 }}
        >
          <Icon>menu</Icon>
        </IconButton>
        <Typography>{translator(props.filter)}</Typography>
      </Toolbar>
    </AppBar>
  </Box>
);
```

:::

:::details App.tsx

```tsx:src/App.tsx
import { useState } from 'react';

import GlobalStyles from '@mui/material/GlobalStyles';
import { createTheme, ThemeProvider } from '@mui/material/styles';
import { indigo, pink } from '@mui/material/colors';

import { ToolBar } from './ToolBar';
import { SideBar } from './SideBar';
import { TodoItem } from './TodoItem';
import { FormDialog } from './FormDialog';
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
    <ThemeProvider theme={theme}>
      <GlobalStyles styles={{ body: { margin: 0, padding: 0 } }} />
      <ToolBar filter={filter} />
      <SideBar onFilter={handleFilter} />
      <FormDialog text={text} onChange={handleChange} onSubmit={handleSubmit} />
      <TodoItem todos={todos} filter={filter} onTodo={handleTodo} />
      <ActionButton todos={todos} onEmpty={handleEmpty} />
    </ThemeProvider>
  );
};
```

:::
