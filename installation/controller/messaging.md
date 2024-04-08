# メッセージング

## インストール

RabbitMQ をインストールする。

```sh
dnf install -y rabbitmq-server
```

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

ユーザ openstack のすべての権限を付与する。

```sh
rabbitmqctl set_permissions openstack ".*" ".*" ".*"
```
