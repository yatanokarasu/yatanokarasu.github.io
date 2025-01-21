---
tags:
  - Windows
  - Node.js
---

# Install Node.js via NVM for Windows

Node.js は、 JavaScript の実行環境の 1 つ。
ここでは、 Node.js のバージョン管理も行える NVM for Windows[^1] を使用してインストールを行う。

## インストール手順

以下いずれかの方法でインストールする。

=== "公式サイトからインストーラを入手してインストール"

    [NVM for Windows 公式サイト](https://github.com/coreybutler/nvm-windows?tab=readme-ov-file#install-nvm-windows)
    に従い、以下の手順でインストールする。

    1.  [最新版のリリースページ](https://github.com/coreybutler/nvm/releases) から
        `nvm-setup.zip` をダウンロードする。

    2.  Zip ファイルに `nvm-setup.exe` が含まれているので、
        展開・実行して任意のフォルダにインストールする。

=== "Windows Package Manager を使ってインストール"

    PowerShell で以下コマンドを実行してインストールする。

    ``` ps1
    winget install `
        -e --id CoreyButler.NVMforWindows `
        -l C:\tools\nvm # (1)!
    ```

    1.  ここでは、インストール先を `C:\tools\nvm` としている

!!! tip

    インストール後、環境変数 `PATH` も更新されているため、
    ターミナルを新しく開けば `nvm` コマンドが使用できるようになっている。

??? "Proxy 環境下で npm コマンドを使う場合"

    プロキシにより `npm` コマンドによる依存モジュールのダウンロードに失敗する場合は、
    PowerShell で以下のコマンドを実行する。もし、プロキシサーバの証明書検証が求められる場合は、
    `cafile` パラメータに信頼済 CA 証明書ファイルを指定する必要もあり。

    ``` ps1
    $user_home = [Environment]::GetEnvironmentVariable('USERPROFILE')

    $prefix_path = "${user_home}\.npm_global" # (1)!

    @"
    proxy=<PROXY-SERVER:PORT>
    https-proxy=<PROXY-SERVER:PORT>
    cafile=<TRUST-CA-CERTFILE>
    "@ >"${user_home}\.npmrc"
    ```

    1.  ここでは、 `%USERPROFILE%\.npm_global` としているが、お好みで変えても問題なし。
        このパスには、 `npm -g install` コマンドでインストールしたパッケージやコマンドが格納される。

## Node.js のインストール

### LTS 版をインストール

LTS 版の Node.js をインストールするには `#!bash nvm install lts` を実行する。
`lts` を指定した場合は、実行した時点での最新の LTS 版がインストールされる。

``` powershell
nvm install lts
```

### 特定のバージョンをインストール

特定のバージョンをインストールしたい場合は、
[インストール可能なバージョンを確認](#インストール可能なバージョンの一覧表示)して、
`#!bash nvm install VERSION` の `VERSION` に指定する。
例えば、 `20.10.0` をインストールしたい場合は `#!bash nvm install 20.10.0` を実行する。

## インストール可能なバージョンの一覧表示

使用したい Node.js のバージョン一覧を表示するには `#!bash nvm list available` を実行する。

``` console
$ nvm list available

|   CURRENT    |     LTS      |  OLD STABLE  | OLD UNSTABLE |
|--------------|--------------|--------------|--------------|
|    21.5.0    |   20.10.0    |   0.12.18    |   0.11.16    |
|    21.4.0    |    20.9.0    |   0.12.17    |   0.11.15    |
|      :       |      :       |      :       |      :       |
|    19.9.0    |   16.20.0    |   0.10.48    |    0.9.10    |

This is a partial list. For a complete list, visit https://nodejs.org/en/download/releases
```

## インストール済の Node.js バージョンを一覧表示

`#!bash nvm install` でインストール済みのバージョン一覧を表示するには `#!bash nvm list` を実行する。
`*` が付いているバージョンが、現在選択されているバージョンになる。
複数のバージョンをインストールしている場合は、[使用したいバージョンに切り替える](#使用したいバージョンを指定)ことも可能。

``` console
$ nvm list

  * 19.9.0 (Currently using 64-bit executable)
      :
    20.10.0
```

## 使用したいバージョンを指定

複数のバージョンをインストールしている場合は、使用するバージョンを切り替えることができる。
使用するバージョンを指定するには `#!bash nvm use VERSION` を実行する。

!!! danger "シンボリックリンクの作成には管理者権限が必要"

    `#!bash nvm use` コマンドは、 `C:\Program Files\nodejs` から `${NVM_HOME}/VERSION` に
    対してシンボリックリンクを作成するため、管理者権限を持ったコマンドプロンプトもしくは
    PowerShell で実行する必要がある。

    [コチラの設定](./../configurations/symlink-as-non-root-user.md)を行っておけば、
    管理者モードでターミナルを開かなくても、一般ユーザでもシンボリックリンクが
    作成できるようになる。

``` console
$ nvm use 20.10.0 # (1)!
Now using node v20.10.0 (64-bit)
```

1.  ここでは、 `20.10.0` を使用する場合を例にしている。

`#!bash nvm list` を実行して、使用するバージョンが変わっていることを確認できる。

``` console
$ nvm list

    19.9.0
      :
  * 20.10.0 (Currently using 64-bit executable)
```

[^1]:   NVM for Windows は、 [Microsoft](https://learn.microsoft.com/en-us/windows/dev-environment/javascript/nodejs-on-windows)、 [npm](https://docs.npmjs.com/cli/v9/configuring-npm/install#windows-node-version-managers)、
[Google](https://cloud.google.com/nodejs/docs/setup?hl=ja#installing_nvm) も利用を推奨している。
