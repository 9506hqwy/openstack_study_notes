# 登録

## ベアメタルノードの作成

ベアメタルノードを作成する。

| オプション                  | 説明                         |
| --------------------------- | ---------------------------- |
| --driver                    | ドライバ                     |

```sh
openstack baremetal node create --driver staging-libvirt
```

```
+------------------------+--------------------------------------+
| Field                  | Value                                |
+------------------------+--------------------------------------+
| chassis_uuid           | None                                 |
| clean_step             | {}                                   |
| console_enabled        | False                                |
| created_at             | 2024-04-30T23:22:23+00:00            |
| driver                 | staging-libvirt                      |
| driver_info            | {}                                   |
| driver_internal_info   | {}                                   |
| extra                  | {}                                   |
| inspection_finished_at | None                                 |
| inspection_started_at  | None                                 |
| instance_info          | {}                                   |
| instance_uuid          | None                                 |
| last_error             | None                                 |
| maintenance            | False                                |
| maintenance_reason     | None                                 |
| name                   | None                                 |
| power_state            | None                                 |
| properties             | {}                                   |
| provision_state        | enroll                               |
| provision_updated_at   | None                                 |
| reservation            | None                                 |
| target_power_state     | None                                 |
| target_provision_state | None                                 |
| updated_at             | None                                 |
| uuid                   | 803559b8-3a5b-4e80-b23d-7bd99fbbc5da |
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
    803559b8-3a5b-4e80-b23d-7bd99fbbc5da
```

UEFI で起動しなかったためファームウェアを BIOS に設定する。

```sh
openstack baremetal node set \
    803559b8-3a5b-4e80-b23d-7bd99fbbc5da \
    --property capabilities='boot_mode:bios'
```

ノードと MAC アドレスをもとにポートを作成する。

```sh
openstack baremetal port create 52:54:00:84:f8:75 \
    --node 803559b8-3a5b-4e80-b23d-7bd99fbbc5da
```

```
+-----------------------+--------------------------------------+
| Field                 | Value                                |
+-----------------------+--------------------------------------+
| address               | 52:54:00:84:f8:75                    |
| created_at            | 2024-04-30T23:25:54+00:00            |
| extra                 | {}                                   |
| internal_info         |                                      |
| is_smartnic           |                                      |
| local_link_connection |                                      |
| node_uuid             | 803559b8-3a5b-4e80-b23d-7bd99fbbc5da |
| physical_network      |                                      |
| portgroup_uuid        |                                      |
| pxe_enabled           |                                      |
| updated_at            | None                                 |
| uuid                  | 847b5449-1713-4512-b398-1d26cde26c56 |
+-----------------------+--------------------------------------+
```

## 設定の検証

ノードを設定を確認する。

```sh
openstack baremetal node validate 803559b8-3a5b-4e80-b23d-7bd99fbbc5da
```

`boot` と `deploy` はデプロイ時に設定されるので `False` で問題ない。

```
+------------+--------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Interface  | Result | Reason                                                                                                                                                                                                                                                    |
+------------+--------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| bios       | False  | Driver staging-libvirt does not support bios (disabled or not implemented).                                                                                                                                                                               |
| boot       | False  | Cannot validate image information for node 803559b8-3a5b-4e80-b23d-7bd99fbbc5da because one or more parameters are missing from its instance_info and insufficent information is present to boot from a remote volume. Missing are: ['kernel', 'ramdisk'] |
| console    | False  | Driver staging-libvirt does not support console (disabled or not implemented).                                                                                                                                                                            |
| deploy     | False  | Cannot validate image information for node 803559b8-3a5b-4e80-b23d-7bd99fbbc5da because one or more parameters are missing from its instance_info and insufficent information is present to boot from a remote volume. Missing are: ['kernel', 'ramdisk'] |
| inspect    | False  | Driver staging-libvirt does not support inspect (disabled or not implemented).                                                                                                                                                                            |
| management | True   |                                                                                                                                                                                                                                                           |
| network    | True   |                                                                                                                                                                                                                                                           |
| power      | True   |                                                                                                                                                                                                                                                           |
| raid       | False  | Driver staging-libvirt does not support raid (disabled or not implemented).                                                                                                                                                                               |
| rescue     | False  | Driver staging-libvirt does not support rescue (disabled or not implemented).                                                                                                                                                                             |
| storage    | True   |                                                                                                                                                                                                                                                           |
+------------+--------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```
