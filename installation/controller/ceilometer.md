# Telemetry Data Collection (Ceilometer)

## データベースの作成

データベースに root でログインする。

```sh
mysql -u root -p59c6b803ff3fd0e68dcd
```

データベース ceilometer を作成する。

```sh
CREATE DATABASE gnocchi;
GRANT ALL PRIVILEGES ON gnocchi.* TO 'gnocchi'@'localhost' IDENTIFIED BY '023e924d8a52c340024f';
GRANT ALL PRIVILEGES ON gnocchi.* TO 'gnocchi'@'%' IDENTIFIED BY '023e924d8a52c340024f';
```

## ユーザの作成

ユーザ ceilometer を作成する。

```sh
openstack user create --domain default --password eb5b4e528d0e10669cfa ceilometer
```

```
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 60dbaf021d1446e7983ff4a0b133497b |
| name                | ceilometer                       |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```

プロジェクト service でユーザ ceilometer にロール admin 権限を追加する。

```sh
openstack role add --project service --user ceilometer admin
```

ユーザ gnocchi を作成する。

```sh
openstack user create --domain default --password e760d03f9f03f676c9e2 gnocchi
```

```
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 7a0b967aff5348fab02238ff18b2b83e |
| name                | gnocchi                          |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```

プロジェクト service でユーザ gnocchi にロール admin 権限を追加する。

```sh
openstack role add --project service --user gnocchi admin
```

## サービスの作成

サービス metering を作成する。

```sh
openstack service create --name ceilometer --description "Telemetry Data Collection" metering
```

```
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Telemetry Data Collection        |
| enabled     | True                             |
| id          | 9a677e1d74604221bd969748c523bf33 |
| name        | ceilometer                       |
| type        | metering                         |
+-------------+----------------------------------+
```

サービス metric を作成する。

```sh
openstack service create --name gnocchi --description "OpenStack Metric Service" metric
```

```
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Metric Service         |
| enabled     | True                             |
| id          | 79d07753d398465f9a9208fbe9e83a17 |
| name        | gnocchi                          |
| type        | metric                           |
+-------------+----------------------------------+
```

## エンドポイントの作成

API エンドポイントを作成する。

```sh
openstack endpoint create --region RegionOne metric public http://controller:8041
```

```
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 5092ef2b13b346b79035e6136823eb3b |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 79d07753d398465f9a9208fbe9e83a17 |
| service_name | gnocchi                          |
| service_type | metric                           |
| url          | http://controller:8041           |
+--------------+----------------------------------+
```

```sh
openstack endpoint create --region RegionOne metric internal http://controller:8041
```

```
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 69b3b20cfeed4b00a3f0ce62128db80a |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 79d07753d398465f9a9208fbe9e83a17 |
| service_name | gnocchi                          |
| service_type | metric                           |
| url          | http://controller:8041           |
+--------------+----------------------------------+
```

```sh
openstack endpoint create --region RegionOne metric admin http://controller:8041
```

```
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 3822f677ae834309805c72507de4426d |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 79d07753d398465f9a9208fbe9e83a17 |
| service_name | gnocchi                          |
| service_type | metric                           |
| url          | http://controller:8041           |
+--------------+----------------------------------+
```

ファイアウォールを開ける。

```sh
firewall-cmd --permanent --zone=internal --add-port=8041/tcp
firewall-cmd --reload
```

## インストール

ceilometer をインストールする。

```sh
dnf install -y \
    openstack-gnocchi-api \
    openstack-gnocchi-metricd \
    python3-gnocchiclient \
    uwsgi-plugin-common \
    uwsgi-plugin-python3 \
    uwsgi \
    openstack-ceilometer-notification \
    openstack-ceilometer-central
```

## gnocchi の設定

設定ファイルを更新する。

```sh
sed \
    -e '/^\[api]/,/^\[/ {
      /^auth_mode =/d
      /^#auth_mode =/aauth_mode = keystone
      /^port =/d
      /^#port =/aport = 8041
      /^uwsgi_mode =/d
      /^#uwsgi_mode =/auwsgi_mode = http-socket
    }' \
    -e '/^\[indexer]/,/^\[/ {
      /^url =/d
      /^#url =/aurl = mysql+pymysql://gnocchi:023e924d8a52c340024f@controller/gnocchi
    }' \
    -e '/^\[storage]/,/^\[/ {
      /^driver =/d
      /^#driver =/adriver = file
      /^file_basepath =/d
      /^#file_basepath =/afile_basepath = /var/lib/gnocchi
    }' \
    -i /etc/gnocchi/gnocchi.conf

sed \
    -e '/^\[keystone_authtoken]/,/^\[/d' \
    -i /etc/gnocchi/gnocchi.conf

cat >> /etc/gnocchi/gnocchi.conf <<EOF

[keystone_authtoken]
www_authenticate_uri = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_name = Default
project_name = service
user_domain_name = Default
username = gnocchi
password = e760d03f9f03f676c9e2
EOF
```

保存先の権限を変更する。

```sh
chown -R gnocchi /var/lib/gnocchi
chgrp -R gnocchi /var/lib/gnocchi
restorecon -R /var/lib/gnocchi
```

## ceilometer の設定

設定ファイルを更新する。

```sh
sed \
    -e '/^\[DEFAULT]/,/^\[/ {
      /^transport_url =/d
      /^#transport_url =/atransport_url = rabbit://openstack:3c17215fe69ba1dad320@controller:5672/
    }' \
    -e '/^\[service_credentials]/,/^\[/ {
      /^auth_type =/d
      /^#auth_type =/aauth_type = password
      /^auth_url =/d
      /^#auth_url =/aauth_url = http://controller:5000
      /^project_domain_name =/d
      /^#project_domain_name =/aproject_domain_name = Default
      /^project_name =/d
      /^#project_name =/aproject_name = service
      /^user_domain_name =/d
      /^#user_domain_name =/auser_domain_name = Default
      /^username =/d
      /^#username =/ausername = ceilometer
      /^password =/d
      /^#password =/apassword = eb5b4e528d0e10669cfa
      /^region_name =/d
      /^#region_name =/aregion_name = RegionOne
      /^interface =/d
      /^#interface =/ainterface = internalURL
    }' \
    -i /etc/ceilometer/ceilometer.conf
```

## gnocchi のデータベースの構築

データベースを構築する。

```sh
gnocchi-upgrade
```

## gnocchi の起動

マシン起動時の自動起動設定とサービスを起動する。

```sh
systemctl enable --now openstack-gnocchi-api
systemctl enable --now openstack-gnocchi-metricd
```

## ceilometer のデータベースの構築

データベースを構築する。

```sh
ceilometer-upgrade
```

## ceilometer の起動

マシン起動時の自動起動設定とサービスを起動する。

```sh
systemctl enable --now openstack-ceilometer-notification
systemctl enable --now openstack-ceilometer-central
```

## cinder の設定

cinder に通知を設定する。

```sh
sed \
    -e '/^\[oslo_messaging_notifications]/,/^\[/ {
      /^driver =/d
      /^#driver =/adriver = messagingv2
    }' \
    -i /etc/cinder/cinder.conf
```

```sh
systemctl restart openstack-cinder-api openstack-cinder-scheduler
systemctl restart openstack-cinder-volume
```

## glance の設定

glance に通知を設定する。

```sh
sed \
    -e '/^\[DEFAULT]/,/^\[/ {
      /^transport_url =/d
      /^#transport_url =/atransport_url = rabbit://openstack:3c17215fe69ba1dad320@controller:5672/
    }' \
    -e '/^\[oslo_messaging_notifications]/,/^\[/ {
      /^driver =/d
      /^#driver =/adriver = messagingv2
    }' \
    -i /etc/glance/glance-api.conf
```

```sh
systemctl restart openstack-glance-api
```

## heat の設定

heat に通知を設定する。

```sh
sed \
    -e '/^\[oslo_messaging_notifications]/,/^\[/ {
      /^driver =/d
      /^#driver =/adriver = messagingv2
    }' \
    -i /etc/heat/heat.conf
```

```sh
systemctl restart openstack-heat-api openstack-heat-api-cfn openstack-heat-engine
```

## neutron の設定

neutron に通知を設定する。

```sh
sed \
    -e '/^\[oslo_messaging_notifications]/,/^\[/ {
      /^driver =/d
      /^#driver =/adriver = messagingv2
    }' \
    -i /etc/neutron/neutron.conf
```

```sh
systemctl restart neutron-server
```

## nova の設定

nova に通知を設定する。

### インストール

Controller Node (iconic) / Compute Node (nova) にインストールする。

```sh
dnf install -y openstack-ceilometer-compute
```

### ceilometer の設定

設定ファイルを更新する。

```sh
sed \
    -e '/^\[DEFAULT]/,/^\[/ {
      /^transport_url =/d
      /^#transport_url =/atransport_url = rabbit://openstack:3c17215fe69ba1dad320@controller:5672/
    }' \
    -e '/^\[service_credentials]/,/^\[/ {
      /^auth_type =/d
      /^#auth_type =/aauth_type = password
      /^auth_url =/d
      /^#auth_url =/aauth_url = http://controller:5000
      /^project_domain_name =/d
      /^#project_domain_name =/aproject_domain_name = Default
      /^project_name =/d
      /^#project_name =/aproject_name = service
      /^user_domain_name =/d
      /^#user_domain_name =/auser_domain_name = Default
      /^username =/d
      /^#username =/ausername = ceilometer
      /^password =/d
      /^#password =/apassword = eb5b4e528d0e10669cfa
      /^region_name =/d
      /^#region_name =/aregion_name = RegionOne
      /^interface =/d
      /^#interface =/ainterface = internalURL
    }' \
    -i /etc/ceilometer/ceilometer.conf
```

### nova の設定

設定ファイルを更新する。

```sh
sed \
    -e '/^\[DEFAULT]/,/^\[/ {
      /^instance_usage_audit_period=/d
      /^#instance_usage_audit_period=/ainstance_usage_audit_period=hour
      /^instance_usage_audit=/d
      /^#instance_usage_audit=/ainstance_usage_audit=true
    }' \
    -e '/^\[notifications]/,/^\[/ {
      /^notify_on_state_change=/d
      /^#notify_on_state_change=/anotify_on_state_change=vm_and_task_state
    }' \
    -e '/^\[oslo_messaging_notifications]/,/^\[/ {
      /^driver =/d
      /^#driver =/adriver = messagingv2
    }' \
    -i /etc/nova/nova.conf
```

### 起動

マシン起動時の自動起動設定とサービスを起動する。

```sh
systemctl enable --now openstack-ceilometer-compute
```

nova を再起動する。

```sh
systemctl restart openstack-nova-compute
```

## 動作確認

メトリクスの対象リソースを表示する。

```sh
openstack metric resource list
```

```
------------------+-----------------+------------------+------------------+----------------------+------------------+----------+------------------+--------------+-----------------------+
| id               | type            | project_id       | user_id          | original_resource_id | started_at       | ended_at | revision_start   | revision_end | creator               |
+------------------+-----------------+------------------+------------------+----------------------+------------------+----------+------------------+--------------+-----------------------+
| fb4470ac-e504-49 | volume          | bccf406c045d401b | 7f3acb28d26943ba | fb4470ac-e504-4959-9 | 2024-05-18T12:14 | None     | 2024-05-18T12:14 | None         | 60dbaf021d1446e7983ff |
| 59-939c-7f4b22fd |                 | 91ba5c7552a124ae | b9510df3a6edf3b0 | 39c-7f4b22fd07b4     | :40.365335+00:00 |          | :40.365342+00:00 |              | 4a0b133497b:bb6848524 |
| 07b4             |                 |                  |                  |                      |                  |          |                  |              | 614439b9b0f89f69ca98c |
|                  |                 |                  |                  |                      |                  |          |                  |              | 37                    |
| 612d7ce9-3030-59 | volume_provider | None             | None             | controller.home.loca | 2024-05-18T12:15 | None     | 2024-05-18T12:15 | None         | 60dbaf021d1446e7983ff |
| 96-b06f-dfc51b2d |                 |                  |                  | l@lvm                | :47.688311+00:00 |          | :47.688320+00:00 |              | 4a0b133497b:bb6848524 |
| 1f74             |                 |                  |                  |                      |                  |          |                  |              | 614439b9b0f89f69ca98c |
|                  |                 |                  |                  |                      |                  |          |                  |              | 37                    |
| a8fbb935-7f2b-49 | instance        | bccf406c045d401b | 7f3acb28d26943ba | a8fbb935-7f2b-49a5-9 | 2024-05-18T12:19 | None     | 2024-05-18T12:19 | None         | 60dbaf021d1446e7983ff |
| a5-983a-1f6e78a1 |                 | 91ba5c7552a124ae | b9510df3a6edf3b0 | 83a-1f6e78a1e0ed     | :30.527068+00:00 |          | :30.527078+00:00 |              | 4a0b133497b:bb6848524 |
| e0ed             |                 |                  |                  |                      |                  |          |                  |              | 614439b9b0f89f69ca98c |
|                  |                 |                  |                  |                      |                  |          |                  |              | 37                    |
+------------------+-----------------+------------------+------------------+----------------------+------------------+----------+------------------+--------------+-----------------------+
```

```sh
openstack metric resource show a8fbb935-7f2b-49a5-983a-1f6e78a1e0ed
```

```
+-----------------------+---------------------------------------------------------------------+
| Field                 | Value                                                               |
+-----------------------+---------------------------------------------------------------------+
| created_by_project_id | bb6848524614439b9b0f89f69ca98c37                                    |
| created_by_user_id    | 60dbaf021d1446e7983ff4a0b133497b                                    |
| creator               | 60dbaf021d1446e7983ff4a0b133497b:bb6848524614439b9b0f89f69ca98c37   |
| ended_at              | None                                                                |
| id                    | a8fbb935-7f2b-49a5-983a-1f6e78a1e0ed                                |
| metrics               | compute.instance.booting.time: 4f218cd6-aeb9-467b-9da3-f999a290e0c2 |
|                       | disk.ephemeral.size: b5d232ea-f477-4d82-a619-ddce96c15377           |
|                       | disk.root.size: f816b4d8-0ef4-423c-9260-9b4177b9199a                |
|                       | memory: 76ea9f63-fc18-4e14-b006-0e960e292d8a                        |
|                       | vcpus: 0fe132c9-32e7-4181-9a40-65cdb2736231                         |
| original_resource_id  | a8fbb935-7f2b-49a5-983a-1f6e78a1e0ed                                |
| project_id            | bccf406c045d401b91ba5c7552a124ae                                    |
| revision_end          | None                                                                |
| revision_start        | 2024-05-18T12:19:30.527078+00:00                                    |
| started_at            | 2024-05-18T12:19:30.527068+00:00                                    |
| type                  | instance                                                            |
| user_id               | 7f3acb28d26943bab9510df3a6edf3b0                                    |
+-----------------------+---------------------------------------------------------------------+
```

メトリクスの値を表示する。

```sh
openstack metric measures show 76ea9f63-fc18-4e14-b006-0e960e292d8a
```

```
+---------------------------+-------------+-------+
| timestamp                 | granularity | value |
+---------------------------+-------------+-------+
| 2024-05-18T23:00:00+09:00 |       300.0 | 256.0 |
+---------------------------+-------------+-------+
```
