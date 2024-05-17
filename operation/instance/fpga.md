# インスタンスの作成 (FPGA)

FPGA を要求するインスタンスを作成する。

## 前提条件

* [](../network/linuxbridge_flat) を作成していること。
* flavor [](../flavor/a1_nano) を作成していること。
* イメージ [](../../installation/controller/glance) でイメージを作成していること。
* セキュリティグループのルール [](../security_group/icmp) を作成していること。
* セキュリティグループのルール [](../security_group/ssh) を作成していること。

## インスタンスの作成

インスタンス instance04 を作成する。

```sh
openstack server create \
    --flavor a1.nano \
    --image cirros \
    --nic net-id=85f372ef-f39a-42bb-a06b-9f9ed8e4e58e \
    --security-group default \
    --key-name mykey \
    instance04
```

```
+-----------------------------+------------------------------------------------+
| Field                       | Value                                          |
+-----------------------------+------------------------------------------------+
| OS-DCF:diskConfig           | MANUAL                                         |
| OS-EXT-AZ:availability_zone |                                                |
| OS-EXT-STS:power_state      | NOSTATE                                        |
| OS-EXT-STS:task_state       | scheduling                                     |
| OS-EXT-STS:vm_state         | building                                       |
| OS-SRV-USG:launched_at      | None                                           |
| OS-SRV-USG:terminated_at    | None                                           |
| accessIPv4                  |                                                |
| accessIPv6                  |                                                |
| addresses                   |                                                |
| adminPass                   | w9DXJXDbQ2Uo                                   |
| config_drive                |                                                |
| created                     | 2024-05-06T04:02:58Z                           |
| flavor                      | a1.nano (9ac967af-d538-4bd6-acd0-f4eb77fb6e00) |
| hostId                      |                                                |
| id                          | f108af6f-97ef-44fb-a657-4f11d6b63c08           |
| image                       | cirros (e83903c4-7fa8-42a7-b693-f5034bc33603)  |
| key_name                    | mykey                                          |
| name                        | instance04                                     |
| progress                    | 0                                              |
| project_id                  | f2aeffb34ff34ffb8959f1cd813655c6               |
| properties                  |                                                |
| security_groups             | name='87fd4685-d317-42fb-a487-28382d2c2750'    |
| status                      | BUILD                                          |
| updated                     | 2024-05-06T04:02:58Z                           |
| user_id                     | 71b5948c75f24c0f841dbf1c4eb4c4a7               |
| volumes_attached            |                                                |
+-----------------------------+------------------------------------------------+
```

## インスタンスの確認

インスタンスが ACTIVE になったことを確認する。

```sh
openstack server list
```

```
+--------------------------------------+---------------------------+---------+-----------------------------------------+--------+---------+
| ID                                   | Name                      | Status  | Networks                                | Image  | Flavor  |
+--------------------------------------+---------------------------+---------+-----------------------------------------+--------+---------+
| f108af6f-97ef-44fb-a657-4f11d6b63c08 | instance04                | ACTIVE  | provider=172.17.0.13                    | cirros | a1.nano |
+--------------------------------------+---------------------------+---------+-----------------------------------------+--------+---------+
```

```sh
openstack server show f108af6f-97ef-44fb-a657-4f11d6b63c08
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
| OS-SRV-USG:launched_at      | 2024-05-06T04:03:11.000000                               |
| OS-SRV-USG:terminated_at    | None                                                     |
| accessIPv4                  |                                                          |
| accessIPv6                  |                                                          |
| addresses                   | provider=172.17.0.13                                     |
| config_drive                |                                                          |
| created                     | 2024-05-06T04:02:58Z                                     |
| flavor                      | a1.nano (9ac967af-d538-4bd6-acd0-f4eb77fb6e00)           |
| hostId                      | 97e1bdc25bc905fab023c5f59b74e31b8b6c1a3e3e9a9e993ff9da13 |
| id                          | f108af6f-97ef-44fb-a657-4f11d6b63c08                     |
| image                       | cirros (e83903c4-7fa8-42a7-b693-f5034bc33603)            |
| key_name                    | mykey                                                    |
| name                        | instance04                                               |
| progress                    | 0                                                        |
| project_id                  | f2aeffb34ff34ffb8959f1cd813655c6                         |
| properties                  |                                                          |
| security_groups             | name='default'                                           |
| status                      | ACTIVE                                                   |
| updated                     | 2024-05-06T04:03:11Z                                     |
| user_id                     | 71b5948c75f24c0f841dbf1c4eb4c4a7                         |
| volumes_attached            |                                                          |
+-----------------------------+----------------------------------------------------------+
```

## アクセラレータリクエストの確認

ARQ を確認する。

```sh
openstack accelerator arq list
```

```
+--------------------------------------+-------+---------------------+--------------------------------------+--------------------+-----------------------------------------------------------------+
| uuid                                 | state | device_profile_name | instance_uuid                        | attach_handle_type | attach_handle_info                                              |
+--------------------------------------+-------+---------------------+--------------------------------------+--------------------+-----------------------------------------------------------------+
| 4b818e1c-c404-434b-b61c-320b656f60f9 | Bound | fpga1               | f108af6f-97ef-44fb-a657-4f11d6b63c08 | TEST_PCI           | {'domain': '0000', 'bus': '0c', 'device': '0', 'function': '1'} |
+--------------------------------------+-------+---------------------+--------------------------------------+--------------------+-----------------------------------------------------------------+
```

```sh
openstack accelerator arq show 4b818e1c-c404-434b-b61c-320b656f60f9
```

```
+---------------------+-----------------------------------------------------------------+
| Field               | Value                                                           |
+---------------------+-----------------------------------------------------------------+
| uuid                | 4b818e1c-c404-434b-b61c-320b656f60f9                            |
| state               | Bound                                                           |
| device_profile_name | fpga1                                                           |
| hostname            | compute.home.local                                              |
| device_rp_uuid      | 96b3d1a8-1155-32f7-92ab-a82c671f5155                            |
| instance_uuid       | f108af6f-97ef-44fb-a657-4f11d6b63c08                            |
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
