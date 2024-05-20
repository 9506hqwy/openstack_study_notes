# インスタンスの作成 (FPGA)

FPGA を要求するインスタンスを作成する。

## 前提条件

* [](../network/ovs_flat) を作成していること。
* flavor [](../flavor/a1_milli) を作成していること。
* イメージ [](../../installation/controller/glance) でイメージを作成していること。
* [](../sshkey/keypair.md) を作成していること。
* セキュリティグループのルール [](../security_group/icmp) を作成していること。
* セキュリティグループのルール [](../security_group/ssh) を作成していること。

## インスタンスの作成

```{tip}
myuser で実行
```

インスタンス instance04 を作成する。

```sh
openstack server create \
    --flavor a1.milli \
    --image cirros062 \
    --nic net-id=53f92676-e6e0-42fb-bde9-48dd2c2506b4 \
    --security-group mysecurity \
    --key-name mykey \
    instance04
```

```
+--------------------------------------+--------------------------------------------------+
| Field                                | Value                                            |
+--------------------------------------+--------------------------------------------------+
| OS-DCF:diskConfig                    | MANUAL                                           |
| OS-EXT-AZ:availability_zone          |                                                  |
| OS-EXT-SRV-ATTR:host                 | None                                             |
| OS-EXT-SRV-ATTR:hypervisor_hostname  | None                                             |
| OS-EXT-SRV-ATTR:instance_name        |                                                  |
| OS-EXT-STS:power_state               | NOSTATE                                          |
| OS-EXT-STS:task_state                | scheduling                                       |
| OS-EXT-STS:vm_state                  | building                                         |
| OS-SRV-USG:launched_at               | None                                             |
| OS-SRV-USG:terminated_at             | None                                             |
| accessIPv4                           |                                                  |
| accessIPv6                           |                                                  |
| addresses                            |                                                  |
| adminPass                            | JMvo83HSVAXx                                     |
| config_drive                         |                                                  |
| created                              | 2024-05-18T16:32:13Z                             |
| flavor                               | a1.milli (82979736-dea6-4e0f-a155-bcfbda0afd49)  |
| hostId                               |                                                  |
| id                                   | c44894d3-433b-451c-9f41-b968e372ebf1             |
| image                                | cirros062 (6793c9b2-7cb6-4796-b477-9e22d985ea2b) |
| key_name                             | mykey                                            |
| name                                 | instance04                                       |
| os-extended-volumes:volumes_attached | []                                               |
| progress                             | 0                                                |
| project_id                           | bccf406c045d401b91ba5c7552a124ae                 |
| properties                           |                                                  |
| security_groups                      | name='a81e0cba-8806-40aa-bde8-c030df1545c9'      |
| status                               | BUILD                                            |
| updated                              | 2024-05-18T16:32:13Z                             |
| user_id                              | 7f3acb28d26943bab9510df3a6edf3b0                 |
+--------------------------------------+--------------------------------------------------+
```

## インスタンスの確認

インスタンスが ACTIVE になったことを確認する。

```sh
openstack server list
```

```
+--------------------------------------+------------+--------+-----------------------+-----------+----------+
| ID                                   | Name       | Status | Networks              | Image     | Flavor   |
+--------------------------------------+------------+--------+-----------------------+-----------+----------+
| c44894d3-433b-451c-9f41-b968e372ebf1 | instance04 | ACTIVE | provider=172.16.0.110 | cirros062 | a1.milli |
+--------------------------------------+------------+--------+-----------------------+-----------+----------+
```

```sh
openstack server show c44894d3-433b-451c-9f41-b968e372ebf1
```

```
+-------------------------------------+-------------------------------------------------------------------------------------------------------------------------------------+
| Field                               | Value                                                                                                                               |
+-------------------------------------+-------------------------------------------------------------------------------------------------------------------------------------+
| OS-DCF:diskConfig                   | MANUAL                                                                                                                              |
| OS-EXT-AZ:availability_zone         | nova                                                                                                                                |
| OS-EXT-SRV-ATTR:host                | compute.home.local                                                                                                                  |
| OS-EXT-SRV-ATTR:hostname            | instance04                                                                                                                          |
| OS-EXT-SRV-ATTR:hypervisor_hostname | compute.home.local                                                                                                                  |
| OS-EXT-SRV-ATTR:instance_name       | instance-0000000b                                                                                                                   |
| OS-EXT-SRV-ATTR:kernel_id           |                                                                                                                                     |
| OS-EXT-SRV-ATTR:launch_index        | 0                                                                                                                                   |
| OS-EXT-SRV-ATTR:ramdisk_id          |                                                                                                                                     |
| OS-EXT-SRV-ATTR:reservation_id      | r-jp0j58b3                                                                                                                          |
| OS-EXT-SRV-ATTR:root_device_name    | /dev/vda                                                                                                                            |
| OS-EXT-SRV-ATTR:user_data           | None                                                                                                                                |
| OS-EXT-STS:power_state              | Running                                                                                                                             |
| OS-EXT-STS:task_state               | None                                                                                                                                |
| OS-EXT-STS:vm_state                 | active                                                                                                                              |
| OS-SRV-USG:launched_at              | 2024-05-18T16:32:20.000000                                                                                                          |
| OS-SRV-USG:terminated_at            | None                                                                                                                                |
| accessIPv4                          |                                                                                                                                     |
| accessIPv6                          |                                                                                                                                     |
| addresses                           | provider=172.16.0.110                                                                                                               |
| config_drive                        |                                                                                                                                     |
| created                             | 2024-05-18T16:32:13Z                                                                                                                |
| description                         | instance04                                                                                                                          |
| flavor                              | description=, disk='1', ephemeral='0', extra_specs.accel:device_profile='fpga1', id='a1.milli', is_disabled=, is_public='True',     |
|                                     | location=, name='a1.milli', original_name='a1.milli', ram='256', rxtx_factor=, swap='0', vcpus='1'                                  |
| hostId                              | 080dcc05cbdea486877d7dddbebaa4864df1dbc3c6da09d3be8eccaa                                                                            |
| host_status                         | UP                                                                                                                                  |
| id                                  | c44894d3-433b-451c-9f41-b968e372ebf1                                                                                                |
| image                               | cirros062 (6793c9b2-7cb6-4796-b477-9e22d985ea2b)                                                                                    |
| key_name                            | mykey                                                                                                                               |
| locked                              | False                                                                                                                               |
| locked_reason                       | None                                                                                                                                |
| name                                | instance04                                                                                                                          |
| progress                            | 0                                                                                                                                   |
| project_id                          | bccf406c045d401b91ba5c7552a124ae                                                                                                    |
| properties                          |                                                                                                                                     |
| security_groups                     | name='mysecurity'                                                                                                                   |
| server_groups                       | []                                                                                                                                  |
| status                              | ACTIVE                                                                                                                              |
| tags                                |                                                                                                                                     |
| trusted_image_certificates          | None                                                                                                                                |
| updated                             | 2024-05-18T16:32:20Z                                                                                                                |
| user_id                             | 7f3acb28d26943bab9510df3a6edf3b0                                                                                                    |
| volumes_attached                    |                                                                                                                                     |
+-------------------------------------+-------------------------------------------------------------------------------------------------------------------------------------+
```

## アクセラレータリクエストの確認

ARQ を確認する。

```sh
openstack accelerator arq list
```

```
+--------------------------------------+-------+---------------------+--------------------------------------+--------------------+----------------------------------------+
| uuid                                 | state | device_profile_name | instance_uuid                        | attach_handle_type | attach_handle_info                     |
+--------------------------------------+-------+---------------------+--------------------------------------+--------------------+----------------------------------------+
| d4c9c228-f678-418c-9a26-97b7331f473b | Bound | fpga1               | c44894d3-433b-451c-9f41-b968e372ebf1 | TEST_PCI           | {'domain': '0000', 'bus': '0c',        |
|                                      |       |                     |                                      |                    | 'device': '0', 'function': '1'}        |
+--------------------------------------+-------+---------------------+--------------------------------------+--------------------+----------------------------------------+
```

```sh
openstack accelerator arq show d4c9c228-f678-418c-9a26-97b7331f473b
```

```
+---------------------+-----------------------------------------------------------------+
| Field               | Value                                                           |
+---------------------+-----------------------------------------------------------------+
| uuid                | d4c9c228-f678-418c-9a26-97b7331f473b                            |
| state               | Bound                                                           |
| device_profile_name | fpga1                                                           |
| hostname            | compute.home.local                                              |
| device_rp_uuid      | 96b3d1a8-1155-32f7-92ab-a82c671f5155                            |
| instance_uuid       | c44894d3-433b-451c-9f41-b968e372ebf1                            |
| attach_handle_type  | TEST_PCI                                                        |
| attach_handle_info  | {'domain': '0000', 'bus': '0c', 'device': '0', 'function': '1'} |
+---------------------+-----------------------------------------------------------------+
```

## デバイスの確認

デバイスが消費されたことを確認する。

```sh
openstack resource provider inventory list 96b3d1a8-1155-32f7-92ab-a82c671f5155
```

```
+----------------+------------------+----------+----------+----------+-----------+-------+------+
| resource_class | allocation_ratio | min_unit | max_unit | reserved | step_size | total | used |
+----------------+------------------+----------+----------+----------+-----------+-------+------+
| FPGA           |              1.0 |        1 |       16 |        0 |         1 |    16 |    1 |
+----------------+------------------+----------+----------+----------+-----------+-------+------+
```
