# NTP サーバ

## インストール

chrony をインストールする。

```sh
dnf install -y chrony
```

## 設定

内部ネットワーク向けに NTP をサービスを有効化する。
外部 NTP サーバは ntp.nict.jp を参照する。

```sh
sed \
    -e 's/^\(pool.*\)/#\1/' \
    \
    -e '/^#pool/aserver ntp.nict.jp iburst' \
    \
    -e '/^#allow/aallow 10.0.0.0/24' \
    \
    -i /etc/chrony.conf
```

ファイアウォールを開ける。

```sh
firewall-cmd --permanent --zone=internal --add-service=ntp
firewall-cmd --reload
```

## 起動

マシン起動時の自動起動設定とサービスを起動する。

```sh
systemctl enable --now chronyd
```

設定を確認する。

```sh
chronyc sources
```
