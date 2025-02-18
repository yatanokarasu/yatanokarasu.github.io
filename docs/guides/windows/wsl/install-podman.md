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

    単にコンテナを使うだけなら不要だが、
    DinD のようにコンテナ内からコンテナホストの Podman
    を操作したい場合などは、以下のコマンドを実行して Podman の UNIX ソケットを作成しておく。  
    (`/run/user/$(id -u)/podman/podman.socket` に作成される)

    ``` shell
    mkdir -p ~/.config/systemd/user/
    sudo cp /usr/lib/systemd/user/podman.{service,socket} ~/.config/systemd/user/
    systemctl --user enable --now podman.socket
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

!!! danger "Windows 側では IPv6 にのみ bind される (2025/02/18追記)"

    上記のように `-p` でコンテナポートをホストに公開しているにもかかわらず、
    ホストマシンの Windows からアクセスできない場合は、以下のようにしてIPv4 にバインドすること。

    ``` shell
    podman run ... -p 127.0.0.1:8080:80 ...
    ```

    理由としては、 Linux (WSL の Ubuntu) 上では `*:8080` として
    IPv4/IPv6 のデュアルスタックで bind されるが、
    Windows 側では `[::1]:8080` と [IPv6 にのみ bind されるため](https://github.com/containers/podman/issues/12292#issuecomment-1034165121)。

    実際、[前述の NGINX 起動後](#podman-でコンテナの動作確認)、
    WSL (Linux) 側とホスト (Windows) 側で違いを確認できる。

    **WSL (Linux) 側**

    `*:8080` なので、いずれの IP address でも 8080/tcp で bind されている。

    ``` console
    $ ss -lnpt | grep -i 8080
    LISTEN 0    4096    *:8080      *:*    users:(("rootlessport",pid=252807,fd=11))
    ```

    **ホスト (Windows) 側**

    `[::1]:8080` と、 IPv6 の Loopback アドレスにのみ bind されている (`0.0.0.0:8080` が無い)。

    ``` console
    PS1> netstat -ano | Select-String '8080'
    TCP     [::1]:8080      [::]:0      LISTENING   13440
    ```
