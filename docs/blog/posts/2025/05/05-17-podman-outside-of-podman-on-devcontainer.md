---
date: 2025-05-17
authors: [yatanokarasu]
tags:
  - Container
  - Devcontainer
  - Podman
  - WSL
categories:
  - Tech Research
slug: podman-outside-of-podman-on-devcontainer
---

# PooP (Podman outside of Podman) を Devcontainer でやってみた

PooP なんて単語が見つからないけど、 DooD (Docker outside of Docker) の Podman 版。
Devcontainer から `podman` コマンドでコンテナを使いたくなったので試してみた。



<!-- more -->

## `podman.sock` を bind マウント

やることは単純で `/var/run/podman.sock` (Rootless なら `${XDG_RUNTIME_DIR}/podman/podman.sock`
(`/run/user/$(id -u)/podman/podman.sock`)) をバインドマウントするだけ。
Devcontainer のイメージは `mcr.microsoft.com/devcontainers/base:bullseye` を使ってみた。

``` json title="devcontainer.json 抜粋"
{
    ...omit...
    "image": "mcr.microsoft.com/devcontainers/base:bullseye",
    ...omit...
    "mounts": [
        {
            // 環境変数では指定できないので、直接指定する
            // ここでは UID=1000 として指定している
            "source": "/run/user/1000/podman/podman.sock",
            "target": "/run/user/1000/podman/podman.sock",
            "type": "bind"
        }
    ],
    ...omit...
}
```

## Devcontainer で Podman をインストールして動作確認

お次に、Devcontainer 内で `podman` コマンドをインストールする。

``` bash
sudo apt update && sudo apt install -y podman
```

あとは、 `--url` オプションに `podman.sock` のパスを指定すれば
ホストの Podman を通して操作が可能になる。

``` bash
podman --url unix:///run/user/1000/podman/podman.sock version
```

ただ、毎回長々しくパスを指定するのもメンドウなので、
環境変数 `CONTAINER_HOST` に `podman.sock` のパスを指定すれば省略可能。

``` bash
export CONTAINER_HOST=unix:///run/user/1000/podman/podman.sock
```

...と、思ったのだけど、なぜか上手くいかない。。。

``` bash
podman version
ERRO[0000] running `/usr/bin/newuidmap 3632 0 1000 1 1 100000 65536`: newuidmap: write to uid_map failed: Operation not permitted
Error: cannot set up namespace using "/usr/bin/newuidmap": exit status 1
```

いろいろ調べたが、おそらく Debian bullseye でインストールされる
Podman のバージョンが 3.x 系で古いせいっぽい。

ということで、 Debian bookworm だと Podman 4.x がインストールされて
`CONTAINER_HOST` を通してオプション指定なしで動作させることが出来た。
(上側が Devcontainer の version、下側が Host 側の version)

``` console
$ podman version
Client:       Podman Engine
Version:      4.3.1
API Version:  4.3.1
Go Version:   go1.19.8
Built:        Thu Jan  1 09:00:00 1970
OS/Arch:      linux/amd64

Server:       Podman Engine
Version:      4.9.3
API Version:  4.9.3
Go Version:   go1.22.2
Built:        Thu Jan  1 09:00:00 1970
OS/Arch:      linux/amd64
```

## 最終的な `devcontainer.json`

`onCreateCommand` でコンテナ起動直後に Podman をインストールするようにして、
環境変数 `CONTAINER_HOST` も最初から設定しておけば、すぐに Podman を使えて便利。

``` json
{
    "name": "Podman outside of Podman",
    "image": "mcr.microsoft.com/devcontainers/base:bookworm",
    "onCreateCommand": "sudo apt update && sudo apt install -y podman podman-compose podman-docker",

    "runArgs": [
        "--name", "poop-workspace",
        "--userns", "keep-id"
    ],

    "mounts": [
        {
            "source": "/run/user/1000/podman/podman.sock",
            "target": "/run/user/1000/podman/podman.sock",
            "type": "bind"
        }
    ],

    "shutdownAction": "stopContainer",
    "containerEnv": {
        "TZ": "Asia/Tokyo",
        "CONTAINER_HOST": "unix:///run/user/1000/podman/podman.sock"
    },

    "containerUser": "vscode"
}

```
