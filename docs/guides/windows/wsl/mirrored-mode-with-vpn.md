---
tags:
  - Windows
  - WSL
  - VPN
---

# Mirror mode for accessing external networks while enabling VPN

!!! warning

    以下環境で検証済

    * **Windows**: Windows 11 Pro 23H2
    * **WSL2**: 2.4.11.0
    * **Linux on WSL**: Ubuntu 24.04.1 LTS

Global Protect など、 WSL ホスト (つまり Windows 上) で VPN が有効な状態だと、
WSL からインターネットのホストに接続できなくなってしまう。
手元に VPN 接続状態を検証できる環境にないが、 DNS 名前解決ができなくなることが原因の模様。

VPN 接続状態でも以下のいずれかの方法でインターネットへの接続が可能になるが、
ここでは WSL2 v2.0.5 以降で使用可能になった Mirrored モードを利用する手順を示す。
    比較的多く情報があるし、導入も難しくない。

1.  Mirrored モードを有効にする

1.  [wsl-vpnkit](https://github.com/sakai135/wsl-vpnkit) を使用する

!!! quote

    wsl-vpnkit が気になる方は、公式ページや周辺情報を探してみて確認して試してもらいたい。

## Mirrored モードを有効にする

`%USERPROFILE%\.wslconfig` を開き、以下の内容を追記する。

``` ini
[wsl2]
networkingMode = mirrored
```

設定の追記が終わったら、 PowerShell で以下のコマンドを実行して WSL2 をシャットダウンする。

``` ps1
wsl --shutdown
```

もう一度 WSL2 (の Linux マシン) を起動しなおせば、
VPN 有効状態でもDNS 名前解決ができてインターネット接続ができるはず。
