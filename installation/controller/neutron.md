# OpenStack Networking (Neutron)

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
| id                  | 3744ebe3c7f3437c832bcf9da2e8ece6 |
| name                | neutron                          |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```

プロジェクト service にロール admin 権限でユーザ neutron を追加する。

```sh
openstack role add --project service --user neutron admin
```

## サービスの作成

neutron サービスを作成する。

```sh
openstack service create --name neutron --description "OpenStack Networking" network
```

```
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Networking             |
| enabled     | True                             |
| id          | 53b4c8d7b26e400a81a7ce187c0bf4cf |
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
| id           | 0d004f2e177c413281d96d967d9a2e85 |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 53b4c8d7b26e400a81a7ce187c0bf4cf |
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
| id           | 7dfe7cf516c54b73896f4a94181f166a |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 53b4c8d7b26e400a81a7ce187c0bf4cf |
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
| id           | 515675b528594faca7af2a6faed17e58 |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 53b4c8d7b26e400a81a7ce187c0bf4cf |
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

neutron と Linux Bridge ML2 プラグインをインストールする。

```sh
dnf install -y \
    openstack-neutron \
    openstack-neutron-ml2 \
    openstack-neutron-linuxbridge \
    ebtables
```

## Neutron の設定

設定ファイルを更新する。

```sh
sed \
    -e '/^\[DEFAULT]/,/^\[/ {
      /^state_path=/d
      /^#state_path=/astate_path=/var/lib/neutron
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
    -i /etc/neutron/neutron.conf
```

## ML2 プラグインの設定

設定ファイルを更新する。

```sh
sed \
    -e '/^\[ml2]/,/^\[/ {
      /^type_drivers =/d
      /^#type_drivers =/atype_drivers = flat,vlan
      /^tenant_network_types =/d
      /^#tenant_network_types =/atenant_network_types =
      /^mechanism_drivers =/d
      /^#mechanism_drivers =/amechanism_drivers = linuxbridge
      /^extension_drivers =/d
      /^#extension_drivers =/aextension_drivers = port_security
    }' \
    -e '/^\[ml2_type_flat]/,/^\[/ {
      /^flat_networks =/d
      /^#flat_networks =/aflat_networks = provider
    }' \
    -e '/^\[securitygroup]/,/^\[/ {
      /^enable_ipset =/d
      /^#enable_ipset =/aenable_ipset = true
    }' \
    -i /etc/neutron/plugins/ml2/ml2_conf.ini
```

## Linux Bridge の設定

設定ファイルを更新する。

```sh
sed \
    -e '/^\[linux_bridge]/,/^\[/ {
      /^physical_interface_mappings =/d
      /^#physical_interface_mappings =/aphysical_interface_mappings = provider:eth0
    }' \
    -e '/^\[vxlan]/,/^\[/ {
      /^enable_vxlan =/d
      /^#enable_vxlan =/aenable_vxlan = false
    }' \
    -e '/^\[securitygroup]/,/^\[/ {
      /^enable_security_group =/d
      /^#enable_security_group =/aenable_security_group = true
      /^firewall_driver =/d
      /^#firewall_driver =/afirewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
    }' \
    -i /etc/neutron/plugins/ml2/linuxbridge_agent.ini
```

br_netfilter モジュールを有効化する。

```sh
cat > /etc/modules-load.d/nuetron.conf <<EOF
br_netfilter
EOF

systemctl restart systemd-modules-load
```

## DHCP の設定

設定ファイルを更新する。

```sh
sed \
    -e '/^\[DEFAULT]/,/^\[/ {
      /^interface_driver =/d
      /^#interface_driver =/ainterface_driver = linuxbridge
      /^dhcp_driver =/d
      /^#dhcp_driver =/adhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
      /^enable_isolated_metadata =/d
      /^#enable_isolated_metadata =/aenable_isolated_metadata = true
    }' \
    -i /etc/neutron/dhcp_agent.ini
```

```{warning}
継続
```

DHCP エージェント がインスタンスの DHCP Discover を受信できないので
既定の firewalld ゾーンを trusted に設定する。

```sh
firewall-cmd --set-default-zone trusted
firewall-cmd --permanent --zone=public --change-interface=eth0
firewall-cmd --reload
```

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

nova を再起動する。

```sh
systemctl restart openstack-nova-api
```

マシン起動時の自動起動設定とサービスを起動する。

```sh
systemctl enable --now neutron-server
systemctl enable --now neutron-linuxbridge-agent
systemctl enable --now neutron-dhcp-agent
systemctl enable --now neutron-metadata-agent
```

## 動作確認

エージェントを表示する。

```sh
openstack network agent list
```

```
+--------------------------------------+--------------------+-----------------------+-------------------+-------+-------+---------------------------+
| ID                                   | Agent Type         | Host                  | Availability Zone | Alive | State | Binary                    |
+--------------------------------------+--------------------+-----------------------+-------------------+-------+-------+---------------------------+
| 145d9d73-cb1c-4bbf-ad28-adb1a9f4a826 | Metadata agent     | controller.home.local | None              | :-)   | UP    | neutron-metadata-agent    |
| e843c356-67a9-418e-96db-3fa6e4210df9 | DHCP agent         | controller.home.local | nova              | :-)   | UP    | neutron-dhcp-agent        |
| f075f5df-7f2f-4482-881f-5f463387be89 | Linux bridge agent | controller.home.local | None              | :-)   | UP    | neutron-linuxbridge-agent |
+--------------------------------------+--------------------+-----------------------+-------------------+-------+-------+---------------------------+
```
