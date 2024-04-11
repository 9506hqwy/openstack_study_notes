# OpenStack Image Service (Glance)

## データベースの作成

データベースに root でログインする。

```sh
mysql -u root -p59c6b803ff3fd0e68dcd
```

データベース keystone を作成する。

```sh
CREATE DATABASE glance;
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' IDENTIFIED BY '5ba24787139e1a1a86b8';
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' IDENTIFIED BY '5ba24787139e1a1a86b8';
```

## 事前準備


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
| id                  | 011104ed999f473db2e12b88780f0ad2 |
| name                | glance                           |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```

プロジェクト service にロール admin 権限でユーザ glance を追加する。

```sh
openstack role add --project service --user glance admin
```

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
| id          | caf161a6fcd94ce8b0cb9bb4e7fed12a |
| name        | glance                           |
| type        | image                            |
+-------------+----------------------------------+
```

API エンドポイントを作成する。

```sh
openstack endpoint create --region RegionOne image public http://controller:9292
```

```
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 25a548c384674472805ce5bd11bc48ee |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | caf161a6fcd94ce8b0cb9bb4e7fed12a |
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
| id           | 87a797eca4384721a79a858286db03e7 |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | caf161a6fcd94ce8b0cb9bb4e7fed12a |
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
| id           | a2ecfac484fb4b19bee90a5f68e24109 |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | caf161a6fcd94ce8b0cb9bb4e7fed12a |
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
curl -sSLO http://download.cirros-cloud.net/0.4.0/cirros-0.4.0-aarch64-disk.img
```

イメージを作成する。

```sh
glance image-create \
    --name "cirros" \
    --file cirros-0.4.0-aarch64-disk.img \
    --disk-format qcow2 \
    --container-format bare \
    --visibility=public
```

```
+------------------+----------------------------------------------------------------------------------+
| Property         | Value                                                                            |
+------------------+----------------------------------------------------------------------------------+
| checksum         | ecc9a5132e7a0f11a4c585f513cd0873                                                 |
| container_format | bare                                                                             |
| created_at       | 2024-04-09T11:55:29Z                                                             |
| disk_format      | qcow2                                                                            |
| id               | 479c1890-edb4-4b69-85c5-40dcef777f42                                             |
| min_disk         | 0                                                                                |
| min_ram          | 0                                                                                |
| name             | cirros                                                                           |
| os_hash_algo     | sha512                                                                           |
| os_hash_value    | 5e23f68b90d5c4763193e08ce149b24df32c53ad8b758b55ef42b045d865cf777edf3812b1219986 |
|                  | 47315c5e9a978bdfda42092148c3b8f85c3395e76d9c11bf                                 |
| os_hidden        | False                                                                            |
| owner            | cb2be036057d4225af1a33e3afb9f891                                                 |
| protected        | False                                                                            |
| size             | 15731712                                                                         |
| status           | active                                                                           |
| tags             | []                                                                               |
| updated_at       | 2024-04-09T11:55:30Z                                                             |
| virtual_size     | 113246208                                                                        |
| visibility       | public                                                                           |
+------------------+----------------------------------------------------------------------------------+
```

ファイルシステム上の格納していることを確認する。

```sh
ls -R /var/lib/glance/images/
```

```
/var/lib/glance/images/:
479c1890-edb4-4b69-85c5-40dcef777f42
```

イメージの登録を確認する。

```sh
openstack image list
```

```
+--------------------------------------+--------+--------+
| ID                                   | Name   | Status |
+--------------------------------------+--------+--------+
| 479c1890-edb4-4b69-85c5-40dcef777f42 | cirros | active |
+--------------------------------------+--------+--------+
```
