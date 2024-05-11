# デプロイ

## 前提条件

* ノードを[](./prepare) していること。
* [](../network/linuxbridge_flat) を作成していること。
* flavor [](../flavor/b1_nano) を作成していること。
* イメージ [](../../installation/controller/glance) でイメージを作成していること。
* セキュリティグループのルール [](../security_group/icmp) を作成していること。
* セキュリティグループのルール [](../security_group/ssh) を作成していること。

## ベアメタルインスタンスの作成

インスタンス bare00 を作成する。

```sh
openstack server create \
    --flavor b1.nano \
    --image cirros \
    --nic net-id=eda78c59-6bc0-4302-927f-3df79bdf57de \
    --security-group default \
    --key-name mykey \
    bare00
```

### イメージの配信

操作を実行すると HTTP サーバにイメージが配置される。


```sh
ls -l /httpboot/
```

```
合計 0
drwxr-xr-x. 2 ironic ironic 50  5月  3 09:09 agent_images
```

```
ls -l /httpboot/agent_images/
```

```
合計 0
lrwxrwxrwx. 1 ironic ironic 64  5月  3 09:09 803559b8-3a5b-4e80-b23d-7bd99fbbc5da -> /var/lib/ironic/images/803559b8-3a5b-4e80-b23d-7bd99fbbc5da/disk
```

```sh
ls -l /var/lib/ironic/images/803559b8-3a5b-4e80-b23d-7bd99fbbc5da/disk
```

```
-rw-r--r--. 2 ironic ironic 46137344  5月  2 21:01 /var/lib/ironic/images/803559b8-3a5b-4e80-b23d-7bd99fbbc5da/disk
```

TFTP を使用してデプロイイメージで PXE ブートして、 HTTP を使用してイメージを取得しディスクに書き込む。

![node deploy pxe boot](../../_static/image/node_cleaing_boot.png "node deploy pxe boot")

## ベアメタルノードの確認

`provision_state` が `active` になったことを確認する。

```sh
openstack baremetal node list
```

```
+--------------------------------------+--------+--------------------------------------+-------------+--------------------+-------------+
| UUID                                 | Name   | Instance UUID                        | Power State | Provisioning State | Maintenance |
+--------------------------------------+--------+--------------------------------------+-------------+--------------------+-------------+
| 803559b8-3a5b-4e80-b23d-7bd99fbbc5da | node01 | d950bb17-7b73-4d39-ab2c-7a52b4302f0a | power on    | active             | False       |
+--------------------------------------+--------+--------------------------------------+-------------+--------------------+-------------+
```


```sh
openstack baremetal node show 803559b8-3a5b-4e80-b23d-7bd99fbbc5da
```

```
+------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field                  | Value                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
+------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| allocation_uuid        | None                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| automated_clean        | None                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| bios_interface         | no-bios                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| boot_interface         | pxe                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| boot_mode              | None                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| chassis_uuid           | None                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| clean_step             | {}                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| conductor              | controller.home.local                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| conductor_group        |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| console_enabled        | False                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| console_interface      | no-console                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| created_at             | 2024-04-30T23:22:23+00:00                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| deploy_interface       | direct                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| deploy_step            | {}                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| description            | None                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| driver                 | staging-libvirt                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| driver_info            | {'libvirt_uri': 'qemu+tcp://baremetal/system', 'sasl_username': 'openstack', 'sasl_password': '******', 'image_download_source': ''}                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| driver_internal_info   | {'clean_steps': None, 'agent_erase_devices_iterations': 1, 'agent_erase_devices_zeroize': True, 'agent_continue_if_secure_erase_failed': False, 'agent_continue_if_ata_erase_failed': False, 'agent_enable_nvme_secure_erase': True, 'agent_enable_ata_secure_erase': True, 'disk_erasure_concurrency': 1, 'agent_erase_skip_read_only': False, 'last_power_state_change': '2024-05-03T00:31:01.047645', 'agent_version': '8.5.3.dev1', 'agent_last_heartbeat': '2024-05-03T00:30:42.535499', 'hardware_manager_version': {'generic_hardware_manager': '1.1'}, 'agent_cached_clean_steps_refreshed': '2024-05-02T12:20:02.727034', 'deploy_steps': None, 'is_whole_disk_image': True, 'agent_cached_deploy_steps_refreshed': '2024-05-03T00:30:29.479031', 'root_uuid_or_disk_id': '0x00000000'} |
| extra                  | {}                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| fault                  | None                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| inspect_interface      | no-inspect                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| inspection_finished_at | None                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| inspection_started_at  | None                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| instance_info          | {'image_source': 'e83903c4-7fa8-42a7-b693-f5034bc33603', 'root_gb': '10', 'swap_mb': '0', 'display_name': 'bare00', 'vcpus': '1', 'nova_host_id': 'controller.home.local', 'memory_mb': '1024', 'local_gb': '10', 'image_type': 'whole-disk', 'image_disk_format': 'raw', 'image_checksum': None, 'image_os_hash_algo': 'sha512', 'image_os_hash_value': 'b795f047a1b10ba0b7c95b43b2a481a59289dc4cf2e49845e60b194a911819d3ada03767bbba4143b44c93fd7f66c96c5a621e28dff51d1196dae64974ce240e', 'image_url': '******', 'image_container_format': 'bare', 'image_tags': [], 'image_properties': {'os_hidden': False, 'virtual_size': 46137344}}                                                                                                                                                      |
| instance_uuid          | d950bb17-7b73-4d39-ab2c-7a52b4302f0a                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| last_error             | None                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| lessee                 | None                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| maintenance            | False                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| maintenance_reason     | None                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| management_interface   | staging-libvirt                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| name                   | node01                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| network_data           | {}                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| network_interface      | flat                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| owner                  | None                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| power_interface        | staging-libvirt                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| power_state            | power on                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| properties             | {'capabilities': 'boot_mode:bios', 'memory_mb': 1024, 'local_gb': 10}                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| protected              | False                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| protected_reason       | None                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| provision_state        | active                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| provision_updated_at   | 2024-05-03T00:31:02+00:00                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| raid_config            | {}                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| raid_interface         | no-raid                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| rescue_interface       | no-rescue                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| reservation            | None                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| resource_class         | baremetal                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| retired                | False                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| retired_reason         | None                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| secure_boot            | None                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| storage_interface      | noop                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| target_power_state     | None                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| target_provision_state | None                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| target_raid_config     | {}                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| traits                 | []                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| updated_at             | 2024-05-03T00:31:02+00:00                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| uuid                   | 803559b8-3a5b-4e80-b23d-7bd99fbbc5da                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| vendor_interface       | no-vendor                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
+------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```

## ベアメタルインスタンスの確認

インスタンスが ACTIVE になったことを確認する。

```sh
openstack server list
```

```
+--------------------------------------+---------------------------+---------+-----------------------------------------+--------+---------+
| ID                                   | Name                      | Status  | Networks                                | Image  | Flavor  |
+--------------------------------------+---------------------------+---------+-----------------------------------------+--------+---------+
| d950bb17-7b73-4d39-ab2c-7a52b4302f0a | bare00                    | ACTIVE  | mgmt=10.0.0.208                         | cirros | b1.nano |
+--------------------------------------+---------------------------+---------+-----------------------------------------+--------+---------+
```

```sh
openstack server show d950bb17-7b73-4d39-ab2c-7a52b4302f0a
```

```
+-----------------------------+----------------------------------------------------------+
| Field                       | Value                                                    |
+-----------------------------+----------------------------------------------------------+
| OS-DCF:diskConfig           | MANUAL                                                   |
| OS-EXT-AZ:availability_zone | nova                                                     |
| OS-EXT-STS:power_state      | Running                                                  |
| OS-EXT-STS:task_state       | None                                                     |
| OS-EXT-STS:vm_state         | active                                                   |
| OS-SRV-USG:launched_at      | 2024-05-03T00:31:03.000000                               |
| OS-SRV-USG:terminated_at    | None                                                     |
| accessIPv4                  |                                                          |
| accessIPv6                  |                                                          |
| addresses                   | mgmt=10.0.0.208                                          |
| config_drive                |                                                          |
| created                     | 2024-05-03T00:09:28Z                                     |
| flavor                      | b1.nano (3ccda944-adcb-478c-b94a-21cd4d321905)           |
| hostId                      | c849c9e4cd816bf06df90145bc484c943b262376880be8f1656b06d2 |
| id                          | d950bb17-7b73-4d39-ab2c-7a52b4302f0a                     |
| image                       | cirros (e83903c4-7fa8-42a7-b693-f5034bc33603)            |
| key_name                    | mykey                                                    |
| name                        | bare00                                                   |
| progress                    | 0                                                        |
| project_id                  | f2aeffb34ff34ffb8959f1cd813655c6                         |
| properties                  |                                                          |
| security_groups             | name='default'                                           |
| status                      | ACTIVE                                                   |
| updated                     | 2024-05-03T00:31:04Z                                     |
| user_id                     | 71b5948c75f24c0f841dbf1c4eb4c4a7                         |
| volumes_attached            |                                                          |
+-----------------------------+----------------------------------------------------------+
```

## ポートの確認

ポートを確認する。

```sh
openstack port show 1c8a7fd2-c54b-46bc-83c8-1b6d4a3ccecb
```

```{note}
ベアメタルネットワークを設定していないため `binding_vif_type` は `binding_failed` になる。
https://docs.openstack.org/ironic/latest/install/configure-networking.html
```

```
+-------------------------+---------------------------------------------------------------------------+
| Field                   | Value                                                                     |
+-------------------------+---------------------------------------------------------------------------+
| admin_state_up          | UP                                                                        |
| allowed_address_pairs   |                                                                           |
| binding_host_id         | 803559b8-3a5b-4e80-b23d-7bd99fbbc5da                                      |
| binding_profile         |                                                                           |
| binding_vif_details     |                                                                           |
| binding_vif_type        | binding_failed                                                            |
| binding_vnic_type       | baremetal                                                                 |
| created_at              | 2024-05-03T00:09:34Z                                                      |
| data_plane_status       | None                                                                      |
| description             |                                                                           |
| device_id               | d950bb17-7b73-4d39-ab2c-7a52b4302f0a                                      |
| device_owner            | compute:nova                                                              |
| device_profile          | None                                                                      |
| dns_assignment          | None                                                                      |
| dns_domain              | None                                                                      |
| dns_name                | None                                                                      |
| extra_dhcp_opts         | ip_version='4', opt_name='server-ip-address', opt_value='10.0.0.11'       |
|                         | ip_version='4', opt_name='67', opt_value='pxelinux.0'                     |
|                         | ip_version='4', opt_name='210', opt_value='/tftpboot/'                    |
|                         | ip_version='4', opt_name='150', opt_value='10.0.0.11'                     |
|                         | ip_version='4', opt_name='66', opt_value='10.0.0.11'                      |
| fixed_ips               | ip_address='10.0.0.208', subnet_id='36b38628-6e3e-4741-8cc1-3261249e186a' |
| id                      | 1c8a7fd2-c54b-46bc-83c8-1b6d4a3ccecb                                      |
| ip_allocation           | None                                                                      |
| mac_address             | 52:54:00:84:f8:75                                                         |
| name                    |                                                                           |
| network_id              | eda78c59-6bc0-4302-927f-3df79bdf57de                                      |
| numa_affinity_policy    | None                                                                      |
| port_security_enabled   | True                                                                      |
| project_id              | f2aeffb34ff34ffb8959f1cd813655c6                                          |
| propagate_uplink_status | None                                                                      |
| qos_network_policy_id   | None                                                                      |
| qos_policy_id           | None                                                                      |
| resource_request        | None                                                                      |
| revision_number         | 13                                                                        |
| security_group_ids      | 87fd4685-d317-42fb-a487-28382d2c2750                                      |
| status                  | DOWN                                                                      |
| tags                    |                                                                           |
| trunk_details           | None                                                                      |
| updated_at              | 2024-05-03T00:31:00Z                                                      |
+-------------------------+---------------------------------------------------------------------------+
```

## 環境の確認

### dnsmasq

DHCP に MAC アドレスと IP アドレスの関連が追加される。

```sh
cat /var/lib/neutron/dhcp/eda78c59-6bc0-4302-927f-3df79bdf57de/host
```

```
52:54:00:84:f8:75,host-10-0-0-208.openstacklocal,10.0.0.208,set:port-1c8a7fd2-c54b-46bc-83c8-1b6d4a3ccecb
```

DNS のエントリが追加される。

```sh
cat /var/lib/neutron/dhcp/eda78c59-6bc0-4302-927f-3df79bdf57de/addn_hosts
```

```
10.0.0.208      host-10-0-0-208.openstacklocal host-10-0-0-208
```

### ネットワーク

イーサネットの情報を確認する。

```sh
ssh -i demo_rsa cirros@10.0.0.208 /sbin/ip addr show
```

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast qlen 1000
    link/ether 52:54:00:84:f8:75 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.208/24 brd 10.0.0.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:ff:fe84:f875/64 scope link
       valid_lft forever preferred_lft forever
```

ルーティングを確認する。

```sh
ssh -i demo_rsa cirros@10.0.0.208 /sbin/ip route
```

```
default via 10.0.0.1 dev eth0
10.0.0.0/24 dev eth0  src 10.0.0.208
169.254.169.254 via 10.0.0.200 dev eth0
```

ホスト名を確認する。

```sh
ssh -i demo_rsa cirros@10.0.0.208 hostname
```

```
bare00
```


