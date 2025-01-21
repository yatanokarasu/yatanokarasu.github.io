---
tags:
  - Windows
  - Node.js
---

# Install Node.js via fnm

Node.js のインストール手順として [NVM を使ったやり方](./nodejs-with-nvm.md) があるけど、
どうやら [FNM (Fast Node Manager)](https://github.com/Schniz/fnm) の方が良さそう。
個人的に感じたメリットは以下。

*   Rust 製なので軽量＆高速

*   バージョン切り替えに symlink ではなく junction を使っているので、
    管理者権限のターミナルでなくて良い。  
    (NVM は symlink でバージョンを切り替えるため、管理者権限のターミナルが必要)

## インストール手順

以下いずれかの方法でインストールする。

=== "GitHub から入手してインストール"

    1.  [FNM の GitHub リポジトリ](https://github.com/Schniz/fnm/releases) から、
        `fnm-windows.zip` をダウンロードする。

    1.  Zip ファイルに `fnm.exe` が含まれているので、
        任意のフォルダに展開する。

=== "Windows Package Manager を使ってインストール"

    PowerShell で以下コマンドを実行してインストールする。

    ``` ps1
    winget install `
        -e --id Schniz.fnm `
        -l C:\tools\fnm # (1)!
    ```

    1.  ここでは、インストール先を `C:\tools\fnm` としている

!!! tip

    インストール後、環境変数 `PATH` も更新されているため、
    ターミナルを新しく開けば `fnm` コマンドが使用できるようになっている。

## 事前準備

`fnm` コマンドを使う前に、ターミナルに環境変数を叩き込む必要がある。
[公式ドキュメント](https://github.com/Schniz/fnm#shell-setup) に従って、
使っているターミナルに合わせたコマンドを入力する。
ココでは Windows 環境であることを想定して、以下の PowerShell コマンドを実行する。

``` powershell
fnm env --use-on-cd --shell powershell | Out-String | Invoke-Expression
```

## Node.js のインストール

まず、どのようなバージョンが提供されているかを `fnm list-remote` コマンドで確認する。

``` console
PS> fnm list-remote
  :
v20.18.1 (Iron)
  :
v22.11.0 (Jod)
  :
v23.6.0
```

インストール可能なバージョンが一覧表示されるので、
希望するバージョンを `fnm install` コマンドでインストールする。

!!! info

    バージョン指定に `v` は付けなくても良い。

``` powershell
fnm install 20.18.1
```

インストールした後、 `fnm list` コマンドでインストール済のバージョンを、
現在使用中のバージョンを確認できる (青くハイライトされているはず)。

``` console
PS> fnm list
* v20.18.1 default
* system
```

もちろん、他のバージョンを追加でインストールすることも可能。

``` powershell
fnm install 22.13.0
```

もう一度 `fnm list` コマンドを実行すると、インストール済バージョンが増えている。

``` console
PS> fnm list
* v20.18.1 default
* v22.13.0
* system
```

## 他のバージョンに切り替え

複数インストールしている場合、 `fnm use` コマンドで Node.js のバージョンを切り替えることができる。
以下は、 Node.js v22.13.0 を使用する場合の例。

``` powershell
fnm use 22.13.0
```

ただし、 `fnm use` はターミナルのカレントセッションでのみ有効なので、
起動するターミナルの全てで使用するバージョンを指定したい場合は、
[次の節](#デフォルトバージョンの切り替え)を参照。

## デフォルトバージョンの切り替え

前節の通り、デフォルトで使用する Node.js バージョンを指定した場合は、
`fnm default` コマンドを使用する。例えば、以下の実行例のように、
現在のデフォルトバージョンが v20.18.1 とする。

``` console
PS> fnm list
* v20.18.1 default
* v22.13.0
* system
```

実際に、現在使用中の Node.js のバージョンを確認しても v20.18.1 が表示される。

``` console
PS> node -v
v20.18.1
```

ここで、デフォルトバージョンを v22.13.0 に切り替える。

``` powershell
fnm default 22.13.0
```

すると、デフォルトバージョンが v22.13.0 になっていることが確認できる。

``` console
PS> fnm list
* v20.18.1
* v22.13.0 default
* system
```

使用中の Node.js のバージョンも v22.13.0 に変わっているし、
新しいターミナルを開いてもデフォルトバージョンが使用可能になっている。

``` console
PS> node -v
v22.13.0
```
