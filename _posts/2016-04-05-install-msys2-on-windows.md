---
layout:         post
description:    "Install Msys2 to use UNIX like commands on Windows OS."
keywords:
  - Msys2
categories:     [Environments]
tags:           [Msys2, Jekyll, fabric, Python, pip]
---

WindowsでUNIX系コマンドを扱うには[cygwin](https://www.cygwin.com/)が有名ですが、ここは[Msys2](https://sourceforge.net/projects/msys2/)を使っていきます。
Msys2はArch Linuxの`pacman`というパッケージ管理ツールが使用できるので、主要なコマンドをは`pacman`を使って簡単にインストールすることができて便利です。

ここでは、このGithub Pageのウェブページを作るための`Jekyll`と、`python`のパッケージ管理ツールの`pip`をインストールするところまでまとめてみます(`python2.7`を対象としています)。

<!-- more -->

64bit環境のWindowsを対象として説明していきます。なので、起動するmsysは`${MSYS_INSTALL_DIR}/mingw64_shell.bat`です。

# Initialization after install msys2

1.  パッケージデータベースの最新化
    
    `-S`オプションで、リポジトリと同期(更新)します。`-y`はデータベースのリフレッシュです。
    
    ~~~
    $ pacman -Sy
    ~~~

2.  インストール済パッケージの更新
    
    `-u`はインストール済みパッケージを更新します。`--needed`で必要なものだけ更新します。
    
    ~~~
    $ pacman -Su --needed
    ~~~

3.  よく使うコマンド類をインストール
    
    インストール可能なパッケージの検索は`-Ss`で検索できますので、事前に確認しましょう。
    
    ~~~
    $ pacman -S mingw-w64-x86_64-{cmake,diffutils,gcc,make,} {git,man-db,tar,vim,wget}
    ~~~


# Install Jekyll

1.  Rubyのインストール
    
    Jekyllを使用するために、`ruby`をインストールします。
    
    ~~~
    $ pacman -S mingw-w64-x86_64-ruby
    ~~~

2.  Jekyllのインストール
    
    `ruby`と一緒に`gem`もインストールされますので、`jekyll`をインストールします。必要であればJekyllプラグインも一緒にインストールしましょう。
    私は、`jekyll-paginate`と`jekyll-gist`をインストールしました。
    
    ~~~
    $ gem install jekyll jekyll-paginate jekyll-gist
    ~~~


# Install Python and pip

ここでは、Python2系のものをインストールしています。

1.  Pythonとpipのインストール
    
    ~~~
    $ pacman -S mingw-w64-x86_64-python2{,-pip}
    ~~~

2.  pipで試しに何かインストールしてみる。
    
    ここではデプロイツールのFabricをインストールしてみます。
    ほんとはAnsibleが良いのですが、WindowsだとAnsibleは使えないので。。。
    
    ~~~
    $ pip install fabric
    ~~~
    
## Fabricの動作テスト

ちょっとFabricのテストしてみます。下記内容の`fabfile.py`を作成します。

<span class="balloon">fabfile.py</span>

~~~ python
def greeting(first="Hello", last="World"):
    """
    prints your name (default: Hello World)
    """
    print("%s %s" % (first, last))
~~~

Fabricは`fab`コマンドで操作します。
`fabfile.py`があるディレクトリで`fab --list`を入力すると、`fabfile.py`で定義されている使用可能なタスク一覧が確認できます。
さらに、関数の最初で記述した文字列が、タスクの説明文として出力されていることも確認できます。

~~~
$ fab --list
Available commands:

    greeting  prints your name (default: Hello World)

~~~

`greeting`タスクを実行してみます。

~~~
$ fab greeting
Hello World

Done.

~~~

`greeting`関数のデフォルト引数に指定された内容が出力されました。続いて、引数も与えてみます。引数を与えるには以下のようにします。

~~~
$ fab greeting:first=Foo,last=Bar
Foo Bar

Done.

~~~
