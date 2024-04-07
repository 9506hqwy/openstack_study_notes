# 事前準備

## ネットワーク

ホスト名を設定する。

```sh
hostnamectl set-hostname controller.home.local
```

ネットワークアダプタを設定する。

```sh
nmcli connection modify ens192 \
    ipv6.method ignore \
    ipv4.method manual \
    ipv4.dns 10.0.0.1 \
    ipv4.dns-search home.local \
    ipv4.addresses 10.0.0.11/24 \
    ipv4.gateway 10.0.0.1 \
    connection.autoconnect yes
nmcli connection up ens192

nmcli connection modify ens224 \
    ipv6.method ignore \
    ipv4.method auto \
    connection.autoconnect yes
nmcli connection up ens224
```

ファイアウォールを設定する。

```sh
firewall-cmd --zone=internal --change-interface=ens192 --permanent
firewall-cmd --reload
```

## NTP クライアント

[](../appendix/time_sync.md) を参照する。

##  OpenStack リポジトリ有効化

[](../appendix/repository_enable.md) を参照する。

##  OpenStack クライアントのインストール

[](../appendix/os_client_install.md) を参照する。

##   SELinux の設定

[](../appendix/os_selinux.md) を参照する。
