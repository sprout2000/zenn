---
title: "QR コンポーネントの作成"
free: true
---

# QR コード表示機能を追加

このウェブアプリの URL を読み取ってもらうための QR コードを表示します。

![](https://storage.googleapis.com/zenn-user-upload/c01760fc3511-20211122.png =360x)
_QR コンポーネント_

## ふたたびステートの追加

QR の表示状態を管理するため、親コンポーネントへステートを追加します。

```jsx:src/App.tsx
const [qrOpen, setQrOpen] = useState(false);
```

そして前章同様に表示・非表示を切り替える関数も作成します。

```jsx:src/App.tsx
const handleToggleQR = () => {
  setQrOpen((qrOpen) => !qrOpen);
};
```

## react-qrcode-logo ライブラリを追加

React で容易に QR を表示できる [react-qrcode-logo](https://www.npmjs.com/package/react-qrcode-logo) という素晴らしいライブラリがありますのでインストールしましょう。

https://www.npmjs.com/package/react-qrcode-logo

```sh:zsh
npm install react-qrcode-logo
```

## QR コンポーネントの作成

もう、とくに説明はいらないかと思われます。

```jsx:src/QR.tsx
import { QRCode } from 'react-qrcode-logo';

import Backdrop from '@mui/material/Backdrop';
import { styled } from '@mui/material/styles';

const TodoBackdrop = styled(Backdrop)(({ theme }) => ({
  zIndex: theme.zIndex.drawer + 1,
  color: '#fff',
  backgroundColor: 'rgba(0, 0, 0, 0.8)',
}));

type Props = {
  open: boolean;
  onClose: () => void;
};

export const QR = (props: Props) => (
  <TodoBackdrop open={props.open} onClick={props.onClose}>
    <QRCode value="https://sprout2000.github.io/todo" />
  </TodoBackdrop>
);
```

https://mui.com/components/backdrop/

```tsx:src/App.tsx
import { QR } from './QR';
```

```diff tsx:src/App.tsx
       <SideBar
         drawerOpen={drawerOpen}
         onFilter={handleFilter}
         onToggleDrawer={handleToggleDrawer}
       />
+      <QR open={qrOpen} onClose={handleToggleQR} />
       <FormDialog
         text={text}
         onChange={handleChange}
         onSubmit={handleSubmit}
        />
```

## SideBar コンポーネントのリストに追加

QR を開くボタンを **SideBar コンポーネント** のリストへ追加し、クリックイベントへ `props` の関数を紐づけます。

![](https://storage.googleapis.com/zenn-user-upload/4062390e86ac-20230222.png =480x)

```diff tsx:src/SideBar.tsx
 type Props = {
   drawerOpen: boolean;
+  onToggleQR: () => void;
   onToggleDrawer: () => void;
   onFilter: (filter: Filter) => void;
 };
```

```diff tsx:src/SideBar.tsx
     <Divider />
+    <ListItem disablePadding>
+      <ListItemButton aria-label="list-share" onClick={props.onToggleQR}>
+        <ListItemIcon>
+          <Icon>share</Icon>
+        </ListItemIcon>
+        <ListItemText secondary="このアプリを共有" />
+      </ListItemButton>
+    </ListItem>
   </List>
```

```tsx:src/App.tsx
    {/* snip */}
    <SideBar
      drawerOpen={drawerOpen}
      onFilter={handleFilter}
      // ↓追加
      onToggleQR={handleToggleQR}
      onToggleDrawer={handleToggleDrawer}
    />
    <QR open={qrOpen} onClose={handleToggleQR} />
    {/* snip */}
```

## この章のソースコード全文

:::details QR.tsx

```tsx:src/QR.tsx
import { QRCode } from 'react-qrcode-logo';

import Backdrop from '@mui/material/Backdrop';
import { styled } from '@mui/material/styles';

const TodoBackdrop = styled(Backdrop)(({ theme }) => ({
  zIndex: theme.zIndex.drawer + 1,
  color: '#fff',
  backgroundColor: 'rgba(0, 0, 0, 0.8)',
}));

type Props = {
  open: boolean;
  onClose: () => void;
};

export const QR = (props: Props) => (
  <TodoBackdrop open={props.open} onClick={props.onClose}>
    <QRCode value="https://sprout2000.github.io/todo" />
  </TodoBackdrop>
);
```

:::

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
  onToggleQR: () => void;
  onToggleDrawer: () => void;
  onFilter: (filter: Filter) => void;
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
            onClick={() => props.onFilter('all')}
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
            onClick={() => props.onFilter('unchecked')}
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
            onClick={() => props.onFilter('checked')}
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
            onClick={() => props.onFilter('removed')}
          >
            <ListItemIcon>
              <Icon>delete</Icon>
            </ListItemIcon>
            <ListItemText secondary="ごみ箱" />
          </ListItemButton>
        </ListItem>
        <Divider />
        <ListItem disablePadding>
          <ListItemButton aria-label="list-share" onClick={props.onToggleQR}>
            <ListItemIcon>
              <Icon>share</Icon>
            </ListItemIcon>
            <ListItemText secondary="このアプリを共有" />
          </ListItemButton>
        </ListItem>
      </List>
    </DrawerList>
  </Drawer>
);
```

:::

:::details App.tsx

```tsx:src/App.tsx
import { useState } from 'react';

import GlobalStyles from '@mui/material/GlobalStyles';
import { createTheme, ThemeProvider } from '@mui/material/styles';
import { indigo, pink } from '@mui/material/colors';

import { QR } from './QR';
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

  const [qrOpen, setQrOpen] = useState(false);
  const [drawerOpen, setDrawerOpen] = useState(false);

  const handleToggleDrawer = () => {
    setDrawerOpen((drawerOpen) => !drawerOpen);
  };

  const handleToggleQR = () => {
    setQrOpen((qrOpen) => !qrOpen);
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
        onFilter={handleFilter}
        onToggleQR={handleToggleQR}
        onToggleDrawer={handleToggleDrawer}
      />
      <QR open={qrOpen} onClose={handleToggleQR} />
      <FormDialog text={text} onChange={handleChange} onSubmit={handleSubmit} />
      <TodoItem todos={todos} filter={filter} onTodo={handleTodo} />
      <ActionButton todos={todos} onEmpty={handleEmpty} />
    </ThemeProvider>
  );
};
```

:::
