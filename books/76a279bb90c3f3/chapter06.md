---
title: "配列ステートの操作には要注意"
free: true
---

## 配列ステートを操作する

前章で作成した `Todos ステート`は **`Todo型`オブジェクトの配列**ですが、配列のステートを操作しなければならない場合、そのステート配列を**直接触ってはいけません**。

例えば、以下のようなステート操作は行ってはいけません。

```js
const [todos, setTodos] = useState([{ value: "現在のタスク" }]);

// ❌ やってはいけない
// ステート配列の末尾に新しい要素を直接に追加
todos.push({ value: "新しいタスク" });
```

:::message alert
原則 1: **「`setState` メソッドを利用しないでステートの値を書き換えてはいけない」**
原則 2: **「配列ステートを直接触ってはいけない」**
:::

その理由は、「ステートの**イミュータビリティ**（**immutability**, 不変性）が保持できなくなる」からですが、これを詳しく見ていきましょう。

## イミュータブルな操作とは？

イミュータブルな操作とは、その操作の対象となった元の値を**不変**（＝イミュータブル）**に保つ**操作のことです。

```javascript
const array1 = [0, 1, 2];
const array2 = [0, 1, 2];

// Array.push()
console.log(array1.push(3)); // --> 4

// スプレッド構文（次章でも解説します）
console.log([...array2, 3]); // --> ▶️(4) [0, 1, 2, 3]
```

上の例での下 2 行では、どちらもそれぞれの元の配列に `3` という要素を追加しています。

![](https://storage.googleapis.com/zenn-user-upload/f62854b5d8ec-20230121.png =400x)
_`3` を追加して出力_

では、元の配列の値はどうなったでしょうか？

```javascript:結果
console.log(array1);
▶️(4) [0, 1, 2, 3]

console.log(array2);
▶️(3) [0, 1, 2]
```

![](https://storage.googleapis.com/zenn-user-upload/ad48353f4aed-20230121.png =400x)
_元の配列の値が変わってしまっている_

`Array.push()` メソッドが元の配列を **変更（＝ミューテート）** してしまったのに対し、スプレッド構文を使った要素の追加では元の配列の **イミュータビリティ（＝不変性）** が保たれています。

ここでの `Array.push()` メソッドが「ミュータブルな操作」、スプレッド構文が「イミュータブルな操作」です。

https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/push

https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Spread_syntax

## なぜ React ではイミュータブルな操作が必要なのか?

では、なぜ React ではイミュータブルな操作が必要とされるのでしょうか？

パフォーマンス上の理由や再レンダリングについての予測可能性のため、React は**コンポーネントの変化をオブジェクトの同一性（差分）チェック**で検知しています。

ミュータブルな操作をしてしまうとコピー元の情報も変更されてしまうため、変更前と変更後の差分を React が検知できなくなってしまいます。

一方、イミュータブルな操作では変更前と変更後の情報をそれぞれ参照しているので、React は差分を検知できます。
また、元のステートが変更されていないので、それを履歴として残すことでその段階まで巻き戻すような機能の実装も可能となります。

ふたたび上でも挙げた例で言うと、`todos` ステートへの以下のような操作はいけません。

```js
// ❌ やってはいけない
// 元配列を変更するかたちでステートを更新
const newTodos = todos.unshift({ value: "新しいタスク" });
setTodos(newTodos);
```

なぜなら、`Array.prototype.push()` や `Array.prototype.unshift()` は破壊的メソッドなので `todos` ステートを直接ミューテート（**mutate**, 書き換え）してしまうからです。

https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/unshift

配列のステートを操作する場合には、 いったんその**コピー**に対して変更を加え、その**変更後のコピーでステートを更新**します。

```jsx:コピーを操作
const todos = [{ value: '最初のタスク' }];

// todos 配列をコピー
const newTodos = todos.slice();
/**
 * またはスプレッド構文（後述）で
 * const newTodos = [...todos]
*/

// コピーした配列へ Todo型オブジェクトの要素を追加
newTodos.unshift({ value: '新しいタスク' });

// それぞれの配列の内容を確認
console.log('=== old todos ===');
console.log(JSON.stringify(todos));

console.log('=== new todos ===');
console.log(JSON.stringify(newTodos));

/**
 * 結果:
 * === old todos ===
 * [{"value":"最初のタスク"}]
 *
 * === new todos ===
 * [{"value":"新しいタスク"},{"value":"最初のタスク"}]
 *
 * 元の配列 (=todos) に影響を与えることなく、
 * コピーした配列 (=newTodos) へ要素が追加されている
 **/
```

こうすることで、更新前のステートである元の配列と、新しいステートの差分を React が検知できるようになります。

![](https://storage.googleapis.com/zenn-user-upload/12dc60f5e80d-20230121.png =400x)
_コピーに変更を加えても元配列には影響がない_

## 参考記事

- [イミュータビリティは何故重要なのか](https://ja.react.dev/learn/tutorial-tic-tac-toe#why-immutability-is-important)（React 公式チュートリアル）
