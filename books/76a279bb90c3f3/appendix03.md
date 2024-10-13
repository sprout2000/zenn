---
title: "（補遺）ユニットテストを導入するには？"
free: true
---

## ユニットテストとは？

ユニットテスト（単体テスト）とは、プログラムを構成するコンポーネントや関数などの比較的小さな単位（ユニット）がそれぞれの機能を正しく果たしているか検証するプロセスです。

これによりユニット自体の不確実性を減らすことができ、後日のリファクタリングにおいても、まだユニットが正しく機能することを確認することができます。また、そのリファクタリングによって障害が引き起こされたとしても、すばやく原因を究明し修正する手助けとなります。

## Jest と Testing Library

React アプリケーションのユニットテストは、[Jest](https://jestjs.io/ja/) と [Testing Library](https://testing-library.com/) を利用して行うことができます。

Jest は、テストファイルを自動で探し出し、テストを実行して期待通りの正しい値を持っているかマッチャー関数を利用してチェックしてくれるフレームワークです。

Testing Library は様々なフレームワークでの UI のテスト環境を提供し、テストしたいコンポーネントの描写やクリックイベントの実行、描写した内容からの要素の取得等に利用されます。

https://jestjs.io/ja/

https://testing-library.com/

## テスト環境のインストール

### Jest のインストール

- Jest 本体と TypeScript 用のプリセット

```sh:zsh
npm i -D jest ts-jest
```

- テスト環境
  JS + DOM によるアプリケーションを Jest でテストするための[テスト環境](https://jestjs.io/ja/docs/next/configuration#testenvironment-string) **jest-environment-jsdom** も必要です。

```sh:zsh
npm i -D jest-environment-jsdom
```

- **ts-node** と **Node.js** の型定義ファイル

Node.js のモジュールを TypeScript のままで実行（≒ `jest.config.ts` などの設定ファイルを評価）できる [ts-node](https://typestrong.org/ts-node/) と、これが利用する Node.js 向け型定義ファイルをインストールします。

```sh:zsh
npm i -D ts-node @types/node
```

### Testing Library のインストール

- React 向け Testing Library 本体

```sh:zsh
npm i -D @testing-library/react
```

- 拡張マッチャー
  Jest に含まれているものだけでは足りないマッチャー関数を提供するライブラリ。

```sh:zsh
npm i -D @testing-library/jest-dom
```

:::message
パッケージマネージャに [pnpm](https://pnpm.io/ja/) を利用している場合、[@types/testing-library\_\_jest-dom
](https://www.npmjs.com/package/@types/testing-library__jest-dom) のインストールも必要です。

```sh
pnpm add -D @types/testing-library__jest-dom
```

:::

- UserEvent
  その名の通り、クリックなどのユーザーイベントをエミュレートしてくれるライブラリ。このライブラリは `@testing-library/dom` に依存しているため、これもインストールします。

```sh:zsh
npm i -D @testing-library/user-event @testing-library/dom
```

## Jest の設定

### `jest.config.ts` の作成

プロジェクト直下へ `jset.config.ts` を作成します。

```ts:jest.config.ts
// 設定オプションの型定義をインポート
import type { JestConfigWithTsJest } from 'ts-jest';

// オプションを設定
const config: JestConfigWithTsJest = {
  // TypeScript 用プリセット
  preset: 'ts-jest',
  // テスト環境
  testEnvironment: 'jsdom',
  // カバレッジ（テストコードの網羅率）を収集
  collectCoverage: true,
  // コンソール上でのみカバレッジを報告
  coverageReporters: ['text'],
};

// 設定をデフォルトエクスポートする
export default config;
```

### NPM スクリプトの追加

`npm run test` のコマンド発行でテストが実行されるように `package.json` へ NPM スクリプトを追加します。

```diff json:package.json
  {
      "scripts": {
        "dev": "vite",
        "build": "tsc && vite build",
        "preview": "vite preview",
+       "test": "jest"
      }
  }
```

なお、スクリプト名が `start` や `test` の場合には、例外的に途中の `run` を省略することもできます。

```sh:zsh
npm test
```

## ユニットテストの書き方

テストファイルは、テストしたいコンポーネントを持つファイルの名前へ **`test`** を挿入したファイル名として作成します。

```diff 例
  % tree src
  src
+ ├── App.test.tsx
  ├── App.tsx
  ├── main.tsx
  └── styles.css
```

```ts:src/App.test.tsx
// 拡張マッチャーをインポート
import '@testing-library/jest-dom';

import { render, screen } from '@testing-library/react';

// コンポーネントをインポート
import { App } from './App';

test('App コンポーネントのテスト', async () => {
  // App コンポーネントをレンダー
  render(<App />);

  /** ここにテストを書く */
});
```

- `render`
  名前の通り、コンポーネントをレンダーします。

- `screen`
  レンダーされたコンポーネント内の要素を取得するときに使います。
  素の JavaScript の `document.getElementBy**` に近い役割です。

```ts:例
// 一番上のボタンを取得
const button0 = screen.getAllByRole('button')[0];

// このボタンが "こんにちは" というラベルを持っているかチェック
expect(button0).toHaveTextContent('こんにちは');
```

- **`test`**
  `test` メソッドでは、第 1 引数の文字列へテストの説明を記述し、第 2 引数のコールバック関数内でコンポーネントをレンダーするなどしてテストを行います。

- **`expect(value)`**
  `expect` は値をテストしたい時に毎回使用する関数です。`expect` のみを呼び出すということはほとんどなく、 代わりに値について何らかの状態がマッチするかチェックする**マッチャー関数**とともに使用します。

上の例で言えば、`button0` に付けられたラベル（文字列）が、期待値と一致することをチェックするために `expect` が使われています。

```typescript:例
expect(button0).toHaveTextContent('こんにちは');
```

この `toHaveTextContent()` がマッチャー関数です。**Jest** や **@testing-library/jest-dom** には、様々な事をテストするのを手助けする数多くのマッチャー関数が用意されています。

具体的なテストコードについては、GitHub 上のサンプルリポジトリを参照してもらい、本書ではもっとも容易な `isTodos.ts` のテストを実例として以下で紹介するに留めます。

## `isTodos()` メソッドをテストする

型ガード関数である `isTodos()` メソッドでは、与えられた引数が `Todo型` オブジェクトであること、およびその配列であることをチェックしています。

```ts:src/lib/isTodos.ts
const isTodo = (arg: any): arg is Todo => {
  return (
    typeof arg === 'object' &&
    typeof arg.id === 'number' &&
    typeof arg.value === 'string' &&
    typeof arg.checked === 'boolean' &&
    typeof arg.removed === 'boolean'
  );
};

export const isTodos = (arg: any): arg is Todo[] => {
  return Array.isArray(arg) && arg.every(isTodo);
};
```

上段の `isTodo()` 関数もテストファイルからアクセスできるようにエクスポートします。

```diff ts:src/lib/isTodos.ts
+ export const isTodo = (arg: any): arg is Todo => {
    // ...
  };
```

では、テストファイルを作成しましょう。

```ts:src/__tests__/isTodos.test.ts
// 両関数をインポート
import { isTodo, isTodos } from '../lib/isTodos';

// Todo型オブジェクトの配列のモック（模型）
const mockTodos = [
  { id: 0, value: 'hello', checked: false, removed: false },
  { id: 1, value: 'bye', checked: true, removed: true },
];

test('Todo型オブジェクトが与えられると `true` となるか？', () => {
  expect(isTodo(mockTodos[0])).toBeTruthy();
});

test('Todo型オブジェクトの配列が与えられると `true` となるか？', () => {
  expect(isTodos(mockTodos)).toBeTruthy();
});
```

テストを実行してみます。

```sh:zsh
npm test
```

![](https://storage.googleapis.com/zenn-user-upload/0d1ddc2bfc55-20230716.png =640x)

1 つのテストファイルとその中の 2 つのテストがパスし、カバレッジがいずれの項目でも **100%** となった旨が表示されました。
