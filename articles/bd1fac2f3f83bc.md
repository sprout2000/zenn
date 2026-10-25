---
title: "macOS の zsh ではこれだけはやっておこう"
emoji: "📺"
type: "tech"
topics:
  - "homebrew"
  - "shell"
  - "mac"
  - "初心者向け"
  - "zsh"
published: true
published_at: "2021-09-11 15:31"
---

# Homebrew を使います

https://brew.sh/

まだ [Homebrew](https://brew.sh/) をインストールしていない場合はこちら:

```shell:shell
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

:::message alert
まだ [Command Line Tools](https://developer.apple.com/download/all/) をインストールしていない場合は、上のインストールに併せてインストールが始まります。
:::

# 1. パスの設定

まず **Homebrew** 向けにパスを通しておく必要があります。

```sh
# デフォルトのパスを確認
% echo $path
/usr/local/bin /usr/bin /bin /usr/sbin /sbin /Library/Apple/usr/bin /Library/Frameworks/Mono.framework/Versions/Current/Commands
```

**Homebrew** のインストール先は `/opt/homebrew` 以下となるので、ここへパスを通しておきます。
ホームディレクトリ (`/Users/ユーザー名`) に `.zshrc` を作成します。

```shell:~/.zshrc
typeset -U path PATH
path=(
  /opt/homebrew/bin(N-/)
  /opt/homebrew/sbin(N-/)
  /usr/bin
  /usr/sbin
  /bin
  /sbin
  /usr/local/bin(N-/)
  /usr/local/sbin(N-/)
  /Library/Apple/usr/bin
)
```

優先してほしい順にパスを指定していきます。

**`(N-/)`** は、もしそのディレクトリが存在していれば **PATH** に追加し、そうでなければ無視してくれるオプションです。
上のパス設定を反映させるには `~/.zshrc` を読み込みます。

```shell:shell
source ~/.zshrc
```

# 2. zsh-completions のインストール

コマンド入力にバシバシ補完を効かせてくれる **zsh-completions** をインストールします。

```shell:shell
brew install zsh-completions
```

```sh
~ 略 ~
=> Caveats
To activate these completions, add the following to your .zshrc:

  if type brew &>/dev/null; then
    FPATH=$(brew --prefix)/share/zsh-completions:$FPATH

    autoload -Uz compinit
    compinit
  fi

You may also need to force rebuild `zcompdump`:

  rm -f ~/.zcompdump; compinit

Additionally, if you receive "zsh compinit: insecure directories" warnings when attempting
to load these completions, you may need to run this:

  chmod -R go-w '/opt/homebrew/share/zsh'
~ 略 ~
```

インストール時の **Caveats** には、`/opt/homebrew/share/zsh` (Intel Mac の場合は `/usr/local/share/zsh`) のみを **`go-w`** せよと表示されていますが、実際には `share` ディレクトリそのものを **`chmod -R go-w`** する必要があります。

```shell:shell
chmod -R go-w /opt/homebrew/share
```

あとは **Caveats** の指示にしたがって、`~/.zshrc` への追記と補完キャッシュファイルの再生成をおこないます。

```shell:~/.zshrc
if type brew &>/dev/null; then
  FPATH=$(brew --prefix)/share/zsh-completions:$FPATH
  autoload -Uz compinit && compinit
fi
```

```sh
% source ~/.zshrc
% rm -f ~/.zcompdump; compinit
```

これでコマンド入力中に `Tab` キーまたは `Ctrl+I` を打鍵するとその後の候補を補完表示してくれます。

:::message alert
この設定だけでは補完が効かない場合は、ホームディレクトリへ以下の `~/.zshenv` ファイルを作成してターミナルを再起動させてください。

```sh:~/.zshenv
. "$HOME/.zshrc"
```

:::

```sh
% sysctl _  # <-- ここで Tab または \C-I を入力

% sysctl _
audit     hw        kperf     machdep   security  vfs
debug     kern      ktrace    net       user      vm

% sysctl hw._ # <-- ここでふたたび Tab

% sysctl hw._
activecpu               cpufrequency            l3cachesize
busfrequency            cpufrequency_max        logicalcpu
busfrequency_max        cpufrequency_min        logicalcpu_max
busfrequency_min        cpusubfamily            memsize
```

:::message
**`_`** はカーソル位置を表しています。
:::

# 3. zsh-autosuggestions のインストール

ターミナルのコマンド履歴に基づいてコマンド候補を表示、入力補完もしてくれる **zsh-autosuggestions** もインストールします。

```shell:shell
brew install zsh-autosuggestions
```

```sh
~ 略 ~
==> Caveats
To activate the autosuggestions, add the following at the end of your .zshrc:

  source /opt/homebrew/share/zsh-autosuggestions/zsh-autosuggestions.zsh

You will also need to force reload of your .zshrc:

  source ~/.zshrc
==> Analytics

~ 略 ~
```

**Caveats** にしたがって `~/.zshrc` へ追記しましょう。

```diff shell:~/.zshrc
  if type brew &>/dev/null; then
    FPATH=$(brew --prefix)/share/zsh-completions:$FPATH
+   source $(brew --prefix)/share/zsh-autosuggestions/zsh-autosuggestions.zsh
    autoload -Uz compinit && compinit
  fi
```

```shell:shell
source ~/.zshrc
```

![](https://storage.googleapis.com/zenn-user-upload/615948f86ee5-20220122.png =360x)
_この状態から Ctrl+E で補完をコンプリート_

# 4. プロンプトの表示を変更

プロンプトのデフォルト値を確認します。

```shell:zsh
zenn@mba ~ $ echo $PROMPT
```

```sh
%n@%h %1~ %#
```

これらは以下のことを意味しています。

- **`%n`:** ユーザー名
- **`%h`:** ホスト名
- **`%~`:** カレントディレクトリ（**`1`** はディレクトリ深度）
- **`%#`:** 一般ユーザーのときは **`%`**、**root** になったときは **`#`** を表示

カレントディレクトリの表示形式にはいくつかのバリエーションがあります。

|  PROMPT   | 機能                 | 実際の表示                        |
| :-------: | :------------------- | :-------------------------------- |
| **`%d`**  | フルパス             | `/Users/zenn/Downloads/myapp:% _` |
| **`%~`**  | フルパス 2           | `~/Downloads/myapp:% _`           |
| **`%c`**  | 相対パス             | `myapp:% _`                       |
| **`%2c`** | ディレクトリ深度 `2` | `Downloads/myapp:% _`             |

ここでは次のように設定しました。

```shell:~/.zshrc
PROMPT="%n ($(arch)):%~"$'\n'"%# "
```

- **`$()`:** カッコ内のコマンド結果を表示します
- **`"$'\n'"`:** プロンプトが長くなってしまったので途中に改行を挿入します

```sh
% source ~/.zshrc

zenn (arm64):~/Downloads/myapp
% _
```

https://zsh.sourceforge.io/Doc/Release/Prompt-Expansion.html

# 5. プロンプトへ色を付ける

**zsh** では簡単にプロンプトへ色をつけることができます。

まず、`~/.zshrc` の冒頭で色を扱うモジュールを有効化し、

```shell:~/.zshrc
autoload -Uz colors && colors
```

プロンプト内の文字列に色を指定します。

```shell:構文
%F{色の名前または色番号}hoge%f
```

**`%F` ~ `%f`** に挟まれた文字列 (`hoge`) に指定した色が付きます。

**色の名前:** `black` `red` `green` `yellow` `blue` `magenta` `cyan` `white`

```shell:~/.zshrc
PROMPT="%F{green}%n%f %F{cyan}($(arch))%f:%F{blue}%~%f"$'\n'"%# "
```

![](https://storage.googleapis.com/zenn-user-upload/30ed61fc1b86769093a05c0c.png =480x)

**色番号**を調べるには以下のスクリプトを実行してみてください。

:::details palette.sh（クリックで展開トグル）

```shell:palette.sh
#!/bin/sh
#
# 256色のカラーパレットを表示する
#  bash と zsh にて実行可能
#

target_shell=$1

if [ -z "$1" ]; then
    target_shell=$(basename "$SHELL")
fi

if [ "$target_shell" = "bash" ]; then
    bash <<< 'for code in {0..255}; do echo -n "[38;05;${code}m $(printf %03d $code)"; [ $((${code} % 16)) -eq 15 ] && echo; done'
elif [ "$target_shell" = "zsh" ]; then
    zsh  <<< 'for code in {000..255}; do print -nP -- "%F{$code}$code %f"; [ $((${code} % 16)) -eq 15 ] && echo; done'
else
    echo "error: Invalid argument ($target)"
    echo "Usage: $0 [bash|zsh]"
fi
```

:::

![](https://storage.googleapis.com/zenn-user-upload/f7154efa4b02e0afcfc49430.png =640x)

https://yonchu.hatenablog.com/entry/2012/10/20/044603

# 6. プロンプトへ Git リポジトリの状態を表示する

![](https://storage.googleapis.com/zenn-user-upload/796f83d31e4c-20230103.png =480x)

まず、**zsh-git-prompt** をインストールします。

```shell:shell
brew install zsh-git-prompt
```

```sh
~ 略 ~
==> Caveats
Make sure zsh-git-prompt is loaded from your .zshrc:
  source "/opt/homebrew/opt/zsh-git-prompt/zshrc.sh"
~ 略 ~

```

**Caveats** の指示に従い、`~/.zshrc` の中でスクリプトを読み込み、

```shell:~/.zshrc
source $(brew --prefix)/opt/zsh-git-prompt/zshrc.sh
```

プロンプトの中で `git_super_status` を展開するだけです。

```shell:~/.zshrc
PROMPT="%F{034}%h%f:%F{020}%~%f $(git_super_status)"$'\n'"%# "
```

:::message
[macOS 12 Monterey](https://www.apple.com/jp/macos/monterey/) 以降ではデフォルトパス内に `python` コマンドが存在しないため、エイリアスを設定しないと `git_super_status` が機能しません。

```shell:~/.zshrc
alias python="python3"
```

:::

## Git リポジトリ以外では `$(git_super_status)` を表示させたくない場合

**zsh** には、**bash** における [`PROMPT_COMMAND`](http://archive.linux.or.jp/JF/JFdocs/Bash-Prompt-HOWTO-3.html) と同等の[フック関数](https://zsh.sourceforge.io/Doc/Release/Functions.html#Special-Functions) **`precmd()`** が用意されています。
**precmd() フック**へ、そのディレクトリが Git リポジトリかどうか判定する関数を追加します。

```shell:~/.zshrc
git_prompt() {
  if [ "$(git rev-parse --is-inside-work-tree 2> /dev/null)" = true ]; then
    PROMPT="%F{034}%h%f:%F{020}%~%f $(git_super_status)"$'\n'"%# "
  else
    PROMPT="%F{034}%h%f:%F{020}%~%f "$'\n'"%# "
  fi
}

precmd() {
  git_prompt
}
```

![](https://storage.googleapis.com/zenn-user-upload/05574fa22437-20230103.png =480x)

# 7. コマンド実行結果のあとに空行を挿入する

コマンドの実行結果が表示されたあと、すぐにプロンプトが表示されるのはやや窮屈です。

```sh
zenn (arm64):~/Downloads
% ls -a
.		..		.DS_Store	.localized	myapp
zenn (arm64):~/Downloads
% _
```

`~/.zshrc` へ以下を追記します。

```shell:~/.zshrc
add_newline() {
  if [[ -z $PS1_NEWLINE_LOGIN ]]; then
    PS1_NEWLINE_LOGIN=true
  else
    printf '\n'
  fi
}

precmd() {
  git_prompt
  add_newline
}
```

コマンド実行後に空行が挿入されるようになりました。

```sh
zenn (arm64):~/Downloads
% source ~/.zshrc

zenn (arm64):~/Downloads
% _
```

# 8. あたらしくインストールされたコマンドを即認識させる

デフォルト状態の **zsh** では、あたらしくインストールされたコマンドをただちに認識してはくれません。

```sh
~% brew install yarn

~% yarn --version
zshell: command not found: yarn

```

`~/.zshrc` の冒頭へ以下の行を追記しましょう。

```shell:~/.zshrc
zstyle ":completion:*:commands" rehash 1
```

# 9. tarball へ macOS の特殊ファイルを含めないようにする

macOS で作成した **`tar.gz`** を Windows などの他の OS で解凍すると、`.DS_Store` ファイルや `._ (ドットアンダーバー)` ファイルなどの特殊ファイルが tarball に含まれてしまっていることがよくあります。
これを防ぐためのエイリアス関数を用意しましょう。

```shell:~/.zshrc
tgz() {
  env COPYFILE_DISABLE=1 tar zcvf "$1" --exclude=".DS_Store" "${@:2}"
}
```

**使い方：**

```sh
~% tgz dotfiles.tgz .zshrc .gitconfig .ssh/

a .zshrc
a .gitconfig
a .ssh
a .ssh/id_ed25519
a .ssh/id_ed25519.pub
a .ssh/config
a .ssh/known_hosts

~% ls
dotfiles.tgz
```

# 10. その他の小技集

### 1. `rm *` でいちいち確認しないで欲しい

![](https://storage.googleapis.com/zenn-user-upload/f6aad330e229-20220907.png =640x)

```shell:~/.zshrc
setopt RM_STAR_SILENT
```

### 2. `curl` などで **`?`** や **`&`** をエスケープなしで使いたい

```shell:~/.zshrc
setopt nonomatch
```

### 3. ファイル名を補完した後に挿入されたスペースが消えてしまう

https://qiita.com/sprout2000/questions/daaa95192fab3ac291ee

```shell:~/.zshrc
ZLE_REMOVE_SUFFIX_CHARS=$''
```

# ここまでの `~/.zshrc` と `~/.zshenv`

```shell:~/.zshrc
autoload -Uz colors && colors
zstyle ":completion:*:commands" rehash 1

setopt RM_STAR_SILENT
setopt nonomatch

ZLE_REMOVE_SUFFIX_CHARS=$''

typeset -U path PATH
path=(
  /opt/homebrew/bin(N-/)
  /opt/homebrew/sbin(N-/)
  /usr/bin
  /usr/sbin
  /bin
  /sbin
  /usr/local/bin(N-/)
  /usr/local/sbin(N-/)
  /Library/Apple/usr/bin
)

if type brew &>/dev/null; then
  FPATH=$(brew --prefix)/share/zsh-completions:$FPATH
  source $(brew --prefix)/share/zsh-autosuggestions/zsh-autosuggestions.zsh
  autoload -Uz compinit && compinit
fi

git_prompt() {
  if [ "$(git rev-parse --is-inside-work-tree 2> /dev/null)" = true ]; then
    PROMPT="%F{034}%h%f:%F{020}%~%f $(git_super_status)"$'\n'"%# "
  else
    PROMPT="%F{034}%h%f:%F{020}%~%f "$'\n'"%# "
  fi
}

add_newline() {
  if [[ -z $PS1_NEWLINE_LOGIN ]]; then
    PS1_NEWLINE_LOGIN=true
  else
    printf '\n'
  fi
}

tgz() {
  env COPYFILE_DISABLE=1 tar zcvf "$1" --exclude=".DS_Store" "${@:2}"
}

precmd() {
  git_prompt
  add_newline
}
```

```sh:~/.zshenv
. "$HOME/.zshrc"
```
