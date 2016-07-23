---
layout:         post
title:          Setup Wi-Fi on CentOS7 minimal
keywords:       [CentOS7]
categories:     [Configurations]
tags:           [CentOS7]
---

CentOS7 installed Intel NUC can not connect Wi-Fi due to has no wifi driver.

~~~
# dmesg | grep wifi
  :
[    3.496067] iwlwifi 0000:02:00.0: request for firmware file 'iwlwifi-7265D-10.ucode' failed.
[    3.496067] iwlwifi 0000:02:00.0: request for firmware file 'iwlwifi-7265D-11.ucode' failed.
[    3.496067] iwlwifi 0000:02:00.0: request for firmware file 'iwlwifi-7265D-12.ucode' failed.
[    3.496154] iwlwifi 0000:02:00.0: no suitable firmware found!
  :

# nmcli d
DEVICE   TYPE      STATE        CONNECTION
wlp2s0   wifi      unmanaged    --
enp0s25  ethernet  unavailable  --
lo       loopback  unmanaged    --
~~~

The following is how to setup Wi-Fi configuration on CentOS 7.

<!-- more -->

## Install Wi-Fi driver

1.  Download Wi-Fi driver
    
    Download the following drivers from [Linux Wireless](https://wireless.wiki.kernel.org/en/users/drivers/iwlwifi#firmware)
    
    *   [Intel® Wireless 7265 3.17+](https://wireless.wiki.kernel.org/_media/en/users/drivers/iwlwifi-7265-ucode-23.15.10.0.tgz)
    
    *   [Intel® Wireless 7265 3.19+](https://wireless.wiki.kernel.org/_media/en/users/drivers/iwlwifi-7265-ucode-25.17.12.0.tgz)

2.  Extract tarball, copy and reboot to load drivers

    ~~~
    # tar zxf iwlwifi-7265-ucode-23.15.10.0.tgz
    # tar zxf iwlwifi-7265-ucode-25.17.12.0.tgz
    # cp iwlwifi-7265-ucode-23.15.10.0/iwlwifi-7265D-10.ucode /lib/firmware/
    # cp iwlwifi-7265-ucode-25.17.12.0/iwlwifi-7265D-12.ucode /lib/firmware/
    # reboot
    ~~~

3.  Check WLAN

    ~~~
    # nmcli d
    DEVICE   TYPE      STATE        CONNECTION
    wlp2s0   wifi      disconnected --
    enp0s25  ethernet  unavailable  --
    lo       loopback  unmanaged    --
    ~~~

## Configure Wi-Fi

1.  Add connection profile
    
    ~~~
    # nmcli c add type wifi ifname wlp2s0 con-name [profile-name] ssid <ESSID>
    ~~~

2.  Edit connection
    We can use `nmtui` graphical console tools.

    ~~~
    # nmcli c edit [profile-name]
    nmcli> set wifi-sec.key-mgmt wpa-psk
    nmcli> set wifi-sec.psk <Wi-Fi passphrase>
    ~~~

3.  Check whether connection is established or not

    ~~~
    # nmcli c up [profile-name]
    # nmcli d
    DEVICE   TYPE      STATE        CONNECTION
    wlp2s0   wifi      connected    [profile-name]
    enp0s25  ethernet  unavailable  --
    lo       loopback  unmanaged    --
    ~~~


If the status is disconnected until above, edit `/etc/wpa_supplicant/wpa_supplicant.conf`.
