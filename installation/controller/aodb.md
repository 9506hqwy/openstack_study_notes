# OpenStack Telemetry Alarming Service (aodh)

## データベースの作成

データベースに root でログインする。

```sh
mysql -u root -p59c6b803ff3fd0e68dcd
```

データベース aodh を作成する。

```sh
CREATE DATABASE aodh;
GRANT ALL PRIVILEGES ON aodh.* TO 'aodh'@'localhost' IDENTIFIED BY '237d687f60cd82bd2472';
GRANT ALL PRIVILEGES ON aodh.* TO 'aodh'@'%' IDENTIFIED BY '237d687f60cd82bd2472';
```

## ユーザの作成

ユーザ aodh を作成する。

```sh
openstack user create --domain default --password 7df3446ca4ec04bb40be aodh
```

```
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | a61918fc49164c6fa6b80714c68a9412 |
| name                | aodh                             |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```

プロジェクト service にロール admin 権限でユーザ aodh を追加する。

```sh
openstack role add --project service --user aodh admin
```

## サービスの作成

サービス alarming を作成する。

```sh
openstack service create --name aodh --description "OpenStack Telemetry Alarming" alarming
```

```
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Telemetry Alarming     |
| enabled     | True                             |
| id          | 7a617f6df901433a82da9b36e3eaccce |
| name        | aodh                             |
| type        | alarming                         |
+-------------+----------------------------------+
```

## エンドポイントの作成

API エンドポイントを作成する。

```sh
openstack endpoint create --region RegionOne alarming public http://controller:8042
```

```
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | ae789ff352dc4ebd9e161fa84c631010 |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 7a617f6df901433a82da9b36e3eaccce |
| service_name | aodh                             |
| service_type | alarming                         |
| url          | http://controller:8042           |
+--------------+----------------------------------+
```

```sh
openstack endpoint create --region RegionOne alarming internal http://controller:8042
```

```
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 728e133996e840978296f7e33fdf49b8 |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 7a617f6df901433a82da9b36e3eaccce |
| service_name | aodh                             |
| service_type | alarming                         |
| url          | http://controller:8042           |
+--------------+----------------------------------+
```

```sh
openstack endpoint create --region RegionOne alarming admin http://controller:8042
```

```
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 27017f929fe5465a95f4969f4ee4193f |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 7a617f6df901433a82da9b36e3eaccce |
| service_name | aodh                             |
| service_type | alarming                         |
| url          | http://controller:8042           |
+--------------+----------------------------------+
```

ファイアウォールを開ける。

```sh
firewall-cmd --permanent --zone=internal --add-port=8042/tcp
firewall-cmd --reload
```

## インストール

aodh をインストールする。

```sh
dnf install -y \
    openstack-aodh-api \
    openstack-aodh-evaluator \
    openstack-aodh-notifier \
    openstack-aodh-listener \
    openstack-aodh-expirer \
    python3-aodhclient
```

## aodh の設定

設定ファイルを更新する。

```sh
sed \
    -e '/^\[DEFAULT]/,/^\[/ {
      /^transport_url =/d
      /^#transport_url =/atransport_url = rabbit://openstack:3c17215fe69ba1dad320@controller:5672/
      /^auth_strategy =/d
      /^#auth_strategy =/aauth_strategy = keystone
    }' \
    -e '/^\[database]/,/^\[/ {
      /^connection =/d
      /^#connection =/aconnection = mysql+pymysql://aodh:237d687f60cd82bd2472@controller/aodh
    }' \
    -e '/^\[keystone_authtoken]/,/^\[/ {
      /^www_authenticate_uri =/d
      /^#www_authenticate_uri =/awww_authenticate_uri = http://controller:5000
      /^memcached_servers =/d
      /^#memcached_servers =/amemcached_servers = controller:11211
      /^auth_type =/d
      /^#auth_type =/aauth_type = password
      /^auth_url =/d
      /^\[keystone_authtoken]/aauth_url = http://controller:5000
      /^project_domain_name =/d
      /^\[keystone_authtoken]/aproject_domain_name = Default
      /^project_name =/d
      /^\[keystone_authtoken]/aproject_name = service
      /^user_domain_name =/d
      /^\[keystone_authtoken]/auser_domain_name = Default
      /^username =/d
      /^\[keystone_authtoken]/ausername = aodh
      /^password =/d
      /^\[keystone_authtoken]/apassword = 7df3446ca4ec04bb40be
    }' \
    -e '/^\[service_credentials]/,/^\[/ {
      /^auth_type =/d
      /^#auth_type =/aauth_type = password
      /^auth_url =/d
      /^#auth_url =/aauth_url = http://controller:5000
      /^project_domain_name =/d
      /^#project_domain_name =/aproject_domain_name = Default
      /^project_name =/d
      /^#project_name =/aproject_name = service
      /^user_domain_name =/d
      /^#user_domain_name =/auser_domain_name = Default
      /^username =/d
      /^#username =/ausername = aodh
      /^password =/d
      /^#password =/apassword = 7df3446ca4ec04bb40be
      /^region_name =/d
      /^#region_name =/aregion_name = RegionOne
      /^interface =/d
      /^#interface =/ainterface = internalURL
    }' \
    -i /etc/aodh/aodh.conf
```

## データベースの構築

データベースを構築する。

```sh
aodh-dbsync
```

## 起動

マシン起動時の自動起動設定とサービスを起動する。

```{note}
*/usr/bin/aodh-api*　の 8000 -> 8042 に変更する。
```

```sh
systemctl enable --now openstack-aodh-api
systemctl enable --now openstack-aodh-evaluator
systemctl enable --now openstack-aodh-notifier
systemctl enable --now openstack-aodh-listener
```
