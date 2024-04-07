# 時刻同期

## インストール

chrony をインストールする。

```sh
dnf install -y chrony
```

## 設定

NTP サーバは gateway を参照する。

```sh
sed \
    -e 's/^\(pool.*\)/#\1/' \
    \
    -e '/^#pool/aserver gateway iburst' \
    \
    -i /etc/chrony.conf
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
