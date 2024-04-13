# OpenStack Compute (Nova)

## データベースの作成

データベースに root でログインする。

```sh
mysql -u root -p59c6b803ff3fd0e68dcd
```

データベース nova を作成する。

```sh
CREATE DATABASE nova_api;
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' IDENTIFIED BY '830e052c9ff3fab2efa4';
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' IDENTIFIED BY '830e052c9ff3fab2efa4';

CREATE DATABASE nova;
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' IDENTIFIED BY '830e052c9ff3fab2efa4';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' IDENTIFIED BY '830e052c9ff3fab2efa4';

CREATE DATABASE nova_cell0;
GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'localhost' IDENTIFIED BY '830e052c9ff3fab2efa4';
GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'%' IDENTIFIED BY '830e052c9ff3fab2efa4';
```

## ユーザの作成

ユーザ nova を作成する。

```sh
openstack user create --domain default --password 5c5cfbce3214db530456 nova
```

```
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | e48c9286086f469ebaf9c492bed365ce |
| name                | nova                             |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```

プロジェクト service にロール admin 権限でユーザ nova を追加する。

```sh
openstack role add --project service --user nova admin
```

## サービスの作成

nova サービスを作成する。

```sh
openstack service create --name nova --description "OpenStack Compute" compute
```

```
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Compute                |
| enabled     | True                             |
| id          | 76e6426236084edd8643d5f1c64c858b |
| name        | nova                             |
| type        | compute                          |
+-------------+----------------------------------+
```

## エンドポイントの作成

API エンドポイントを作成する。


```sh
openstack endpoint create --region RegionOne compute public http://controller:8774/v2.1
```

```
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 6f8cc58e54464cc29d35e12344e7abdb |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 76e6426236084edd8643d5f1c64c858b |
| service_name | nova                             |
| service_type | compute                          |
| url          | http://controller:8774/v2.1      |
+--------------+----------------------------------+
```

```sh
openstack endpoint create --region RegionOne compute internal http://controller:8774/v2.1
```

```
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 17af5e954037419d92f255ddbc8dbb72 |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 76e6426236084edd8643d5f1c64c858b |
| service_name | nova                             |
| service_type | compute                          |
| url          | http://controller:8774/v2.1      |
+--------------+----------------------------------+
```

```sh
openstack endpoint create --region RegionOne compute admin http://controller:8774/v2.1
```

```
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 96b7a84dab9c4e66819ea0ffae800b0d |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 76e6426236084edd8643d5f1c64c858b |
| service_name | nova                             |
| service_type | compute                          |
| url          | http://controller:8774/v2.1      |
+--------------+----------------------------------+
```

ファイアウォールを開ける。

```sh
firewall-cmd --permanent --zone=internal --add-port=6080/tcp
firewall-cmd --permanent --zone=internal --add-port=8774/tcp
firewall-cmd --reload
```

## インストール

nova をインストールする。

```sh
dnf install -y \
    openstack-nova-api \
    openstack-nova-conductor \
    openstack-nova-novncproxy \
    openstack-nova-scheduler
```

## Nova の設定

設定ファイルを更新する。

```sh
sed \
    -e '/^\[DEFAULT]/,/^\[/ {
      /^enabled_apis=/d
      /^#enabled_apis=/aenabled_apis=osapi_compute,metadata
      /^transport_url=/d
      /^#transport_url=/atransport_url=rabbit://openstack:3c17215fe69ba1dad320@controller:5672/
      /^my_ip=/d
      /^#my_ip=/amy_ip=10.0.0.11
      /^state_path=/d
      /^#state_path=/astate_path=/var/lib/nova
    }' \
    -e '/^\[api_database]/,/^\[/ {
      /^connection=/d
      /^#connection=/aconnection=mysql+pymysql://nova:830e052c9ff3fab2efa4@controller/nova_api
    }' \
    -e '/^\[database]/,/^\[/ {
      /^connection=/d
      /^#connection=/aconnection=mysql+pymysql://nova:830e052c9ff3fab2efa4@controller/nova
    }' \
    -e '/^\[api]/,/^\[/ {
      /^auth_strategy=/d
      /^#auth_strategy=/aauth_strategy=keystone
    }' \
    -e '/^\[keystone_authtoken]/,/^\[/ {
      /^www_authenticate_uri=/d
      /^#www_authenticate_uri=/awww_authenticate_uri=http://controller:5000
      /^memcached_servers=/d
      /^#memcached_servers=/amemcached_servers=controller:11211
      /^auth_type=/d
      /^#auth_type=/aauth_type=password
      /^auth_url=/d
      /^\[keystone_authtoken]/aauth_url=http://controller:5000
      /^project_domain_name=/d
      /^\[keystone_authtoken]/aproject_domain_name=Default
      /^project_name=/d
      /^\[keystone_authtoken]/aproject_name=service
      /^user_domain_name=/d
      /^\[keystone_authtoken]/auser_domain_name=Default
      /^username=/d
      /^\[keystone_authtoken]/ausername=nova
      /^password=/d
      /^\[keystone_authtoken]/apassword=5c5cfbce3214db530456
    }' \
    -e '/^\[service_user]/,/^\[/ {
      /^send_service_user_token=/d
      /^#send_service_user_token=/asend_service_user_token=true
      /^auth_type=/d
      /^#auth_type=/aauth_type=password
      /^auth_url=/d
      /^#auth_url=/aauth_url=http://controller:5000
      /^project_domain_name=/d
      /^#project_domain_name=/aproject_domain_name=Default
      /^project_name=/d
      /^#project_name=/aproject_name=service
      /^user_domain_name=/d
      /^#user_domain_name=/auser_domain_name=Default
      /^username=/d
      /^#username=/ausername=nova
      /^password=/d
      /^#password=/apassword=5c5cfbce3214db530456
      /^auth_strategy=/d
      /^\[service_user]/aauth_strategy=keystone
    }' \
    -e '/^\[vnc]/,/^\[/ {
      /^enabled=/d
      /^#enabled=/aenabled=true
      /^server_listen=/d
      /^#server_listen=/aserver_listen=$my_ip
      /^server_proxyclient_address=/d
      /^#server_proxyclient_address=/aserver_proxyclient_address=$my_ip
    }' \
    -e '/^\[glance]/,/^\[/ {
      /^api_servers=/d
      /^#api_servers=/aapi_servers=http://controller:9292
    }' \
    -e '/^\[oslo_concurrency]/,/^\[/ {
      /^lock_path=/d
      /^#lock_path=/alock_path=$state_path/tmp
    }' \
    -e '/^\[placement]/,/^\[/ {
      /^auth_type=/d
      /^#auth_type=/aauth_type=password
      /^auth_url=/d
      /^#auth_url=/aauth_url=http://controller:5000
      /^project_domain_name=/d
      /^#project_domain_name=/aproject_domain_name=Default
      /^project_name=/d
      /^#project_name=/aproject_name=service
      /^user_domain_name=/d
      /^#user_domain_name=/auser_domain_name=Default
      /^username=/d
      /^#username=/ausername=placement
      /^password=/d
      /^#password=/apassword=b4b6936f87c338567d3c
      /^region_name=/d
      /^#region_name=/aregion_name=RegionOne
    }' \
    -i /etc/nova/nova.conf
```

## データベースの構築

データベースを構築する。

```sh
su -s /bin/sh -c "nova-manage api_db sync" nova
su -s /bin/sh -c "nova-manage cell_v2 map_cell0" nova
su -s /bin/sh -c "nova-manage cell_v2 create_cell --name=cell1 --verbose" nova
su -s /bin/sh -c "nova-manage db sync" nova
```

cell0 と cell1 の作成を確認する。

```sh
su -s /bin/sh -c "nova-manage cell_v2 list_cells" nova
```

```
+-------+--------------------------------------+------------------------------------------+-------------------------------------------------+----------+
|  Name |                 UUID                 |              Transport URL               |               Database Connection               | Disabled |
+-------+--------------------------------------+------------------------------------------+-------------------------------------------------+----------+
| cell0 | 00000000-0000-0000-0000-000000000000 |                  none:/                  | mysql+pymysql://nova:****@controller/nova_cell0 |  False   |
| cell1 | 3d7b9db7-4e4a-4f7a-b91a-e054e57906c3 | rabbit://openstack:****@controller:5672/ |    mysql+pymysql://nova:****@controller/nova    |  False   |
+-------+--------------------------------------+------------------------------------------+-------------------------------------------------+----------+
```

## 起動

マシン起動時の自動起動設定とサービスを起動する。

```sh
systemctl enable --now openstack-nova-api
systemctl enable --now openstack-nova-scheduler
systemctl enable --now openstack-nova-conductor
systemctl enable --now openstack-nova-novncproxy
```

## 動作確認

サービスを表示する。

```sh
openstack compute service list
```

```
+--------------------------------------+----------------+-----------------------+----------+---------+-------+----------------------------+
| ID                                   | Binary         | Host                  | Zone     | Status  | State | Updated At                 |
+--------------------------------------+----------------+-----------------------+----------+---------+-------+----------------------------+
| dfd02086-0a60-42bc-8555-6b2aa8bc5e39 | nova-scheduler | controller.home.local | internal | enabled | up    | 2024-04-13T06:16:36.000000 |
| 704c4065-cc51-4f1d-80e9-45f8ba3ceed2 | nova-conductor | controller.home.local | internal | enabled | up    | 2024-04-13T06:16:30.000000 |
+--------------------------------------+----------------+-----------------------+----------+---------+-------+----------------------------+
```
