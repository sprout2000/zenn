---
title: "（補遺）なぜコールバック関数をカッコ () 付きで渡すとダメなのか？"
free: true
---

## なぜカッコ `()` 付きで渡すとダメなのか？

第 7 章で『イベントリスナーに渡す関数は、 アロー関数の `() => hoge()` もしくは 引数なしの `hoge` そのものです』と述べました。ここでは、その理由をもう少し詳しく見ていきましょう。

JavaScript では、**関数も値の一つ**です。奇妙にも感じられますが、**値**なので他の変数に代入することも、他の関数の引数とすることも可能となります。

```js
// 関数宣言
function five() {
  return 5;
}

// 関数宣言も値なので変数へ代入可能
const myFive = five;

// そもそもアロー関数式で定義、つまり変数に代入しても良い
const arrowFive = () => 5;
```

**関数を値として扱うには後ろのカッコ **`()`** をつけず**に書きます (=**`five`**)。逆に、カッコを付けると関数の中の処理の呼び出しとなります。

```js
const copy = five; // <-- ただの「値」の代入なのでコピーされるだけ
const answer = five(); // --> 関数が実行されて、結果は "answer:5" となる
```

そして[非同期処理](https://developer.mozilla.org/ja/docs/Learn/JavaScript/Asynchronous/Introducing)であるイベントリスナーは、「**値としての関数**を引数として受け取る関数」（高階関数といいます）に該当します。

https://jsprimer.net/basic/async/

つまり、イベントリスナーでは**値としての関数（アロー関数含む）を登録するのみ**で、まだ実行はしません。クリックなどのイベントが発火して初めて登録済みの関数を実行するわけです。

```js:例
// イベントハンドラー
const handleClick = () => console.log('Hello!');

// 🟢 クリックイベントの発火に備え、コールバック関数を "値" として登録しておく
button.addEventListener('click', handleClick);
button.addEventListener('click', () => handleClick());

// ❌ カッコを付けるとクリックされる前に、すぐに関数内の処理が呼び出されてしまう
button.addEventListener('click', handleClick());
button.addEventListener('click', console.log('Hello!'));
```

これが、イベントリスナーにカッコ **`()`** 付きで関数を与えない、つまり関数の即時実行として渡さない理由です。

さらに React コンポーネントにおいては、それが `setState` メソッドを実行する関数だった場合、コンポーネントの再レンダリングをトリガーするため、「レンダー → `setState` の即時実行 → レンダー → （以下略）」の無限ループとなってしまう訳です。

https://sbfl.net/blog/2019/02/08/javascript-callback-func/

:::message
JSX 内でのイベントリスナーの取り扱いについては、[chap.26](https://zenn.dev/sprout2000/books/76a279bb90c3f3/viewer/chapter26) 章末の囲み記事も参考にしてください。
:::
