---
tags:
  - Windows
  - WSL
  - Git
  - Git Credential Manager
---

# Use GCM from WSL

WSL から GitHub private リポジトリなど認証を必要とするリポジトリを利用時、
毎回認証を要求されることを避けたい。
Linux 版 GCM (Git Credential Manager) を導入することも可能であれば、
Windows の資格情報に保存して WSL ホスト (Windows マシン) からも
Git の資格情報を使用できるようにしたい。

ここでは、 WSL ホストに GCM をインストールしつつ、 WSL から GCM を利用する手順を示す。

## GCM のインストールと設定

1.  もし Git for Windows など、 Windows の Git クライアントをインストールしていなければ、
    PowerShell で以下のコマンドを実行して Git for Windows をインストールする  
    (GCM 単体では `git.exe` が見つからないことでエラーになる)

    ``` ps1
    winget install -e --id Git.Git -l C:\tools\git  # (1)!
    ```

    1.  ココでは `C:\tools\git` にインストールしている

1.  同様に、 PowerShell で以下のコマンドを実行して GCM をインストールする

    ``` ps1
    winget install -e --id Git.GCM -l C:\tools\gcm  # (1)!
    ```

    1.  ココでは `C:\tools\gcm` にインストールしている

1.  WSL の `${HOME}/.gitconfig` に GCM を `credential.helper` として設定する

    ``` ps1
    wsl -- git config --global credential.helper `
        "$(wsl wslpath "'C:\tools\gcm\git-credential-helper.exe'")"  # (1)!
    ```

    1.  直前でインストールした GCM のパスを指定する

以上でインストールと設定は完了。 `git.exe` のパスを有効にする必要があるので、
一度起動済の WSL ターミナルを全て終了させて、
`#!powershell wsl --shutdown` で WSL をシャットダウンしてから起動しなおすこと。
