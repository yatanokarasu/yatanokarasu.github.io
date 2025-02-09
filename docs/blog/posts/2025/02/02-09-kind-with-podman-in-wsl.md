---
date: 2025-02-09
authors: [yatanokarasu]
tags:
  - Container
  - CGroup
  - Devcontainer
  - Kubernetes
  - kind
  - Podman
  - WSL
categories:
  - Tech Research
slug: kind-with-podman-in-wsl
---

# Podman in WSL で kind を使って Kubernetes で遊びたい

Kubernetes で遊ぶにはいろんな便利なものがあるけど、
今回は [kind](https://kind.sigs.k8s.io/) を使って環境構築を試してみる。

<!-- more -->

!!! info

    * **OS**: Windows 11 Pro 23H2
    * **WSL**: Ubuntu 24.04.01 LTS
    * **Podman**: v4.9.3 (インストール手順は[コチラ](./../../../../guides/windows/wsl/install-podman.md))

まずは[公式ドキュメント](https://kind.sigs.k8s.io/docs/user/quick-start/#installing-from-release-binaries)に従って `kind` コマンドをインストール。

``` shell
# For AMD64 / x86_64
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.26.0/kind-linux-amd64
# For ARM64
[ $(uname -m) = aarch64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.26.0/kind-linux-arm64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

さっそく起動してみる。が、以下のようなエラーがでた。
WSL2 では、互換性のため cgoupv1 を使っているようなので、
まずは WSL2 を cgroupv2 で動かしてあげる必要があるみたい。

``` console
$ kind create cluster
enabling experimental podman provider
ERROR: failed to create cluster: running kind with rootless provider requires cgroup v2, see https://kind.sigs.k8s.io/docs/user/rootless/
```

cgroupv2 にするには、 Windows 側で `%USERPROFILE\.wslconfig` に以下の内容を追加する。

!!! info "`/etc/wsl.conf` と `%USERPROFILE%\.wslconfig` の違い"

    Windows ホスト側の `%USERPROFILE%\.wslconfig` は、WSL 全体のグローバル設定用。
    詳しくは [MS 公式サイト](https://learn.microsoft.com/ja-jp/windows/wsl/wsl-config#configuration-settings-for-wslconf) 参照。

``` ini title="%USERPROFILE%\.wslconfig"
[wsl2]
kernelCommandLine = cgroup_no_v1=all
```

ファイル作成を終えたら、一旦 WSL をシャットダウンして起動しなおす。

``` powershell
wsl --shutdown
```

起動しなおした後、以下のコマンドを実行して `cgroup2fs` と出れば OK。

``` console
$ stat -fc %T /sys/fs/cgroup/
cgroup2fs
```

もう一度 kind クラスタを作成して上手くいけば OK。

``` console
$ kind create cluster
enabling experimental podman provider
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.32.0) 🖼
 ✓ Preparing nodes 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Have a question, bug, or feature request? Let us know! https://kind.sigs.k8s.io/#community 🙂
```

出力に出ている通り、 `kubectl cluster-info` コマンドで動作確認してみる。

``` console
$ kubectl cluster-info --context kind-kind
Kubernetes control plane is running at https://127.0.0.1:36217
CoreDNS is running at https://127.0.0.1:36217/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

無事クラスタが動いてそう。 `kubectl get` などで、他の情報も確認できた。

``` console
$ kubectl get nodes
NAME                STATUS  ROLES           AGE   VERSION
kind-control-plane  Ready   control-plane   14m   v1.32.0
```
