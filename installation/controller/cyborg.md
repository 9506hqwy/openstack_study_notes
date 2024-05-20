# Accelerator (Cyborg)

## データベースの作成

データベースに root でログインする。

```sh
mysql -u root -p59c6b803ff3fd0e68dcd
```

データベース cyborg を作成する。

```sh
CREATE DATABASE cyborg;
GRANT ALL PRIVILEGES ON cyborg.* TO 'cyborg'@'localhost' IDENTIFIED BY 'da7c58277dc7e86d72a0';
GRANT ALL PRIVILEGES ON cyborg.* TO 'cyborg'@'%' IDENTIFIED BY 'da7c58277dc7e86d72a0';
```

## ユーザの作成

ユーザ cyborg を作成する。

```sh
openstack user create --domain default --password 2927e8e7587d5ccaf836 cyborg
```

```
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 24a53e2b6a9945778af0a335f179f9cf |
| name                | cyborg                           |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```

プロジェクト service でユーザ cyborg にロール admin 権限を追加する。

```sh
openstack role add --project service --user cyborg admin
```

## サービスの作成

サービス accelerator を作成する。

```sh
openstack service create --name cyborg --description "Acceleration" accelerator
```

```
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Acceleration                     |
| enabled     | True                             |
| id          | 40b4834a2f0842658d4b98d1789db484 |
| name        | cyborg                           |
| type        | accelerator                      |
+-------------+----------------------------------+
```

## エンドポイントの作成

API エンドポイントを作成する。

```sh
openstack endpoint create --region RegionOne accelerator public http://controller:6666/v2
```

```
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 4f76f93e9c1347d9bdbd4f9987347945 |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 40b4834a2f0842658d4b98d1789db484 |
| service_name | cyborg                           |
| service_type | accelerator                      |
| url          | http://controller:6666/v2        |
+--------------+----------------------------------+
```

```sh
openstack endpoint create --region RegionOne accelerator internal http://controller:6666/v2
```

```
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 43f2042dd0804c3080a0c9ad30eebb89 |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 40b4834a2f0842658d4b98d1789db484 |
| service_name | cyborg                           |
| service_type | accelerator                      |
| url          | http://controller:6666/v2        |
+--------------+----------------------------------+
```

```sh
openstack endpoint create --region RegionOne accelerator admin http://controller:6666/v2
```

```
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | e52c2c115fc34ccdae6c4a5c387fd3cb |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 40b4834a2f0842658d4b98d1789db484 |
| service_name | cyborg                           |
| service_type | accelerator                      |
| url          | http://controller:6666/v2        |
+--------------+----------------------------------+
```

ファイアウォールを開ける。

```sh
firewall-cmd --permanent --zone=internal --add-port=6666/tcp
firewall-cmd --reload
```

## インストール

```{tip}
Python 3.8 以降
```

### OS ユーザの作成

サービス cyborg 用のユーザを作成する。

```sh
useradd -r -M -s /sbin/nologin cyborg
```

### インストール

cyborg をインストールする。

```sh
dnf install -y gcc python3.12-devel

python3.12 -m venv /opt/cyborg
source /opt/cyborg/bin/activate

pip install --upgrade pip
pip install \
    openstack-cyborg \
    pymysql \
    python-memcached \
    amqp \
    python-openstackclient \
    python-cyborgclient

chgrp -R cyborg /opt/cyborg

mkdir -p /etc/cyborg
cp -r /opt/cyborg/etc/cyborg /etc
chgrp -R cyborg /etc/cyborg/*

mkdir -p /var/lib/cyborg
chown -R cyborg /var/lib/cyborg
chgrp -R cyborg /var/lib/cyborg
```

## cyborg の設定

設定ファイルを作成する。

```sh
cat > /etc/cyborg/cyborg.conf <<EOF
[DEFAULT]
transport_url = rabbit://openstack:3c17215fe69ba1dad320@controller:5672/
state_path = /var/lib/cyborg

[api]
host_ip = 0.0.0.0

[database]
connection = mysql+pymysql://cyborg:da7c58277dc7e86d72a0@controller/cyborg
max_pool_size = 20
max_overflow = 200

[service_catalog]
auth_type = password
auth_url = http://controller:5000
project_domain_name = Default
project_name = service
user_domain_name = Default
username = cyborg
password = 2927e8e7587d5ccaf836

[placement]
auth_type = password
auth_url = http://controller:5000
project_domain_name = Default
project_name = service
user_domain_name = Default
username = placement
password = b4b6936f87c338567d3c

[nova]
auth_type = password
auth_url = http://controller:5000
project_domain_name = Default
project_name = service
user_domain_name = Default
username = nova
password = 5c5cfbce3214db530456

[keystone_authtoken]
www_authenticate_uri = http://controller:5000
memcached_servers = localhost:11211
auth_type = password
auth_url = http://controller:5000
project_domain_name = Default
project_name = service
user_domain_name = Default
username = cyborg
password = 2927e8e7587d5ccaf836
EOF
```

```sh
chgrp cyborg /etc/cyborg/cyborg.conf
```

## データベースの構築

データベースを構築する。

```sh
cyborg-dbsync --config-file /etc/cyborg/cyborg.conf upgrade
```

## systemd ユニットファイルの作成

*/usr/lib/systemd/system/openstack-cyborg-api.service* を作成する。

```ini
[Unit]
Description=OpenStack Cyborg API service
After=network.target

[Service]
Type=simple
User=cyborg
ExecStart=/opt/cyborg/bin/cyborg-api --config-file=/etc/cyborg/cyborg.conf
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

*/usr/lib/systemd/system/openstack-cyborg-conductor.service* を作成する。

```ini
[Unit]
Description=OpenStack Cyborg Conductor service
After=network.target

[Service]
Type=simple
User=cyborg
ExecStart=/opt/cyborg/bin/cyborg-conductor --config-file=/etc/cyborg/cyborg.conf
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

## 起動

マシン起動時の自動起動設定とサービスを起動する。

```sh
systemctl enable --now openstack-cyborg-api
systemctl enable --now openstack-cyborg-conductor
```
