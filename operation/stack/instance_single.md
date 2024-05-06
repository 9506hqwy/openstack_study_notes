# インスタンス(シングル)

## 前提条件

* flavor [](../flavor/m1_nano) を作成していること。
* イメージ [](../../installation/controller/glance) でイメージを作成していること。
* セキュリティグループのルール [](../security_group/icmp) を作成していること。
* セキュリティグループのルール [](../security_group/ssh) を作成していること。

## テンプレート

ネットワーク ID を指定してインスタンスを作成するテンプレート *nano-template.yaml* を作成する。

```yaml
heat_template_version: 2015-04-30
description: Template to deploy nano instance.

parameters:
  NetID:
    type: string
    description: Network ID to use for the instance.

resources:
  server:
    type: OS::Nova::Server
    properties:
      image: cirros
      flavor: m1.nano
      key_name: mykey
      networks:
      - network: { get_param: NetID }

outputs:
  instance_name:
    description: Name of the instance.
    value: { get_attr: [ server, name ] }
  instance_ip:
    description: IP address of the instance.
    value: { get_attr: [server, first_address ] }
```

## スタックの作成

NetID に [](../network/ovs_flat) を指定してスタック stack を展開する。

```sh
openstack stack create -t nano-template.yaml --parameter 'NetID=85f372ef-f39a-42bb-a06b-9f9ed8e4e58e' stack
```

```
+---------------------+--------------------------------------+
| Field               | Value                                |
+---------------------+--------------------------------------+
| id                  | 9ae958d7-352c-4f08-8bcd-87da4b1c9d6c |
| stack_name          | stack                                |
| description         | Template to deploy nano instance.    |
| creation_time       | 2024-04-28T02:41:59Z                 |
| updated_time        | None                                 |
| stack_status        | CREATE_IN_PROGRESS                   |
| stack_status_reason | Stack CREATE started                 |
+---------------------+--------------------------------------+
```

## スタックの確認

スタックを確認する。

```sh
openstack stack list
```

```
+--------------------------------------+------------+-----------------+----------------------+--------------+
| ID                                   | Stack Name | Stack Status    | Creation Time        | Updated Time |
+--------------------------------------+------------+-----------------+----------------------+--------------+
| 9ae958d7-352c-4f08-8bcd-87da4b1c9d6c | stack      | CREATE_COMPLETE | 2024-04-28T02:41:59Z | None         |
+--------------------------------------+------------+-----------------+----------------------+--------------+
```

```sh
openstack stack show 9ae958d7-352c-4f08-8bcd-87da4b1c9d6c
```

```
+-----------------------+----------------------------------------------------------------------------------------------------------------------+
| Field                 | Value                                                                                                                |
+-----------------------+----------------------------------------------------------------------------------------------------------------------+
| id                    | 9ae958d7-352c-4f08-8bcd-87da4b1c9d6c                                                                                 |
| stack_name            | stack                                                                                                                |
| description           | Template to deploy nano instance.                                                                                    |
| creation_time         | 2024-04-28T02:41:59Z                                                                                                 |
| updated_time          | None                                                                                                                 |
| stack_status          | CREATE_COMPLETE                                                                                                      |
| stack_status_reason   | Stack CREATE completed successfully                                                                                  |
| parameters            | NetID: 85f372ef-f39a-42bb-a06b-9f9ed8e4e58e                                                                          |
|                       | OS::project_id: f2aeffb34ff34ffb8959f1cd813655c6                                                                     |
|                       | OS::stack_id: 9ae958d7-352c-4f08-8bcd-87da4b1c9d6c                                                                   |
|                       | OS::stack_name: stack                                                                                                |
|                       |                                                                                                                      |
| outputs               | - description: No description given                                                                                  |
|                       |   output_key: instance_ip                                                                                            |
|                       |   output_value: 172.17.0.193                                                                                         |
|                       | - description: Name of the instance.                                                                                 |
|                       |   output_key: instance_name                                                                                          |
|                       |   output_value: stack-server-izoksltvuphu                                                                            |
|                       |                                                                                                                      |
| links                 | - href: http://controller:8004/v1/f2aeffb34ff34ffb8959f1cd813655c6/stacks/stack/9ae958d7-352c-4f08-8bcd-87da4b1c9d6c |
|                       |   rel: self                                                                                                          |
|                       |                                                                                                                      |
| deletion_time         | None                                                                                                                 |
| notification_topics   | []                                                                                                                   |
| capabilities          | []                                                                                                                   |
| disable_rollback      | True                                                                                                                 |
| timeout_mins          | None                                                                                                                 |
| stack_owner           | myuser                                                                                                               |
| parent                | None                                                                                                                 |
| stack_user_project_id | d005f687dbb64f81b40b6419006a92b8                                                                                     |
| tags                  | []                                                                                                                   |
|                       |                                                                                                                      |
+-----------------------+----------------------------------------------------------------------------------------------------------------------+
```

スタックの出力を確認する。

```sh
openstack stack output show --all stack
```

```
+---------------+-----------------------------------------------+
| Field         | Value                                         |
+---------------+-----------------------------------------------+
| instance_ip   | {                                             |
|               |   "output_key": "instance_ip",                |
|               |   "description": "No description given",      |
|               |   "output_value": "172.17.0.193"              |
|               | }                                             |
| instance_name | {                                             |
|               |   "output_key": "instance_name",              |
|               |   "description": "Name of the instance.",     |
|               |   "output_value": "stack-server-izoksltvuphu" |
|               | }                                             |
+---------------+-----------------------------------------------+
```

## リソースの確認

スタック stack 内のリソースを確認する。

```sh
openstack stack resource list stack
```

```
+---------------+--------------------------------------+------------------+-----------------+----------------------+
| resource_name | physical_resource_id                 | resource_type    | resource_status | updated_time         |
+---------------+--------------------------------------+------------------+-----------------+----------------------+
| server        | ca9841dc-3768-4544-9d99-1a452ff99eff | OS::Nova::Server | CREATE_COMPLETE | 2024-04-28T02:42:00Z |
+---------------+--------------------------------------+------------------+-----------------+----------------------+
```

```sh
openstack server list
```

```
+--------------------------------------+---------------------------+---------+-----------------------------+--------+---------+
| ID                                   | Name                      | Status  | Networks                    | Image  | Flavor  |
+--------------------------------------+---------------------------+---------+-----------------------------+--------+---------+
| ca9841dc-3768-4544-9d99-1a452ff99eff | stack-server-izoksltvuphu | ACTIVE  | provider=172.17.0.193       | cirros | m1.nano |
+--------------------------------------+---------------------------+---------+-----------------------------+--------+---------+
```

```sh
openstack server show ca9841dc-3768-4544-9d99-1a452ff99eff
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
| OS-SRV-USG:launched_at      | 2024-04-28T02:42:14.000000                               |
| OS-SRV-USG:terminated_at    | None                                                     |
| accessIPv4                  |                                                          |
| accessIPv6                  |                                                          |
| addresses                   | provider=172.17.0.193                                    |
| config_drive                |                                                          |
| created                     | 2024-04-28T02:42:04Z                                     |
| flavor                      | m1.nano (0)                                              |
| hostId                      | 97e1bdc25bc905fab023c5f59b74e31b8b6c1a3e3e9a9e993ff9da13 |
| id                          | ca9841dc-3768-4544-9d99-1a452ff99eff                     |
| image                       | cirros (e83903c4-7fa8-42a7-b693-f5034bc33603)            |
| key_name                    | mykey                                                    |
| name                        | stack-server-izoksltvuphu                                |
| progress                    | 0                                                        |
| project_id                  | f2aeffb34ff34ffb8959f1cd813655c6                         |
| properties                  |                                                          |
| security_groups             | name='default'                                           |
| status                      | ACTIVE                                                   |
| updated                     | 2024-04-28T02:42:14Z                                     |
| user_id                     | 71b5948c75f24c0f841dbf1c4eb4c4a7                         |
| volumes_attached            |                                                          |
+-----------------------------+----------------------------------------------------------+
```
