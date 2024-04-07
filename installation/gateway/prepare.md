# 事前準備

## ネットワーク

ホスト名を設定する。

```sh
hostnamectl set-hostname gateway.home.local
```

ネットワークアダプタを設定する。

```sh
nmcli connection modify ens192 \
    ipv6.method ignore \
    ipv4.method manual \
    ipv4.dns-search home.local \
    ipv4.addresses 10.0.0.1/24 \
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
firewall-cmd --zone=public --add-masquerade --permanent
firewall-cmd --permanent --direct --add-rule ipv4 nat POSTROUTING 0 -o ens224 -j MASQUERADE
firewall-cmd --permanent --direct --add-rule ipv4 filter FORWARD 0 -i ens192 -o ens224 -j ACCEPT
firewall-cmd --permanent --direct --add-rule ipv4 filter FORWARD 0 -i ens224 -o ens192 -m state --state RELATED,ESTABLISHED -j ACCEPT
firewall-cmd --reload
```
