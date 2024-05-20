# Compute (Nova)

## インストール

nova をインストールする。

```sh
dnf install -y libvirt openstack-nova-compute
```

## Nova の設定

設定ファイルを更新する。

`novncproxy_base_url` はコンソールを表示するクライントが解決できるホスト名を指定する。

```sh
sed \
    -e '/^\[DEFAULT]/,/^\[/ {
      /^compute_driver=/d
      /^#compute_driver=/acompute_driver=libvirt.LibvirtDriver
      /^enabled_apis=/d
      /^#enabled_apis=/aenabled_apis=osapi_compute,metadata
      /^transport_url=/d
      /^#transport_url=/atransport_url=rabbit://openstack:3c17215fe69ba1dad320@controller:5672/
      /^my_ip=/d
      /^#my_ip=/amy_ip=10.0.0.31
      /^state_path=/d
      /^#state_path=/astate_path=/var/lib/nova
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
      /^#server_listen=/aserver_listen=0.0.0.0
      /^server_proxyclient_address=/d
      /^#server_proxyclient_address=/aserver_proxyclient_address=$my_ip
      /^novncproxy_base_url=/d
      /^#novncproxy_base_url=/anovncproxy_base_url=http://controller:6080/vnc_auto.html
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

仮想化支援機能がないため QEMU を使用する。

```sh
sed -e '/^#virt_type=/avirt_type=qemu' -i /etc/nova/nova.conf
```

ファイアウォールを開ける。

```sh
firewall-cmd --permanent --zone=internal --add-service=vnc-server
firewall-cmd --reload
```

## 起動

マシン起動時の自動起動設定とサービスを起動する。

```sh
systemctl enable --now libvirtd
systemctl enable --now openstack-nova-compute
```

## 登録

Controller Node で下記のコマンドを実行してデータベースに Compute Node があるか確認する。

```sh
openstack compute service list --service nova-compute
```

```
+--------------------------------------+--------------+--------------------+------+---------+-------+----------------------------+
| ID                                   | Binary       | Host               | Zone | Status  | State | Updated At                 |
+--------------------------------------+--------------+--------------------+------+---------+-------+----------------------------+
| a6339453-5193-4fdf-948b-c25d1c05944c | nova-compute | compute.home.local | nova | enabled | up    | 2024-05-09T11:58:34.000000 |
+--------------------------------------+--------------+--------------------+------+---------+-------+----------------------------+
```

Compute Node を検出する。

```sh
su -s /bin/sh -c "nova-manage cell_v2 discover_hosts --verbose" nova
```

```
Found 2 cell mappings.
Skipping cell0 since it does not contain hosts.
Getting computes from cell 'cell1': f97c2c7f-5010-4b97-aef3-7f910006d261
Checking host mapping for compute host 'compute.home.local': 6c53c76c-39d1-45dc-a5f5-293e91abd3e4
Creating host mapping for compute host 'compute.home.local': 6c53c76c-39d1-45dc-a5f5-293e91abd3e4
Found 1 unmapped computes in cell: f97c2c7f-5010-4b97-aef3-7f910006d261
```

## 動作確認

Controller Node でサービスを表示する。

```sh
openstack compute service list
```

```
+--------------------------------------+----------------+-----------------------+----------+---------+-------+----------------------------+
| ID                                   | Binary         | Host                  | Zone     | Status  | State | Updated At                 |
+--------------------------------------+----------------+-----------------------+----------+---------+-------+----------------------------+
| 85f4f538-7249-4871-a558-4576809b11e2 | nova-scheduler | controller.home.local | internal | enabled | up    | 2024-05-09T12:00:02.000000 |
| 3c2f4649-7236-4289-bb16-7558dde4a791 | nova-conductor | controller.home.local | internal | enabled | up    | 2024-05-09T12:00:02.000000 |
| a6339453-5193-4fdf-948b-c25d1c05944c | nova-compute   | compute.home.local    | nova     | enabled | up    | 2024-05-09T12:00:04.000000 |
+--------------------------------------+----------------+-----------------------+----------+---------+-------+----------------------------+
```
