# OpenStack Bare Metal Provisioning (Ironic)

## データベースの作成

データベースに root でログインする。

```sh
mysql -u root -p59c6b803ff3fd0e68dcd
```

データベース ironic を作成する。

```sh
CREATE DATABASE ironic CHARACTER SET utf8mb3;
GRANT ALL PRIVILEGES ON ironic.* TO 'ironic'@'localhost' IDENTIFIED BY '383e3b4468252474df9f';
GRANT ALL PRIVILEGES ON ironic.* TO 'ironic'@'%' IDENTIFIED BY '383e3b4468252474df9f';
```

## ユーザの作成

ユーザ ironic を作成する。

```sh
openstack user create --domain default --password bdda64a33b3a75728a9c ironic
```

```
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 87dcf50b3c0140f8926908cb652da440 |
| name                | ironic                           |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```

プロジェクト service にロール admin 権限でユーザ ironic を追加する。

```sh
openstack role add --project service --user ironic admin
```

## サービスの作成

サービス baremetal を作成する。

```sh
openstack service create --name ironic --description "OpenStack Bare Metal" baremetal
```

```
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Bare Metal             |
| enabled     | True                             |
| id          | 8cfff9b140264c058778c66d21a9d7e2 |
| name        | ironic                           |
| type        | baremetal                        |
+-------------+----------------------------------+
```

## エンドポイントの作成

API エンドポイントを作成する。

```sh
openstack endpoint create --region RegionOne baremetal public http://controller:6385/
```

```
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 8b6d462f29934bd2aa12cf317d5277ee |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 8cfff9b140264c058778c66d21a9d7e2 |
| service_name | ironic                           |
| service_type | baremetal                        |
| url          | http://controller:6385/          |
+--------------+----------------------------------+
```

```sh
openstack endpoint create --region RegionOne baremetal internal http://controller:6385/
```

```
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 535dc54865934417a78b4346a907d816 |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 8cfff9b140264c058778c66d21a9d7e2 |
| service_name | ironic                           |
| service_type | baremetal                        |
| url          | http://controller:6385/          |
+--------------+----------------------------------+
```

```sh
openstack endpoint create --region RegionOne baremetal admin http://controller:6385/
```

```
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | af4377422d984fbe83834f27c6e05096 |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 8cfff9b140264c058778c66d21a9d7e2 |
| service_name | ironic                           |
| service_type | baremetal                        |
| url          | http://controller:6385/          |
+--------------+----------------------------------+
```

ファイアウォールを開ける。

```sh
firewall-cmd --permanent --zone=internal --add-port=6385/tcp
firewall-cmd --reload
```

## デプロイイメージの作成

デプロイイメージをダウンロードする。

```sh
curl -sSOL https://tarballs.opendev.org/openstack/ironic-python-agent/tinyipa/files/tinyipa-stable-yoga.gz
curl -sSOL https://tarballs.opendev.org/openstack/ironic-python-agent/tinyipa/files/tinyipa-stable-yoga.vmlinuz
```

イメージを登録する。


```sh
glance image-create \
    --name "deploy-vmlinuz" \
    --file tinyipa-stable-yoga.vmlinuz \
    --disk-format aki  \
    --container-format aki \
    --visibility=public
```

```
+------------------+----------------------------------------------------------------------------------+
| Property         | Value                                                                            |
+------------------+----------------------------------------------------------------------------------+
| checksum         | ab718135381e05da26455c216ca698a3                                                 |
| container_format | aki                                                                              |
| created_at       | 2024-04-30T12:12:58Z                                                             |
| disk_format      | aki                                                                              |
| id               | 8ab4a55e-01d7-4d1e-98d6-188478edeae2                                             |
| min_disk         | 0                                                                                |
| min_ram          | 0                                                                                |
| name             | deploy-vmlinuz                                                                   |
| os_hash_algo     | sha512                                                                           |
| os_hash_value    | 4d423b2d5fb3095597b1b1ec5fb77efc5b7a3fe6758d31b25da65d77e1d30cf7ba2e984bfd278123 |
|                  | 5b411abc4b01c4b4edd50c92513edd7761aeac2d2e0c38cb                                 |
| os_hidden        | False                                                                            |
| owner            | 1e3ac7ae10e24515a0956beaa1d8073c                                                 |
| protected        | False                                                                            |
| size             | 5521344                                                                          |
| status           | active                                                                           |
| tags             | []                                                                               |
| updated_at       | 2024-04-30T12:12:59Z                                                             |
| virtual_size     | Not available                                                                    |
| visibility       | public                                                                           |
+------------------+----------------------------------------------------------------------------------+
```

```sh
glance image-create \
    --name "deploy-initrd" \
    --file tinyipa-stable-yoga.gz \
    --disk-format ari  \
    --container-format ari \
    --visibility=public
```

```
+------------------+----------------------------------------------------------------------------------+
| Property         | Value                                                                            |
+------------------+----------------------------------------------------------------------------------+
| checksum         | 29cfc9ea71ef4dd8254be5c1de407b32                                                 |
| container_format | ari                                                                              |
| created_at       | 2024-04-30T12:14:08Z                                                             |
| disk_format      | ari                                                                              |
| id               | af500ca4-7f4e-4496-870b-8a9064e89c8d                                             |
| min_disk         | 0                                                                                |
| min_ram          | 0                                                                                |
| name             | deploy-initrd                                                                    |
| os_hash_algo     | sha512                                                                           |
| os_hash_value    | 9b6b151a769f1ac7054810f98f204635f7fe61532ff657aacc2178305d1a0930da856461eb83b07e |
|                  | 9fa55f3cb6e4ee44b0ce6ce1501f45ef69008cfb8d50a50b                                 |
| os_hidden        | False                                                                            |
| owner            | 1e3ac7ae10e24515a0956beaa1d8073c                                                 |
| protected        | False                                                                            |
| size             | 76779172                                                                         |
| status           | active                                                                           |
| tags             | []                                                                               |
| updated_at       | 2024-04-30T12:14:09Z                                                             |
| virtual_size     | Not available                                                                    |
| visibility       | public                                                                           |
+------------------+----------------------------------------------------------------------------------+
```

## インストール

ironic と [ステージングドライバ](https://ironic-staging-drivers.readthedocs.io/en/latest/) をインストールする。

```sh
dnf install -y \
    openstack-ironic-api \
    openstack-ironic-conductor \
    openstack-ironic-dnsmasq-tftp-server \
    openstack-ironic-staging-drivers \
    python3-libvirt \
    cyrus-sasl-md5 \
    cyrus-sasl-scram \
    python3-ironicclient \
    syslinux \
    syslinux-tftpboot \
    openstack-nova-compute
```

## ironic の設定

設定ファイルを更新する。

```sh
sed \
    -e '/^\[DEFAULT]/,/^\[/ {
      /^transport_url =/d
      /^#transport_url =/atransport_url = rabbit://openstack:3c17215fe69ba1dad320@controller:5672/
      /^auth_strategy =/d
      /^#auth_strategy =/aauth_strategy = keystone
      /^my_ip =/d
      /^#my_ip =/amy_ip = 10.0.0.11
      /^enabled_hardware_types =/d
      /^#enabled_hardware_types =/aenabled_hardware_types = staging-libvirt
      /^enabled_bios_interfaces =/d
      /^#enabled_bios_interfaces =/aenabled_bios_interfaces = no-bios
      /^enabled_boot_interfaces =/d
      /^#enabled_boot_interfaces =/aenabled_boot_interfaces = pxe
      /^enabled_deploy_interfaces =/d
      /^#enabled_deploy_interfaces =/aenabled_deploy_interfaces = direct
      /^enabled_inspect_interfaces =/d
      /^#enabled_inspect_interfaces =/aenabled_inspect_interfaces = no-inspect
      /^enabled_management_interfaces =/d
      /^#enabled_management_interfaces =/aenabled_management_interfaces = staging-libvirt
      /^enabled_power_interfaces =/d
      /^#enabled_power_interfaces =/aenabled_power_interfaces = staging-libvirt
      /^enabled_raid_interfaces =/d
      /^#enabled_raid_interfaces =/aenabled_raid_interfaces = no-raid
      /^enabled_vendor_interfaces =/d
      /^#enabled_vendor_interfaces =/aenabled_vendor_interfaces = no-vendor
    }' \
    -e '/^\[database]/,/^\[/ {
      /^connection =/d
      /^#connection =/aconnection = mysql+pymysql://ironic:383e3b4468252474df9f@controller/ironic
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
      /^\[keystone_authtoken]/ausername = ironic
      /^password =/d
      /^\[keystone_authtoken]/apassword = bdda64a33b3a75728a9c
    }' \
    -e '/^\[glance]/,/^\[/ {
      /^auth_url =/d
      /^#auth_url =/aauth_url = http://controller:5000
      /^auth_type =/d
      /^#auth_type =/aauth_type = password
      /^project_domain_name =/d
      /^#project_domain_name =/aproject_domain_name = Default
      /^project_name =/d
      /^#project_name =/aproject_name = service
      /^user_domain_name =/d
      /^#user_domain_name =/auser_domain_name = Default
      /^username =/d
      /^#username =/ausername = ironic
      /^password =/d
      /^#password =/apassword = bdda64a33b3a75728a9c
    }' \
    -e '/^\[neutron]/,/^\[/ {
      /^auth_url =/d
      /^#auth_url =/aauth_url = http://controller:5000
      /^auth_type =/d
      /^#auth_type =/aauth_type = password
      /^project_domain_name =/d
      /^#project_domain_name =/aproject_domain_name = Default
      /^project_name =/d
      /^#project_name =/aproject_name = service
      /^user_domain_name =/d
      /^#user_domain_name =/auser_domain_name = Default
      /^username =/d
      /^#username =/ausername = ironic
      /^password =/d
      /^#password =/apassword = bdda64a33b3a75728a9c
      /^cleaning_network =/d
      /^#cleaning_network =/acleaning_network = eda78c59-6bc0-4302-927f-3df79bdf57de
    }' \
    -e '/^\[service_catalog]/,/^\[/ {
      /^auth_url =/d
      /^#auth_url =/aauth_url = http://controller:5000
      /^auth_type =/d
      /^#auth_type =/aauth_type = password
      /^project_domain_name =/d
      /^#project_domain_name =/aproject_domain_name = Default
      /^project_name =/d
      /^#project_name =/aproject_name = service
      /^user_domain_name =/d
      /^#user_domain_name =/auser_domain_name = Default
      /^username =/d
      /^#username =/ausername = ironic
      /^password =/d
      /^#password =/apassword = bdda64a33b3a75728a9c
    }' \
    -e '/^\[conductor]/,/^\[/ {
      /^deploy_kernel =/d
      /^#deploy_kernel =/adeploy_kernel = 8ab4a55e-01d7-4d1e-98d6-188478edeae2
      /^deploy_ramdisk =/d
      /^#deploy_ramdisk =/adeploy_ramdisk = af500ca4-7f4e-4496-870b-8a9064e89c8d
    }' \
    -i /etc/ironic/ironic.conf
```

## nova の設定

設定ファイルを更新する。

```sh
sed \
    -e '/^\[DEFAULT]/,/^\[/ {
      /^compute_driver=/d
      /^#compute_driver=/acompute_driver = ironic.IronicDriver
      /^reserved_host_memory_mb=/d
      /^#reserved_host_memory_mb=/areserved_host_memory_mb = 0
    }' \
    -e '/^\[filter_scheduler]/,/^\[/ {
      /^track_instance_changes=/d
      /^#track_instance_changes=/atrack_instance_changes = false
    }' \
    -i /etc/nova/nova.conf

sed \
    -e '/^\[ironic]/,/^\[/d' \
    -i /etc/nova/nova.conf

cat >> /etc/nova/nova.conf <<EOF

[ironic]
auth_url = http://controller:5000
auth_type = password
project_domain_name = Default
project_name = service
user_domain_name = Default
username = ironic
password = bdda64a33b3a75728a9c
EOF
```

サービスを再起動する。

```sh
systemctl restart openstack-nova-scheduler
```

## TFTP の設定

設定ファイルを更新する。

```sh
sed \
    -e '/^listen-address=/d' \
    -e '/^#listen-address=/alisten-address=10.0.0.11' \
    -i /etc/ironic/dnsmasq-tftp-server.conf
```

権限を更新する。

```sh
restorecon -R /tftpboot
chown -R ironic /tftpboot
```

ファイアウォールを開ける。

```sh
firewall-cmd --permanent --zone=internal --add-service=tftp
firewall-cmd --reload
```

## HTTP の設定

イメージを配信するための HTTP サーバを用意する。

```sh
mkdir /httpboot
chown ironic /httpboot
restorecon -R /httpboot
```

```sh
cat > /etc/httpd/conf.d/openstack-ironic-httpboot.conf <<EOF
Listen 8080
<VirtualHost *:8080>
  DocumentRoot "/httpboot"
  <Directory "/httpboot">
    Options Indexes FollowSymLinks
    Require all granted
  </Directory>
</VirtualHost>
EOF
```

再起動する。

```sh
systemctl restart httpd
```

## データベースの構築

データベースを構築する。

```sh
ironic-dbsync --config-file /etc/ironic/ironic.conf create_schema
```

## 起動

マシン起動時の自動起動設定とサービスを起動する。

```sh
systemctl enable --now openstack-ironic-dnsmasq-tftp-server
systemctl enable --now openstack-ironic-api
systemctl enable --now openstack-ironic-conductor
systemctl enable --now openstack-nova-compute
```

## 検出

Controller Node の nova-compute を検出する。

```sh
su -s /bin/sh -c "nova-manage cell_v2 discover_hosts --verbose" nova
```

```
Found 2 cell mappings.
Skipping cell0 since it does not contain hosts.
Getting computes from cell 'cell1': 3d7b9db7-4e4a-4f7a-b91a-e054e57906c3
Checking host mapping for compute host 'controller.home.local': 803559b8-3a5b-4e80-b23d-7bd99fbbc5da
Creating host mapping for compute host 'controller.home.local': 803559b8-3a5b-4e80-b23d-7bd99fbbc5da
Found 1 unmapped computes in cell: 3d7b9db7-4e4a-4f7a-b91a-e054e57906c3
```

登録を確認する。

```sh
openstack compute service list
```

```
+--------------------------------------+----------------+-----------------------+----------+---------+-------+----------------------------+
| ID                                   | Binary         | Host                  | Zone     | Status  | State | Updated At                 |
+--------------------------------------+----------------+-----------------------+----------+---------+-------+----------------------------+
| dfd02086-0a60-42bc-8555-6b2aa8bc5e39 | nova-scheduler | controller.home.local | internal | enabled | up    | 2024-05-02T03:27:57.000000 |
| 704c4065-cc51-4f1d-80e9-45f8ba3ceed2 | nova-conductor | controller.home.local | internal | enabled | up    | 2024-05-02T03:27:58.000000 |
| 920d37ad-0984-42ef-907d-26cf205858ca | nova-compute   | compute.home.local    | nova     | enabled | up    | 2024-05-02T03:27:59.000000 |
| 91b41c79-a125-49b3-b160-560f1e38c4a1 | nova-compute   | controller.home.local | nova     | enabled | up    | 2024-05-02T03:27:56.000000 |
+--------------------------------------+----------------+-----------------------+----------+---------+-------+----------------------------+
```

## 動作確認

ドライバを表示する。

```sh
openstack baremetal driver list
```

```
+---------------------+-----------------------+
| Supported driver(s) | Active host(s)        |
+---------------------+-----------------------+
| staging-libvirt     | controller.home.local |
+---------------------+-----------------------+
```
