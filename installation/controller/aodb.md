# Telemetry Alarming (aodh)

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
| id                  | d605f85c5a7b4722903afcc3657a2293 |
| name                | aodh                             |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```

プロジェクト service でユーザ aodh にロール admin 権限を追加する。

```sh
openstack role add --project service --user aodh admin
```

## サービスの作成

サービス alarming を作成する。

```sh
openstack service create --name aodh --description "Telemetry Alarming" alarming
```

```
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Telemetry Alarming               |
| enabled     | True                             |
| id          | 6d563adee2144fe09d2f6e4f62a9b0aa |
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
| id           | e30bc33622ee4b47a654bd6ab1316a4a |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 6d563adee2144fe09d2f6e4f62a9b0aa |
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
| id           | 47ff02939d9a494eb56deb63af33b11f |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 6d563adee2144fe09d2f6e4f62a9b0aa |
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
| id           | 3800a723d75f4bb68bda11b63aa72c82 |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 6d563adee2144fe09d2f6e4f62a9b0aa |
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
*/usr/bin/aodh-api* の 8000 -> 8042 に変更する。
```

```sh
systemctl enable --now openstack-aodh-api
systemctl enable --now openstack-aodh-evaluator
systemctl enable --now openstack-aodh-notifier
systemctl enable --now openstack-aodh-listener
```
