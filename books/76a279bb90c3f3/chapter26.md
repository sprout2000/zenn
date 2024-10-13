---
title: "コラム：React コンポーネントのライフサイクル"
free: true
---

## コンポーネントのライフサイクル

React コンポーネントには、_ライフサイクル_ というものがあります。
ここでは、かなり大雑把ですが、**マウント**（コンポーネントの画面への描画）〜**アップデート**（コンポーネントの更新・再描画）〜**アンマウント**（画面への描画の停止）の各段階のことをライフサイクルと呼ぶことにします。

React コンポーネントをクラス構文で記述していた時代（※）は、このライフサイクルはとても分かりやすいものでした。なぜなら、そのインスタンスの作成・削除やデータ更新による DOM への反映そのものがライフサイクルであったからです。

```jsx:例
// クラスコンポーネントの例
class Counter extends Component<CounterProps, CounterState> {
  constructor(props: CounterProps) {
    super(props);
    this.state = { count: 0 };
  }

  handleIncrement = () => {
    this.setState((prevState) => ({
      count: prevState.count + 1,
    }));
  };

  render() {
    return (
      <div>
        <p>Count: {this.state.count}</p>
        <button onClick={this.handleIncrement}>Increment</button>
      </div>
    );
  }
}
```

また、各ライフサイクルのそれぞれの段階に実行される **`componentDidMount()`**、**`componentDidUpdate()`** そして **`componentWillUnmount()`** といったライフサイクルメソッドも用意されていました。

:::message
現在でもクラス構文によってコンポーネントを作成すること自体は可能ですが、今後の新しいプロジェクトで採用するメリットはほぼないため、本書ではクラスコンポーネントの詳細については触れません。
:::

https://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/

しかし、Hooks 時代の関数コンポーネントにおいては、これらのメソッドを **`useEffect` フック**ひとつ（※）で処理するようになったため、コードの記述量は減ったものの、各ライフサイクルを意識することが難しくなりました。

:::message
他に `useLayoutEffect` フックなどもありますが、本書では割愛します。
:::

## Hooks 時代のライフサイクル

一般的な関数コンポーネントは、以下のような構成で作成されます。

```jsx
function Example() {
  /**
   * 初期化
   * ステートやイベント処理関数の定義など
   */

  useEffect(() => {
    // 副作用

    return () => {
      // 副作用のクリーンアップ
    };
  });

  // レンダー
  return <div>{/* ... */}</div>;
}
```

これらは、必ずしも上から順に実行されるわけではなく、簡略化して図示すると以下のような順番で実行されます。

![](https://storage.googleapis.com/zenn-user-upload/a45a0cd9796b-20230222.png)

ここで注意が必要なのは、以下の 3 点です。

- マウント時にも副作用は走る
- アップデート時には副作用の**前に**クリーンアップ関数が実行されるが、マウント時にこれは走らない
- アップデート時における副作用実行の可否は依存配列でコントロールする _（前章参照）_

:::message alert
厳密に言うと、「レンダー」とは React が関数コンポーネントの本体を呼び出すことであり、「描画」はブラウザが画面に DOM を反映する動作のことなので両者は区別すべきものです。
しかし、入門書である本書では、これらを峻別すべき場面は登場しないため、どちらもほぼ同じ意味として取り扱っています。
:::

### クリーンアップ関数とは？

本書の Todo アプリでは、副作用のクリーンアップを必要とするようなサイドエフェクトは登場しません。
しかし、例えば何らかの理由で `useEffect` フック内のイベントリスナーで、コンポーネント**外**で発生するイベントを検知しなければならないような場合を考えてみましょう。

```jsx
const listener = () => {
  console.log("ウィンドウサイズが変更されました！");
};

useEffect(() => {
  // ウィンドウサイズ変更に対応するイベントリスナー
  window.addEventListener("resize", listener);
}, []);
```

このコード例での `useEffect` フックの使い方では、React アプリケーションはあっと言う間に[メモリリーク](http://ja.wikipedia.org/w/index.php?curid=44656)を起こしてしまいます。
なぜなら、上掲のイラストで示した通り、**副作用はマウント・アップデートごとに実行される**ため、その都度イベントリスナーが**多重登録**されてしまうからです。

このような事態を防ぐために、`useEffect` フックではクリーンアップという仕組みが用意されています。

```jsx
useEffect(() => {
  window.addEventListener("resize", listener);

  // 副作用のクリーンアップ
  return () => {
    // イベントリスナーの除去
    window.removeEventListener("resize", listener);
  };
}, []);
```

このクリーンアップ関数を定義しておくことで、上図の通り、コンポーネントの更新時には副作用実行の**前**に、従前のイベントリスナーを除去できるようになります。

:::message

### JSX でのイベント処理

一方で、これまでたくさんの以下のようなイベントリスナーを JSX 内に設定してきました。

```jsx
<button onClick={() => console.log("Hello!")}>Greet</button>
```

これらのイベントリスナーを手動で除去する必要がなかったのは、React がイベントリスナーを `addEventListener()` で直接登録するのではなく、仮想 DOM の中に設定する仕組み（[SyntheticEvent](https://ja.react.dev/reference/react-dom/components/common#react-event-object) といいます）を提供しているからです。
このため、React コンポーネントが再描画されたときに、古いイベントリスナーが無効になり、新しいイベントリスナーが自動的に設定されるようになっています。
これが、これまで JSX 内のイベントリスナーを除去する必要がなかった理由です。
:::
