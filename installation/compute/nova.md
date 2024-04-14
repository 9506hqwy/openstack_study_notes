# OpenStack Compute (Nova)

## インストール

nova をインストールする。

```sh
dnf install -y openstack-nova-compute
```

## Nova の設定

設定ファイルを更新する。

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
| 920d37ad-0984-42ef-907d-26cf205858ca | nova-compute | compute.home.local | nova | enabled | up    | 2024-04-13T07:06:11.000000 |
+--------------------------------------+--------------+--------------------+------+---------+-------+----------------------------+
```

Compute Node を検出する。

```sh
su -s /bin/sh -c "nova-manage cell_v2 discover_hosts --verbose" nova
```

```
Found 2 cell mappings.
Skipping cell0 since it does not contain hosts.
Getting computes from cell 'cell1': 3d7b9db7-4e4a-4f7a-b91a-e054e57906c3
Checking host mapping for compute host 'compute.home.local': 30cb800f-e095-4a5c-a05a-fdfafdeb435a
Creating host mapping for compute host 'compute.home.local': 30cb800f-e095-4a5c-a05a-fdfafdeb435a
Found 1 unmapped computes in cell: 3d7b9db7-4e4a-4f7a-b91a-e054e57906c3
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
| dfd02086-0a60-42bc-8555-6b2aa8bc5e39 | nova-scheduler | controller.home.local | internal | enabled | up    | 2024-04-13T07:12:51.000000 |
| 704c4065-cc51-4f1d-80e9-45f8ba3ceed2 | nova-conductor | controller.home.local | internal | enabled | up    | 2024-04-13T07:12:51.000000 |
| 920d37ad-0984-42ef-907d-26cf205858ca | nova-compute   | compute.home.local    | nova     | enabled | up    | 2024-04-13T07:12:50.000000 |
+--------------------------------------+----------------+-----------------------+----------+---------+-------+----------------------------+
```
