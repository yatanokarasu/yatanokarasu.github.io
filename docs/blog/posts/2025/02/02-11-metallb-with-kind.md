---
date: 2025-02-12
authors: [yatanokarasu]
tags:
  - Container
  - Devcontainer
  - kind
  - Kubernetes
  - LoadBalancer
  - MetalLB
  - Podman
  - WSL
categories:
  - Tech Research
slug: installing-metallb-on-kind
---

# kind で MetalLB を使って type=LoadBalancer を使いたい

kind を使って Kubernetes クラスタを作ったとしても、そのままだと
`type=LoadBalancer` の Service 作ってクラスタ外からアクセスさせることができない。

今回は [MetalLB](https://metallb.io/) を使って、 kind で作った Kubernetes クラスタで
`type=LoadBalancer` の Service を作ってみた
(といっても、公式ドキュメントに従って、ただただコピペしただけだけど。)

<!-- more -->

## MetalLB のインストール

[公式ドキュメント](https://metallb.universe.tf/installation/)に従って以下のコマンドをコピペする。
何か kube-proxy に ARP に関する設定をしてるっぽいけど、細かいことは調べてないのでよくわからない。
たぶん、 MetalLB にパケットが流れるように ARP に対して固定で応答を返すようなことをしているのかな。

``` shell
# see what changes would be made, returns nonzero returncode if different
kubectl get configmap kube-proxy -n kube-system -o yaml | \
sed -e "s/strictARP: false/strictARP: true/" | \
kubectl diff -f - -n kube-system

# actually apply the changes, returns nonzero returncode on errors only
kubectl get configmap kube-proxy -n kube-system -o yaml | \
sed -e "s/strictARP: false/strictARP: true/" | \
kubectl apply -f - -n kube-system
```

kube-proxy の設定が終わったら、MetalLB をインストールする (MetalLB のマニフェストを叩き込むだけ)。

``` bash
kubectl apply \
    -f https://raw.githubusercontent.com/metallb/metallb/v0.14.9/config/manifests/metallb-native.yaml
```

インストールはコレだけだけど、このままじゃ IP アドレス払い出しが上手くいかないので、次章の設定もしていく。

## MetalLB の設定

まずは、 MetalLB が使えるアドレス帯のために、
kind が提供するコンテナネットワークのネットワークアドレスを調べる。
IPv4 と IPv6 のアドレスが出力されると思うが、 IPv4 のものを控えておく (ココでは `10.89.2.0/24`)。

``` console
$ podman network inspect kind | grep -i subnet\"
      "subnet": "...omit (ipv6)...",
      "subnet": "10.89.2.0/24",
```

以下の内容の MetalLB のコンフィグを作る。

``` yaml
---
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: demo-pool
  namespace: metallb-system
spec:
  addresses:
  - 10.89.2.2-10.89.2.255 # (1)!
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: demo-advertisement
  namespace: metallb-system
```

1.  最下位 bit が `1` (ココでは `10.89.2.1/32`) は
    Gateway アドレスとして使用されているので
    `10.89.2.2` から `10.89.2.255` までを指定する

あとは作ったコンフィグを `kubectl apply` するだけ。

## 動作確認

試しに `type=LoadBalancer` Service を作って、 EXTERNAL-IP が割り振られることを確認する。
以下のような適当な Service を作る。

``` bash
kubectl apply -f - <<-__EOF__
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: echo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: echo
  template:
    metadata:
      labels:
        app: echo
      annotations:
        sidecar.istio.io/inject: "false"
    spec:
      containers:
      - image: docker.io/library/nginx
        name: nginx
        ports:
        - name: http
          containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: echo
spec:
  ports:
    - name: http
      port: 80
      targetPort: http
      protocol: TCP
  selector:
    app: echo
  type: LoadBalancer
__EOF__
```

ちゃんと Service に EXTERNAL-IP が割り振られているし、
アドレス帯も kind のコンテナネットワークのものが使われている。

``` console
$ kubectl get svc echo
NAME   TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
echo   LoadBalancer   10.96.65.120   10.89.2.3     80:30934/TCP   12s
```

ということで、 kind コンテナネットワークに別のコンテナを作って疎通確認。

``` console
$ podman run --rm --network kind docker.io/library/busybox wget -qS 10.89.2.3
  HTTP/1.1 200 OK
  Server: nginx/1.27.4
    :
```

!!! tip

    いちいち、コンテナネットワークに別のコンテナを作ったりせず、
    ホスト側からアクセスしたい場合は、 kind 作成時に
    コンテナポートとホストポートのマッピングを行うコンフィグを指定する。
    以下は http/https をマッピングしたい場合の例。

    ``` yaml
    kind: Cluster
    apiVersion: kind.x-k8s.io/v1alpha4
    nodes:
    - role: control-plane
    kubeadmConfigPatches:
    - |
        kind: InitConfiguration
        nodeRegistration:
        kubeletExtraArgs:
          node-labels: "ingress-ready=true"
    extraPortMappings:
    - containerPort: 80
        hostPort: 80
        protocol: TCP
    - containerPort: 443
        hostPort: 443
        protocol: TCP
    ```
