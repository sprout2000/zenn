---
title: "（補遺）プログラミング環境の準備"
free: true
---

# プログラミングに必要なツールたち

JavaScript / TypeScript でのプログラミングを進めていく上で、最低限必要なツールが以下のものです。

- Web ブラウザ
- プログラミングに適したフォント
- JavaScript / TypeScript 実行環境
- CLI（コマンドラインインターフェイス）のためのターミナルアプリとコマンドシェル
- コードエディタ
- Git コマンドおよび GitHub アカウント/リポジトリ

Web ブラウザには、[Google Chrome](https://www.google.com/intl/ja_jp/chrome/) または [Microsoft Edge](https://www.microsoft.com/ja-jp/edge) の利用をおすすめしますが、すでにインストール済みの他のブラウザでも構いません（ただし、Internet Explorer と Safari を除く）。

https://www.google.com/intl/ja_jp/chrome/

https://www.microsoft.com/ja-jp/edge

これ以外のツールについて、順に見ていきましょう。

# プログラミング向けフォントのインストール

ターミナルやコードエディタで使用するフォントには、読みやすさや目の疲れにくさが求められます。
本書では、英文には [Cascadia Code](https://github.com/microsoft/cascadia-code/releases) フォント、和文には [白源](https://github.com/yuru7/HackGen)フォントをおすすめします。

それぞれのリリースページ（↑）から、**"CascadiaCode-2111.01.zip"** と **"HackGen_v2.8.0.zip"** をダウンロードしてインストールしましょう（バージョン番号は適宜読み替えてください）。

:::message
Windows では、すでに「Windows ターミナル」（後述）がインストールされている場合、**Cascadia Code** フォントもインストール済みである可能性が高いです。
:::

ダウンロードした zip を解凍してフォントファイルを適当な場所へ展開します。
macOS では、そのフォントファイルをダブルクリックすると **Font Book.app** で開かれ、インストール可能となります。

![](https://storage.googleapis.com/zenn-user-upload/0789158effe2-20230305.png =360x)
_Font Book.app on macOS_

Windows では、そのフォントファイルを右クリックすることでインストールするメニューが表示されます。

![](https://storage.googleapis.com/zenn-user-upload/68c74976d008-20230305.png)
_右クリック（コンテキスト）メニュー_

# JavaScript 実行環境 Node.js

[Node.js](https://nodejs.org/ja/) は、JavaScript がブラウザー上で実行されるのと同じように、OS 上で動作する JavaScript アプリケーションを開発するための環境を提供します。
Node.js では、JavaScript を使って OS 側のアプリケーションを構築することができ、Web アプリケーション、ネットワークアプリケーション、コマンドラインツールなどの開発に利用されます。

https://nodejs.org/ja/

:::message
以下のスクリーンショットは Windows での様子ですが、macOS でも要領は同じです。
:::

[Node.js のサイト](https://git-scm.com/download/win) から LTS （長期サポート）版のインストーラをダウンロードし、これを実行します。
基本的に **"Next"** をボタンを連打するだけでインストールが完了します。

![](https://storage.googleapis.com/zenn-user-upload/d92f53cf1bd1-20230304.png =480x)
_nodejs.org_

![](https://storage.googleapis.com/zenn-user-upload/e87ad6b91124-20230304.png)
_使用許諾ライセンスへの同意_

![](https://storage.googleapis.com/zenn-user-upload/fc22b84354c7-20230304.png)
_インストール先の選択_

![](https://storage.googleapis.com/zenn-user-upload/9e0cc10ada48-20230304.png)
_インストールするモジュールの選択（そのままで可）_

![](https://storage.googleapis.com/zenn-user-upload/0ad70900bbca-20230304.png)
_（※ Windows 版のみ）C コンパイラと Chocolatey をインストールするか否か（しない）_

![](https://storage.googleapis.com/zenn-user-upload/22ce014d108e-20230304.png)
_**Install** ボタンのクリックでインストール完了_

# ターミナルとコマンドシェル

コマンドラインインターフェイス (CLI) のためのターミナルアプリとコマンドシェルを用意します。

## macOS の場合

標準で搭載されている「ターミナル.app」（「アプリケーション」→「ユーティリティ」に収録されています）を利用しましょう。デフォルトで [zsh](https://www.zsh.org/) という UNIX コマンドシェルが利用できます。

![](https://storage.googleapis.com/zenn-user-upload/3db369798311-20230305.png)

ターミナルの「設定」（Command キー + カンマ **`,`**）→ 「プロファイル」画面を開き、上でインストールしたフォントを指定します。その他の設定項目はお好みで良いでしょう。

![](https://storage.googleapis.com/zenn-user-upload/acaef5703f18-20230305.png)

### Command Line Tools (CLT) のインストール

Git コマンドなどを始めとした UNIX コマンド群を利用するには、[Command Line Tools](https://developer.apple.com/jp/xcode/resources/) のインストールも必要です。上で確認したターミナルから以下のコマンドを実行してインストールを完了させてください。

```sh:zsh
xcode-select --install
```

:::message
CLT の代わりに、Apple 統合開発環境 [Xcode](https://developer.apple.com/jp/xcode/) をインストールすることも選択可能ですが、容量が大きく、インストールにも多くの時間がかかります。どうしても必要となるまではインストールしておかなくとも問題はないでしょう。
:::

まだ UNIX コマンドシェルの使い方に慣れていない場合は、以下のオンライン学習サービスの [Command Line コース](https://prog-8.com/courses/commandline)（無料）がおすすめです。

https://prog-8.com/courses/commandline

また、**zsh** の設定については以下の記事を参考にしてください。

https://zenn.dev/sprout2000/articles/bd1fac2f3f83bc

## Windows の場合

Windows では、[Windows ターミナル](https://learn.microsoft.com/ja-jp/windows/terminal/install) と [Git Bash](https://git-scm.com/) の組み合わせがおすすめです。

### Windows ターミナル

「Windows ターミナル」は最初から Windows に含まれているか、もしそうでなければ [Microsoft Store](https://apps.microsoft.com/store/apps?hl=ja-jp&gl=jp) アプリから入手することが可能です。

![](https://storage.googleapis.com/zenn-user-upload/90d8c052d0f3-20230304.png =480x)
_Microsoft Store アプリで "Windows ターミナル" を検索_

Windows では、標準のコマンドシェルとして [PowerShell](https://learn.microsoft.com/ja-jp/powershell/) が用意されています。もしもすでに PowerShell の扱いに慣れているのであれば、これをそのまま利用しても構いません。

しかし、本書では UNIX シェルである [Git Bash](https://git-scm.com/) の導入方法を紹介します。少なくとも Bash であれば、[Linux](https://ja.wikipedia.org/wiki/Linux) をはじめとする UNIX 系 OS や [WSL](https://ja.wikipedia.org/wiki/Windows_Subsystem_for_Linux) (Windows Subsystem for Linux) など、より多くの環境でそのスキルが役立つだろうからです。

そして、まだ UNIX コマンドシェルの使い方に慣れていないのであれば、以下のオンライン学習サービスの [Command Line コース](https://prog-8.com/courses/commandline)（無料）を一通り試すことをおすすめします。

https://prog-8.com/courses/commandline

### Git Bash とは？

[Git Bash](https://git-scm.com/) は、Windows 環境で Git を使用するためのコマンドラインインターフェースであると同時に、UNIX コマンドシェル である [Bash](https://ja.wikipedia.org/wiki/Bash) の機能を提供します。

:::message
Git コマンドそのものについては後述します。
:::

また、Git Bash には UNIX シェルで使用される主要なコマンド群も同梱されています。これにより SSH キーの生成や Git リポジトリのクローンなど、Git の基本的な操作に必要なすべてのツールが揃います。

### Git Bash のインストール

**Git Bash** の[ダウンロードページ](https://git-scm.com/download/win) からインストーラ **"64-bit Git for Windows Setup"** をダウンロードし、これを実行します。

![](https://storage.googleapis.com/zenn-user-upload/2c5121671698-20230304.png)
_インストーラのダウンロード_

インストール手順としては、基本的に **Next** ボタンの連打でも支障ありませんが、随時以下の注意書きに従って設定を行ってください。

![](https://storage.googleapis.com/zenn-user-upload/7949d2e3356b-20230304.png)
_使用許諾ライセンス (GPL) への同意_

![](https://storage.googleapis.com/zenn-user-upload/2f8fbb03d681-20230304.png)
_インストール先の指定_

![](https://storage.googleapis.com/zenn-user-upload/5321cbd4334c-20230321.png)
_ファイルの関連付けなどの設定（※）_

:::message
ターミナルに Windows ターミナルを利用する場合、**"Add a Git Bash Profile to Windows Terminal"** へチェックを入れてください（デフォルトではオフ）。
Git Bash 用プロファイルが Window ターミナルへ自動的に追加されます。
:::

![](https://storage.googleapis.com/zenn-user-upload/28fba3989856-20230304.png)
_スタートメニューで表示する名前の設定_

![](https://storage.googleapis.com/zenn-user-upload/fff6a242114e-20230304.png)
_標準エディタの設定（そのままで可）_

![](https://storage.googleapis.com/zenn-user-upload/bc8f49641b47-20230321.png)
_デフォルトのブランチ名の設定（※）_

:::message
デフォルトで作成されるブランチ（後述）の名前は、**"main"** に設定しておくことをお勧めします。
:::

![](https://storage.googleapis.com/zenn-user-upload/ca88d31a6db6-20230304.png)
_`git` コマンドのための PATH 設定（そのままで可）_

![](https://storage.googleapis.com/zenn-user-upload/48344fb5c803-20230304.png)
_付属の SSH コマンドを利用するか否か（そのままで可）_

![](https://storage.googleapis.com/zenn-user-upload/5453604d6cd9-20230304.png)
_OpenSSL ライブラリを利用するか否か（そのままで可）_

![](https://storage.googleapis.com/zenn-user-upload/8a99cbe05f7d-20230304.png)
_改行コードの設定（そのままで可）_

![](https://storage.googleapis.com/zenn-user-upload/a0a0b18016cc-20230304.png)
_仮想ターミナルの設定（本書では Windows ターミナルを利用するため関係がない）_

![](https://storage.googleapis.com/zenn-user-upload/c2b417641b40-20230304.png)
_`git pull` コマンドの設定（そのままで可）_

![](https://storage.googleapis.com/zenn-user-upload/d53e8556dd40-20230304.png)
_パスワード入力ダイアログの設定（そのままで可）_

![](https://storage.googleapis.com/zenn-user-upload/eca486263b54-20230304.png)
_ファイルシステム関連の設定（そのままで可）_

![](https://storage.googleapis.com/zenn-user-upload/1b2328b4ac20-20230304.png)
_実験的機能も使うか否か（何も選択せずに Install ボタンをクリック）_

![](https://storage.googleapis.com/zenn-user-upload/06a6901bcae5-20230321.png)
_インストール完了_

![](https://storage.googleapis.com/zenn-user-upload/f70c90dcdad5-20230321.png)
_Windows ターミナルのプルダウンメニューへ Git Bash プロファイルが追加されている_

### 手動で Windows ターミナルへ Git Bash を追加する

上の Git Bash のインストールオプションで **"Add a Git Bash Profile to Windows Terminal"** を選択しなかった場合や、[Scoop](https://scoop.sh/) などのパッケージマネージャーで Git (Bash) をインストールした場合には、手動で Windows ターミナルへ Git Bash 用のプロファイルを追加する必要があります。

Control キー + カンマ **`,`** もしくはタブバーのプルダウンメニューからクリックで「設定」画面を開きます。

![](https://storage.googleapis.com/zenn-user-upload/235008c13a41-20230305.png)

![](https://storage.googleapis.com/zenn-user-upload/b75c2191aa70-20230305.png)
_左下の「プロファイル追加」**`+`** ボタンをクリック、「`+ 新しい空のプロファイル`」を選択_

![](https://storage.googleapis.com/zenn-user-upload/cb0e28201d45-20230304.png)
_プロファイルの名前を「Git Bash」に指定_

![](https://storage.googleapis.com/zenn-user-upload/2b63a3fbfef9-20230304.png)
_「コマンドライン」には Bash の実行ファイルの場所を指定_

![](https://storage.googleapis.com/zenn-user-upload/45a1456bdcac-20230304.png)
_「開始ディレクトリ」にはユーザーのホームディレクトリを指定_

![](https://storage.googleapis.com/zenn-user-upload/e079b218d42b-20230304.png)
_タブに表示するアイコンをお好みで指定し、「保存」をクリック_

![](https://storage.googleapis.com/zenn-user-upload/ab3e0f194b87-20230304.png)
_タブバーのプルダウンメニューに "Git Bash" が追加された_

![](https://storage.googleapis.com/zenn-user-upload/201109aea428-20230304.png =640x)
_Git Bash を起動した様子_

# コードエディタ Visual Studio Code

コードエディタには、[WebStorm](https://www.jetbrains.com/webstorm/) などを利用しても構いませんが、本書では無償で利用できる [Visual Studio Code](https://code.visualstudio.com/)（以下、VS Code）を推奨します。

## macOS の場合

**VS Code** の[サイト](https://code.visualstudio.com/)から zip ファイルをダウンロードし、解凍した **Visual Studio Code.app** を「アプリケーション」フォルダへドラッグ＆ドロップするだけです。

![](https://storage.googleapis.com/zenn-user-upload/c763ba599dfa-20230304.png =360x)
_Intel, Apple Silicon の両方に対応した Universal バイナリをダウンロード_

![](https://storage.googleapis.com/zenn-user-upload/1655b9dfac5e-20230304.png =480x)
_zip をダブルクリックして解凍_

## Windows の場合

**VS Code** の[サイト](https://code.visualstudio.com/)からインストーラをダウンロードし、これを実行します。
これも基本的に「次へ」ボタンの連打で問題ありません。

![](https://storage.googleapis.com/zenn-user-upload/ca39d6ee5ef2-20230304.png)
_"Download for Windows" からインストーラをダウンロード_

![](https://storage.googleapis.com/zenn-user-upload/0371e97bb94b-20230304.png)
_使用許諾契約への同意_

![](https://storage.googleapis.com/zenn-user-upload/e006b9dd764a-20230304.png)
_お好みでオプションを設定（全部チェックでも良いでしょう）_

![](https://storage.googleapis.com/zenn-user-upload/126158b61e51-20230304.png)
_「インストール」ボタンのクリックでインストール完了_

## VS Code の初回起動とプログラミング用フォントの設定

VS Code では、初回起動時に「表示言語を日本語に変更しますか？」と訊いてくれます。そのままクリックして VS Code を再起動させましょう。拡張機能 [Japanese Language Pack for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=MS-CEINTL.vscode-language-pack-ja) が自動的にインストールされます。

![](https://storage.googleapis.com/zenn-user-upload/f3db124e330a-20230305.png)

### プログラミング用フォントを設定

VS Code が再び起動したら、「Ctrl キー + カンマ **`,`**」（macOS では「Command キー + カンマ **`,`**」）で設定画面を開き、**Editor: Font Family** へ上でインストールした 2 つのフォントとフォールバックとしての等幅フォントを指定します。

![](https://storage.googleapis.com/zenn-user-upload/b698c198b28d-20230305.png)

```json:settings.json
{
  "editor.fontFamily": "'Cascadia Code', HackGen, monospace"
}
```

## ESLint と Prettier 用プラグインをインストール

JavaScript 向けリンター（文法チェッカー）である [ESLint](https://eslint.org/) とフォーマッター（コード整形ツール）である [Prettier]() を VSCode で有効に使うためのプラグインをそれぞれインストールします。

https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint

https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode

# Git の設定と GitHub アカウントの作成

前項まででインストールが完了した Git コマンドを設定していきましょう。

## Git と GitHub とは？

Git とは、分散型のバージョン管理システムの一つで、プログラムのソースコードやドキュメントなどのファイル変更履歴を管理するためのツールです。

Git では、リモートとローカルに 2 つのリポジトリ（※）を持つことができます。ローカルリポジトリは、開発者が変更を行いバージョンを管理する場所で、リモートリポジトリは、開発者が第三者とコードを共有する場所となります。

:::message
Git レポジトリは、プロジェクトの全体的な変更履歴を保存する場所であり、ファイルやディレクトリのスナップショットを保持します。
:::

この「リモートリポジトリ置き場」として、もっとも多くの人々に利用されているオンラインサービスが [GitHub](https://github.com/) です。

![](https://storage.googleapis.com/zenn-user-upload/3a551bbce938-20230520.png)

## GitHub アカウントの作成

まず最初に、リモートリポジトリを作成する [GitHub](https://github.com/) へアカウントを登録します。

[GitHub](https://github.com/) へアクセスし、右上の **Sign up** をクリックします（すでに GitHub アカウントを作成済みの場合は **"Sign in"** をクリックしてログインしてください）。

![](https://storage.googleapis.com/zenn-user-upload/4385be105256-20230309.png)
_github.com_

基本的には、順に質問に答えていき、登録メールアドレスへ送られた認証コードを入力するだけです。

![](https://storage.googleapis.com/zenn-user-upload/d4e2874dce56-20230309.png)
_メールアドレス、パスワード、ユーザー名を設定_

![](https://storage.googleapis.com/zenn-user-upload/a9be49024269-20230309.png)
_認証コードを入力_

## Git の設定（ユーザー情報）

では、Git へ GitHub アカウントとメールアドレスを設定します。**`--global`** オプションを付与することで、今後作成するすべてのリポジトリでのデフォルト値となります。

```sh:zsh
% git config --global user.name "GitHubアカウント名"
% git config --global user.email "GitHubへ登録したメールアドレス"
```

また、デフォルトで作られるブランチ（後述）の名前は、**`main`** となるように設定しておくのが無難でしょう _（※Git Bash for Windows のインストールオプションで設定済みの場合を除く）_。

```sh:zsh
git config --global init.defaultBranch main
```

現在の設定を確認するには `--list` オプションを使います。

```sh:shell
% git config --global --list

user.name=taro_foo
user.email=foo@example.com
init.defaultbranch=main
```

また、**`--global`** オプションを用いて設定した項目は、ユーザーのホームディレクトリへ **`~/.gitconfig`** ファイルとして保存されます。

```sh:shell
% cat ~/.gitconfig

[user]
    name = taro_foo
    email = foo@example.com
[init]
    defaultBranch = main
```

## SSH 鍵の作成

ターミナルから以下のコマンドを実行し、新しい SSH 鍵を作成します。

:::message
SSH（Secure Shell）は、ネットワークに接続されたコンピューター間で安全に通信を行うための暗号化プロトコルです。SSH は、リモートコンピューターにログインしたり、ファイルを転送したりするために使用されます。
:::

```sh:shell
ssh-keygen -t ed25519 -C "GitHubに登録したメールアドレス"
```

この後は Enter キー連打でも構いませんが、適宜パスフレーズなどを設定しておくことをおすすめします。
上記コマンド実行の結果、ホームディレクトリへ **~/.ssh** ディレクトリが作成されました。

```sh:shell
% ls ~/.ssh
id_ed25519  id_ed25519.pub
```

### GitHub アカウントへ SSH 公開鍵を登録する

上で作成した SSH 鍵のうち、その公開鍵（**`pub`拡張子付きのファイル**）を GitHub アカウントに紐づけましょう。

1. GitHub サイト右上のプルダウンメニュー（アバター）から **Settings** を選択します。

![](https://storage.googleapis.com/zenn-user-upload/dcffbd4e1196-20230309.png =256x)

2. 左ペインの **"SSH and GPG keys"** から **"New SSH key"** をクリックします。

![](https://storage.googleapis.com/zenn-user-upload/fd8ff22d1c76-20230309.png)

3. `Title` へ適当なタイトルを付け、 `Key` へ **`~/.ssh/id_ed25519.pub`** の内容をコピー＆ペーストします。

```sh:shell
# 公開鍵をクリップボードへコピー
# macOS
% cat ~/.ssh/id_ed25519.pub | pbcopy

# Windows (Git Bash)
$ cat ~/.ssh/id_ed25519.pub > /dev/clipboard
```

![](https://storage.googleapis.com/zenn-user-upload/e449066f9ea8-20230309.png)

:::message alert
コピべするのは **`*.pub`** が付いているほうのファイルです。拡張子なしのほうは秘密鍵なので大切に扱ってください。
:::

### SSH 経由で GitHub へ接続できるかチェックする

**`ssh github`** のコマンド実行のみで GitHub へ SSH 接続できるように **`~/.ssh/config`** ファイルを作成します。

```sh:.ssh/config
Host github
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519
```

SSH 接続してみましょう。

:::message
ここで以下のように聞かれた場合は、**`yes`** とタイプしてください。

```sh
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

:::

```sh:shell
$ ssh github

PTY allocation request failed on channel 0
Hi sprout2000! You've successfully authenticated, but GitHub does not provide shell access.
Connection to github.com closed.
```

「認証できたけど、GitHub ではシェルアクセスを提供していないので切断するよ」という旨のメッセージが表示されれば成功です。

# Git と GitHub の基本的な使い方

プロジェクトを Git リポジトリとすることで、コードの変更履歴を管理したり、ある時点からの変更を分岐させたりできるようになります。
また、それを GitHub 上のリモートリポジトリへプッシュしておくと、コードやその変更履歴を共有しつつ、第三者との共同作業を進めることも可能となります。

## Git リポジトリとして初期化する

そのプロジェクト（フォルダ）を Git リポジトリとして初期化するには、**`git init`** コマンドを実行します。

```sh:shell
git init
```

例えば、本書の[第 2 章](https://zenn.dev/sprout2000/books/76a279bb90c3f3/viewer/chapter02)での Vite プロジェクト作成直後に `git init` を行うと次のようなメッセージが表示されます。

```sh
Initialized empty Git repository in /Users/foo/vite-project/.git/
# 空っぽの Git リポジトリが /Users/~/.git/ に初期作成されました
```

現在の Git リポジトリの状態を表示するには、**`git status`** コマンドを利用します。

```sh:shell
% git status

On branch main  # main ブランチでの作業です

No commits yet  # まだコミットされたものはありません

Untracked files:  # 履歴として捕捉されていないファイル
  (use "git add <file>..." to include in what will be committed)
  # （コミットに含めるのであれば "git add <file>" を使ってください）
	.gitignore
	index.html
	package.json
	public/
	src/
	tsconfig.json
	tsconfig.node.json
	vite.config.ts

nothing added to commit but untracked files present (use "git add" to track)
# コミットに追加するものはありませんが、未捕捉のファイルが存在しています
```

## コードの作成・変更を履歴として記録する

コードに対して行った変更などを履歴として記録することを **「コミットする」** と言います。
Git では、ファイルへの変更や未捕捉の新しいファイルの作成などをいったん **`git add`** コマンドでステージングした後、コミットメッセージを付けてコミットします。

![](https://storage.googleapis.com/zenn-user-upload/c7ebaff17b47-20230312.png)

```sh:shell
# ステージング
% git add .

# ステータスを確認
% git status

On branch main

No commits yet

Changes to be committed:  # コミットされる予定のファイル
  (use "git rm --cached <file>..." to unstage)
  # 「アンステージ」するには "git rm --cached <file>..." を使ってください
	new file:   .gitignore
	new file:   index.html
	new file:   package.json
	new file:   public/vite.svg
	new file:   src/App.css
	new file:   src/App.tsx
	new file:   src/assets/react.svg
	new file:   src/index.css
	new file:   src/main.tsx
	new file:   src/vite-env.d.ts
	new file:   tsconfig.json
	new file:   tsconfig.node.json
	new file:   vite.config.ts
```

`git add` コマンドの引数へファイル名やフォルダ名でなく、ピリオド **`.`** や `-A` オプションを与えるとすべての変更・作成したものがステージングされます（くわしくは **`man git-add`** コマンドでヘルプを参照してください）。

:::message
ステージングしたくない、つまり Git に管理させたくないファイル（パスワードやトークンを記述した環境変数ファイル **`.env`** など）やフォルダがある場合は、それらをプロジェクトフォルダ直下の **`.gitignore`** ファイルへ指定しておく必要があります。

```sh:.gitignoreの例
dist
node_modules
.env
.DS_Store
```

:::

では、コミットメッセージを **`-m`** オプションで指定してコミットしましょう。

```sh:shell
% git commit -m "第2章まで完了"

[main (root-commit) b688e4e] 第2章まで完了
 13 files changed, 254 insertions(+)
 create mode 100644 .gitignore
 create mode 100644 index.html
 create mode 100644 package.json
 create mode 100644 public/vite.svg
 create mode 100644 src/App.css
 create mode 100644 src/App.tsx
 create mode 100644 src/assets/react.svg
 create mode 100644 src/index.css
 create mode 100644 src/main.tsx
 create mode 100644 src/vite-env.d.ts
 create mode 100644 tsconfig.json
 create mode 100644 tsconfig.node.json
 create mode 100644 vite.config.ts
```

これまでのコミット履歴は `git log` コマンドで参照することができます。

```sh:shell
% git log

commit b688e4ee7d6b266874d155ffa544ed3b6414db49 (HEAD -> main)
Author: foobar <foobar@example.jp>
Date:   Sat Mar 5 16:40:22 2022 +0900

    第2章まで完了
```

## GitHub にリモートリポジトリを作成する

上で作業中のローカルリポジトリをリンクさせるため、 GitHub へリモートリポジトリを作成しましょう。

1. GitHub サイトの右上プルダウンメニュー（**`+`**）から **"New repository"** を選択します。

![](https://storage.googleapis.com/zenn-user-upload/6c0dd2eee2be-20230309.png =360x)

2. リポジトリへ適当な名前を付けて **"Create repository"** をクリックします。

![](https://storage.googleapis.com/zenn-user-upload/e2c00ba9e85d-20230310.png)
![](https://storage.googleapis.com/zenn-user-upload/d32007496832-20230310.png)

3. **"Code"** タブに **"Quick setup"** が表示されれば、リポジトリ作成の完了です。
   コピーボタンをクリックして SSH アドレスをクリップボードへコピーしておきましょう。

![](https://storage.googleapis.com/zenn-user-upload/483f5642fe07-20230310.png)

## ローカルリポジトリを GitHub 上のリモートリポジトリへプッシュする

では、前々項で作成したローカルリポジトリへ戻り、これを GitHub 上のリモートリポジトリとリンクさせます。

### リモートリポジトリを紐付ける

```sh:shell
git remote add origin git@github.com:sprout2000/zenn-todo.git
```

**`git remote add`** は、Git リポジトリへそのリモートリポジトリを設定するためのコマンドです。
このコマンドを使用して、ローカルリポジトリへリモートリポジトリの名前（デフォルトでは `origin`）と URL（前項でクリップボードへコピーしたもの） を登録して、リモートリポジトリにアクセスできるようにします。

### リモートへプッシュする

現在の手元のリポジトリを GitHub 上のリポジトリへ反映させる、つまりプッシュします。

```sh:shell
git push --set-upstream origin main
```

**`--set-upstream`** オプション（短縮形は **`-u`**）は、手元のリポジトリの**ブランチ**（ここでは `main`）（※）**を初めて**リモートへプッシュするときに使います。

:::message
ブランチとは、いまのところはコードの分岐の一つくらいに考えておいてください。
本書ではコードを分岐させることはないので、最初のブランチ (=`main`) のみが存在していることになります。
:::

このオプションにより、ローカルブランチ `main` をリモートブランチ `origin/main` にプッシュし、次回以降のプッシュやプル時にこのリモートブランチが自動的に追跡されるようになります。

なお、次回以降はこの **`-u`** オプションは使いません。そして、`origin main` の部分も省略可能となります。

```sh:次回以降
% git push  # リモートへプッシュ
# or
% git pull  # リモートと同期（プル）
```
