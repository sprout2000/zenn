---
title: "AlertDialog コンポーネントの作成と ActionButton コンポーネントのアップデート"
free: true
---

# 残りのコンポーネントを完成させる

前章で作成した **FormDialog コンポーネント** を呼び出す **ActionButton コンポーネント** の再実装と、「ごみ箱を空にする」が実行されたときに表示する **AlertDialog コンポーネント** の作成を行なっていきましょう。

## AlertDialog コンポーネント

次項の **ActionButton** に必要となるため、まず **AlertDialog** を先に作成します。

![](https://storage.googleapis.com/zenn-user-upload/fc98aa17022c-20211122.png =360x)
_AlertDialog コンポーネント_

https://mui.com/components/dialogs/

特に注意すべき点はありませんが、以下のステートとコールバック関数を親コンポーネントに追加する必要があります。

```tsx:src/App.tsx
const [alertOpen, setAlertOpen] = useState(false);
```

```tsx:src/App.tsx
const handleToggleAlert = () => {
  setAlertOpen((alertOpen) => !alertOpen);
};
```

```tsx:src/App.tsx
import { AlertDialog } from './AlertDialog';
```

```diff tsx:src/App.tsx
+      <AlertDialog
+        alertOpen={alertOpen}
+        onEmpty={handleEmpty}
+        onToggleAlert={handleToggleAlert}
+      />
       <TodoItem todos={todos} filter={filter} onTodo={handleTodo} />
```

## ActionButton コンポーネント

![](https://storage.googleapis.com/zenn-user-upload/34e0b0b19b4a-20211122.png =360x)
_ActionButoon コンポーネント_

https://mui.com/components/floating-action-button/

こちらも前項と同様です。
ごみ箱が空かどうか知るために `todos ステート` を `props` にとり、アイコン表示に反映させます。

```tsx:src/ActionButton.tsx
const removed = props.todos.filter((todo) => todo.removed).length !== 0;
```

親コンポーネントにインポートするにあたっては、`AlertDialog` と `FormDialog` の表示・非表示をコントロールすると同時に、それらの状態に対応して自身のアイコンも変化させるため、多くの `props` が必要となります。

```tsx:src/App.tsx
import { ActionButton } from './ActionButton';
```

```diff tsx:src/App.tsx
       <TodoItem todos={todos} filter={filter} onTodo={handleTodo} />
       <ActionButton
+        todos={todos}
+        filter={filter}
+        alertOpen={alertOpen}
+        dialogOpen={dialogOpen}
+        onToggleAlert={handleToggleAlert}
+        onToggleDialog={handleToggleDialog}
-        onEmpty={handleEmpty}
        />
     </ThemeProvider>
```

## 本章のソースコード全文

:::details AlertDialog.tsx

```tsx:src/AlertDialog.tsx
import Button from '@mui/material/Button';
import Dialog from '@mui/material/Dialog';
import DialogTitle from '@mui/material/DialogTitle';
import DialogActions from '@mui/material/DialogActions';
import DialogContent from '@mui/material/DialogContent';
import DialogContentText from '@mui/material/DialogContentText';

import { styled } from '@mui/material/styles';

type Props = {
  alertOpen: boolean;
  onEmpty: () => void;
  onToggleAlert: () => void;
};

const Alert = styled(Dialog)(() => ({
  fontFamily: '-apple-system, BlinkMacSystemFont, Roboto, sans-serif',
}));

export const AlertDialog = (props: Props) => (
  <Alert open={props.alertOpen} onClose={props.onToggleAlert}>
    <DialogTitle>アラート</DialogTitle>
    <DialogContent>
      <DialogContentText>本当にごみ箱を完全に空にしますか？</DialogContentText>
      <DialogContentText>この操作は取り消しできません。</DialogContentText>
    </DialogContent>
    <DialogActions>
      <Button
        aria-label="alert-cancel"
        onClick={props.onToggleAlert}
        color="primary"
      >
        キャンセル
      </Button>
      <Button
        aria-label="alert-ok"
        onClick={() => {
          props.onToggleAlert();
          props.onEmpty();
        }}
        color="secondary"
        autoFocus
      >
        OK
      </Button>
    </DialogActions>
  </Alert>
);
```

:::

:::details ActionButton.tsx

```tsx:src/ActionButton.tsx
import Fab from '@mui/material/Fab';
import Icon from '@mui/material/Icon';

import { styled } from '@mui/material/styles';

type Props = {
  todos: Todo[];
  filter: Filter;
  alertOpen: boolean;
  dialogOpen: boolean;
  onToggleAlert: () => void;
  onToggleDialog: () => void;
};

const FabButton = styled(Fab)({
  position: 'fixed',
  right: 15,
  bottom: 15,
});

export const ActionButton = (props: Props) => {
  const removed = props.todos.filter((todo) => todo.removed).length !== 0;

  return (
    <>
      {props.filter === 'removed' ? (
        <FabButton
          aria-label="fab-delete-button"
          color="secondary"
          onClick={props.onToggleAlert}
          disabled={!removed || props.alertOpen}
        >
          <Icon>delete</Icon>
        </FabButton>
      ) : (
        <FabButton
          aria-label="fab-add-button"
          color="secondary"
          onClick={props.onToggleDialog}
          disabled={props.filter === 'checked' || props.dialogOpen}
        >
          <Icon>create</Icon>
        </FabButton>
      )}
    </>
  );
};
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

  const handleSort = (filter: Filter) => {
    setFilter(filter);
  };

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
