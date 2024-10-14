---
title: "SideBar コンポーネントの MUI 化"
free: true
---

# コンポーネントを MUI 化する

引き続いて **SideBar コンポーネント** を MUI 化しましょう。
先に見た通り、ツールバーのメニューボタンをクリックすることでドロワーを表示します。

![](https://storage.googleapis.com/zenn-user-upload/ffb75096a9a8-20211122.png =360x)
_SideBar コンポーネント_

## ステートの追加

まずはドロワーの状態を管理するため、親コンポーネントである `App.tsx` へステートを追加します。

```jsx:src/App.tsx
const [drawerOpen, setDrawerOpen] = useState(false);
```

そして、このステートを反転する関数も作成します。

```jsx:src/App.tsx
const handleToggleDrawer = () => {
  setDrawerOpen((drawerOpen) => !drawerOpen);
};
```

これらを **SideBar コンポーネント** の `props` としましょう。

```jsx:src/SideBar.tsx
type Props = {
  drawerOpen: boolean;
  onToggleDrawer: () => void;
  onSort: (filter: Filter) => void;
};
```

## SideBar コンポーネントの作成

では、**SideBar コンポーネント** 本体を全面的に書き換えましょう。

### styled の利用

**MUI v5** においても、**styled-components** ライクな `styled` モジュール を利用して、必要なカスタムパーツを作成できます。

```jsx:構文
const CustomPart = styled({/* HTML タグ or MUI モジュール */})(() => {
  // カスタマイズ内容
})
```

https://mui.com/system/styled/

このようなカスタムパーツも、テーマの場合と同様にコンポーネントの**外側**で作成します。

```jsx:src/SideBar.tsx
// モジュールとカラーのインポート
import { styled } from '@mui/material/styles';
import { indigo, lightBlue, pink } from '@mui/material/colors';

// カスタマイズする MUI モジュールをインポート
import Avatar from '@mui/material/Avatar';

// ドロワー内リストの幅をカスタマイズ
const DrawerList = styled('div')(() => ({
  width: 250,
}));

// ドロワーヘッダーのサイズ・色などをカスタマイズ
const DrawerHeader = styled('div')(() => ({
  height: 150,
  display: 'flex',
  flexDirection: 'column',
  justifyContent: 'center',
  alignItems: 'center',
  padding: '1em',
  backgroundColor: indigo[500],
  color: '#ffffff',
  fontFamily: '-apple-system, BlinkMacSystemFont, Roboto, sans-serif',
}));

// ヘッダー内に表示するアバターのカスタマイズ
const DrawerAvatar = styled(Avatar)(({ theme }) => ({
  backgroundColor: pink[500],
  width: theme.spacing(6),
  height: theme.spacing(6),
}));

// ...
```

### ドロワー本体を作成

ドロワー本体とその中のリストは、以下のようになります。
手順は前章の **ToolBar コンポーネント** と同様なので詳しい説明は省きますが、なるべく自分の手で MUI コンポーネントを実装してみてください。

:::message
章末にソースコード全文を掲載しています。
:::

https://mui.com/material-ui/react-list/

```tsx:src/SideBar.tsx
// ...

// バージョンを `package.json` の "version" エントリから取得
import pjson from '../package.json';

export const SideBar = (props: Props) => (
  <Drawer
    variant="temporary"
    // 開閉状態のステートからの props
    open={props.drawerOpen}
    // 開閉トグルのための props
    onClose={props.onToggleDrawer}
  >
    {/* カスタムパーツ DrawerList */}
    <DrawerList role="presentation" onClick={props.onToggleDrawer}>
      {/* ヘッダーとアバター */}
      <DrawerHeader>
        <DrawerAvatar>
          <Icon>create</Icon>
        </DrawerAvatar>
        {/* バージョン表示 */}
        <p>TODO v{pjson.version}</p>
      </DrawerHeader>
      {/* リスト （元の <select>~</select>） */}
      <List>
        <ListItem disablePadding>
          {/* フィルター選択のための props を設定 */}
          <ListItemButton
            aria-label="list-all"
            onClick={() => props.onSort('all')}
          >
            <ListItemIcon>
              <Icon>subject</Icon>
            </ListItemIcon>
            <ListItemText secondary="すべてのタスク" />
          </ListItemButton>
        </ListItem>
        {/* 以下、同様なので省略 */}
        {/* ... */}
        <Divider />  {/* <-- 区切り線 */}
      </List>
      {/* リストここまで */}
    </DrawerList>
  </Drawer>
);
```

### 親コンポーネントへインポート

```tsx:App.tsx
import { SideBar } from './SideBar';
```

```diff tsx:App.tsx
       <GlobalStyles styles={{ body: { margin: 0, padding: 0 } }} />
       <ToolBar filter={filter} />
+      <SideBar
+        drawerOpen={drawerOpen}
+        onToggleDrawer={handleToggleDrawer}
+        onSort={handleFilter}
+      />
       <FormDialog
         text={text}
         onChange={handleChange}
         onSubmit={handleSubmit}
       />
       <TodoItem todos={todos} filter={filter} onTodo={handleTodo} />
```

https://mui.com/components/drawers/

## ツールバーのメニューボタンからドロワーを開く

親コンポーネントから、 ドロワーをトグルする関数 `onToggleDrawer()` を **ToolBar コンポーネント** へ `props` として渡します。

```jsx:src/App.tsx
<ToolBar filter={filter} onToggleDrawer={handleToggleDrawer} />
```

```diff jsx:src/ToolBar.tsx
  // props を追加して...
  type Props = {
    filter: Filter;
+   onToggleDrawer: () => void;
  };
```

```diff jsx:src/Toolbar.tsx
         // メニューボタンへ紐付け
          <Toolbar>
            <IconButton
              aria-label="menu-button"
              size="large"
              edge="start"
              color="inherit"
              sx={{ mr: 2 }}
+             onClick={props.onToggleDrawer}
            >
              <Icon>menu</Icon>
            </IconButton>
```

![](https://storage.googleapis.com/zenn-user-upload/60e590409010-20230222.png =480x)
_途中経過_

## この章のソースコード全文

:::details SideBar.tsx

```tsx:src/SideBar.tsx
import Icon from '@mui/material/Icon';
import List from '@mui/material/List';
import Avatar from '@mui/material/Avatar';
import Drawer from '@mui/material/Drawer';
import Divider from '@mui/material/Divider';
import ListItem from '@mui/material/ListItem';
import ListItemIcon from '@mui/material/ListItemIcon';
import ListItemText from '@mui/material/ListItemText';
import ListItemButton from '@mui/material/ListItemButton';

import { styled } from '@mui/material/styles';
import { indigo, lightBlue, pink } from '@mui/material/colors';

import pjson from '../package.json';

type Props = {
  drawerOpen: boolean;
  onToggleDrawer: () => void;
  onSort: (filter: Filter) => void;
};

const DrawerList = styled('div')(() => ({
  width: 250,
}));

const DrawerHeader = styled('div')(() => ({
  height: 150,
  display: 'flex',
  flexDirection: 'column',
  justifyContent: 'center',
  alignItems: 'center',
  padding: '1em',
  backgroundColor: indigo[500],
  color: '#ffffff',
  fontFamily: '-apple-system, BlinkMacSystemFont, Roboto, sans-serif',
}));

const DrawerAvatar = styled(Avatar)(({ theme }) => ({
  backgroundColor: pink[500],
  width: theme.spacing(6),
  height: theme.spacing(6),
}));

export const SideBar = (props: Props) => (
  <Drawer
    variant="temporary"
    open={props.drawerOpen}
    onClose={props.onToggleDrawer}
  >
    <DrawerList role="presentation" onClick={props.onToggleDrawer}>
      <DrawerHeader>
        <DrawerAvatar>
          <Icon>create</Icon>
        </DrawerAvatar>
        <p>TODO v{pjson.version}</p>
      </DrawerHeader>
      <List>
        <ListItem disablePadding>
          <ListItemButton
            aria-label="list-all"
            onClick={() => props.onSort('all')}
          >
            <ListItemIcon>
              <Icon>subject</Icon>
            </ListItemIcon>
            <ListItemText secondary="すべてのタスク" />
          </ListItemButton>
        </ListItem>
        <ListItem disablePadding>
          <ListItemButton
            aria-label="list-unchecked"
            onClick={() => props.onSort('unchecked')}
          >
            <ListItemIcon>
              <Icon sx={{ color: lightBlue[500] }}>radio_button_unchecked</Icon>
            </ListItemIcon>
            <ListItemText secondary="現在のタスク" />
          </ListItemButton>
        </ListItem>
        <ListItem disablePadding>
          <ListItemButton
            aria-label="list-checked"
            onClick={() => props.onSort('checked')}
          >
            <ListItemIcon>
              <Icon sx={{ color: pink.A200 }}>check_circle_outline</Icon>
            </ListItemIcon>
            <ListItemText secondary="完了したタスク" />
          </ListItemButton>
        </ListItem>
        <ListItem disablePadding>
          <ListItemButton
            aria-label="list-removed"
            onClick={() => props.onSort('removed')}
          >
            <ListItemIcon>
              <Icon>delete</Icon>
            </ListItemIcon>
            <ListItemText secondary="ごみ箱" />
          </ListItemButton>
        </ListItem>
        <Divider />
      </List>
    </DrawerList>
  </Drawer>
);
```

:::

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
  onToggleDrawer: () => void;
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
          onClick={props.onToggleDrawer}
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

  const [drawerOpen, setDrawerOpen] = useState(false);

  const handleToggleDrawer = () => {
    setDrawerOpen((drawerOpen) => !drawerOpen);
  };

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
      <ToolBar filter={filter} onToggleDrawer={handleToggleDrawer} />
      <SideBar
        drawerOpen={drawerOpen}
        onSort={handleFilter}
        onToggleDrawer={handleToggleDrawer}
      />
      <FormDialog text={text} onChange={handleChange} onSubmit={handleSubmit} />
      <TodoItem todos={todos} filter={filter} onTodo={handleTodo} />
      <ActionButton todos={todos} onEmpty={handleEmpty} />
    </ThemeProvider>
  );
};
```

:::
