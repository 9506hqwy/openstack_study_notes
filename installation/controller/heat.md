# OpenStack Orchestration Service (Heat)

## データベースの作成

データベースに root でログインする。

```sh
mysql -u root -p59c6b803ff3fd0e68dcd
```

データベース heat を作成する。

```sh
CREATE DATABASE heat;
GRANT ALL PRIVILEGES ON heat.* TO 'heat'@'localhost' IDENTIFIED BY '08e9d96fd510ddab2732';
GRANT ALL PRIVILEGES ON heat.* TO 'heat'@'%' IDENTIFIED BY '08e9d96fd510ddab2732';
```

## ユーザの作成

ユーザ heat を作成する。

```sh
openstack user create --domain default --password 2fac8b8307cabd60f3b8 heat
```

```
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | c06980b35417450ca97bfaac0c74b704 |
| name                | heat                             |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```

プロジェクト service にロール admin 権限でユーザ heat を追加する。

```sh
openstack role add --project service --user heat admin
```

## サービスの作成

サービス orchestration を作成する。

```sh
openstack service create --name heat --description "OpenStack Orchestration" orchestration
```

```
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Orchestration          |
| enabled     | True                             |
| id          | 89491554f1af418cab877eaec203b56d |
| name        | heat                             |
| type        | orchestration                    |
+-------------+----------------------------------+
```

サービス cloudformation を作成する。

```sh
openstack service create --name heat-cfn --description "OpenStack Orchestration" cloudformation
```

```
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Orchestration          |
| enabled     | True                             |
| id          | 81e32f4141d745c2b399502a5e854093 |
| name        | heat-cfn                         |
| type        | cloudformation                   |
+-------------+----------------------------------+
```

## エンドポイントの作成

サービス orchestration の API エンドポイントを作成する。

```sh
openstack endpoint create --region RegionOne orchestration public http://controller:8004/v1/%\(tenant_id\)s
```

```
+--------------+-----------------------------------------+
| Field        | Value                                   |
+--------------+-----------------------------------------+
| enabled      | True                                    |
| id           | a5228818885e423d97b8c9a0684204fe        |
| interface    | public                                  |
| region       | RegionOne                               |
| region_id    | RegionOne                               |
| service_id   | 89491554f1af418cab877eaec203b56d        |
| service_name | heat                                    |
| service_type | orchestration                           |
| url          | http://controller:8004/v1/%(tenant_id)s |
+--------------+-----------------------------------------+
```

```sh
openstack endpoint create --region RegionOne orchestration internal http://controller:8004/v1/%\(tenant_id\)s
```

```
+--------------+-----------------------------------------+
| Field        | Value                                   |
+--------------+-----------------------------------------+
| enabled      | True                                    |
| id           | 5b7c389a8e004d7a9e3258c38a9b36a7        |
| interface    | internal                                |
| region       | RegionOne                               |
| region_id    | RegionOne                               |
| service_id   | 89491554f1af418cab877eaec203b56d        |
| service_name | heat                                    |
| service_type | orchestration                           |
| url          | http://controller:8004/v1/%(tenant_id)s |
+--------------+-----------------------------------------+
```

```sh
openstack endpoint create --region RegionOne orchestration admin http://controller:8004/v1/%\(tenant_id\)s
```

```
+--------------+-----------------------------------------+
| Field        | Value                                   |
+--------------+-----------------------------------------+
| enabled      | True                                    |
| id           | 06c2e43d04b6466ebeb41f98dd2058c6        |
| interface    | admin                                   |
| region       | RegionOne                               |
| region_id    | RegionOne                               |
| service_id   | 89491554f1af418cab877eaec203b56d        |
| service_name | heat                                    |
| service_type | orchestration                           |
| url          | http://controller:8004/v1/%(tenant_id)s |
+--------------+-----------------------------------------+
```

ファイアウォールを開ける。

```sh
firewall-cmd --permanent --zone=internal --add-port=8004/tcp
firewall-cmd --reload
```

サービス cloudformation の API エンドポイントを作成する。

```sh
openstack endpoint create --region RegionOne cloudformation public http://controller:8000/v1
```

```
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | d9edb9f30e794f03a3af7f182ade7668 |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 81e32f4141d745c2b399502a5e854093 |
| service_name | heat-cfn                         |
| service_type | cloudformation                   |
| url          | http://controller:8000/v1        |
+--------------+----------------------------------+
```

```sh
openstack endpoint create --region RegionOne cloudformation internal http://controller:8000/v1
```

```
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | f3f7c38b110a4a2fa399db94b6c4f3bb |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 81e32f4141d745c2b399502a5e854093 |
| service_name | heat-cfn                         |
| service_type | cloudformation                   |
| url          | http://controller:8000/v1        |
+--------------+----------------------------------+
```

```sh
openstack endpoint create --region RegionOne cloudformation admin http://controller:8000/v1
```

```
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 67bd76a0dacd4450a68b961a4b9cc50e |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 81e32f4141d745c2b399502a5e854093 |
| service_name | heat-cfn                         |
| service_type | cloudformation                   |
| url          | http://controller:8000/v1        |
+--------------+----------------------------------+
```

ファイアウォールを開ける。

```sh
firewall-cmd --permanent --zone=internal --add-port=8000/tcp
firewall-cmd --reload
```

## スタック用のドメイン作成

ドメインを作成する。

```sh
openstack domain create --description "Stack projects and users" heat
```

```
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Stack projects and users         |
| enabled     | True                             |
| id          | 8a3ceeb0876a4378b058ce669e361d9a |
| name        | heat                             |
| options     | {}                               |
| tags        | []                               |
+-------------+----------------------------------+
```

ドメインの管理ユーザ heat_domain_admin を作成する。

```sh
openstack user create --domain heat --password 3d99ff8995969fbe113b heat_domain_admin
```

```
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | 8a3ceeb0876a4378b058ce669e361d9a |
| enabled             | True                             |
| id                  | 77fc8955fda94af38e13cbfd9511ce97 |
| name                | heat_domain_admin                |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```

ドメイン heat にロール admin 権限でユーザ heat_domain_admin を追加する。

```sh
openstack role add --domain heat --user-domain heat --user heat_domain_admin admin
```

スタック管理用ユーザ権限 heat_stack_owner を作成する。

```sh
openstack role create heat_stack_owner
```

```
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | None                             |
| domain_id   | None                             |
| id          | 80737431e09445b899b8825fb8a97685 |
| name        | heat_stack_owner                 |
| options     | {}                               |
+-------------+----------------------------------+
```

プロジェクト myproject にロール heat_stack_owner 権限でユーザ myuser を追加する。

```sh
openstack role add --project myproject --user myuser heat_stack_owner
```

スタックで作成するユーザに付与する権限 heat_stack_user を作成する。

```sh
openstack role create heat_stack_user
```

```
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | None                             |
| domain_id   | None                             |
| id          | b5194c5b757b4d919684a41f6bae95f8 |
| name        | heat_stack_user                  |
| options     | {}                               |
+-------------+----------------------------------+
```

## インストール

heat をインストールする。

```sh
dnf install -y \
    openstack-heat-api \
    openstack-heat-api-cfn \
    openstack-heat-engine \
    python3-heatclient
```

## heat の設定

設定ファイルを更新する。

```sh
sed \
    -e '/^\[DEFAULT]/,/^\[/ {
      /^transport_url =/d
      /^#transport_url =/atransport_url = rabbit://openstack:3c17215fe69ba1dad320@controller:5672/
      /^stack_user_domain_name =/d
      /^#stack_user_domain_name =/astack_user_domain_name = heat
      /^stack_domain_admin =/d
      /^#stack_domain_admin =/astack_domain_admin = heat_domain_admin
      /^stack_domain_admin_password =/d
      /^#stack_domain_admin_password =/astack_domain_admin_password = 3d99ff8995969fbe113b
      /^heat_metadata_server_url =/d
      /^#heat_metadata_server_url =/aheat_metadata_server_url = http://controller:8000
      /^heat_waitcondition_server_url =/d
      /^#heat_waitcondition_server_url =/aheat_waitcondition_server_url = http://controller:8000/v1/waitcondition
    }' \
    -e '/^\[database]/,/^\[/ {
      /^connection =/d
      /^#connection =/aconnection = mysql+pymysql://heat:08e9d96fd510ddab2732@controller/heat
    }' \
    -e '/^\[trustee]/,/^\[/ {
      /^auth_type =/d
      /^#auth_type =/aauth_type = password
      /^auth_url =/d
      /^#auth_url =/aauth_url = http://controller:5000
      /^username =/d
      /^#username =/ausername = heat
      /^user_domain_name =/d
      /^#user_domain_name =/auser_domain_name = Default
      /^password =/d
      /^#password =/apassword = 2fac8b8307cabd60f3b8
    }' \
    -e '/^\[clients_keystone]/,/^\[/ {
      /^auth_uri =/d
      /^#auth_uri =/aauth_uri = http://controller:5000
    }' \
    -i /etc/heat/heat.conf

sed \
    -e '/^\[keystone_authtoken]/,/^\[/d' \
    -i /etc/heat/heat.conf

cat >> /etc/heat/heat.conf <<EOF

[keystone_authtoken]
www_authenticate_uri = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_name = Default
project_name = service
user_domain_name = Default
username = heat
password = 2fac8b8307cabd60f3b8
EOF
```

## データベースの構築

データベースを構築する。

```sh
su -s /bin/sh -c "heat-manage db_sync" heat
```

## 起動

マシン起動時の自動起動設定とサービスを起動する。

```sh
systemctl enable --now openstack-heat-api
systemctl enable --now openstack-heat-api-cfn
systemctl enable --now openstack-heat-engine
```

## 動作確認

サービスを表示する。

```sh
openstack orchestration service list
```

```
+-----------------------+-------------+--------------------------------------+-----------------------+--------+----------------------------+--------+
| Hostname              | Binary      | Engine ID                            | Host                  | Topic  | Updated At                 | Status |
+-----------------------+-------------+--------------------------------------+-----------------------+--------+----------------------------+--------+
| controller.home.local | heat-engine | 2dc00945-3c46-45ac-907a-a24c65c78fc4 | controller.home.local | engine | 2024-04-28T02:11:35.000000 | up     |
| controller.home.local | heat-engine | 0893d2be-3c62-43a1-86cd-2291f21ac133 | controller.home.local | engine | 2024-04-28T02:11:35.000000 | up     |
| controller.home.local | heat-engine | 44c8d9f4-8634-4d6e-b307-57cbf6b75c1c | controller.home.local | engine | 2024-04-28T02:11:35.000000 | up     |
| controller.home.local | heat-engine | 14e8a72e-9fef-4b4c-9ae2-a15a0ee38127 | controller.home.local | engine | 2024-04-28T02:11:35.000000 | up     |
+-----------------------+-------------+--------------------------------------+-----------------------+--------+----------------------------+--------+
```
