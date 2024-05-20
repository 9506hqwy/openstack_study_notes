# メッセージング

## インストール

RabbitMQ をインストールする。

```sh
dnf install -y rabbitmq-server
```

## 設定

ファイアウォールを開ける。

```sh
firewall-cmd --permanent --zone=internal --add-service=amqp
firewall-cmd --reload
```

## 起動

マシン起動時の自動起動設定とサービスを起動する。

```sh
systemctl enable --now rabbitmq-server
```

## 初期設定

ユーザ openstack を追加する。

```sh
rabbitmqctl add_user openstack 3c17215fe69ba1dad320
```

```
Adding user "openstack" ...
Done. Don't forget to grant the user permissions to some virtual hosts! See 'rabbitmqctl help set_permissions' to learn more.
```

ユーザ openstack にすべての権限を付与する。

```sh
rabbitmqctl set_permissions openstack ".*" ".*" ".*"
```

```
Setting permissions for user "openstack" in vhost "/" ...
```
