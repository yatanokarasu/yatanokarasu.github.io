---
date: 2025-03-04
authors: [yatanokarasu]
tags:
  - Container
  - Devcontainer
  - Podman
  - WSL
categories:
  - Tech Research
slug: podman-compose-with-devcontainer
---

# Devcontainer で Podman Compose を利用する

Devcontainer を使いながら複数のコンテナを起動したい場合は
Podman Compose (Docker Compose) が利用できる。
が、以下を実現する Devcontainer を作ろうとしてずいぶん試行錯誤したので、備忘としてメモしておく。

*   Devcontainer の起動に失敗する  
    (**こちらの問題は、 Compose には関係ないことに後で気づいた**)
*   [UID のマッピング防止](./../02/02-04-chowning-in-podman.md)に `--userns=keep-id` オプションが使えない

<!-- more -->

## この話の前提

今回、まずはじめに以下のような devcontainer.json と compose.yaml を作成した (Java 開発 & PlantUML で文書執筆環境を作ろうとしたことが目的)。

``` json title=".devcontainer/devcontainer.json"
{
    "name": "Java Workspace",
    "dockerComposeFile": [
        "compose.yaml"
    ],
    "service": "java-workspace",
    "workspaceFolder": "/home/vscode/workspace",

    "containerUser": "vscode"
}
```

``` yaml title=".devcontainer/compose.yaml"
services:
  java-workspace:
    image: mcr.microsoft.com/vscode/devcontainers/java:21-bullseye
    name: java-workspace
  plantuml:
    image: docker.io/plantuml/plantuml-server:jetty
    name: plantuml-server
```

## Devcontainer の起動に失敗する件について

結論から言ってしまえば、使ってるコンテナイメージの問題で、
TTY を付けてあげれば解決した。。。
使用している Dockerfile をちゃんと見れば済む話だった。

流れを追って説明すると、まず Devcontainer を起動しようとすると以下のエラーが出た。

``` console linenums="274"
    :
Error: can only create exec sessions on running containers: container state improper
Start: Run in container:  (command -v getent >/dev/null 2>&1 && getent passwd 'vscode' || grep -E '^vscode|^[^:]*:[^:]*:vscode:' /etc/passwd || true)
Stdin closed!
Error: An error occurred setting up the container.
    :
```

見ての通り `Stdin closed` ということで、
標準入力が閉じられておりコマンド実行が出来ないことが原因と分かる。

ここで、使用している [mcr.microsoft.com/vscode/devcontainers/java:21-bullseye の Dockerfile](https://github.com/devcontainers/images/blob/main/src/java/.devcontainer/Dockerfile) からベースイメージを辿っていくと、
最終的に [debian:bullseye](https://github.com/debuerreotype/docker-debian-artifacts/blob/dist-amd64/bullseye/Dockerfile) でやっと `CMD` 定義だけされていることが確認できる。

``` dockerfile title="debian:bullseye の Dockerfile" linenums="1"
FROM scratch
ADD oci/blobs/rootfs.tar.gz /
CMD ["bash"]
```

`CMD` で `bash` を指定しているので、
コンテナ起動時に TTY が割りあてられていないことで `bash` プロセスが終了し、
コンテナも終了していたというわけだった。

ということで、原因がわかれば解決策は非常に単純。
`compose.yaml` に以下の定義を追加すれば無事 Devcontainer が起動した。

``` diff
--- before
+++ after
@@ -2,6 +2,7 @@
   java-workspace:
     image: mcr.microsoft.com/vscode/devcontainers/java:21-bullseye
     name: java-workspace
+    tty: true
   plantuml:
    image: docker.io/plantuml/plantuml-server:jetty
```


## userns=keep-id を指定できない件

Compose を使わない `.devcontainer.json` であれば
`runArgs` で引数を指定できたけれど、
Compose の場合は使えない。
しかし、ユーザ名前空間の指定は環境変数でもできるので、
`compose.yaml` と同じディレクトリに `.env` を置いて以下の定義を入れれば解決。

``` ini title=".devcontainer/.env"
PODMAN_USERNS=keep-id
```
