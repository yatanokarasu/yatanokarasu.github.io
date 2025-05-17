---
tags:
  - CGroup
  - Rootless Container
  - Windows
  - WSL
---

# Disable CGroup v1 on WSL

!!! info

    本記事は、以下の内容で確認。

    * **OS**: Windows 11 Pro 23H2
    * **WSL**: Ubuntu 24.04.01 LTS
    * **Podman**: v4.9.3 (インストール手順は[コチラ](./../windows/wsl/install-podman.md))

[公式ドキュメント](https://kind.sigs.k8s.io/docs/user/rootless/)にも記載されている通り、
Podman のような Rootless provider を実行する場合は CGroup v2 が必須となるが、
WSL はデフォルトが CGroup v1 となっているため、 CGroup v1 を無効にする必要がある。

## CGroup v1 無効化

WSL で CGroup v2 を有効にするには、
Windows 側で `%USERPROFILE\.wslconfig` に以下の内容を追加する。

``` ini title="%USERPROFILE%\.wslconfig"
[wsl2]
kernelCommandLine = cgroup_no_v1=all
```

`.wslconfig` への追記を終えたら、一旦 WSL をシャットダウンする。

``` powershell
wsl --shutdown
```

再度 WSL を起動しなおした後、以下のコマンドを実行して `cgroup2fs` と出力されることを確認する。

``` console
$ stat -fc %T /sys/fs/cgroup/
cgroup2fs
```
