# Accelerator (Cyborg)

## インストール

```{tip}
Python 3.8 以降
```

### OS ユーザの作成

サービス cyborg 用のユーザを作成する。

```sh
useradd -r -M -s /sbin/nologin cyborg
```

### インストール

cyborg をインストールする。

```sh
dnf install -y gcc python3.12-devel

python3.12 -m venv /opt/cyborg
source /opt/cyborg/bin/activate

pip install --upgrade pip
pip install \
    openstack-cyborg \
    python-memcached \
    amqp

chgrp -R cyborg /opt/cyborg

mkdir -p /etc/cyborg
cp -r /opt/cyborg/etc/cyborg /etc
chgrp -R cyborg /etc/cyborg/*

mkdir -p /var/lib/cyborg
chown -R cyborg /var/lib/cyborg
chgrp -R cyborg /var/lib/cyborg
```

## cyborg の設定

設定ファイルを作成する。

```sh
cat > /etc/cyborg/cyborg.conf <<EOF
[DEFAULT]
transport_url = rabbit://openstack:3c17215fe69ba1dad320@controller:5672/
state_path = /var/lib/cyborg

[keystone_authtoken]
www_authenticate_uri = http://controller:5000
memcached_servers = localhost:11211
auth_type = password
auth_url = http://controller:5000
project_domain_name = Default
project_name = service
user_domain_name = Default
username = cyborg
password = 2927e8e7587d5ccaf836
EOF
```

```sh
chgrp cyborg /etc/cyborg/cyborg.conf
```

## systemd ユニットファイルの作成

*/usr/lib/systemd/system/openstack-cyborg-agent.service* を作成する。

```ini
[Unit]
Description=OpenStack Cyborg Agent service
After=network.target

[Service]
Type=simple
User=cyborg
ExecStart=/opt/cyborg/bin/cyborg-agent --config-file=/etc/cyborg/cyborg.conf
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

## 起動

マシン起動時の自動起動設定とサービスを起動する。

```sh
systemctl enable --now openstack-cyborg-agent
```

## 動作確認

Controller Node で確認する。

```sh
openstack resource provider list
```

nova のリソースプロバイダを親子関係が作成される。

```
+--------------------------------------+-------------------------------+------------+--------------------------------------+--------------------------------------+
| uuid                                 | name                          | generation | root_provider_uuid                   | parent_provider_uuid                 |
+--------------------------------------+-------------------------------+------------+--------------------------------------+--------------------------------------+
| 6c53c76c-39d1-45dc-a5f5-293e91abd3e4 | compute.home.local            |         23 | 6c53c76c-39d1-45dc-a5f5-293e91abd3e4 | None                                 |
| 96b3d1a8-1155-32f7-92ab-a82c671f5155 | compute_home_local_FakeDevice |          2 | 6c53c76c-39d1-45dc-a5f5-293e91abd3e4 | 6c53c76c-39d1-45dc-a5f5-293e91abd3e4 |
+--------------------------------------+-------------------------------+------------+--------------------------------------+--------------------------------------+
```

FPGA デバイスを確認する。

```sh
openstack resource provider inventory list 96b3d1a8-1155-32f7-92ab-a82c671f5155
```

```
+----------------+------------------+----------+----------+----------+-----------+-------+------+
| resource_class | allocation_ratio | min_unit | max_unit | reserved | step_size | total | used |
+----------------+------------------+----------+----------+----------+-----------+-------+------+
| FPGA           |              1.0 |        1 |       16 |        0 |         1 |    16 |    0 |
+----------------+------------------+----------+----------+----------+-----------+-------+------+
```

特性を表示する。

```sh
openstack resource provider trait list 96b3d1a8-1155-32f7-92ab-a82c671f5155
```

```
+--------------------+
| name               |
+--------------------+
| CUSTOM_FAKE_DEVICE |
+--------------------+
```

アクセラレータデバイスを確認する。

```sh
openstack accelerator device list
```

```
+--------------------------------------+------+--------+--------------------+------------------------------------------------+
| uuid                                 | type | vendor | hostname           | std_board_info                                 |
+--------------------------------------+------+--------+--------------------+------------------------------------------------+
| 1e7bd002-091a-415b-90eb-255a597c6bb4 | FPGA | 0xABCD | compute.home.local | {"device_id": "0xabcd", "class": "Fake class"} |
+--------------------------------------+------+--------+--------------------+------------------------------------------------+
```

```sh
openstack accelerator device show 1e7bd002-091a-415b-90eb-255a597c6bb4
```

```
+-------------------+------------------------------------------------+
| Field             | Value                                          |
+-------------------+------------------------------------------------+
| created_at        | 2024-05-18T16:11:18+00:00                      |
| updated_at        | None                                           |
| uuid              | 1e7bd002-091a-415b-90eb-255a597c6bb4           |
| type              | FPGA                                           |
| vendor            | 0xABCD                                         |
| model             | miss model info                                |
| hostname          | compute.home.local                             |
| std_board_info    | {"device_id": "0xabcd", "class": "Fake class"} |
| vendor_board_info | fake_vendor_info                               |
| status            |                                                |
+-------------------+------------------------------------------------+
```

割り当て可能なデバイスを確認する。

```sh
openstack accelerator deployable list
```

```
+--------------------------------------+-------------------------------+-----------+
| uuid                                 | name                          | device_id |
+--------------------------------------+-------------------------------+-----------+
| 94716f72-c4c5-44d6-b00c-ffdab9c69b16 | compute_home_local_FakeDevice |         1 |
+--------------------------------------+-------------------------------+-----------+
```

```sh
openstack accelerator deployable show 94716f72-c4c5-44d6-b00c-ffdab9c69b16
```

```
+------------+--------------------------------------+
| Field      | Value                                |
+------------+--------------------------------------+
| created_at | 2024-05-18T16:11:18+00:00            |
| updated_at | 2024-05-18T16:11:20+00:00            |
| uuid       | 94716f72-c4c5-44d6-b00c-ffdab9c69b16 |
| name       | compute_home_local_FakeDevice        |
+------------+--------------------------------------+
```

```{warning}
ログに以下が発生する。
RuntimeError: do not call blocking functions from the mainloop
```
