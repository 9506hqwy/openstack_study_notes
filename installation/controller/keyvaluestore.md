# 分散Key/Valueストア

## インストール

etcd をインストールする。

```sh
dnf install -y etcd
```

## 設定

外部から接続できるように設定する。

```sh
sed \
    -e '/^#ETCD_LISTEN_PEER_URLS=/aETCD_LISTEN_PEER_URLS="http://10.0.0.11:2380"' \
    -e 's|^\(ETCD_LISTEN_CLIENT_URLS=\).*|\1"http://10.0.0.11:2379"|' \
    -e 's/^\(ETCD_NAME=\).*/\1"controller"/' \
    -e '/^#ETCD_INITIAL_ADVERTISE_PEER_URLS=/aETCD_INITIAL_ADVERTISE_PEER_URLS="http://10.0.0.11:2380"' \
    -e 's|^\(ETCD_ADVERTISE_CLIENT_URLS=\).*|\1"http://10.0.0.11:2379"|' \
    -e '/^#ETCD_INITIAL_CLUSTER=/aETCD_INITIAL_CLUSTER="controller=http://10.0.0.11:2380"' \
    -e '/^#ETCD_INITIAL_CLUSTER_TOKEN=/aETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster-01"' \
    -e '/^#ETCD_INITIAL_CLUSTER_STATE=/aETCD_INITIAL_CLUSTER_STATE="new"' \
    -i /etc/etcd/etcd.conf
```

ファイアウォールを開ける。

```sh
firewall-cmd --permanent --zone=internal --add-service=etcd-server
firewall-cmd --reload
```

## 起動

マシン起動時の自動起動設定とサービスを起動する。

```sh
systemctl enable --now etcd
```
