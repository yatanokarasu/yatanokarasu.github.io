---
date: 2025-02-10
authors: [yatanokarasu]
tags:
  - Container
  - CGroup
  - Devcontainer
  - Kubernetes
  - k3d
  - Podman
  - WSL
categories:
  - Tech Research
slug: kind-with-podman-in-wsl
---

# Podman in WSL で k3d を使って Kubernetes で遊びたい

前回は [kind を使って Kubernetes クラスタを構築してみたけど](./02-09-kind-with-podman-in-wsl.md)、
今度は [k3d](https://k3d.io/) を使って環境構築を試してみる。

<!-- more -->

!!! info

    * **OS**: Windows 11 Pro 23H2
    * **WSL**: Ubuntu 24.04.01 LTS
    * **Podman**: v4.9.3 (インストール手順は[コチラ](./../../../../guides/windows/wsl/install-podman.md))

まずは[公式ドキュメント](hhttps://k3d.io/stable/#releases)に従って `k3d` コマンドをインストール。

``` shell
curl -s https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash
```

[前回の kind の時](./02-09-kind-with-podman-in-wsl.md)と同様、 WSL2 で CGroupv2 になってない場合は、
以下のファイルを作成して、 `#!powershell wsl --shutdown` を実行して WSL を再起動しておく。

``` ini title="%USERPROFILE%\.wslconfig"
[wsl2]
kernelCommandLine = cgroup_no_v1=all
```

では、さっそく k3d クラスタを作成してみる。が、以下のようなエラーがでた。
k3d は固定で Docker の Socket ファイルを参照しようとするみたい。

``` console
$ k3d cluster create test
   :
ERRO[0000] Failed to get nodes for cluster 'test': ...omit...: Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
   :
```

ということで、環境変数 `DOCKER_HOST` に podman の Socket ファイルを指定してみる。

!!! warning

    Podman の Socket ファイルが存在しない場合は、以下のコマンドを実行する。

    ``` shell
    mkdir -p ~/.config/systemd/user/;
    sudo cp /usr/lib/systemd/user/podman.{service,socket} ~/.config/systemd/user/;
    systemctl --user daemon-reload
    ```

``` bash
export DOCKER_HOST="unix:///run/user/$(id -u)/podman/podman.sock"
```

もう一度クラスタ作成を試みるが、まだ Docker の Socket ファイルでエラーが出る。

``` console
$ k3d cluster create test
   :
ERRO[0006] failed to ensure tools node: failed to run k3d-tools node for cluster 'test': ...omit...: mkdir /var/run/docker.sock: permission denied
   :
```

オプションの `--trace` を有効にして詳細なログを見てみると、
環境変数 `DOCKER_SOCK` というのも参照しているみたい。

``` console
$ k3d cluster create test --trace
DEBU[0001] DOCKER_SOCK=/var/run/docker.sock
   :
```

ということで、もう一度環境変数を設定する (`DOCKER_SOCK` には `unix://` のスキームは不要)。

``` bash
export DOCKER_SOCK=${DOCKER_HOST#*://}
```

再度クラスタ作成。上手く進んでる感じがしたが、
どうも API Server 用の Server コンテナ起動で刺さってる様子。。。

``` console
k3d cluster create test
   :
INFO[0007] Starting node 'k3d-test-server-0' # ココで刺さる
```

別のターミナルで `k3d-test-server-0` コンテナのログを追ってみると、
CGroup 周りで何かエラーが出ている。

``` console
$ podman logs -f k3d-test-server-0
   :
time="...omit..." level=fatal msg="failed to find cpuset cgroup (v2)"
   :
```

CGroup の設定を確認してみると、たしかに `cpuset` はなさそう。

``` console
$ cat /sys/fs/cgroup/user.slice/user-$(id -u).slice/user@$(id -u).service/cgroup.controllers
cpu memory pids
```

ということで、 `cpuset` についても CGroup に委任できるように設定を追加する。

!!! warning

    ただ、これをやるとパフォーマンスが落ちてしまう様子 (だから、デフォルトでは有効になっていないとのこと)。
    そういう意味だと、WSL で Kubernetes を遊ぶ場合は kind の方が良いかも。

``` bash
sudo sh -c '
   mkdir -p /etc/systemd/system/user@.service.d/
   echo "[Service]\nDelegate=cpuset" >>/etc/systemd/system/user@.service.d/delegate.conf
   systemctl daemon-reload
'
```

もう一度確認してみると `cpuset` が含まれている。
もし、設定が反映がされていない場合は、 `#!powershell wsl --shutdown` をして
WSL を再起動すれば OK。

``` console
$ cat /sys/fs/cgroup/user.slice/user-$(id -u).slice/user@$(id -u).service/cgroup.controllers
cpuset cpu memory pids
```

さすがにもう大丈夫だろう？って思ったけど、やっぱりまだ `k3d-test-server-0` で刺さる。
もう一度 `k3d-test-server-0` のコンテナログを追うと、
Kubelet のユーザ名前空間で動かすための機能フラグが必要っぽい。

``` console
$ podman logs -f k3d-test-server-0`
   :
E0209 ... 99 container_manager_linux.go:438] "Updating kernel flag failed (Hint: enable KubeletInUserNamespace feature flag to ignore the error)" err="open /proc/sys/kernel/panic: permission denied" flag="kernel/panic"
```

ということで、以下のように実行することで無事 k3d クラスタを作成することが出来た。

``` console
$ k3d cluster create test \
   --k3s-arg '--kubelet-arg=feature-gates=KubeletInUserNamespace=true@server:*;agent:*'
INFO[0002] Prep: Network
INFO[0002] Re-using existing network 'k3d-test' (fa174386851d374a5884829e5c357fac46f92111ccf558b100371bb1f83ff853)
INFO[0002] Created image volume k3d-test-images
INFO[0002] Starting new tools node...
INFO[0003] Creating node 'k3d-test-server-0'
INFO[0004] Creating LoadBalancer 'k3d-test-serverlb'
INFO[0004] Starting node 'k3d-test-tools'
INFO[0006] Using the k3d-tools node to gather environment information
INFO[0010] HostIP: using network gateway 10.89.1.1 address
INFO[0010] Starting cluster 'test'
INFO[0010] Starting servers...
INFO[0011] Starting node 'k3d-test-server-0'
INFO[0027] All agents already running.
INFO[0027] Starting helpers...
INFO[0030] Starting node 'k3d-test-serverlb'
INFO[0040] Injecting records for hostAliases (incl. host.k3d.internal) and for 2 network members into CoreDNS configmap...
INFO[0045] Cluster 'test' created successfully!
INFO[0047] You can now use it like this:
kubectl cluster-info
```

出力に出ている通り、 `kubectl cluster-info` コマンドで動作確認してみる。

``` console
$ kubectl cluster-info
Kubernetes control plane is running at https://0.0.0.0:45715
CoreDNS is running at https://0.0.0.0:45715/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
Metrics-server is running at https://0.0.0.0:45715/api/v1/namespaces/kube-system/services/https:metrics-server:https/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

無事クラスタが動いてそう。 `kubectl get` などで、他の情報も確認できた。

``` console
$ kubectl get nodes
NAME                STATUS   ROLES                  AGE   VERSION
k3d-test-server-0   Ready    control-plane,master   68s   v1.31.4+k3s1
```
