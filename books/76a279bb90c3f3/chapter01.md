---
title: "はじめに"
free: true
---

# この本について

本書では、[React](https://ja.react.dev/) 16.8 で導入された [Hooks](https://ja.react.dev/reference/react) の機能を使って、ハンズオン形式で To Do リストを管理するためのアプリケーション（以下、Todo アプリ）を作成していきます。

@[codepen](https://codepen.io/sprout2000_jp/pen/ExvemWj?default-tab=result)

また、現在のフロントエンド開発環境でデファクト・スタンダードとなりつつある [TypeScript](https://www.typescriptlang.org/ja/) を利用することによって[型安全](https://typescript-jp.gitbook.io/deep-dive/getting-started/why-typescript)な実装の実現も目指します。

https://www.typescriptlang.org/ja/

さらに後半部分では、この Todo アプリへ UI フレームワーク [MUI](https://mui.com/) を導入することでマテリアルデザインのユーザーインターフェイス (UI) を与え、[プログレッシブウェブアプリ](https://developer.mozilla.org/ja/docs/Web/Progressive_web_apps) (PWA) として [GitHub Pages](https://docs.github.com/ja/pages) へ公開します。

![](https://storage.googleapis.com/zenn-user-upload/36c99f0b2f3c-20230108.png =240x)
_PWA on iOS_

https://sprout2000.github.io/todo/

## 本書が対象とする読者

この本では、以下のような人を読者として想定しています。

- JavaScript に多少触れたことがある人
- [ターミナル](https://learn.microsoft.com/ja-jp/windows/terminal/install)アプリ、[Node.js](https://nodejs.org/ja/) および [Visual Studio Code](https://code.visualstudio.com/) （以下、VS Code）をインストール・設定済みである人
- コマンドシェルの操作をある程度習得している人

:::message
プログラミング環境の準備については、巻末の[「（補遺）プログラミング環境の準備」](https://zenn.dev/sprout2000/books/76a279bb90c3f3/viewer/appendix04)も参照してください。
:::

まだ JavaScript に触れたことがない人には、まず以下のサイトで基礎を学ぶことをおすすめします。

https://jsprimer.net/

# React とは？

React は、Web アプリケーションの UI を構築するための JavaScript ライブラリです。
React の特長としては、以下のようなものが挙げられます。

### コンポーネント志向

React ではコンポーネントという単位で UI 部品を設計できるので、再利用性が高く、保守がしやすいです。そして、これらのコンポーネントを組み合わせて、完全なアプリケーションを構築することができます。

![](https://storage.googleapis.com/zenn-user-upload/e2eaf726591c22b995cf8884.png =480x)
_各コンポーネントで構成されたアプリケーション_

### JSX

[JSX](https://ja.react.dev/learn/writing-markup-with-jsx) (JavaScript XML) は、JavaScript の拡張文法です。React では、JSX を使用することで HTML に似たような記法を使用して、JavaScript 内で UI を直感的に構築することができます。

```jsx:JSX の例
// 関数コンポーネント App
function App() {
  return (
    <div style={{ textAlign: "center" }}>
      <h1>Hello, world!</h1>
    </div>
  );
}
```

![](https://storage.googleapis.com/zenn-user-upload/aa1831fef060-20230424.png)
_JSX で UI を構築_

### ステートと仮想 DOM による差分レンダリング

React では、**コンポーネントが持つ状態（ステート）に応じて、リアクティブに DOM を書き換える**ことができます。つまり、ユーザー操作による UI の更新をサーバーとの通信を必要とせず実現します。そして、その手段として用いられるのが「仮想 DOM」です。

![](https://storage.googleapis.com/zenn-user-upload/5326b81de423-20230424.gif)
_リアクティブに DOM を書き換え_

仮想 DOM とは、[DOM](https://developer.mozilla.org/ja/docs/Web/API/Document_Object_Model)（ドキュメントオブジェクトモデル）ツリーを抽象化した JavaScript オブジェクトのことです。

React では、この仮想 DOM を操作しリアル DOM と同期させ、その差分のみを反映させるため、UI を更新するたびにブラウザが画面全体を再描画する必要がありません。

![](https://storage.googleapis.com/zenn-user-upload/4419c4ea171b-20230221.png =480x)
_差分アップデート_

# なぜ TypeScript を使うのか？

[TypeScript](https://www.typescriptlang.org/ja/) は、JavaScript のスーパーセットであり、JavaScript の文法に型システムを加えたものです。

### 型による静的チェック

TypeScript の最大の特徴が **「型による静的チェック」** 機能です。

型チェックはプログラムを実行せずとも、プログラムの欠陥に気づくことができます。バグは発見が遅れるほど修正コストが高くつきますが、TypeScript ではコーディング中に頻繁にチェックすることができ、バグの早期発見によって修正コストも抑えることができます。

![](https://storage.googleapis.com/zenn-user-upload/77d0ff935046-20230108.png)
_Visual Studio Code でのエラー表示の例_

型があることで、プログラムの可読性や理解しやすさが上がったり、エディターの補完機能を活かすことができ、コーディングの効率も良くなります。

また、コンパイル時には型エラーを検出することができるため、リリース前に問題を発見することも可能となります。

### 強力な型推論機能

加えて TypeScript では、変数や関数の戻り値などの型が明示的に指定されていない場合でも、[型推論](https://typescript-jp.gitbook.io/deep-dive/type-system/type-inference)機能によって適切な型を自動的に割り当ててくれます。

```ts:型推論の例
const a = 123;  // a は number型
const b = "hello";  // b は string型
const c = [1, 2, 3];  // c は number[]型 (number型を要素とする配列)

function add(x: number, y: number) {
  return x + y;  // add 関数は number型を返す
}
```

型推論によって型が自動的に割り当てられるため、コードをより簡潔に記述できます。もちろん、明示的に型を指定することで、プログラムの読みやすさや保守性が向上する場合もあるため、必要に応じて意図的に指定しておくこともできます。

```ts:明示的な型指定
let z: string;
z = 1000;  // <-- エラー
```

https://typescript-jp.gitbook.io/deep-dive/
