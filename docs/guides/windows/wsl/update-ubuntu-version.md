---
tags:
  - Windows
  - WSL
---

# Update Ubuntu version on WSL2

インストール済の Ubuntu on WSL2 のバージョンを上げる手順は以下の通り。

!!! warning

    ココでは、本記事執筆 2025/01/09 時点の Ubuntu 22.04.05 LTS から Ubuntu 24.04.01 LTS にアップデートする手順を例とする。
    バージョンは適宜環境や時期に合わせて読み替えること。

1.  現在のバージョンを確認しておく

    ``` console
    $ grep -i /etc/os-release
      :
    VERSION="22.04.05 LTS (Jammy Jellyfish)"
      :
    ```

1.  インストールされているパッケージとその依存関係をアップデートしておく

    ``` bash
    sudo apt update && sudo apt upgrade
    ```

1.  ディストリビューションのアップデート 兼 不要なパッケージの削除

    ``` bash
    sudo apt full-upgrade
    ```

    !!! tip

        `apt dist-upgrade` と `apt full-upgrade` は等価らしい。  
        https://askubuntu.com/questions/770135/apt-full-upgrade-versus-apt-get-dist-upgrade

        ただし、 `apt-get -h` では `dist-upgrade` が、 `apt -h` では `full-upgrade` が表示されるので、
        `apt` コマンドを使用する場合は `full-upgrade` とする方が良いのかも。

1.  リリースアップデートのパッケージをインストール

    ``` bash
    $ sudo apt install -y update-manager-core
    ```

1.  LTS 版にアップデートするように指定しておく

    ``` bash
    $ sudo sed -i '/^Prompt/ s,=.*,=lts,' /etc/update-manager/releaes-upgrades
    ```

1.  リリースアップデートを実行

    ``` bash
    sudo do-release-upgrade
    ```

    途中、いろいろ同意を求められることもあるが `y` と答えて進める。
    全部終わったら、一旦 WSL2 のターミナルを閉じる。

6.  念のため、 WSL2 をシャットダウンする

    ``` batch
    wsl --shutdown
    ```

7.  改めて WSL2 を起動すると、バージョンが上がっているはず

    ``` console
    $ grep -i /etc/os-release
      :
    VERSION="24.04.01 LTS (Noble Numbat)"
      :
    ```
