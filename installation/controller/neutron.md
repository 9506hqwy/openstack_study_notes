# Networking (Neutron)

## データベースの作成

データベースに root でログインする。

```sh
mysql -u root -p59c6b803ff3fd0e68dcd
```

データベース neutron を作成する。

```sh
CREATE DATABASE neutron;
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' IDENTIFIED BY '7d58a6544954d3272e44';
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' IDENTIFIED BY '7d58a6544954d3272e44';
```

## ユーザの作成

ユーザ neutron を作成する。

```sh
openstack user create --domain default --password 76283d854fd24b78f90b neutron
```

```
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 46347461b532408586b5f63cb59d6aea |
| name                | neutron                          |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```

プロジェクト service でユーザ neutron にロール admin 権限を追加する。

```sh
openstack role add --project service --user neutron admin
```

## サービスの作成

neutron サービスを作成する。

```sh
openstack service create --name neutron --description "Networking" network
```

```
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Networking                       |
| enabled     | True                             |
| id          | f1c57ae2421a4b1196d90cbd5aee209c |
| name        | neutron                          |
| type        | network                          |
+-------------+----------------------------------+
```

## エンドポイントの作成

API エンドポイントを作成する。

```sh
openstack endpoint create --region RegionOne network public http://controller:9696
```

```
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | d0f928fc84534182ad1ffd8d6835eca6 |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | f1c57ae2421a4b1196d90cbd5aee209c |
| service_name | neutron                          |
| service_type | network                          |
| url          | http://controller:9696           |
+--------------+----------------------------------+
```

```sh
openstack endpoint create --region RegionOne network internal http://controller:9696
```

```
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 8d42dacfffa84b60977475924270f1de |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | f1c57ae2421a4b1196d90cbd5aee209c |
| service_name | neutron                          |
| service_type | network                          |
| url          | http://controller:9696           |
+--------------+----------------------------------+
```

```sh
openstack endpoint create --region RegionOne network admin http://controller:9696
```

```
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | cf4038de6dd64eff8597858fe2492eee |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | f1c57ae2421a4b1196d90cbd5aee209c |
| service_name | neutron                          |
| service_type | network                          |
| url          | http://controller:9696           |
+--------------+----------------------------------+
```

ファイアウォールを開ける。

```sh
firewall-cmd --permanent --zone=internal --add-port=9696/tcp
firewall-cmd --reload
```

## インストール

neutron と ML2 プラグインをインストールする。

```sh
dnf install -y \
    openstack-neutron \
    openstack-neutron-ml2
```

Linux Bridge を使用する場合は下記をインストールする。

```sh
dnf install -y openstack-neutron-linuxbridge ebtables
```

Open vSwitch を使用する場合は下記をインストールする。

```sh
dnf install -y openstack-neutron-openvswitch NetworkManager-ovs
```

## Neutron の設定

設定ファイルを更新する。

```sh
sed \
    -e '/^\[DEFAULT]/,/^\[/ {
      /^state_path =/d
      /^#state_path =/astate_path = /var/lib/neutron
      /^core_plugin =/d
      /^#core_plugin =/acore_plugin = ml2
      /^service_plugins =/d
      /^#service_plugins =/aservice_plugins =
      /^transport_url =/d
      /^#transport_url =/atransport_url = rabbit://openstack:3c17215fe69ba1dad320@controller:5672/
      /^auth_strategy =/d
      /^#auth_strategy =/aauth_strategy = keystone
      /^notify_nova_on_port_status_changes =/d
      /^#notify_nova_on_port_status_changes =/anotify_nova_on_port_status_changes  = true
      /^notify_nova_on_port_data_changes =/d
      /^#notify_nova_on_port_data_changes =/anotify_nova_on_port_data_changes = true
    }' \
    -e '/^\[database]/,/^\[/ {
      /^connection =/d
      /^#connection =/aconnection = mysql+pymysql://neutron:7d58a6544954d3272e44@controller/neutron
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
      /^\[keystone_authtoken]/ausername = neutron
      /^password =/d
      /^\[keystone_authtoken]/apassword = 76283d854fd24b78f90b
    }' \
    -e '/^\[nova]/,/^\[/ {
      /^region_name =/d
      /^#region_name =/aregion_name = RegionOne
      /^auth_url =/d
      /^#auth_url =/aauth_url = http://controller:5000
      /^auth_type =/d
      /^#auth_type =/aauth_type = password
      /^password =/d
      /^#password =/apassword = 5c5cfbce3214db530456
      /^project_domain_name =/d
      /^#project_domain_name =/aproject_domain_name = Default
      /^project_name =/d
      /^#project_name =/aproject_name = service
      /^user_domain_name =/d
      /^#user_domain_name =/auser_domain_name = Default
      /^username =/d
      /^#username =/ausername = nova
    }' \
    -e '/^\[oslo_concurrency]/,/^\[/ {
      /^lock_path =/d
      /^# lock_path =/alock_path = $state_path/tmp
    }' \
    -e '/^\[experimental]/,/^\[/ {
      /^linuxbridge =/d
      /^#linuxbridge =/alinuxbridge = true
    }' \
    -i /etc/neutron/neutron.conf
```

## メカニズムドライバの設定

Linux Bridge と Open vSwitch のどちらかを設定する。

### Linux Bridge の場合

* [](./neutron_linuxbridge/ml2_plugin.md)
* [](./neutron_linuxbridge/agent.md)
* [](./neutron_linuxbridge/dhcp.md)

### Open vSwitch の場合

* [](./neutron_ovs/ml2_plugin.md)
* [](./neutron_ovs/agent.md)
* [](./neutron_ovs/dhcp.md)

## メタデータの設定

設定ファイルを更新する。

```sh
sed \
    -e '/^\[DEFAULT]/,/^\[/ {
      /^nova_metadata_host =/d
      /^#nova_metadata_host =/anova_metadata_host = controller
      /^metadata_proxy_shared_secret =/d
      /^#metadata_proxy_shared_secret =/ametadata_proxy_shared_secret = 44cb41ccbed49e089ab4
    }' \
    -i /etc/neutron/metadata_agent.ini
```

## Nova の設定

設定ファイルを更新する。

```sh
sed \
    -e '/^\[neutron]/,/^\[/ {
      /^service_metadata_proxy=/d
      /^#service_metadata_proxy=/aservice_metadata_proxy=true
      /^metadata_proxy_shared_secret =/d
      /^#metadata_proxy_shared_secret =/ametadata_proxy_shared_secret = 44cb41ccbed49e089ab4
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
      /^#username=/ausername=neutron
      /^password=/d
      /^#password=/apassword=76283d854fd24b78f90b
      /^region_name=/d
      /^#region_name=/aregion_name=RegionOne
    }' \
    -i /etc/nova/nova.conf
```

## SELinux の設定

ポリシーを変更する。

```sh
setsebool -P os_neutron_dac_override on
setsebool -P os_dnsmasq_dac_override on
```

[Bug 1850973](https://bugzilla.redhat.com/show_bug.cgi?id=1850973) を参照。

## データベースの構築

設定ファイルを作成する。

```sh
ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini
```

データベースを構築する。

```sh
su -s /bin/sh -c "neutron-db-manage \
    --config-file /etc/neutron/neutron.conf \
    --config-file /etc/neutron/plugins/ml2/ml2_conf.ini \
    upgrade head" neutron
```

## 起動

メカニズムドライバの種類によって手順を選択する。

* [](./neutron_linuxbridge/startup.md)
* [](./neutron_ovs/startup.md)

## 動作確認

メカニズムドライバの種類によって手順を選択する。

* [](./neutron_linuxbridge/confirm.md)
* [](./neutron_ovs/confirm.md)

## VLAN ネットワークの設定

VLAN ネットワークを使用する場合はメカニズムドライバの種類によって手順を選択する。

* [](./neutron_linuxbridge/vlan.md)
* [](./neutron_ovs/vlan.md)

## VXLAN ネットワークの設定

VXLAN ネットワークを使用する場合はメカニズムドライバの種類によって手順を選択する。

* [](./neutron_linuxbridge/vxlan.md)
* [](./neutron_ovs/vxlan.md)
