---
title: "todos ステートを展開してページに表示する"
free: true
---

## `todos` ステート配列を `Array.map()` メソッドで展開する

`todos` ステートを展開し、タスク一覧としてページに表示します。
具体的には、**`todos （＝配列）`** を非破壊メソッドである [Array.prototype.map()](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/map) を使って `<li> ~ </li>` タグへ展開します。

`Array.map()` メソッドは、与えられた関数を配列のすべての要素に対して呼び出し、その結果からなる**新しい配列を生成**します。

```js:Array.map()
// 例:
const items = [0, 1, 2];
const newItems = items.map((item) => item * 2); // --> [0, 2, 4]

/** for 文で同義を書いた場合 */
const newItems = [];
for (let i = 0; i < items.length; i++) {
  newItems.push(items[i] * 2);
}
```

https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/map

```jsx:src/App.tsx
    <div>
      <form onSubmit={/* 省略 */}>
        {/* 省略 */}
      </form>
      <ul>
        {todos.map((todo) => {
          return <li>{todo.value}</li>;
        })}
      </ul>
    </div>
```

![](https://storage.googleapis.com/zenn-user-upload/3d5d699e5d5e-20230222.png =480x)

ただし、これだけでは各 `<li>` に **key** プロパティが設定されていないため、以下のような警告が表示されてしまいます。

[チュートリアル：React の導入 - key を選ぶ（公式）](https://ja.react.dev/learn/tutorial-tic-tac-toe#picking-a-key)

![](https://storage.googleapis.com/zenn-user-upload/14df5ec9ee85-20230222.png =640x)

> 警告：リストのそれぞれの要素は一意な _key_ プロパティを持っているべきです

## リストをレンダーするときの _key_ の重要性

なぜリストの各項目に _key_ プロパティが必要となるのでしょうか？

React はリストをレンダーする際、どのアイテムが変更になったのか特定できる必要があります。リストのアイテムは追加された可能性も、削除された可能性も、並び替えられた可能性も、中身自体が変更になった可能性もあるからです。

変更・追加・削除・並び替えを検知するためには、リストの各項目を特定する一意な識別子が必要です。

この一意な識別子こそが _key_ プロパティであり、上の警告は『各項目を特定できないため、リストに変更が加えられても**正しく再レンダーできない**可能性があります』という意味で表示されているのです。

https://ja.react.dev/learn/rendering-lists#rules-of-keys

次章では、この _key_ プロパティを各項目へ与えるため、`Todo型`オブジェクトの仕様について再考します。
