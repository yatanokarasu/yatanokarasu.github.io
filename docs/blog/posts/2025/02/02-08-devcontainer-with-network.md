---
date: 2025-02-08
authors: [yatanokarasu]
tags:
  - Container
  - Devcontainer
  - Podman
  - WSL
categories:
  - Tech Research
slug: devcontainer-with-networking
---

# Devcontainer 用のコンテナネットワーク

各々の Devcontainer は個別のネットワーク空間に閉じられてしまう。
もし別コンテナのサーバプロセスと連携させたい場合は、
別途コンテナネットワークを用意して同一ネットワーク上にコンテナを作成する必要がある。

<!-- more -->

すでに希望するコンテナネットワークが存在してるのであれば、
`.devcontainer/devcontainer.json` の `runArgs` に `podman` (もしくは `docker`) コマンドの
`--network` オプションを指定すれば OK。

``` json title=".devcontainer/devcontainer.json"
    ...snip...
    "runArgs": [
        "--network", "devnet" // (1)!
    ],
    ...snip...
```

1.  ココでは devnet という名前にしている

ただ、事前に用意するためのコマンドをいちいち打つのもメンドウなので、
同じく `initializeCommand` にネットワークの存在をチェックして無ければ作成するコマンドを指定しておくと便利。

``` json title=".devcontainer/devcontainer.json"
    ...snip...
    "runArgs": [
        "--network", "devnet"
    ],

    "initializeCommand": "podman network ls --noheading \\
        | awk '{print $2}' | grep -q devnet || podman network create devnet",
    ...snip...
```

動作確認用に NGINX を実行してみる。

``` shell
podman run --rm -d \
    --name text-nginx \
    docker.io/library/nginx:latest # (1)!
```

1.  コンテナ名は何でも良いが、 `test-nginx` とした

先ほど起動している Devcontainer から以下のコマンドを実行すると、
ちゃんと、コンテナ `text-nginx` の NGINX から応答が来ることが確認できる。

``` console
(devcontainer)$ curl -I test-nginx
HTTP/1.1 200 OK
Server: nginx/x.y.z
...snip...
```
