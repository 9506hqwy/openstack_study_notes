# OpenStack Telemetry Data Collection Service (Ceilometer)

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
| id                  | 734cf77af2224faeb078f0f6f288e95b |
| name                | ceilometer                       |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```

プロジェクト service にロール admin 権限でユーザ ceilometer を追加する。

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
| id                  | 6dc05ca3821746b581ff68a81a2ce40d |
| name                | gnocchi                          |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```

プロジェクト service にロール admin 権限でユーザ gnocchi を追加する。

```sh
openstack role add --project service --user gnocchi admin
```

## サービスの作成

サービス metering を作成する。

```sh
openstack service create --name ceilometer --description "OpenStack Telemetry" metering
```

```
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Telemetry              |
| enabled     | True                             |
| id          | 81042649a15b42dba7119e7cf348b647 |
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
| id          | 653fe2117d7f44739905148c6949c0f9 |
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
| id           | 8dbe3d618c1941459b73fb447549882e |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 653fe2117d7f44739905148c6949c0f9 |
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
| id           | 8460448c9f3d4157b1becd0efecfb6fc |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 653fe2117d7f44739905148c6949c0f9 |
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
| id           | 2cb3afa742994b4da2ca762e1c1906e6 |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 653fe2117d7f44739905148c6949c0f9 |
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
+--------------------------------------+----------------------------+----------------------------------+----------------------------------+-----------------------------------------------------------------------+----------------------------------+----------+----------------------------------+--------------+-------------------------------------------------------------------+
| id                                   | type                       | project_id                       | user_id                          | original_resource_id                                                  | started_at                       | ended_at | revision_start                   | revision_end | creator                                                           |
+--------------------------------------+----------------------------+----------------------------------+----------------------------------+-----------------------------------------------------------------------+----------------------------------+----------+----------------------------------+--------------+-------------------------------------------------------------------+
| 2c6cc641-295d-5c93-88f9-979ed5816158 | volume_provider_pool       | None                             | None                             | controller.home.local@lvm#LVM                                         | 2024-05-04T00:53:47.422208+00:00 | None     | 2024-05-04T00:53:47.422215+00:00 | None         | 734cf77af2224faeb078f0f6f288e95b:39dd96b7af3a420096b892fabd45900a |
| a5aa1fc8-fcef-5098-9d43-0f808abbbc92 | volume_provider            | None                             | None                             | controller.home.local@lvm                                             | 2024-05-04T00:53:47.480495+00:00 | None     | 2024-05-04T00:53:47.480507+00:00 | None         | 734cf77af2224faeb078f0f6f288e95b:39dd96b7af3a420096b892fabd45900a |
| 45ee5234-e888-4ab8-91d1-d8d646a382ae | instance                   | f2aeffb34ff34ffb8959f1cd813655c6 | 71b5948c75f24c0f841dbf1c4eb4c4a7 | 45ee5234-e888-4ab8-91d1-d8d646a382ae                                  | 2024-05-04T01:15:56.975016+00:00 | None     | 2024-05-04T01:15:56.975080+00:00 | None         | 734cf77af2224faeb078f0f6f288e95b:39dd96b7af3a420096b892fabd45900a |
| 5aa817f3-b106-42b0-95d6-f2a8d1b6bea0 | instance                   | f2aeffb34ff34ffb8959f1cd813655c6 | 71b5948c75f24c0f841dbf1c4eb4c4a7 | 5aa817f3-b106-42b0-95d6-f2a8d1b6bea0                                  | 2024-05-04T01:15:57.241725+00:00 | None     | 2024-05-04T01:15:57.241733+00:00 | None         | 734cf77af2224faeb078f0f6f288e95b:39dd96b7af3a420096b892fabd45900a |
| 36b496b2-4976-4bf1-88c0-09f44385fd19 | instance                   | f2aeffb34ff34ffb8959f1cd813655c6 | 71b5948c75f24c0f841dbf1c4eb4c4a7 | 36b496b2-4976-4bf1-88c0-09f44385fd19                                  | 2024-05-04T01:15:57.816804+00:00 | None     | 2024-05-04T01:15:57.816810+00:00 | None         | 734cf77af2224faeb078f0f6f288e95b:39dd96b7af3a420096b892fabd45900a |
| ca9841dc-3768-4544-9d99-1a452ff99eff | instance                   | f2aeffb34ff34ffb8959f1cd813655c6 | 71b5948c75f24c0f841dbf1c4eb4c4a7 | ca9841dc-3768-4544-9d99-1a452ff99eff                                  | 2024-05-04T01:15:58.053447+00:00 | None     | 2024-05-04T01:15:58.053453+00:00 | None         | 734cf77af2224faeb078f0f6f288e95b:39dd96b7af3a420096b892fabd45900a |
| 97b95570-f21d-4ea0-a2f9-0c1541f87d96 | instance                   | f2aeffb34ff34ffb8959f1cd813655c6 | 71b5948c75f24c0f841dbf1c4eb4c4a7 | 97b95570-f21d-4ea0-a2f9-0c1541f87d96                                  | 2024-05-04T01:15:58.056011+00:00 | None     | 2024-05-04T01:15:58.056020+00:00 | None         | 734cf77af2224faeb078f0f6f288e95b:39dd96b7af3a420096b892fabd45900a |
| fd18af2f-9ff2-5f00-9546-2fb0149c4908 | instance_disk              | f2aeffb34ff34ffb8959f1cd813655c6 | 71b5948c75f24c0f841dbf1c4eb4c4a7 | 36b496b2-4976-4bf1-88c0-09f44385fd19-vda                              | 2024-05-04T01:19:52.964157+00:00 | None     | 2024-05-04T01:19:52.964163+00:00 | None         | 734cf77af2224faeb078f0f6f288e95b:39dd96b7af3a420096b892fabd45900a |
| a564ecb0-1ea0-5e74-812d-749242f06834 | instance_network_interface | f2aeffb34ff34ffb8959f1cd813655c6 | 71b5948c75f24c0f841dbf1c4eb4c4a7 | instance-00000029-36b496b2-4976-4bf1-88c0-09f44385fd19-tap8811e933-5c | 2024-05-04T01:19:53.161558+00:00 | None     | 2024-05-04T01:19:53.161564+00:00 | None         | 734cf77af2224faeb078f0f6f288e95b:39dd96b7af3a420096b892fabd45900a |
+--------------------------------------+----------------------------+----------------------------------+----------------------------------+-----------------------------------------------------------------------+----------------------------------+----------+----------------------------------+--------------+-------------------------------------------------------------------+
```

```sh
openstack metric resource show 36b496b2-4976-4bf1-88c0-09f44385fd19
```

```
+-----------------------+-------------------------------------------------------------------+
| Field                 | Value                                                             |
+-----------------------+-------------------------------------------------------------------+
| created_by_project_id | 39dd96b7af3a420096b892fabd45900a                                  |
| created_by_user_id    | 734cf77af2224faeb078f0f6f288e95b                                  |
| creator               | 734cf77af2224faeb078f0f6f288e95b:39dd96b7af3a420096b892fabd45900a |
| ended_at              | None                                                              |
| id                    | 36b496b2-4976-4bf1-88c0-09f44385fd19                              |
| metrics               | cpu: 2b63ad10-0ef3-48ec-97e6-57656a691595                         |
|                       | disk.ephemeral.size: f5232beb-0cb2-4a9a-bd63-12111cc4ef6e         |
|                       | disk.root.size: c6120ff7-875a-4620-8436-550c7247db10              |
|                       | memory.usage: 42f1714f-cdd8-4bd5-a0ee-53707fb57345                |
|                       | memory: d1396e85-e41e-4d04-b3e5-03d4f31362d5                      |
|                       | vcpus: 326660e5-501b-4993-8008-fa20c0c292b0                       |
| original_resource_id  | 36b496b2-4976-4bf1-88c0-09f44385fd19                              |
| project_id            | f2aeffb34ff34ffb8959f1cd813655c6                                  |
| revision_end          | None                                                              |
| revision_start        | 2024-05-04T01:15:57.816810+00:00                                  |
| started_at            | 2024-05-04T01:15:57.816804+00:00                                  |
| type                  | instance                                                          |
| user_id               | 71b5948c75f24c0f841dbf1c4eb4c4a7                                  |
+-----------------------+-------------------------------------------------------------------+
```

メトリクスの値を表示する。

```sh
openstack metric measures show 2b63ad10-0ef3-48ec-97e6-57656a691595
```

```
+---------------------------+-------------+---------------+
| timestamp                 | granularity |         value |
+---------------------------+-------------+---------------+
| 2024-05-04T11:35:00+09:00 |       300.0 | 47200000000.0 |
+---------------------------+-------------+---------------+
```
