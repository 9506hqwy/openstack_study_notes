# キャッシュシステム

## インストール

Memcached と Python のライブラリをインストールする。

```sh
dnf install -y memcached python3-memcached
```

## 設定

外部から接続できるように設定する。

```sh
sed -e 's/^\(OPTIONS=\).*/\1"-l 127.0.0.1,controller"/' -i /etc/sysconfig/memcached
```

ファイアウォールを開ける。

```sh
firewall-cmd --permanent --zone=internal --add-service=memcache
firewall-cmd --reload
```

## 起動

マシン起動時の自動起動設定とサービスを起動する。

```sh
systemctl enable --now memcached
```
