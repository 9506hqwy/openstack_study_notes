# 登録

## ベアメタルノードの作成

ベアメタルノードを作成する。

| オプション | 説明     |
| ---------- | -------- |
| --driver   | ドライバ |

```sh
openstack baremetal node create --driver staging-libvirt
```

```text
+------------------------+--------------------------------------+
| Field                  | Value                                |
+------------------------+--------------------------------------+
| allocation_uuid        | None                                 |
| automated_clean        | None                                 |
| bios_interface         | no-bios                              |
| boot_interface         | pxe                                  |
| boot_mode              | None                                 |
| chassis_uuid           | None                                 |
| clean_step             | {}                                   |
| conductor              | controller.home.local                |
| conductor_group        |                                      |
| console_enabled        | False                                |
| console_interface      | no-console                           |
| created_at             | 2024-05-19T01:54:01+00:00            |
| deploy_interface       | direct                               |
| deploy_step            | {}                                   |
| description            | None                                 |
| driver                 | staging-libvirt                      |
| driver_info            | {}                                   |
| driver_internal_info   | {}                                   |
| extra                  | {}                                   |
| fault                  | None                                 |
| inspect_interface      | no-inspect                           |
| inspection_finished_at | None                                 |
| inspection_started_at  | None                                 |
| instance_info          | {}                                   |
| instance_uuid          | None                                 |
| last_error             | None                                 |
| lessee                 | None                                 |
| maintenance            | False                                |
| maintenance_reason     | None                                 |
| management_interface   | staging-libvirt                      |
| name                   | None                                 |
| network_data           | {}                                   |
| network_interface      | flat                                 |
| owner                  | be94f4411bd74f249f5e25f642209b82     |
| power_interface        | staging-libvirt                      |
| power_state            | None                                 |
| properties             | {}                                   |
| protected              | False                                |
| protected_reason       | None                                 |
| provision_state        | enroll                               |
| provision_updated_at   | None                                 |
| raid_config            | {}                                   |
| raid_interface         | no-raid                              |
| rescue_interface       | no-rescue                            |
| reservation            | None                                 |
| resource_class         | None                                 |
| retired                | False                                |
| retired_reason         | None                                 |
| secure_boot            | None                                 |
| storage_interface      | noop                                 |
| target_power_state     | None                                 |
| target_provision_state | None                                 |
| target_raid_config     | {}                                   |
| traits                 | []                                   |
| updated_at             | None                                 |
| uuid                   | e95249d5-0605-498c-9120-706ef05ec240 |
| vendor_interface       | no-vendor                            |
+------------------------+--------------------------------------+
```

## ベアメタルノードの設定

ノードの情報を設定する。

```sh
openstack baremetal node set \
    --bios-interface no-bios \
    --boot-interface pxe \
    --console-interface no-console \
    --deploy-interface direct \
    --inspect-interface no-inspect \
    --raid-interface no-raid \
    --rescue-interface no-rescue \
    --vendor-interface no-vendor \
    --driver-info libvirt_uri=qemu+tcp://baremetal/system \
    --driver-info sasl_username=openstack \
    --driver-info sasl_password=4398982633ff4915310c \
    --resource-class baremetal \
    e95249d5-0605-498c-9120-706ef05ec240
```

UEFI で起動しなかったためファームウェアを BIOS に設定する。

```sh
openstack baremetal node set \
    e95249d5-0605-498c-9120-706ef05ec240 \
    --property capabilities='boot_mode:bios'
```

ノードと MAC アドレスをもとにポートを作成する。

```sh
openstack baremetal port create 52:54:00:7e:e6:01 \
    --node e95249d5-0605-498c-9120-706ef05ec240
```

```text
+-----------------------+--------------------------------------+
| Field                 | Value                                |
+-----------------------+--------------------------------------+
| address               | 52:54:00:7e:e6:01                    |
| created_at            | 2024-05-19T01:56:59+00:00            |
| extra                 | {}                                   |
| internal_info         | {}                                   |
| is_smartnic           | False                                |
| local_link_connection | {}                                   |
| node_uuid             | e95249d5-0605-498c-9120-706ef05ec240 |
| physical_network      | None                                 |
| portgroup_uuid        | None                                 |
| pxe_enabled           | True                                 |
| updated_at            | None                                 |
| uuid                  | e9cf3712-1f2d-44cd-8198-df65768c7cb6 |
+-----------------------+--------------------------------------+
```

## 設定の検証

ノードを設定を確認する。

```sh
openstack baremetal node validate e95249d5-0605-498c-9120-706ef05ec240
```

`deploy` はデプロイ時に設定されるので `False` で問題ない。

```text
+------------+--------+------------------------------------------------------------------------------------------------------------------------------------------------------+
| Interface  | Result | Reason                                                                                                                                               |
+------------+--------+------------------------------------------------------------------------------------------------------------------------------------------------------+
| bios       | False  | Driver staging-libvirt does not support bios (disabled or not implemented).                                                                          |
| boot       | True   |                                                                                                                                                      |
| console    | False  | Driver staging-libvirt does not support console (disabled or not implemented).                                                                       |
| deploy     | False  | Node e95249d5-0605-498c-9120-706ef05ec240 failed to validate deploy image info. Some parameters were missing. Missing are:                           |
|            |        | ['instance_info.image_source']                                                                                                                       |
| firmware   | False  | Driver staging-libvirt does not support firmware (disabled or not implemented).                                                                      |
| inspect    | False  | Driver staging-libvirt does not support inspect (disabled or not implemented).                                                                       |
| management | True   |                                                                                                                                                      |
| network    | True   |                                                                                                                                                      |
| power      | True   |                                                                                                                                                      |
| raid       | False  | Driver staging-libvirt does not support raid (disabled or not implemented).                                                                          |
| rescue     | False  | Driver staging-libvirt does not support rescue (disabled or not implemented).                                                                        |
| storage    | True   |                                                                                                                                                      |
+------------+--------+------------------------------------------------------------------------------------------------------------------------------------------------------+
```
