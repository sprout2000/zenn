---
title: "FormDialog コンポーネントとTodoItem コンポーネントを MUI で化粧直しする"
free: true
---

# コンポーネントを MUI 化する

これまで素の HTML タグで描画していた **FormDialog** と **TodoItem** の両コンポーネントに MUI コンポーネントを被せていきます。

## FormDialog コンポーネント

![](https://storage.googleapis.com/zenn-user-upload/77ec92bfe342-20211122.png =360x)
_FormDialog コンポーネント_

https://mui.com/components/dialogs/

まず、親コンポーネントへ三たびステートとそれを反転する関数を追加します。

```jsx:src/App.tsx
// ダイアログの状態を管理するステート
const [dialogOpen, setDialogOpen] = useState(false);
```

```jsx:src/App.tsx
const handleToggleDialog = () => {
  setDialogOpen((dialogOpen) => !dialogOpen);
  // フォームへの入力をクリア
  setText('');
};
```

このステートとコールバック関数を **FormDialog コンポーネント**や次章で作成する **ActionButton コンポーネント**へ `props` として渡します。

また、`handleSubmit` イベントが発火するとこのダイアログを閉じる必要があるため、親コンポーネントの `handleSubmit()` へ上のステートを反転させる処理を追加します。

```diff jsx:src/App.tsx
   const handleSubmit = () => {
     if (!text) {
       // 何も入力されなかったとき
+      setDialogOpen((dialogOpen) => !dialogOpen);
       return;
     }

     const newTodo: Todo = {
       value: text,
       id: new Date().getTime(),
       checked: false,
       removed: false,
     };

     setTodos((todos) => [newTodo, ...prevState]);
     setText('');
     // FormDialog コンポーネントを閉じる
+    setDialogOpen((dialogOpen) => !todos);
  };
```

基本的なロジックに変更はないので、ソースコードは章末に掲載するにとどめますが、ただ一つの注意点は、 **handleChange(e) のイベントの型が変わった**ことです。

```jsx:src/FormDialog.tsx
type Props = {
  text: string;
  dialogOpen: boolean;
  onSubmit: () => void;
  onChange: (
    e: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement>
  ) => void;
  onToggleDialog: () => void;
};
```

当然に親コンポーネントの呼び出し側関数にも修正が必要です。

```jsx:src/App.tsx
  const handleChange = (
    e: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement>
  ) => {
    setText(e.target.value);
  };
```

上記の `props` を呼び出し側の親コンポーネントにも追加しましょう。

```diff jsx:src/App.tsx
  <FormDialog
    text={text}
+   dialogOpen={dialogOpen}
    onChange={handleChange}
    onSubmit={handleSubmit}
+   onToggleDialog={handleToggleDialog}
  />
```

## TodoItem コンポーネント

![](https://storage.googleapis.com/zenn-user-upload/df2f87811903-20230206.png)
_TodoCard コンテナ (TodoItem コンポーネント)_

こちらでは特に注意すべき点はありません。素の HTML タグをそれぞれ以下の MUI コンポーネントへ着せ替えさせるだけです。

- `<input type="text">` => **TextField** コンポーネント
- `<input type="submit">` => **Button** コンポーネント

https://mui.com/components/text-fields/

https://mui.com/components/buttons/

### コンテナの作成

また、これまでの `<ul> ~ </ul>` タグに代えて、これらの **TodoCard** をラップするコンテナも作っておきましょう。

```jsx:src/TodoItem.tsx
const Container = styled('div')({
  margin: '0 auto',
  maxWidth: '640px',
  fontFamily: '-apple-system, BlinkMacSystemFont, Roboto, sans-serif',
});
```

```jsx:src/TodoItem.tsx
    <Container>
      {filteredTodos.map((todo) => {
        return (
          <TodoCard key={todo.id}>
        {/* 省略 */}
    </Container>
```

:::message
この時点では入力フォームを表示することができないので、次章の **Action Button** コンポーネントの再実装まで新しいタスクの追加はできません。
:::

![](https://storage.googleapis.com/zenn-user-upload/b03db1dafa1f-20230222.png =480x)
_途中経過_

## この章のソースコード全文

:::details FormDialog.tsx

```tsx:src/FormDialog.tsx
import Button from '@mui/material/Button';
import Dialog from '@mui/material/Dialog';
import TextField from '@mui/material/TextField';
import DialogActions from '@mui/material/DialogActions';

type Props = {
  text: string;
  dialogOpen: boolean;
  onSubmit: () => void;
  onChange: (
    e: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement>
  ) => void;
  onToggleDialog: () => void;
};

export const FormDialog = (props: Props) => (
  <Dialog fullWidth open={props.dialogOpen} onClose={props.onToggleDialog}>
    <form
      onSubmit={(e) => {
        e.preventDefault();
        props.onSubmit();
      }}
    >
      <div style={{ margin: '1em' }}>
        <TextField
          aria-label="todo-input"
          variant="standard"
          style={{
            width: '100%',
            fontSize: '16px',
            fontFamily: '-apple-system, BlinkMacSystemFont, Roboto, sans-serif',
          }}
          label="タスクを入力..."
          onChange={(e) => props.onChange(e)}
          value={props.text}
          autoFocus
        />
        <DialogActions>
          <Button
            aria-label="form-add"
            color="secondary"
            onClick={props.onSubmit}
          >
            追加
          </Button>
        </DialogActions>
      </div>
    </form>
  </Dialog>
);
```

:::

:::details TodoItem.tsx

```tsx:src/TodoItem.tsx
import Icon from '@mui/material/Icon';
import Card from '@mui/material/Card';
import TextField from '@mui/material/TextField';
import Typography from '@mui/material/Typography';

import { styled } from '@mui/material/styles';
import { lightBlue, pink, grey } from '@mui/material/colors';

type Props = {
  todos: Todo[];
  filter: Filter;
  onTodo: <T extends Todo['id'], U extends keyof Todo, V extends Todo[U]>(
    id: T,
    key: U,
    value: V
  ) => void;
};

const Container = styled('div')({
  margin: '0 auto',
  maxWidth: '640px',
  fontFamily: '-apple-system, BlinkMacSystemFont, Roboto, sans-serif',
});

const TodoCard = styled(Card)(({ theme }) => ({
  marginTop: theme.spacing(1),
  marginLeft: theme.spacing(2),
  marginRight: theme.spacing(2),
  padding: theme.spacing(1),
  fontFamily: '-apple-system, BlinkMacSystemFont, Roboto, sans-serif',
}));

const Form = styled('div')(({ theme }) => ({
  marginTop: theme.spacing(1),
  marginLeft: theme.spacing(1),
  marginRight: theme.spacing(1),
  fontSize: '16px',
}));

const ButtonContainer = styled('div')(({ theme }) => ({
  marginTop: theme.spacing(1),
  display: 'flex',
  flexDirection: 'row',
  justifyContent: 'space-between',
}));

const Button = styled('button')(() => ({
  display: 'flex',
  flexDirection: 'row',
  justifyContent: 'center',
  alignItems: 'center',
  background: 'none',
  border: 'none',
  cursor: 'pointer',
  outline: 'none',
}));

const Trash = styled('button')(() => ({
  background: 'none',
  border: 'none',
  cursor: 'pointer',
  outline: 'none',
}));

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
    <Container>
      {filteredTodos.map((todo) => {
        return (
          <TodoCard key={todo.id}>
            <Form>
              <TextField
                aria-label={`todo-${todo.value}`}
                fullWidth
                variant="standard"
                value={todo.value}
                onChange={(e) => props.onTodo(todo.id, 'value', e.target.value)}
                disabled={todo.checked || todo.removed}
              />
            </Form>
            <ButtonContainer>
              <Button
                aria-label={`todo-check-${todo.value}`}
                onClick={() => props.onTodo(todo.id, 'checked', !todo.checked)}
                disabled={props.filter === 'removed'}
              >
                {todo.checked ? (
                  <Icon
                    aria-label={`todo-removed-${todo.value}`}
                    style={{
                      color: props.filter !== 'removed' ? pink.A200 : grey[500],
                    }}
                  >
                    check_circle_outline
                  </Icon>
                ) : (
                  <Icon
                    aria-label={`todo-uncheck-${todo.value}`}
                    style={{
                      color:
                        props.filter !== 'removed' ? lightBlue[500] : grey[500],
                    }}
                  >
                    radio_button_unchecked
                  </Icon>
                )}
                <Typography
                  style={{
                    userSelect: 'none',
                    color:
                      todo.checked && props.filter !== 'removed'
                        ? pink.A200
                        : grey[500],
                  }}
                >
                  Done
                </Typography>
              </Button>
              <Trash
                aria-label={`todo-trash-${todo.value}`}
                onClick={() => props.onTodo(todo.id, 'removed', !todo.removed)}
              >
                {todo.removed ? (
                  <Icon
                    aria-label={`todo-undo-${todo.value}`}
                    style={{ color: lightBlue[500] }}
                  >
                    undo
                  </Icon>
                ) : (
                  <Icon
                    aria-label={`todo-delete-${todo.value}`}
                    style={{ color: grey[500] }}
                  >
                    delete
                  </Icon>
                )}
              </Trash>
            </ButtonContainer>
          </TodoCard>
        );
      })}
    </Container>
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
      <TodoItem todos={todos} filter={filter} onTodo={handleTodo} />
      <ActionButton todos={todos} onEmpty={handleEmpty} />
    </ThemeProvider>
  );
};
```

:::
