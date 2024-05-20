# Orchestration (Heat)

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

プロジェクト service でユーザ heat にロール admin 権限を追加する。

```sh
openstack role add --project service --user heat admin
```

## サービスの作成

サービス orchestration を作成する。

```sh
openstack service create --name heat --description "Orchestration" orchestration
```

```
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Orchestration                    |
| enabled     | True                             |
| id          | 0f8a5236f4ad4b10a8c27e49ea4cdeef |
| name        | heat                             |
| type        | orchestration                    |
+-------------+----------------------------------+
```

サービス cloudformation を作成する。

```sh
openstack service create --name heat-cfn --description "Orchestration CFN" cloudformation
```

```
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Orchestration CFN                |
| enabled     | True                             |
| id          | 375ff7c307bd47d3acd7abc31c933ae6 |
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
| id           | 7b8338bb42444515bc843e06ac8e4218        |
| interface    | public                                  |
| region       | RegionOne                               |
| region_id    | RegionOne                               |
| service_id   | 0f8a5236f4ad4b10a8c27e49ea4cdeef        |
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
| id           | f8f16d25e8024555abe9de8498feffe1        |
| interface    | internal                                |
| region       | RegionOne                               |
| region_id    | RegionOne                               |
| service_id   | 0f8a5236f4ad4b10a8c27e49ea4cdeef        |
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
| id           | d55eaed796c943a19833ab9231ae083f        |
| interface    | admin                                   |
| region       | RegionOne                               |
| region_id    | RegionOne                               |
| service_id   | 0f8a5236f4ad4b10a8c27e49ea4cdeef        |
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
| id           | e4b883a6455d4376bf1f3413d08eda1f |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 375ff7c307bd47d3acd7abc31c933ae6 |
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
| id           | a2e188b5fe6b4debb1579227fa73689f |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 375ff7c307bd47d3acd7abc31c933ae6 |
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
| id           | 01683be8a4494b13be22020909672ce2 |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 375ff7c307bd47d3acd7abc31c933ae6 |
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
| id          | 9c4f6a35d5044e8484ec72eddcdfd575 |
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
| domain_id           | 9c4f6a35d5044e8484ec72eddcdfd575 |
| enabled             | True                             |
| id                  | eae88383f7ed444eb984b2ed1ea7ed6c |
| name                | heat_domain_admin                |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```

ドメイン heat でユーザ heat_domain_admin にロール admin 権限を追加する。

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
| id          | 9b5e41dd28b643cd87670646c26db497 |
| name        | heat_stack_owner                 |
| options     | {}                               |
+-------------+----------------------------------+
```

プロジェクト myproject でユーザ myuser にロール heat_stack_owner 権限を追加する。

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
| id          | f97f4d66e000446ca46f07f7bcb21ba9 |
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
      /^\[keystone_authtoken]/ausername = heat
      /^password =/d
      /^\[keystone_authtoken]/apassword = 2fac8b8307cabd60f3b8
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
| controller.home.local | heat-engine | ec7d98a1-62c2-45ce-bdfc-e0d3c461327a | controller.home.local | engine | 2024-05-18T02:49:16.000000 | up     |
| controller.home.local | heat-engine | 7d7b10b9-3585-4f10-b162-1261b80bab53 | controller.home.local | engine | 2024-05-18T02:49:16.000000 | up     |
| controller.home.local | heat-engine | c72cd696-53fe-4c5e-aece-462e5abf2136 | controller.home.local | engine | 2024-05-18T02:49:16.000000 | up     |
| controller.home.local | heat-engine | f741590f-0023-49ff-8f29-38c4c99d23bf | controller.home.local | engine | 2024-05-18T02:49:16.000000 | up     |
+-----------------------+-------------+--------------------------------------+-----------------------+--------+----------------------------+--------+
```
