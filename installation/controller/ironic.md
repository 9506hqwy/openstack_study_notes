# Bare Metal Provisioning (Ironic)

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
| id                  | dabbb8a2ad1849e98e8f4910f8b0e1ad |
| name                | ironic                           |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```

プロジェクト service でユーザ ironic にロール admin 権限を追加する。

```sh
openstack role add --project service --user ironic admin
```

## サービスの作成

サービス baremetal を作成する。

```sh
openstack service create --name ironic --description "Bare Metal Provisioning" baremetal
```

```
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Bare Metal Provisioning          |
| enabled     | True                             |
| id          | 2f754716a0b44a6292f7d4d593a3c7b3 |
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
| id           | c23ada8824e04450a53962941ee9e31a |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 2f754716a0b44a6292f7d4d593a3c7b3 |
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
| id           | 9044e6f45c514e838451b00b31dd799b |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 2f754716a0b44a6292f7d4d593a3c7b3 |
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
| id           | 25b079194ca1424bb265a56e61bfd0a9 |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 2f754716a0b44a6292f7d4d593a3c7b3 |
| service_name | ironic                           |
| service_type | baremetal                        |
| url          | http://controller:6385/          |
+--------------+----------------------------------+
```

ファイアウォールを開ける。

```sh
firewall-cmd --permanent --zone=internal --add-port=6385/tcp
firewall-cmd --permanent --zone=internal --add-port=8080/tcp
firewall-cmd --reload
```

## デプロイイメージの作成

デプロイイメージをダウンロードする。

```sh
curl -sSOL https://tarballs.opendev.org/openstack/ironic-python-agent/tinyipa/files/tinyipa-stable-2024.1.gz
curl -sSOL https://tarballs.opendev.org/openstack/ironic-python-agent/tinyipa/files/tinytinyipa-stable-2024.1.vmlinuz
```

イメージを登録する。


```sh
glance image-create \
    --name "deploy-vmlinuz" \
    --file tinyipa-stable-2024.1.vmlinuz \
    --disk-format aki  \
    --container-format aki \
    --visibility=public
```

```
+------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field            | Value                                                                                                                                                   |
+------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------+
| checksum         | 872001fdd5d80dee905654241cc6b4e7                                                                                                                        |
| container_format | aki                                                                                                                                                     |
| created_at       | 2024-05-18T05:52:42Z                                                                                                                                    |
| disk_format      | aki                                                                                                                                                     |
| file             | /v2/images/403d8dde-5ad7-43a5-a3fb-7bf39b6f8d9f/file                                                                                                    |
| id               | 403d8dde-5ad7-43a5-a3fb-7bf39b6f8d9f                                                                                                                    |
| min_disk         | 0                                                                                                                                                       |
| min_ram          | 0                                                                                                                                                       |
| name             | deploy-vmlinuz                                                                                                                                          |
| owner            | be94f4411bd74f249f5e25f642209b82                                                                                                                        |
| properties       | os_hash_algo='sha512',                                                                                                                                  |
|                  | os_hash_value='f9c40c5847d4d985d7ab955636db2701ed97d5f388f4b0ce8333567f5f39f89689c6c57dd5221826da8bc94c14fbb49ed37db96abe3587157082b809c6de3f57',       |
|                  | os_hidden='False', stores='fs'                                                                                                                          |
| protected        | False                                                                                                                                                   |
| schema           | /v2/schemas/image                                                                                                                                       |
| size             | 5924896                                                                                                                                                 |
| status           | active                                                                                                                                                  |
| tags             |                                                                                                                                                         |
| updated_at       | 2024-05-18T05:52:42Z                                                                                                                                    |
| visibility       | public                                                                                                                                                  |
+------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------+
```

```sh
glance image-create \
    --name "deploy-initrd" \
    --file tinyipa-stable-2024.1.gz \
    --disk-format ari  \
    --container-format ari \
    --visibility=public
```

```
+------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field            | Value                                                                                                                                                   |
+------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------+
| checksum         | 093ecf129d93df24464dfb781f5b5b61                                                                                                                        |
| container_format | ari                                                                                                                                                     |
| created_at       | 2024-05-18T05:52:59Z                                                                                                                                    |
| disk_format      | ari                                                                                                                                                     |
| file             | /v2/images/7c856f98-5e4d-466e-bdc0-53024851ecd0/file                                                                                                    |
| id               | 7c856f98-5e4d-466e-bdc0-53024851ecd0                                                                                                                    |
| min_disk         | 0                                                                                                                                                       |
| min_ram          | 0                                                                                                                                                       |
| name             | deploy-initrd                                                                                                                                           |
| owner            | be94f4411bd74f249f5e25f642209b82                                                                                                                        |
| properties       | os_hash_algo='sha512',                                                                                                                                  |
|                  | os_hash_value='473335bfcdf48110cb03a7b79c2bd3e7ec732cd7051805450045d41c6deee47c7eb260bf331448035c6781281a43d60215c61d6585081929e222762483f5775b',       |
|                  | os_hidden='False', stores='fs'                                                                                                                          |
| protected        | False                                                                                                                                                   |
| schema           | /v2/schemas/image                                                                                                                                       |
| size             | 97912241                                                                                                                                                |
| status           | active                                                                                                                                                  |
| tags             |                                                                                                                                                         |
| updated_at       | 2024-05-18T05:52:59Z                                                                                                                                    |
| visibility       | public                                                                                                                                                  |
+------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------+
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
      /^#username =/ausername = glance
      /^password =/d
      /^#password =/apassword = 7a69c4834de4f3ed2cfa
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
      /^#username =/ausername = neutron
      /^password =/d
      /^#password =/apassword = 76283d854fd24b78f90b
      /^cleaning_network =/d
      /^#cleaning_network =/acleaning_network = 02451125-f735-49a4-b41a-31d962d6a1c0
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
      /^#deploy_kernel =/adeploy_kernel = 403d8dde-5ad7-43a5-a3fb-7bf39b6f8d9f
      /^deploy_ramdisk =/d
      /^#deploy_ramdisk =/adeploy_ramdisk = 7c856f98-5e4d-466e-bdc0-53024851ecd0
    }' \
    -e '/^\[agent]/,/^\[/ {
      /^image_download_source =/d
      /^#image_download_source =/aimage_download_source = http
    }' \
    -e '/^\[deploy]/,/^\[/ {
      /^http_url =/d
      /^#http_url =/ahttp_url = http://10.0.0.11:8080
    }' \
    -e '/^\[oslo_policy]/,/^\[/ {
      /^enforce_scope =/d
      /^#enforce_scope =/aenforce_scope = false
      /^enforce_new_defaults =/d
      /^#enforce_new_defaults =/aenforce_new_defaults = false
    }' \
    -i /etc/ironic/ironic.conf
```

`oslo_policy` は
[Upgrade Notes](https://docs.openstack.org/releasenotes/ironic/2024.1.html#relnotes-24-0-0-stable-2024-1-upgrade-notes)
を参照。

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
    -e '/^\[ironic]/,/^\[/ {
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
      /^#username=/ausername=ironic
      /^password=/d
      /^#password=/apassword=bdda64a33b3a75728a9c
    }' \
    -i /etc/nova/nova.conf
```

サービスを再起動する。

```sh
systemctl restart openstack-nova-scheduler
```

## Neutron の設定

既定の Policy では Neutron Port の作成に失敗するためポリシーを変更する。

```sh
cat > /etc/neutron/policy.yaml <<EOF
"create_port:binding:profile": "rule:admin_only or rule:service_api"
"update_port:binding:profile": "rule:admin_only or rule:service_api"
EOF
```
[Policy: binding operations are prohibited for service role](https://bugs.launchpad.net/neutron/+bug/2052937) を参照。

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
mkdir -p /tftpboot/grub
chown -R ironic /tftpboot
chgrp -R ironic /tftpboot
restorecon -R /tftpboot
```

ファイアウォールを開ける。

```sh
firewall-cmd --permanent --zone=internal --add-service=tftp
firewall-cmd --reload
```

## HTTP の設定

イメージを配信するための HTTP サーバを用意する。

```sh
mkdir -p /httpboot/grub
chown -R ironic /httpboot
chgrp -R ironic /httpboot
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
Getting computes from cell 'cell1': f97c2c7f-5010-4b97-aef3-7f910006d261
Found 0 unmapped computes in cell: f97c2c7f-5010-4b97-aef3-7f910006d261
```

登録を確認する。

```sh
openstack compute service list
```

```
+--------------------------------------+----------------+-----------------------+----------+---------+-------+----------------------------+
| ID                                   | Binary         | Host                  | Zone     | Status  | State | Updated At                 |
+--------------------------------------+----------------+-----------------------+----------+---------+-------+----------------------------+
| 85f4f538-7249-4871-a558-4576809b11e2 | nova-scheduler | controller.home.local | internal | enabled | up    | 2024-05-19T01:42:15.000000 |
| 3c2f4649-7236-4289-bb16-7558dde4a791 | nova-conductor | controller.home.local | internal | enabled | up    | 2024-05-19T01:42:14.000000 |
| a02d831e-cac8-468e-86d4-0074af0f31d7 | nova-compute   | compute.home.local    | nova     | enabled | up    | 2024-05-19T01:42:20.000000 |
| 624dac1e-2bd6-4f11-9c98-fabbc197c7c3 | nova-compute   | controller.home.local | nova     | enabled | up    | 2024-05-19T01:42:13.000000 |
+--------------------------------------+----------------+-----------------------+----------+---------+-------+----------------------------+
```

## 動作確認

ドライバを表示する。

```{note}
`oslo_policy` で以前のリリースの動作に戻していない場合は System Scoped で確認する。
```

```sh
OS_PROJECT_NAME="" \
OS_PROJECT_DOMAIN_NAME="" \
openstack --os-system-scope all baremetal driver list
```

```
+---------------------+-----------------------+
| Supported driver(s) | Active host(s)        |
+---------------------+-----------------------+
| staging-libvirt     | controller.home.local |
+---------------------+-----------------------+
```
