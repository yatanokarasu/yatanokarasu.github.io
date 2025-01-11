---
tags:
  - Windows
  - WSL
  - Podman
---

# Install Podman

!!! warning

    本記事は、以下の内容で確認。詳しく確認はしていないけど、
    Ubuntu がこれより古い環境では Podman が上手く動かなかった。

    * **OS**: Windows 11 Pro 23H2
    * **WSL**: Ubuntu 24.04.01 LTS

## Podman のインストール

インストール自体は `apt` コマンドで簡単にインストール可能。

1.  パッケージを更新する

    ``` bash
    sudo apt update
    ```

2.  Podman をインストールする

    ``` bash
    sudo apt install -y podman
    ```

!!! info

    (本記事では、一旦手順は割愛するが)
    もし、 Windows 側からインストールした Podman を操作したいのであれば、
    以下のコマンドを実行してサービスとソケットを作成しておくと良い。

    ``` shell
    mkdir -p ~/.config/systemd/user/ &&
    sudo cp /usr/lib/systemd/user/podman.{service,socket} ~/.config/systemd/user/
    ```
