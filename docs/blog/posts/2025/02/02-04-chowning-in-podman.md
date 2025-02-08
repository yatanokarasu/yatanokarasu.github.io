---
date: 2025-02-04
authors: [yatanokarasu]
tags:
  - Container
  - Devcontainer
  - Podman
  - WSL
categories:
  - Tech Research
slug: auto-chowning-in-container-with-podman
---

# Podman で起動したコンテナ内の UID とホストから見た UID が勝手に変わる件について調べてみた

!!! danger "(2025/02/08) 追記"

    いろいろ試してて、 [Workspace の bind mount 周り](#devcontainer-で-root-以外で-workspace-と-bind-mount-したい)について
    理解が足りなてかったところがあったので追記。

    結論としては、自前の Dockerfile 作ったりしなくて良かった。  
    （まぁ、普通に考えたら、 MS さんがそんな面倒なことさせるわけもないか。。。）

Podman in WSL2 で [Devcontainer](https://containers.dev/) を拵えようとしてて、
コンテナ内のユーザが `root` というのはセキュリティ的にどうなのかな？って思ったのがキッカケ。

その中で、 bind mount してコンテナ内からホスト側にもファイルを残そうと書き出すと、
なぜかコンテナ内の UID/GID と異なるオーナでファイルが保存されたり、
編集しようとすると `Permission Denied` って怒られたりする。

「何でこんなことなるの？」って思って、 Podman で作ったコンテナについて調べてみてたら、
どうやら以下のような挙動をしてるっぽい。

<!-- more -->

*   Dockerfile とかで `USER` 未指定もしくは `USER root` で動いているようなコンテナの場合、
    `root` で動いてるのに、ファイルを作ったりすると WSL の一般ユーザのパーミッションになっている

*   コンテナ内で UID (UserID) が `1000` のユーザでファイルを作ると、
    コンテナの外では `100999` とか桁違いな UID でファイルが出来上がったりする


## 結論

Linux カーネルのユーザ名前空間の仕組みで制御されているっぽい。
[RedHat ブログ](https://www.redhat.com/ja/blog/understanding-root-inside-and-outside-container) や
[赤帽エンジニアブログ](https://rheb.hatenablog.com/entry/2020/04/03/Podman%E3%81%A8%E3%83%A6%E3%83%BC%E3%82%B6%E5%90%8D%E5%89%8D%E7%A9%BA%E9%96%93%3A_%E6%9C%80%E9%AB%98%E3%81%AE%E7%B5%84%E3%81%BF%E3%81%82%E3%82%8F%E3%81%9B) にとっても詳しく書いてました。

コンテナ内の UID/GID とホストから見た UID/GID を上手くマッピングさせることで
コンテナ内ではいろいろ出来るけど、ホスト側には影響させないようにする仕組みなのでしょうね。
不勉強で全然知らなかった。。。

## 勝手に UID が変わらないようにする方法

かと言って、冒頭のように、
ホストのディレクトリを bind mount して双方向にやり取りしたいなぁって時には
この仕組みはちょっと不便。

そんな時は、コンテナ起動時に `#!bash --userns=keep-id` オプションを付けると良いみたい。
もしくは、環境変数 `#!bash PODMAN_USERNS=keep-id` を有効にした上で起動する。
毎回指定するのが面倒な場合は `~/.config/containers/containers.conf` ファイルを
以下のような内容で作成する。

``` ini title="~/.config/containers/containers.json"
[containers]
userns = "keep-id"
```

!!! warning "2025/02/08 追記 その2"

    `~/.config/containers/containers.conf` で、全コンテナに対して提供してしまうと、
    たとえば docker.io/library/nginx のように 80/tcp で LISTEN しようとするプロセスは
    `root` 権限が必要となるため、コンテナが起動しなくなってしまう。

    なので、基本的には `--userns=keep-id` や環境変数 `#!bash PODMAN_USERNS=keep-id` を指定して
    ホスト側とやり取りしたいコンテナだけに適用するのが望ましいかも。


## Devcontainer で root 以外で workspace と bind mount したい

Devcontainer で `root` 以外のユーザで作業しつつ、ホスト側の
ワークスペースを bind mount する場合は、 `userns` パラメータをいじるだけでは足りない。
==`devcontainer.json` の `containerUser` に、ホスト側と同じ UID/GID を持ったコンテナ内のユーザ名を
指定する必要がある。== 大体の場合、 WSL インストール時に作成したユーザは UID/GID 共に 1000 のはずなので、
コンテナ内のユーザも UID/GID が 1000 のものを指定すれば良い。

例えば、 Microsoft が提供している [vscode-devcontainers](https://hub.docker.com/r/microsoft/vscode-devcontainers) とかは、
[Commons Utilities](https://github.com/devcontainers/features/tree/main/src/common-utils) を使って `vscode (UID/GID=1000)` の
ユーザを用意してくれている[^1] ので、 `#!json "containerUser": "vscode"` を指定すると良い。

### (2025/02/08 追記版) 実際の成功例

`userns` を設定済であれば、以下の内容で devcontainer ↔️ コンテナ間でファイルの読み書きができるはず。
今回は Microsoft から提供されている [devcontainers/base の dev-bullseye (Debian 11)](https://mcr.microsoft.com/en-us/artifact/mar/devcontainers/base/tags) を利用した、おそらくどの OS イメージでも同様に `vscode` ユーザが使えると思う。

``` json title=".devcontainer/devcontainer.json"
{
    "name": "Test",

    "image": "mcr.microsoft.com/vscode/devcontainers/base:bullseye",

    // 今回は vscode ユーザ直下に workspace というフォルダに bind mount させている
    "workspaceFolder": "/home/vscode/workspace",
    "workspaceMount": "source=${localWorkspaceFolder},target=/home/vscode/workspace,type=bind",

    // userns=keep-id を指定する
    "runArgs": [
        "--userns", "keep-id"
    ],

    "containerUser": "vscode"
}
```

!!! danger "(2025/02/08) 追記"

    以降の内容は不要でした (ただ、せっかく調査した記録なので、残しておきます)。

    たとえば、 [コンテナ内のユーザの作成について](#コンテナ内のユーザの作成について)で記載している
    Commons Utilities なんかは、前述の通り[^1]すでに取り込まれています。

### やり方

結論から言えば、以下が必要になる。

*   Dockerfile の末尾の `USER UID[:GID]` に上記の UID/GID を指定する
*   コンテナ内に、 WSL ユーザと同じ UID/GID を持つユーザを作っておく

#### Dockerfile の USER ディレクトリの指定について

Devcontainer のコンテナイメージビルド時の挙動を調べてみると、
以下のようなコマンドでコンテナイメージをビルドしようとしていた (一部のパラメータだけ抜粋)。

``` shell
podman buildx build
  ...snip...
  -f /tmp/.../Dockerfile-with-features  # (1)!
  --build-arg _DEV_CONTAINERS_IMAGE_USER=root  # (2)!
  ...snip...
```

1.  自動的に合成されたコンテナイメージの定義ファイル
2.  定義ファイルの末尾にある `USER` ディレクトリの引数として使用 (定義がないと `root` を指定)

例えば、 `devcontainer.json` で指定したコンテナイメージの末尾の `USER` が `1000:1000` と定義されていた場合、
合成された定義ファイルの中身は以下のようになっており、

``` dockerfile title="合成されたコンテナイメージ定義ファイル抜粋"
# ---- ココから
USER 1000:1000
# ---- ココまでが devcontainers.json で指定したイメージの中身

...snip...

ARG _DEV_CONTAINERS_IMAGE_USER=root
USER $_DEV_CONTAINERS_image_user
...snip...
```

ビルド時の引数も `#!bash --build-arg _DEV_CONTAINERS_IMAGE_USER=1000:1000` となる。

#### コンテナ内のユーザの作成について

自分で頑張って Dockerfile 内に `useradd` とかして作っても良いのだけど、
`sudo` もさせたいとか考え出すといろいろと大変。。。

そんな時は、 [Dev Container Features](https://containers.dev/features) にある
[Commons Utilities](https://github.com/devcontainers/features/tree/main/src/common-utils) を使うと
簡単にユーザ作成とか `sudoer` の設定をしてくれる優れモノ。

`devcontainer.json` の `features` に以下の定義を加えてあげれば OK 。

``` json title=".devcontainer/devcontainer.json の抜粋"
    ...snip...
    "features": {
        "ghcr.io/devcontainers/features/common-utils:2": {
            // "installZsh": "false",
            // "installOhMyZsh": "false",
            // "installOhMyZshConfig": "false",
            "username": "vscode",
            "userUid": "1000",
            "userGid": "1000"
        }
    },
    ...snip...
```

上記のように、ユーザ名と UID/GID を指定して上げれば、
コンテナイメージをビルドする際に自動的に合成してくれてユーザを作ってくれる。
デフォルトでは ZSH をインストールするようなので、
不要であれば上記のコメントアウト箇所を解除すれば良い。

### 実際の成功例

以上の3つが整えば、 Devcontainer を `root` 以外で作業しつつ、
WSL 環境のディレクトリのファイルを使って相互作業ができるようになっているはず。

実際にざっくり試した環境は以下の通り。コンテナ内のユーザは `vscode (uid=1000, gid=1000)` にしています。

``` dockerfile title=".devcontainer/Dockerfile"
FROM mcr.microsoft.com/vscode/devcontainers/base:bullseye

USER 1000:1000
```

``` json title=".devcontainer/devcontainer.json"
{
    "name": "Test",

    "build": {
        "dockerfile": "./Dockerfile",
        "context": "."
    },

    "workspaceFolder": "/home/vscode/workspace",
    "workspaceMount": "source=${localWorkspaceFolder},target=/home/vscode/workspace,type=bind",

    "features": {
        "ghcr.io/devcontainers/features/common-utils:2": {
            "installZsh": "false",
            "installOhMyZsh": "false",
            "installOhMyZshConfig": "false",
            "upgradePackages": "true",
            "username": "vscode",
            "userUid": "1000",
            "userGid": "1000"
        }
    }
}
```

[^1]: https://github.com/devcontainers/images/blob/4fd0a81cc6442d2d2c95a1380fd21de203d60d1b/src/base-debian/.devcontainer/devcontainer.json#L7-L13
