# OpenStack Image Service (Glance)

## データベースの作成

データベースに root でログインする。

```sh
mysql -u root -p59c6b803ff3fd0e68dcd
```

データベース glance を作成する。

```sh
CREATE DATABASE glance;
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' IDENTIFIED BY '5ba24787139e1a1a86b8';
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' IDENTIFIED BY '5ba24787139e1a1a86b8';
```

## ユーザの作成

ユーザ glance を作成する。

```sh
openstack user create --domain default --password 7a69c4834de4f3ed2cfa glance
```

```
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 79c03c2f523b479ab98565add8c72552 |
| name                | glance                           |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```

プロジェクト service にロール admin 権限でユーザ glance を追加する。

```sh
openstack role add --project service --user glance admin
```

## サービスの作成

image サービスを作成する。

```sh
openstack service create --name glance --description "OpenStack Image" image
```

```
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Image                  |
| enabled     | True                             |
| id          | 1207733aef284c37a3d4e26952802528 |
| name        | glance                           |
| type        | image                            |
+-------------+----------------------------------+
```

## エンドポイントの作成

API エンドポイントを作成する。

```sh
openstack endpoint create --region RegionOne image public http://controller:9292
```

```
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | e11c6bc806fa46878cef909b83f681bc |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 1207733aef284c37a3d4e26952802528 |
| service_name | glance                           |
| service_type | image                            |
| url          | http://controller:9292           |
+--------------+----------------------------------+
```

```sh
openstack endpoint create --region RegionOne image internal http://controller:9292
```

```
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | f991841602d84c248f5c81b20cf233c5 |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 1207733aef284c37a3d4e26952802528 |
| service_name | glance                           |
| service_type | image                            |
| url          | http://controller:9292           |
+--------------+----------------------------------+
```

```sh
openstack endpoint create --region RegionOne image admin http://controller:9292
```

```
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | e81ca38e2fa5474597b26f8b6863ad48 |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 1207733aef284c37a3d4e26952802528 |
| service_name | glance                           |
| service_type | image                            |
| url          | http://controller:9292           |
+--------------+----------------------------------+
```

ファイアウォールを開ける。

```sh
firewall-cmd --permanent --zone=internal --add-port=9292/tcp
firewall-cmd --reload
```

## インストール

glance をインストールする。

```sh
dnf install -y openstack-glance
```

## Glance の設定

設定ファイルを更新する。

```sh
sed \
    -e '/^\[database]/,/^\[/ {
      /^connection =/d
      /^#connection =/aconnection = mysql+pymysql://glance:5ba24787139e1a1a86b8@controller/glance
    }' \
    -e '/^\[keystone_authtoken]/,/^\[/ {
      /^www_authenticate_uri =/d
      /^#www_authenticate_uri =/awww_authenticate_uri = http://controller:5000
      /^memcached_servers =/d
      /^#memcached_servers =/amemcached_servers = controller:11211
      /^auth_type =/d
      /^#auth_type =/aauth_type = password
      /^auth_url=/d
      /^\[keystone_authtoken]/aauth_url = http://controller:5000
      /^project_domain_name=/d
      /^\[keystone_authtoken]/aproject_domain_name = Default
      /^project_name=/d
      /^\[keystone_authtoken]/aproject_name = service
      /^user_domain_name=/d
      /^\[keystone_authtoken]/auser_domain_name = Default
      /^username=/d
      /^\[keystone_authtoken]/ausername = glance
      /^password=/d
      /^\[keystone_authtoken]/apassword = 7a69c4834de4f3ed2cfa
    }' \
    -e '/^\[paste_deploy]/,/^\[/ {
      /^flavor =/d
      /^#flavor =/aflavor = keystone
    }' \
    -e '/^\[glance_store]/,/^\[/ {
      /^stores =/d
      /^#stores =/astores = file,http
      /^default_store =/d
      /^#default_store =/adefault_store = file
      /^filesystem_store_datadir =/d
      /^#filesystem_store_datadir =/afilesystem_store_datadir = /var/lib/glance/images
    }' \
    -i /etc/glance/glance-api.conf
```

## データベースの構築

データベースを構築する。

```sh
su -s /bin/sh -c "glance-manage db_sync" glance
```

## 起動

マシン起動時の自動起動設定とサービスを起動する。

```sh
systemctl enable --now openstack-glance-api
```

## イメージの登録

cirros をダウンロードする。

```sh
curl -sSLO http://download.cirros-cloud.net/0.4.0/cirros-0.4.0-x86_64-disk.img
file cirros-0.4.0-x86_64-disk.img
```

```
cirros-0.4.0-x86_64-disk.img: QEMU QCOW Image (v3), 46137344 bytes
```

イメージを作成する。

```sh
glance image-create \
    --name "cirros" \
    --file cirros-0.4.0-x86_64-disk.img \
    --disk-format qcow2 \
    --container-format bare \
    --visibility=public
```

```
+------------------+----------------------------------------------------------------------------------+
| Property         | Value                                                                            |
+------------------+----------------------------------------------------------------------------------+
| checksum         | 443b7623e27ecf03dc9e01ee93f67afe                                                 |
| container_format | bare                                                                             |
| created_at       | 2024-04-13T05:38:27Z                                                             |
| disk_format      | qcow2                                                                            |
| id               | e83903c4-7fa8-42a7-b693-f5034bc33603                                             |
| min_disk         | 0                                                                                |
| min_ram          | 0                                                                                |
| name             | cirros                                                                           |
| os_hash_algo     | sha512                                                                           |
| os_hash_value    | 6513f21e44aa3da349f248188a44bc304a3653a04122d8fb4535423c8e1d14cd6a153f735bb0982e |
|                  | 2161b5b5186106570c17a9e58b64dd39390617cd5a350f78                                 |
| os_hidden        | False                                                                            |
| owner            | 1e3ac7ae10e24515a0956beaa1d8073c                                                 |
| protected        | False                                                                            |
| size             | 12716032                                                                         |
| status           | active                                                                           |
| tags             | []                                                                               |
| updated_at       | 2024-04-13T05:38:28Z                                                             |
| virtual_size     | 46137344                                                                         |
| visibility       | public                                                                           |
+------------------+----------------------------------------------------------------------------------+
```

ファイルシステム上に格納していることを確認する。

```sh
ls -R /var/lib/glance/images/
```

```
/var/lib/glance/images/:
e83903c4-7fa8-42a7-b693-f5034bc33603
```

イメージの登録を確認する。

```sh
openstack image list
```

```
+--------------------------------------+--------+--------+
| ID                                   | Name   | Status |
+--------------------------------------+--------+--------+
| e83903c4-7fa8-42a7-b693-f5034bc33603 | cirros | active |
+--------------------------------------+--------+--------+
```
