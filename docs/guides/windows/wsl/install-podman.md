---
tags:
  - Windows
  - WSL
  - Podman
---

# Install Podman on WSL

WSL の Ubuntu に Podman をインストールしてコンテナを扱える環境を構築する。

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

## Podman でコンテナの動作確認

試しに NGINX のコンテナを動かしてみる。

``` shell
podman run --rm -p 8080:80 docker.io/library/nginx:latest
```

??? tip "コンテナイメージの完全修飾名の省略"
    Podman は、コンテナイメージをレジストリ URL も含んだ完全修飾名で指定することを期待しているので、
    以下のようにコンテナイメージ名だけ指定してもエラーがでる。

    ``` console
    $ podman run --rm -p 8080:80 nginx:latest
    Error: short-name "nginx:latest" did not resolve to an alias and no unqualified-search registries are defined in "/etc/containers/registries.conf"
    ```

    本来、完全修飾名で指定することが望ましいけども、自分の開発マシンだったりでわざわざ記述することが面倒な場合は、
    エラーに表示されている通り `/etc/containers/registries.conf` の `unqualified-search-registries` に
    省略したいレジストリ名を追加する。以下は `docker.io` を省略したい場合の例。

    ``` diff
    --- before
    +++ after
    @@ -19,6 +19,7 @@
    #
    # # An array of host[:port] registries to try when pulling an unqualified image, in order.
    # unqualified-search-registries = ["example.com"]
    +unqualified-search-registries = ["docker.io"]
    #
    # [[registry]]
    # # The "prefix" field is used to choose the relevant [[registry]] TOML table;
    ```

    追加した後は、イメージ名だけコンテナを Pull することができる (1行目のメッセージから `unqualified-search` が行われていることが分かる)。

    ``` console
    $ podman run --rm -p 8080:80 nginx:latest
    Resolving "nginx" using unqualified-search registries (/etc/containers/registries.conf)
    Trying to pull docker.io/library/nginx:latest...
        :
    ```

WSL 上の Podman で起動したコンテナのポートは、ホスト OS (Windows) のブラウザからもアクセス可能。
試しに PowerShell で `curl` (`Invoke-WebRequest`) を送ると NGINX のコンテンツが返ってくる。

``` console
PS> $(curl http://localhost:8080/).Content
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```
