---
title: "おわりに"
free: true
---

## PWA をインストールしてもらうには？

[chap.29](https://zenn.dev/sprout2000/books/76a279bb90c3f3/viewer/chapter29) でおこなった設定が正常に機能していれば、Service Worker がサイトで稼働中であるため、このアプリを **PWA** としてインストールできます。

https://developers.google.cn/web/fundamentals/primers/service-workers?hl=ja

### Android OS の場合

ページを開いてしばらくすると、アドレスバー下にインストールバナーが表示されます（※）。

:::message
ただし、一定の条件下ではインストールバナーが表示されない場合もあります。
また、Android OS のバージョンによってはバナーの表示場所が異なる場合があります。
:::

![](https://storage.googleapis.com/zenn-user-upload/19093611a3e1-20221231.png =480x)

インストールバナーが表示されないときは、メニューから「アプリをインストール」を選択してもらいます。

![](https://storage.googleapis.com/zenn-user-upload/62c183cac56d-20221231.png =240x)

その後の確認ダイアログで **インストール** をタップしてもらえばインストール完了です。

![](https://storage.googleapis.com/zenn-user-upload/dbb43617af1e75c0f7c82022.png)

### iOS の場合

iOS の場合、原則としてインストールバナーは出ません。
まずサイトを開いて「共有」メニューから「ホーム画面に追加」。

![](https://storage.googleapis.com/zenn-user-upload/e42ef5fb802a24db64f202f5.png =360x)

![](https://storage.googleapis.com/zenn-user-upload/71d565d9e9ba55ccc2f2612c.png =360x)

追加ボタンのタップでインストール完了です。

![](https://storage.googleapis.com/zenn-user-upload/1f2e68628edce42e9690a1ac.png =360x)

![](https://storage.googleapis.com/zenn-user-upload/5f7a6bc07901b89af89ad007.png =360x)

まるでネイティブアプリであるかのような表示と使用感が確認できるでしょう。

![](https://storage.googleapis.com/zenn-user-upload/afb4ca4f85a799effd7f4870.png =360x)
