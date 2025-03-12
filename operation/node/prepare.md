# 準備

## 管理状態に変更

ノードを管理状態にする。

```sh
openstack baremetal node manage e95249d5-0605-498c-9120-706ef05ec240
```

`provision_state` が `manageable` になったことを確認する。

```sh
openstack baremetal node show e95249d5-0605-498c-9120-706ef05ec240
```

```text
+------------------------+---------------------------------------------------------------------------------------------------------+
| Field                  | Value                                                                                                   |
+------------------------+---------------------------------------------------------------------------------------------------------+
| allocation_uuid        | None                                                                                                    |
| automated_clean        | None                                                                                                    |
| bios_interface         | no-bios                                                                                                 |
| boot_interface         | pxe                                                                                                     |
| boot_mode              | None                                                                                                    |
| chassis_uuid           | None                                                                                                    |
| clean_step             | {}                                                                                                      |
| conductor              | controller.home.local                                                                                   |
| conductor_group        |                                                                                                         |
| console_enabled        | False                                                                                                   |
| console_interface      | no-console                                                                                              |
| created_at             | 2024-05-19T01:54:01+00:00                                                                               |
| deploy_interface       | direct                                                                                                  |
| deploy_step            | {}                                                                                                      |
| description            | None                                                                                                    |
| driver                 | staging-libvirt                                                                                         |
| driver_info            | {'libvirt_uri': 'qemu+tcp://baremetal/system', 'sasl_username': 'openstack', 'sasl_password': '******'} |
| driver_internal_info   | {}                                                                                                      |
| extra                  | {}                                                                                                      |
| fault                  | None                                                                                                    |
| inspect_interface      | no-inspect                                                                                              |
| inspection_finished_at | None                                                                                                    |
| inspection_started_at  | None                                                                                                    |
| instance_info          | {}                                                                                                      |
| instance_uuid          | None                                                                                                    |
| last_error             | None                                                                                                    |
| lessee                 | None                                                                                                    |
| maintenance            | False                                                                                                   |
| maintenance_reason     | None                                                                                                    |
| management_interface   | staging-libvirt                                                                                         |
| name                   | None                                                                                                    |
| network_data           | {}                                                                                                      |
| network_interface      | flat                                                                                                    |
| owner                  | be94f4411bd74f249f5e25f642209b82                                                                        |
| power_interface        | staging-libvirt                                                                                         |
| power_state            | power off                                                                                               |
| properties             | {'capabilities': 'boot_mode:bios'}                                                                      |
| protected              | False                                                                                                   |
| protected_reason       | None                                                                                                    |
| provision_state        | manageable                                                                                              |
| provision_updated_at   | 2024-05-19T02:18:21+00:00                                                                               |
| raid_config            | {}                                                                                                      |
| raid_interface         | no-raid                                                                                                 |
| rescue_interface       | no-rescue                                                                                               |
| reservation            | None                                                                                                    |
| resource_class         | baremetal                                                                                               |
| retired                | False                                                                                                   |
| retired_reason         | None                                                                                                    |
| secure_boot            | None                                                                                                    |
| storage_interface      | noop                                                                                                    |
| target_power_state     | None                                                                                                    |
| target_provision_state | None                                                                                                    |
| target_raid_config     | {}                                                                                                      |
| traits                 | []                                                                                                      |
| updated_at             | 2024-05-19T02:18:21+00:00                                                                               |
| uuid                   | e95249d5-0605-498c-9120-706ef05ec240                                                                    |
| vendor_interface       | no-vendor                                                                                               |
+------------------------+---------------------------------------------------------------------------------------------------------+
```

## 利用可能状態に変更

ノードを利用可能状態にする。

```sh
openstack baremetal node provide e95249d5-0605-498c-9120-706ef05ec240
```

PXE ブートして cleaning が開始される。

```sh
openstack baremetal node list
```

```text
+--------------------------------------+------+---------------+-------------+--------------------+-------------+
| UUID                                 | Name | Instance UUID | Power State | Provisioning State | Maintenance |
+--------------------------------------+------+---------------+-------------+--------------------+-------------+
| e95249d5-0605-498c-9120-706ef05ec240 | None | None          | power on    | clean wait         | False       |
+--------------------------------------+------+---------------+-------------+--------------------+-------------+
```

![node cleaning pxe boot](../../_static/image/node_cleaing_boot.png "node cleaning pxe boot")

セキュア消去に対応していないので `shred` で消去する。

![node cleaning vda](../../_static/image/node_cleaing_vda.png "node cleaning vda")

cleaning が完了すると自動的にシャットダウンする。

```sh
openstack baremetal node list
```

```text
+--------------------------------------+------+---------------+-------------+--------------------+-------------+
| UUID                                 | Name | Instance UUID | Power State | Provisioning State | Maintenance |
+--------------------------------------+------+---------------+-------------+--------------------+-------------+
| e95249d5-0605-498c-9120-706ef05ec240 | None | None          | power off   | available          | False       |
+--------------------------------------+------+---------------+-------------+--------------------+-------------+
```

`provision_state` が `available` になったことを確認する。

```sh
openstack baremetal node show e95249d5-0605-498c-9120-706ef05ec240
```

```text
+------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field                  | Value                                                                                                                                                |
+------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------+
| allocation_uuid        | None                                                                                                                                                 |
| automated_clean        | None                                                                                                                                                 |
| bios_interface         | no-bios                                                                                                                                              |
| boot_interface         | pxe                                                                                                                                                  |
| boot_mode              | None                                                                                                                                                 |
| chassis_uuid           | None                                                                                                                                                 |
| clean_step             | {}                                                                                                                                                   |
| conductor              | controller.home.local                                                                                                                                |
| conductor_group        |                                                                                                                                                      |
| console_enabled        | False                                                                                                                                                |
| console_interface      | no-console                                                                                                                                           |
| created_at             | 2024-05-19T01:54:01+00:00                                                                                                                            |
| deploy_interface       | direct                                                                                                                                               |
| deploy_step            | {}                                                                                                                                                   |
| description            | None                                                                                                                                                 |
| driver                 | staging-libvirt                                                                                                                                      |
| driver_info            | {'libvirt_uri': 'qemu+tcp://baremetal/system', 'sasl_username': 'openstack', 'sasl_password': '******'}                                              |
| driver_internal_info   | {'clean_steps': None, 'agent_erase_devices_iterations': 1, 'agent_erase_devices_zeroize': True, 'agent_continue_if_secure_erase_failed': False,      |
|                        | 'agent_continue_if_ata_erase_failed': False, 'agent_enable_nvme_secure_erase': True, 'agent_enable_ata_secure_erase': True,                          |
|                        | 'disk_erasure_concurrency': 4, 'agent_erase_skip_read_only': False, 'last_power_state_change': '2024-05-19T02:51:29.952783', 'agent_version':        |
|                        | '9.11.1.dev1', 'agent_last_heartbeat': '2024-05-19T02:51:29.454640', 'hardware_manager_version': {'generic_hardware_manager': '1.2'},                |
|                        | 'agent_cached_clean_steps_refreshed': '2024-05-19T02:45:21.769614'}                                                                                  |
| extra                  | {}                                                                                                                                                   |
| fault                  | None                                                                                                                                                 |
| inspect_interface      | no-inspect                                                                                                                                           |
| inspection_finished_at | None                                                                                                                                                 |
| inspection_started_at  | None                                                                                                                                                 |
| instance_info          | {}                                                                                                                                                   |
| instance_uuid          | None                                                                                                                                                 |
| last_error             | None                                                                                                                                                 |
| lessee                 | None                                                                                                                                                 |
| maintenance            | False                                                                                                                                                |
| maintenance_reason     | None                                                                                                                                                 |
| management_interface   | staging-libvirt                                                                                                                                      |
| name                   | None                                                                                                                                                 |
| network_data           | {}                                                                                                                                                   |
| network_interface      | flat                                                                                                                                                 |
| owner                  | be94f4411bd74f249f5e25f642209b82                                                                                                                     |
| power_interface        | staging-libvirt                                                                                                                                      |
| power_state            | power off                                                                                                                                            |
| properties             | {'capabilities': 'boot_mode:bios'}                                                                                                                   |
| protected              | False                                                                                                                                                |
| protected_reason       | None                                                                                                                                                 |
| provision_state        | available                                                                                                                                            |
| provision_updated_at   | 2024-05-19T02:51:31+00:00                                                                                                                            |
| raid_config            | {}                                                                                                                                                   |
| raid_interface         | no-raid                                                                                                                                              |
| rescue_interface       | no-rescue                                                                                                                                            |
| reservation            | None                                                                                                                                                 |
| resource_class         | baremetal                                                                                                                                            |
| retired                | False                                                                                                                                                |
| retired_reason         | None                                                                                                                                                 |
| secure_boot            | None                                                                                                                                                 |
| storage_interface      | noop                                                                                                                                                 |
| target_power_state     | None                                                                                                                                                 |
| target_provision_state | None                                                                                                                                                 |
| target_raid_config     | {}                                                                                                                                                   |
| traits                 | []                                                                                                                                                   |
| updated_at             | 2024-05-19T02:51:32+00:00                                                                                                                            |
| uuid                   | e95249d5-0605-498c-9120-706ef05ec240                                                                                                                 |
| vendor_interface       | no-vendor                                                                                                                                            |
+------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------+
```

### cleaning の動作

デプロイイメージを使用してディスクを消去する。

操作を実行すると TFTP サーバに PXE ブートに必要なファイルが配置される。

```sh
ls -l /tftpboot
```

```text
合計 1224

(...)

lrwxrwxrwx  1 ironic ironic     43  5月 19 11:41 52:54:00:7e:e6:01.conf -> e95249d5-0605-498c-9120-706ef05ec240/config
drwxr-xr-x  2 ironic ironic     63  5月 19 11:41 e95249d5-0605-498c-9120-706ef05ec240
lrwxrwxrwx  1 ironic ironic     43  5月 19 11:41 grub.cfg-01-52-54-00-7e-e6-01 -> e95249d5-0605-498c-9120-706ef05ec240/config
-rw-r--r--. 1 ironic ironic  42720  8月  3  2022 pxelinux.0
```

```sh
ls -l /tftpboot/e95249d5-0605-498c-9120-706ef05ec240/
```

```text
合計 101412
-rw-r--r-- 1 ironic ironic      827  5月 19 11:41 config
-rw-r--r-- 2 ironic ironic  5924896  5月 19 11:41 deploy_kernel
-rw-r--r-- 2 ironic ironic 97912241  5月 19 11:41 deploy_ramdisk
```

```sh
cat /tftpboot/e95249d5-0605-498c-9120-706ef05ec240/config
```

```text
default deploy

label deploy
kernel e95249d5-0605-498c-9120-706ef05ec240/deploy_kernel
append initrd=e95249d5-0605-498c-9120-706ef05ec240/deploy_ramdisk selinux=0 troubleshoot=0 text nofb vga=normal ipa-debug=1 ipa-api-url=http://controller:6385 ipa-global-request-id=req-2d086240-25bd-4d5d-9624-c0592db89358
ipappend 2

label boot_whole_disk
COM32 chain.c32
append mbr:{{ DISK_IDENTIFIER }}

label boot_ramdisk
kernel no_kernel
append initrd=no_ramdisk root=/dev/ram0 text nofb vga=normal ipa-debug=1 ipa-api-url=http://controller:6385 ipa-global-request-id=req-2d086240-25bd-4d5d-9624-c0592db89358

label boot_anaconda
kernel no_kernel
append initrd=no_ramdisk text nofb vga=normal ipa-debug=1 ipa-api-url=http://controller:6385 ipa-global-request-id=req-2d086240-25bd-4d5d-9624-c0592db89358 inst.ks= inst.stage2=
ipappend 2
```
