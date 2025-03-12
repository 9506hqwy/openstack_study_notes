# Block Storage (Cinder)

## データベースの作成

データベースに root でログインする。

```sh
mysql -u root -p59c6b803ff3fd0e68dcd
```

データベース cinder を作成する。

```sh
CREATE DATABASE cinder;
GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'localhost' IDENTIFIED BY '909ad266b6b2b52c6a6d';
GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'%' IDENTIFIED BY '909ad266b6b2b52c6a6d';
```

## ユーザの作成

ユーザ cinder を作成する。

```sh
openstack user create --domain default --password e2c046c01e44c27725c3 cinder
```

```text
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 340253b003604fcf8f16f45da4338528 |
| name                | cinder                           |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```

プロジェクト service でユーザ cinder にロール admin 権限を追加する。

```sh
openstack role add --project service --user cinder admin
```

## サービスの作成

```sh
openstack service create --name cinderv3 --description "Block Storage" volumev3
```

```text
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Block Storage                    |
| enabled     | True                             |
| id          | 24c8854b87f6413b8d1ced5314edb942 |
| name        | cinderv3                         |
| type        | volumev3                         |
+-------------+----------------------------------+
```

## エンドポイントの作成

API エンドポイントを作成する。

```sh
openstack endpoint create --region RegionOne volumev3 public http://controller:8776/v3/%\(project_id\)s
```

```text
+--------------+------------------------------------------+
| Field        | Value                                    |
+--------------+------------------------------------------+
| enabled      | True                                     |
| id           | 487b1f9bcb304f3ab54cad462f77032b         |
| interface    | public                                   |
| region       | RegionOne                                |
| region_id    | RegionOne                                |
| service_id   | 24c8854b87f6413b8d1ced5314edb942         |
| service_name | cinderv3                                 |
| service_type | volumev3                                 |
| url          | http://controller:8776/v3/%(project_id)s |
+--------------+------------------------------------------+
```

```sh
openstack endpoint create --region RegionOne volumev3 internal http://controller:8776/v3/%\(project_id\)s
```

```text
+--------------+------------------------------------------+
| Field        | Value                                    |
+--------------+------------------------------------------+
| enabled      | True                                     |
| id           | 56c19b3d68c64f49a9ea4d5dc101ecb2         |
| interface    | internal                                 |
| region       | RegionOne                                |
| region_id    | RegionOne                                |
| service_id   | 24c8854b87f6413b8d1ced5314edb942         |
| service_name | cinderv3                                 |
| service_type | volumev3                                 |
| url          | http://controller:8776/v3/%(project_id)s |
+--------------+------------------------------------------+
```

```sh
openstack endpoint create --region RegionOne volumev3 admin http://controller:8776/v3/%\(project_id\)s
```

```text
+--------------+------------------------------------------+
| Field        | Value                                    |
+--------------+------------------------------------------+
| enabled      | True                                     |
| id           | e962d34aaec7488fab9d542b70b1634e         |
| interface    | admin                                    |
| region       | RegionOne                                |
| region_id    | RegionOne                                |
| service_id   | 24c8854b87f6413b8d1ced5314edb942         |
| service_name | cinderv3                                 |
| service_type | volumev3                                 |
| url          | http://controller:8776/v3/%(project_id)s |
+--------------+------------------------------------------+
```

ファイアウォールを開ける。

```sh
firewall-cmd --permanent --zone=internal --add-port=8776/tcp
firewall-cmd --reload
```

## サービストークンの設定

[OSSA-2023-003](https://docs.openstack.org/cinder/latest/configuration/block-storage/service-token.html)
対応のためサービストークンを設定する。

ロール service 権限を作成する。

```sh
openstack role create service
```

```text
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | None                             |
| domain_id   | None                             |
| id          | 0fb5fb6a77d24878ae95964bb3c9aa97 |
| name        | service                          |
| options     | {}                               |
+-------------+----------------------------------+
```

プロジェクト service でユーザ cinder にロール service 権限を追加する。

```sh
openstack role add --user cinder --project service service
```

ロールが割り当たったことを確認する。

```sh
openstack role assignment list --user cinder --project service --names
```

```text
+---------+----------------+-------+-----------------+--------+--------+-----------+
| Role    | User           | Group | Project         | Domain | System | Inherited |
+---------+----------------+-------+-----------------+--------+--------+-----------+
| service | cinder@Default |       | service@Default |        |        | False     |
| admin   | cinder@Default |       | service@Default |        |        | False     |
+---------+----------------+-------+-----------------+--------+--------+-----------+
```

プロジェクト service でユーザ nova にロール service 権限を追加する。

```sh
openstack role add --user nova --project service service
```

ロールが割り当たったことを確認する。

```sh
openstack role assignment list --user nova --project service --names
```

```text
+---------+--------------+-------+-----------------+--------+--------+-----------+
| Role    | User         | Group | Project         | Domain | System | Inherited |
+---------+--------------+-------+-----------------+--------+--------+-----------+
| service | nova@Default |       | service@Default |        |        | False     |
| admin   | nova@Default |       | service@Default |        |        | False     |
+---------+--------------+-------+-----------------+--------+--------+-----------+
```

## インストール

cinder をインストールする。

```sh
dnf install -y openstack-cinder
```

## Cinder の設定

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
    }' \
    -e '/^\[database]/,/^\[/ {
      /^connection =/d
      /^#connection =/aconnection = mysql+pymysql://cinder:909ad266b6b2b52c6a6d@controller/cinder
    }' \
    -e '/^\[keystone_authtoken]/,/^\[/ {
      /^www_authenticate_uri =/d
      /^#www_authenticate_uri =/awww_authenticate_uri = http://controller:5000
      /^memcached_servers =/d
      /^#memcached_servers =/amemcached_servers = controller:11211
      /^auth_type =/d
      /^#auth_type =/aauth_type = password
      /^service_token_roles =/d
      /^#service_token_roles =/aservice_token_roles = service
      /^service_token_roles_required =/d
      /^#service_token_roles_required =/aservice_token_roles_required = true
      /^auth_url =/d
      /^\[keystone_authtoken]/aauth_url = http://controller:5000
      /^project_domain_name =/d
      /^\[keystone_authtoken]/aproject_domain_name = Default
      /^project_name =/d
      /^\[keystone_authtoken]/aproject_name = service
      /^user_domain_name =/d
      /^\[keystone_authtoken]/auser_domain_name = Default
      /^username =/d
      /^\[keystone_authtoken]/ausername = cinder
      /^password =/d
      /^\[keystone_authtoken]/apassword = e2c046c01e44c27725c3
    }' \
    -e '/^\[oslo_concurrency]/,/^\[/ {
      /^lock_path =/d
      /^#lock_path =/alock_path = /var/lib/cinder/tmp
    }' \
    -i /etc/cinder/cinder.conf
```

## データベースの構築

データベースを構築する。

```sh
su -s /bin/sh -c "cinder-manage db sync" cinder
```

## Nova の設定

Compute Node を設定する。

```sh
sed \
    -e '/^\[cinder]/,/^\[/ {
      /^os_region_name =/d
      /^#os_region_name=/aos_region_name = RegionOne
    }' \
    -i /etc/nova/nova.conf
```

サービスを再起動する。

```sh
systemctl restart openstack-nova-compute
```

## 起動

マシン起動時の自動起動設定とサービスを起動する。

```sh
systemctl enable --now openstack-cinder-api
systemctl enable --now openstack-cinder-scheduler
```

## iSCSI ターゲットの作成

### インストール

iSCSI ターゲットをインストールする。

```sh
dnf install -y targetcli
```

### LVM ボリュームグループの作成

*/dev/sda* を物理ボリュームとして作成する。

```sh
pvcreate /dev/sda
```

```text
  Physical volume "/dev/sda" successfully created.
```

物理ボリュームから LVM ボリュームグループを作成する。

```sh
vgcreate cinder-volumes /dev/sda
```

```text
  Volume group "cinder-volumes" successfully created
```

LVM ボリュームグループを確認する。

```sh
vgdisplay cinder-volumes
```

```text
  --- Volume group ---
  VG Name               cinder-volumes
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <32.00 GiB
  PE Size               4.00 MiB
  Total PE              8191
  Alloc PE / Size       0 / 0
  Free  PE / Size       8191 / <32.00 GiB
  VG UUID               rvw5jA-EYOu-TJNl-kLsa-DyQP-m60z-46yLGS
```

### Cinder の設定

設定ファイルを更新する。

```sh
sed \
    -e '/^\[DEFAULT]/,/^\[/ {
      /^enabled_backends =/d
      /^#enabled_backends =/aenabled_backends = lvm
      /^glance_api_servers =/d
      /^#glance_api_servers =/aglance_api_servers = http://controller:9292
    }' \
    -i /etc/cinder/cinder.conf

sed \
    -e '/^\[lvm]/,/^\[/d' \
    -i /etc/cinder/cinder.conf

cat >> /etc/cinder/cinder.conf <<EOF

[lvm]
volume_driver = cinder.volume.drivers.lvm.LVMVolumeDriver
volume_group = cinder-volumes
target_protocol = iscsi
target_helper = lioadm
EOF
```

### ファイアウォールの設定

iSCSI ターゲットを許可する。

```sh
firewall-cmd --permanent --zone=internal --add-service=iscsi-target
firewall-cmd --reload
```

### 起動

cinder を再起動する。

```sh
systemctl restart openstack-cinder-api openstack-cinder-scheduler
```

マシン起動時の自動起動設定とサービスを起動する。

```sh
systemctl enable --now openstack-cinder-volume
```

Compute Node で iSCSI イニシエータを起動する。

```sh
systemctl enable --now iscsid
```

## 動作確認

サービスを表示する。

```sh
openstack volume service list
```

```text
+------------------+---------------------------+------+---------+-------+----------------------------+
| Binary           | Host                      | Zone | Status  | State | Updated At                 |
+------------------+---------------------------+------+---------+-------+----------------------------+
| cinder-scheduler | controller.home.local     | nova | enabled | up    | 2024-05-18T04:08:35.000000 |
| cinder-volume    | controller.home.local@lvm | nova | enabled | up    | 2024-05-18T04:08:41.000000 |
+------------------+---------------------------+------+---------+-------+----------------------------+
```
