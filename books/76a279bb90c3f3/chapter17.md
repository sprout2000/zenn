---
title: "UI コンポーネントの構成を考える"
free: true
---

# UI コンポーネントの構成を考える

UI コンポーネントの構成を[第 1 章](https://zenn.dev/sprout2000/books/76a279bb90c3f3/viewer/chapter01)で掲げた完成形イメージから逆算して検討します。

## 全体の App シェル

まずアプリ全体の構成、[App シェル](https://developers.google.com/web/fundamentals/architecture/app-shell?hl=ja)をイメージします。

![](https://storage.googleapis.com/zenn-user-upload/2d73d2e43bd7d0248b075ca1.png =360x)
_App コンポーネント_

https://developers.google.com/web/fundamentals/architecture/app-shell?hl=ja

- 上部ツールバーへ現在の **`filter ステート`** の値を表示する
- 左上のメニューボタンのクリックでドロワーを開く
- 開かれたドロワーからフィルターの選択が出来るようにリストを表示する
- 右下の FAB (Floating Action Button) のクリックで新しいタスクを入力するダイアログを表示する

## App シェルを構成する各コンポーネントを想定する

つぎに各部分に分解してみましょう。

![](https://storage.googleapis.com/zenn-user-upload/e2eaf726591c22b995cf8884.png =480x)

- **ToolBar コンポーネント** については上で見た通り
- **`todos ステート`** を `<ul> ~ </ul>` タグへ展開していた部分は **TodoItem** コンポーネントへ
- **TodoItem** を構成する 1 つ 1 つの `todo` は **TodoCard** コンテナーへ

- **ActionButton コンポーネント** の Floating Action Button（以下、**FAB**）のクリックで **FormDialog コンポーネント**を表示

![](https://storage.googleapis.com/zenn-user-upload/77ec92bfe342-20211122.png =360x)
_FormDialog コンポーネント_

## ActionButton コンポーネント

- `filter ステート` が `checked` のときは鉛筆アイコンを無効化

![](https://storage.googleapis.com/zenn-user-upload/cbfc8971b15b-20211122.png =240x)

- `filter ステート` が `removed` のときは「ごみ箱を空にする」ボタンを表示

![](https://storage.googleapis.com/zenn-user-upload/34e0b0b19b4a-20211122.png =240x)

- 「ごみ箱を空にする」実行前に **AlertDialog コンポーネント**を表示

![](https://storage.googleapis.com/zenn-user-upload/fc98aa17022c-20211122.png =360x)
_AlertDialog コンポーネント_

- 「ごみ箱を空にする」実行後はごみ箱アイコンを無効化

![](https://storage.googleapis.com/zenn-user-upload/5e90afebb756-20211122.png =240x)

## TodoItem コンポーネント

**TodoCard** コンテナーを細かく見ていきましょう。

![](https://storage.googleapis.com/zenn-user-upload/df2f87811903-20230206.png)

- 上段のフォームには `todo.value` の値を表示
- 下段左に `todo.checked` の値を表すアイコンを配置し、クリックで `handleTodo()` 関数を実行
- 下段右のごみ箱アイコンのクリックで `handleTodo()` 関数を実行

## SideBar コンポーネント

右上のメニューボタンから開かれるドロワーを見てみましょう。

![](https://storage.googleapis.com/zenn-user-upload/ffb75096a9a8-20211122.png =360x)
_SideBar コンポーネント_

- 最上部にはドロワーヘッダーとバージョン等の情報
- `<select> ~ </select>` タグで選択していたフィルターをリストとして展開
- 「このアプリを共有」クリックでこのアプリの URL をエンコードした QR コードを表示

![](https://storage.googleapis.com/zenn-user-upload/c01760fc3511-20211122.png =360x)
_QR コンポーネント_

## まとめ

以上でひと通りのコンポーネントが確認できました。次章以降は、この UI を作成するための UI フレームワークを導入し、各コンポーネントを React コンポーネントとして実装していくこととなります。
