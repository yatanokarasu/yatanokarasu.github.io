---
layout:         post
title:          "The beginnings of docker"
keywords:       [Docker, Virtualization, Docker Machine]
categories:     [Virtualization]
tags:           [Docker]
---

I have started to study [Docker](https://www.docker.com/) that is popular with the public.
However, Docker runs on Linux distribution like [CoreOS](https://coreos.com/) or [CentOS](https://www.centos.org/),
so we need to client and enviroments to run Docker if use Windows OS.
Now, we can use [Docker Machine](https://www.docker.com/products/docker-machine) to run docker on Windows OS.

巷で流行りの[Docker](https://www.docker.com)を勉強し始めました。
しかし、Dockerは[CoreOS](https://coreos.com/)や[CentOS](https://www.centos.org/)等のLinuxディストリビューショ上で動作するので、
Windows OSを使用する場合はDockerを動作させるクライアントと環境が必要です。
そこで、Windows OS上でDockerを動作させるために[Docker Machine](https://www.docker.com/products/docker-machine)を使うことができます。

<!-- more -->

# Environments

HostOS
:   Windows 10 Home (64bit)

VirtualBox
:   5.0.16 r105871をインストール済み


# Download Docker Machine

詳細は[公式ドキュメント](https://docs.docker.com/machine/install-machine/)を確認します。
ここではお試しとして、[ココ](https://github.com/docker/machine/releases/)から`docker-machine-Windows-x86_64.exe`だけダウンロードします。

ダウンロードしたら下記コマンドを実行して、正常に動作するか確認します。お試しなので、実行ファイルのリネームとか`PATH`設定はスキップします。

{% highlight raw %}
> docker-machine-Windows-x86_64.exe ls
NAME   ACTIVE   DRIVER   STATE   URL   SWARM   DOCKER   ERRORS

>
{% endhighlight %}

下記コマンドを実行して、DockerホストをダウンロードおよびVirtualBoxに追加します。

{% highlight raw %}
> docker-machine-Windows-x86_64.exe create --driver=virtualbox test-dev
Creating CA: C:\Users\${USERNAME}\.docker\machine\certs\ca.pem
Creating client certificate: C:\Users\${USERNAME}\.docker\machine\certs\cert.pem
Running pre-create checks...
(test-dev) Image cache directory does not exist, creating it at C:\Users\${USERNAME}\.docker\machine\cache...
(test-dev) No default Boot2Docker ISO found locally, downloading the latest release...
(test-dev) Latest release for github.com/boot2docker/boot2docker is v1.10.2
(test-dev) Downloading C:\Users\${USERNAME}\.docker\machine\cache\boot2docker.iso from https://github.com/boot2docker/boot2docker/releases/download/v1.10.2/boot2docker.iso...
(test-dev) 0%....10%....20%....30%....40%....50%....60%....70%....80%....90%....100%
Creating machine...
(test-dev) Copying C:\Users\${USERNAME}\.docker\machine\cache\boot2docker.iso to C:\Users\${USERNAME}\.docker\machine\machines\test-dev\boot2docker.iso...
(test-dev) Creating VirtualBox VM...
(test-dev) Creating SSH key...
(test-dev) Starting the VM...
(test-dev) Check network to re-create if needed...
(test-dev) Windows might ask for the permission to create a network adapter. Sometimes, such confirmation window is minimized in the taskbar.
(test-dev) Found a new host-only adapter: "VirtualBox Host-Only Ethernet Adapter #6"
(test-dev) Windows might ask for the permission to configure a network adapter. Sometimes, such confirmation window is minimized in the taskbar.
(test-dev) Windows might ask for the permission to configure a dhcp server. Sometimes, such confirmation window is minimized in the taskbar.
(test-dev) Waiting for an IP...
Waiting for machine to be running, this may take a few minutes...
Detecting operating system of created instance...
Waiting for SSH to be available...
Detecting the provisioner...
Provisioning with boot2docker...
Copying certs to the local machine directory...
Copying certs to the remote machine...
Setting Docker configuration on the remote daemon...
Checking connection to Docker...
Docker is up and running!
To see how to connect your Docker Client to the Docker Engine running on this virtual machine, run: docker-machine-Windows-x86_64.exe env test-dev

>
{% endhighlight %}

DockerホストにSSH接続してみます。

{% highlight raw %}
> docker-machine-Windows-x86_64.exe ssh test-dev
                        ##         .
                  ## ## ##        ==
               ## ## ## ## ##    ===
           /"""""""""""""""""\___/ ===
      ~~~ {~~ ~~~~ ~~~ ~~~~ ~~~ ~ /  ===- ~~~
           \______ o           __/
             \    \         __/
              \____\_______/
 _                 _   ____     _            _
| |__   ___   ___ | |_|___ \ __| | ___   ___| | _____ _ __
| '_ \ / _ \ / _ \| __| __) / _` |/ _ \ / __| |/ / _ \ '__|
| |_) | (_) | (_) | |_ / __/ (_| | (_) | (__|   <  __/ |
|_.__/ \___/ \___/ \__|_____\__,_|\___/ \___|_|\_\___|_|
Boot2Docker version 1.10.2, build master : 611be10 - Mon Feb 22 22:47:06 UTC 2016
Docker version 1.10.2, build c3959b1
docker@test-dev:~$
{% endhighlight %}

今日はここまで。
