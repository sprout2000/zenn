---
title: "Todo（タスク）の仕様を考える (その 1)"
free: true
---

## タスクの構成要素

ひとつのタスク (todo) を JavaScript オブジェクトと考えると、そのオブジェクトにはタスクの内容を保持するプロパティ（変数）が必要となります。ここでは、これを **`value`** プロパティとします。

```ts
// タスクを表すオブジェクト
const todo01 = {
  value: "美容院へ行く",
};
```

Todo 入力フォームに入力された**テキスト文字列**が代入されるので、この **`value`** プロパティは **`string型`** となります。

これから作成される複数の todo のひな型として **`Todo型`オブジェクト** の[型エイリアス](https://typescriptbook.jp/reference/values-types-variables/type-alias)を定義しましょう。型エイリアスは、 以下のように **type** キーワードを使って定義します。

```jsx:App.tsx
// "Todo型" の定義
type Todo = {
  // プロパティ value は文字列型
  value: string;
};

export const App = () => { /* ... */}
```

型エイリアスは、既存の型を再利用して新しい型をカスタム定義する方法です。型エイリアスを使用すると、複雑な型を定義することができ、コードの可読性を向上させることができます。

なお、型エイリアスの名前は**大文字**（[アッパー・キャメルケース](https://qiita.com/deerboy/items/f035b9044edf9a51aff7)）で始まるようにすることを強く推奨します。これは **「値ではなく、型である」** ことを明示するためです。

また、**`: string`** のような部分を[型注釈](https://typescriptbook.jp/reference/values-types-variables/type-annotation)といい、その変数にどんな値が代入可能かを指定したり、関数の場合には引数と戻り値の型を指定したりできます。

```ts:型注釈
const price: number = 1000; // <-- 数値型
const isCheap: boolean = true; // <-- 論理型

// 引数に文字列型と数値型を取り、文字列型を返す
function treat(name: string, times: number): string => {
  return `${name} さんに食事を ${times} 回ごちそうしました`
}
```

## タスクリストを保持するステートの作成

複数のタスクをリストとして保持しておくステートも作成しましょう。

### `todos` ステート

このタスクリスト、つまり複数の _todo_ は、**`Todo型`オブジェクトの配列: `Todo[]`** となります。

![](https://storage.googleapis.com/zenn-user-upload/86645df9f8ac-20230320.png =360x)

```jsx:App.tsx
export const App = () => {
  const [text, setText] = useState('');
  // 追加
  const [todos, setTodos] = useState<Todo[]>([]);

  return (<div>{/* ... */}</div>)
```

**`useState<>`** の中に型を指定しておくと、これと型が異なる値をステートに代入できなくなるため、ステートの[型安全性](https://typescript-jp.gitbook.io/deep-dive/getting-started/why-typescript)が常に保証されます。

https://typescript-jp.gitbook.io/deep-dive/getting-started/why-typescript

上記のコードで言うと、`todos` ステートの初期値は空配列 **`[]`** となっていますが、このステートには **Todo 型オブジェクトの配列**以外の値を代入することができません。
例えば、以下のような `setState` メソッドの実行はエラーとなります。

```jsx
// ❌ Todo型の配列じゃない
setTodos([0, 1, 2]);
```

```jsx
// 🟢 これは OK!
setTodos([{ valude: "次のタスク" }, { value: "最初のタスク" }]);
```
