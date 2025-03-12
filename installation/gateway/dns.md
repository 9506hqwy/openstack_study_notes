# DNS サーバ

## インストール

dnsmasq をインストールする。

```sh
dnf install -y dnsmasq
```

## 設定

内部ネットワーク向けに DNS をサービスを有効化する。

```sh
sed \
    -e 's/^#\(domain-needed\)/\1/ #ドメン名がない場合は上位サーバに問い合わせない。' \
    \
    -e 's/^#\(bogus-priv\)/\1/ #プライベート IP アドレスは上位サーバに逆引きを要求しない。' \
    \
    -e 's/^#\(interface=\).*/\1eth2/1 #機能を有効にするインターフェースを指定する。' \
    \
    -e 's/^#\(expand-hosts\)/\1/ #/etc/hosts のホスト名にドメイン名を自動的に付与する。' \
    -e "s/^#\(domain=\)thekelleys.org.uk/\1$(hostname -d)/1" \
    \
    -i /etc/dnsmasq.conf
```

エントリを */etc/hosts* に追加する。

```sh
cat >> /etc/hosts <<EOF
10.0.0.254 gateway
10.0.0.11  controller
10.0.0.31  compute
10.0.0.51  baremetal
EOF
```

ファイアウォールを開ける。

```sh
firewall-cmd --permanent --zone=internal --add-service=dns
firewall-cmd --reload
```

## systemd ユニットファイルの設定

マシンの起動時に下記のログが出力されるためネットワークが起動した後で dnsmasq を起動する。

```text
dnsmasq: unknown interface eth2
```

```sh
sed \
    -e 's/^\(After\)/#\1/' \
    -e '/^#After/aAfter=network-online.target' \
    -e '/^#After/aRequires=network-online.target' \
    -i /usr/lib/systemd/system/dnsmasq.service
```

## 起動

マシン起動時の自動起動設定とサービスを起動する。

```sh
systemctl enable --now dnsmasq
```
