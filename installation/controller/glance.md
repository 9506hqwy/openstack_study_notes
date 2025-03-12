# Image (Glance)

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

```text
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | ba5fd9acb1ca4f5fbab866ad888f74c2 |
| name                | glance                           |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```

プロジェクト service でユーザ glance にロール admin 権限を追加する。

```sh
openstack role add --project service --user glance admin
```

## サービスの作成

image サービスを作成する。

```sh
openstack service create --name glance --description "Image" image
```

```text
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Image                            |
| enabled     | True                             |
| id          | 2b1559b0cef34d43800eca074985191d |
| name        | glance                           |
| type        | image                            |
+-------------+----------------------------------+
```

## エンドポイントの作成

API エンドポイントを作成する。

```sh
openstack endpoint create --region RegionOne image public http://controller:9292
```

```text
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 613c9f5e7008483fa148626b2def8ed3 |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 2b1559b0cef34d43800eca074985191d |
| service_name | glance                           |
| service_type | image                            |
| url          | http://controller:9292           |
+--------------+----------------------------------+
```

```sh
openstack endpoint create --region RegionOne image internal http://controller:9292
```

```text
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | c9c5b8bc019e42f38c9f01cfb5686193 |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 2b1559b0cef34d43800eca074985191d |
| service_name | glance                           |
| service_type | image                            |
| url          | http://controller:9292           |
+--------------+----------------------------------+
```

```sh
openstack endpoint create --region RegionOne image admin http://controller:9292
```

```text
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 496c44c7aed9485ba05ce1fb2e137118 |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 2b1559b0cef34d43800eca074985191d |
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
dnf install -y openstack-glance python3-glanceclient
```

## Glance の設定

設定ファイルを更新する。

```sh
sed \
    -e '/^\[DEFAULT]/,/^\[/ {
      /^enabled_backends =/d
      /^#enabled_backends =/aenabled_backends = fs:file
    }' \
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
      /^auth_url =/d
      /^\[keystone_authtoken]/aauth_url = http://controller:5000
      /^project_domain_name =/d
      /^\[keystone_authtoken]/aproject_domain_name = Default
      /^project_name =/d
      /^\[keystone_authtoken]/aproject_name = service
      /^user_domain_name =/d
      /^\[keystone_authtoken]/auser_domain_name = Default
      /^username =/d
      /^\[keystone_authtoken]/ausername = glance
      /^password =/d
      /^\[keystone_authtoken]/apassword = 7a69c4834de4f3ed2cfa
    }' \
    -e '/^\[paste_deploy]/,/^\[/ {
      /^flavor =/d
      /^#flavor =/aflavor = keystone
    }' \
    -e '/^\[glance_store]/,/^\[/ {
      /^default_backend =/d
      /^#default_backend =/adefault_backend = fs
    }' \
    -e '/^\[file]/,/^\[/ {
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

```{tip}
v0.4.0 は SSH 公開鍵をインポートしても公開鍵認証が機能しないため v0.6.2 を使用する。
```

```sh
curl -sSLO http://download.cirros-cloud.net/0.6.2/cirros-0.6.2-x86_64-disk.img
file cirros-0.6.2-x86_64-disk.img
```

```text
cirros-0.6.2-x86_64-disk.img: QEMU QCOW2 Image (v3), 117440512 bytes
```

イメージを作成する。

```sh
glance image-create \
    --name "cirros062" \
    --file cirros-0.6.2-x86_64-disk.img \
    --disk-format qcow2 \
    --container-format bare \
    --visibility=public
```

```text
+------------------+----------------------------------------------------------------------------------+
| Property         | Value                                                                            |
+------------------+----------------------------------------------------------------------------------+
| checksum         | c8fc807773e5354afe61636071771906                                                 |
| container_format | bare                                                                             |
| created_at       | 2024-05-10T15:53:21Z                                                             |
| disk_format      | qcow2                                                                            |
| id               | 18b72eca-3cf5-40fb-b4fa-441938b13964                                             |
| min_disk         | 0                                                                                |
| min_ram          | 0                                                                                |
| name             | cirros062                                                                        |
| os_hash_algo     | sha512                                                                           |
| os_hash_value    | 1103b92ce8ad966e41235a4de260deb791ff571670c0342666c8582fbb9caefe6af07ebb11d34f44 |
|                  | f8414b609b29c1bdf1d72ffa6faa39c88e8721d09847952b                                 |
| os_hidden        | False                                                                            |
| owner            | be94f4411bd74f249f5e25f642209b82                                                 |
| protected        | False                                                                            |
| size             | 21430272                                                                         |
| status           | active                                                                           |
| stores           | fs                                                                               |
| tags             | []                                                                               |
| updated_at       | 2024-05-10T15:53:21Z                                                             |
| virtual_size     | 117440512                                                                        |
| visibility       | public                                                                           |
+------------------+----------------------------------------------------------------------------------+
```

ファイルシステム上に格納していることを確認する。

```sh
ls -R /var/lib/glance/images/
```

```text
/var/lib/glance/images/:
18b72eca-3cf5-40fb-b4fa-441938b13964
```

virtio ビデオドライバをサポートしていないため vga に変更する。

```sh
openstack image set --property hw_video_model=vga cirros062
```

イメージの登録を確認する。

```sh
openstack image list
```

```text
+--------------------------------------+-----------+--------+
| ID                                   | Name      | Status |
+--------------------------------------+-----------+--------+
| 18b72eca-3cf5-40fb-b4fa-441938b13964 | cirros062 | active |
+--------------------------------------+-----------+--------+
```
