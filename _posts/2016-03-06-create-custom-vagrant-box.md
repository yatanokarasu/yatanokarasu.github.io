---
layout:         post
title:          "How to create custom vagrant box"
description:    "Web上にはvagrantのboxファイルがいくつも公開されているが、自分用に設定ファイルなどをカスタマイズしたboxファイルを作成することもできる。"
keyword:        Vagrant
categories:     [Virtualization]
tags:           [Vagrant, VirtualBox]
---

You can use vagrant box file published by [Vagrantbox.es](http://www.vagrantbox.es) or created by yourself.

- - -

Vagrantで使用するboxファイルは[Vagrantbox.es](http://www.vagrantbox.es)などで公開されているものを使用できますが、
自分用に設定ファイルをカスタマイズしたboxファイルを作成することができます。

<!-- more -->

# Environments

Environments to create custom box file are as follows:

Host OS
:   Windows 10 Home Edition

Guest OS
:   [CentOS 6.7 x86_64 minimal](http://ftp.jaist.ac.jp/pub/Linux/CentOS/6.7/isos/x86_64/)

VirtualBox
:   5.0.16 r105871

Vagrant
:   1.8.1


# Procedures

## Prepare virtual image

Select [New] menu on toolbar.  
[新規]を選択して仮想マシンを作成します。
{% include modal-image
    id="vm-create"
    src="/images/posts/20160306/virtual-box-new-creation.png"
    class="half center"
%}

{: #ref-vm-name}
Set virtual machine name. This name will be used later when Vagrant box file.  
VM名を設定します。この名前は、Vagrant boxファイルを作成する際に使用します。
{% include modal-image
    id="vm-name"
    src="/images/posts/20160306/virtual-box-vm-name.png"
    class="half center"
%}}

Assign virtual machine memory. RAM size is enough 512MB.  
仮想マシンのメモリを割り当てます。512MBで良いです。
{% include modal-image
    id="vm-memory"
    src="/images/posts/20160306/virtual-box-vm-memory.png"
    class="half center"
%}

Create virtual HDD.  
仮想HDDを作成します。
{% include modal-image
    id="create-vm-hdd"
    src="/images/posts/20160306/virtual-box-vm-hdd-creation.png"
    class="half center"
%}

Select filetype of virtual HDD. You should select VDI (VirtualBox Disk Image).  
仮想HDDのファイルタイプを選択します。VDIを選択すると良いです。
{% include modal-image
    id="create-vm-filetype"
    src="/images/posts/20160306/virtual-box-vm-filetype.png"
    class="half center"
%}

Set the path in which virtual machine is stored.  
仮想マシンを格納するパスを設定します。
{% include modal-image
    id="create-vm-hdd"
    src="/images/posts/20160306/virtual-box-vm-path.png"
    class="half center"
%}


## Configure virtual machine

You have to modify several settings to use this vm as vagrant box.  
Vagrant boxファイルとして使用するために、いくつかの設定を変更します。


### Disable Audio　and USB.

Disable audio.  
オーディオを無効化します。
{% include modal-image
    id="create-vm-disable-audio"
    src="/images/posts/20160306/virtual-box-disable-audio.png"
    class="half center"
%}

Similarly, disable USB too.  
同様に、USBも無効化します。
{% include modal-image
    id="create-vm-disable-usb"
    src="/images/posts/20160306/virtual-box-disable-usb.png"
    class="half center"
%}


### Port forwarding configuration

Enable port forward for SSH.  
SSHのためのポートフォワード設定を行います。
{% include modal-image
    id="port-forwarding"
    src="/images/posts/20160306/virtual-box-set-port-forwarding.png"
    class="half center"
%}

You have to add rule from `127.0.0.1:2222` to `:22` (Guest IP is empty).  
`127.0.0.1:2222`から`:22`(ゲストIPは空)へのルールを追加します。
{% include modal-image
    id="add-port-forward-for-ssh"
    src="/images/posts/20160306/virtual-box-add-port-forward-for-ssh.png"
    class="half center"
%}


## Configure OS/Kernel parameters

Login to virtual machine, and do as follows by `root` user.  
仮想マシンにログインして、以下を`root`ユーザで行います。


### Configure NIC

Enable NIC on boot.  
ブート時のNICを有効にします。

~~~
# sed -i 's:^ONBOOT=.*!ONBOOT=yes!' /etc/sysconfig/network-scripts/ifcfg-eth0
# service network restart
~~~

Delete UUID assigned NIC.  
NICに割り当てられているUUIDを削除します。

~~~
# ln -snf /dev/null /etc/udev/rules.d/70-persistent-net.rules
# sed -i -e '/^HWADDR=.*$/d' -e '/^UUID=.*$/d' /etc/sysconfig/network-scripts/ifcfg-eth0
~~~


### Install VirtualBox Guest Additions

Update installed packages, and then reboot.  
インストール済みのパッケージたちを更新して再起動します。

~~~
# yum -y update
# shutdown -r now
~~~


After reboot, insert VirtualBox Guest Additions's image CD to virtual media drive, and then update packages.  
再起動後、VirtualBox Guest AdditionsのイメージCDをマウントし、アップデートを行います。

{% include modal-image
    id="additional-disk"
    src="/images/posts/20160306/virtual-box-insert-additional-disk.png"
    class="half center"
%}

~~~
# yum -y install kernel-devel gcc perl
# mount /dev/sr0 /media
# sh /media/VBoxLinuxAdditions.run
# umount /media
~~~


### Customize as you like

You can install other packages and add additional configuratino as you like.
I did as follows:  
お好みで他の設定追加とパッケージインストールをします。私は以下のことを行いました。


#### Disable unnecessary services on boot

~~~
# chkconfig --del postfix
# chkconfig --del ip6tables
# chkconfig --del auditd
# chkconfig --del mdmonitor
# chkconfig --del rdisc
# chkconfig --del restorecond
# chkconfig --del saslauthd
~~~


#### Install useful packages

~~~
# yum -y install man nslookup rsync tree vim 
~~~


#### Vim configuration

<span class="balloon">/etc/skel/.exrc, /root/.exrc</span>

~~~ vim
"---- window settings ----
set title

set list
set number

"---- status line ----
set laststatus=2
set showmode
set showcmd
set ruler

"---- encoding ----
set termencoding=utf-8
set encoding=utf-8
set fileformats=unix,dos,mac
set fileencoding=utf-8
set fileencodings=utf-8,shift-jis

"---- color settings ----
syntax on

"---- others ----
set autoindent
set expandtab
set shiftwidth=4
set tabstop=4

set hidden
set showmatch

set backspace=indent,eol,start
set cursorline
~~~


#### bash configuration

<span class="balloon">/etc/skel/.bashrc, /etc/root/.bashrc</span>

~~~ shell
alias ls="ls --color=auto -F"
alias vi="vim"
alias cp="cp -i"
alias mv="mv -i"
alias rm="rm -i"

alias grep="grep --color=auto"
alias egrep="egrep --color=auto"

export EDITOR=vim

# bash history
export HISTCONTROL=ignoreboth
export HISTIGNORE="fg*:bg*:history*:cd*:ls*"
export HISTSIZE=10000
export HISTTIMEFORMAT="%Y/%m/%d %T: "
~~~


### Configure SSH

Disable `UseDNS`.  
`UseDNS`を無効化します。

~~~
sed -i "s:^#UseDNS.*$:UseDNS no:" /etc/ssh/sshd_config
~~~


### Create vagrant user

Create `vagrant` user with password `vagrant` .  
ユーザ名/パスワードともに`vagrant`でユーザを作成します。

~~~
# useradd vagrant -G wheel
# passwd vagrant
# echo "vagrant ALL=(ALL) NOPASSWD: ALL" >>/etc/sudoers
# sed -i "s:^.*requiretty:#Default requiretty:" /etc/sudoers
~~~


### Setup SSH public key

Setup SSH public key of `vagrant` user.   
`vagrant`ユーザのSSH公開鍵を設定します。

~~~
# su - vagrant
$ mkdir ~/.ssh
$ chmod 0700 ~/.ssh
$ curl -L -o ~/.ssh/authorized_keys https://raw.githubusercontent.com/mitchellh/vagrant/master/keys/vagrant.pub
$ chmod 0600 ~/.ssh/authorized_keys
~~~


That's end of configuration of virtual machine.  
以上で仮想マシンの設定は終わりです。


- - -

## Create and add vagrant box file

In Host OS, creates vagrant box file with the following command.
[VM name](#ref-vm-name) will be specified `--base` option.
Without `--output` option, the name of the box file is `package.box`.  
ホストOS上で、以下のコマンドでvagrant boxファイルを作成します。[仮想マシン名](#ref-vm-name)を`--base`オプションに指定します。
`--output`オプション無しでは、boxファイルの名前は`package.box`です。

~~~
> vagrant package --base CentOS6.7_x64
~~~

After that, add the box file to vagrant with name as you like.
続いて、作成したboxファイルを名前を指定してVagrantに追加します。

~~~
> vagrant box add --name centos6.7_x64 package.box
~~~

