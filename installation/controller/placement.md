# OpenStack Placement Service

## データベースの作成

データベースに root でログインする。

```sh
mysql -u root -p59c6b803ff3fd0e68dcd
```

データベース placement を作成する。

```sh
CREATE DATABASE placement;
GRANT ALL PRIVILEGES ON placement.* TO 'placement'@'localhost' IDENTIFIED BY '251ccf2f93a6bd6ef9ee';
GRANT ALL PRIVILEGES ON placement.* TO 'placement'@'%' IDENTIFIED BY '251ccf2f93a6bd6ef9ee';
```

## ユーザの作成

ユーザ placement を作成する。

```sh
openstack user create --domain default --password b4b6936f87c338567d3c placement
```

```
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 025321ec417543aa88c52681d1b9dd5f |
| name                | placement                        |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```

プロジェクト service にロール admin 権限でユーザ placement を追加する。

```sh
openstack role add --project service --user placement admin
```

## サービスの作成

placement サービスを作成する。

```sh
openstack service create --name placement --description "Placement API" placement
```

```
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Placement API                    |
| enabled     | True                             |
| id          | 2993c33f14264beb991dfb17ec90a042 |
| name        | placement                        |
| type        | placement                        |
+-------------+----------------------------------+
```

## エンドポイントの作成

API エンドポイントを作成する。

```sh
openstack endpoint create --region RegionOne placement public http://controller:8778
```

```
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 6891c4866ad9439fb167bdd59daf57e0 |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 2993c33f14264beb991dfb17ec90a042 |
| service_name | placement                        |
| service_type | placement                        |
| url          | http://controller:8778           |
+--------------+----------------------------------+
```

```sh
openstack endpoint create --region RegionOne placement internal http://controller:8778
```

```
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | cf75b865f2d141b3ba1c7ed71922d944 |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 2993c33f14264beb991dfb17ec90a042 |
| service_name | placement                        |
| service_type | placement                        |
| url          | http://controller:8778           |
+--------------+----------------------------------+
```

```sh
openstack endpoint create --region RegionOne placement admin http://controller:8778
```

```
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 8ea9433a3d764f92a0b254532daa6326 |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 2993c33f14264beb991dfb17ec90a042 |
| service_name | placement                        |
| service_type | placement                        |
| url          | http://controller:8778           |
+--------------+----------------------------------+
```

ファイアウォールを開ける。

```sh
firewall-cmd --permanent --zone=internal --add-port=8778/tcp
firewall-cmd --reload
```

## インストール

placement をインストールする。

```sh
dnf install -y openstack-placement-api
```

## Placement の設定

設定ファイルを更新する。

```sh
sed \
    -e '/^\[placement_database]/,/^\[/ {
      /^connection =/d
      /^#connection =/aconnection = mysql+pymysql://placement:251ccf2f93a6bd6ef9ee@controller/placement
    }' \
    -e '/^\[api]/,/^\[/ {
      /^auth_strategy =/d
      /^#auth_strategy =/aauth_strategy = keystone
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
      /^\[keystone_authtoken]/ausername = placement
      /^password =/d
      /^\[keystone_authtoken]/apassword = b4b6936f87c338567d3c
    }' \
    -i /etc/placement/placement.conf
```

[Bug 1430540](https://bugzilla.redhat.com/show_bug.cgi?id=1430540) を追加する。

```sh
cat >> /etc/httpd/conf.d/00-placement-api.conf <<EOF
<Directory /usr/bin>
  <IfVersion >= 2.4>
    Require all granted
  </IfVersion>
  <IfVersion < 2.4>
    Order allow,deny
    Allow from all
  </IfVersion>
</Directory>
EOF
```

## データベースの構築

データベースを構築する。

```sh
su -s /bin/sh -c "placement-manage db sync" placement
```

## 起動

HTTP サーバを再起動する。

```sh
systemctl restart httpd
```

## 動作確認

動作確認する。

```sh
placement-status upgrade check
```

```
+----------------------------------------------------------------------+
| Upgrade Check Results                                                |
+----------------------------------------------------------------------+
| Check: Missing Root Provider IDs                                     |
| Result: Success                                                      |
| Details: None                                                        |
+----------------------------------------------------------------------+
| Check: Incomplete Consumers                                          |
| Result: Success                                                      |
| Details: None                                                        |
+----------------------------------------------------------------------+
| Check: Policy File JSON to YAML Migration                            |
| Result: Failure                                                      |
| Details: Your policy file is JSON-formatted which is deprecated. You |
|   need to switch to YAML-formatted file. Use the                     |
|   ``oslopolicy-convert-json-to-yaml`` tool to convert the            |
|   existing JSON-formatted files to YAML in a backwards-              |
|   compatible manner: https://docs.openstack.org/oslo.policy/         |
|   latest/cli/oslopolicy-convert-json-to-yaml.html.                   |
+----------------------------------------------------------------------+
```

```sh
dnf install -y python3-osc-placement
```

```sh
openstack --os-placement-api-version 1.2 resource class list --sort-column name
```

```
+----------------------------------------+
| name                                   |
+----------------------------------------+
| DISK_GB                                |
| FPGA                                   |
| IPV4_ADDRESS                           |
| MEMORY_MB                              |
| MEM_ENCRYPTION_CONTEXT                 |
| NET_BW_EGR_KILOBIT_PER_SEC             |
| NET_BW_IGR_KILOBIT_PER_SEC             |
| NET_PACKET_RATE_EGR_KILOPACKET_PER_SEC |
| NET_PACKET_RATE_IGR_KILOPACKET_PER_SEC |
| NET_PACKET_RATE_KILOPACKET_PER_SEC     |
| NUMA_CORE                              |
| NUMA_MEMORY_MB                         |
| NUMA_SOCKET                            |
| NUMA_THREAD                            |
| PCI_DEVICE                             |
| PCPU                                   |
| PGPU                                   |
| SRIOV_NET_VF                           |
| VCPU                                   |
| VGPU                                   |
| VGPU_DISPLAY_HEAD                      |
+----------------------------------------+
```

```sh
openstack --os-placement-api-version 1.6 trait list --sort-column name
```

```
+---------------------------------------+
| name                                  |
+---------------------------------------+
| COMPUTE_ACCELERATORS                  |
| COMPUTE_ARCH_AARCH64                  |
| COMPUTE_ARCH_MIPSEL                   |
| COMPUTE_ARCH_PPC64LE                  |
| COMPUTE_ARCH_RISCV64                  |
```
