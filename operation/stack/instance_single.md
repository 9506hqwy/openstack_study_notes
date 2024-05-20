# インスタンス(シングル)

## 前提条件

* flavor [](../flavor/m1_milli) を作成していること。
* イメージ [](../../installation/controller/glance) でイメージを作成していること。
* セキュリティグループのルール [](../security_group/icmp) を作成していること。
* セキュリティグループのルール [](../security_group/ssh) を作成していること。

## テンプレート

ネットワーク ID を指定してインスタンスを作成するテンプレート *milli-template.yaml* を作成する。

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
      image: cirros062
      flavor: m1.milli
      key_name: mykey
      networks:
      - network: { get_param: NetID }
      security_groups:
      - mysecurity

outputs:
  instance_name:
    description: Name of the instance.
    value: { get_attr: [ server, name ] }
  instance_ip:
    description: IP address of the instance.
    value: { get_attr: [server, first_address ] }
```

## スタックの作成

```{tip}
myuser で実行
```

NetID に [](../network/ovs_flat) を指定してスタック stack を展開する。

```sh
openstack stack create -t milli-template.yaml --parameter 'NetID=53f92676-e6e0-42fb-bde9-48dd2c2506b4' stack
```

```
+---------------------+--------------------------------------+
| Field               | Value                                |
+---------------------+--------------------------------------+
| id                  | bcf618e8-7e8a-4b58-8808-40e669af88fb |
| stack_name          | stack                                |
| description         | Template to deploy nano instance.    |
| creation_time       | 2024-05-18T02:57:28Z                 |
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
+--------------------------------------+------------+----------------------------------+-----------------+----------------------+--------------+
| ID                                   | Stack Name | Project                          | Stack Status    | Creation Time        | Updated Time |
+--------------------------------------+------------+----------------------------------+-----------------+----------------------+--------------+
| bcf618e8-7e8a-4b58-8808-40e669af88fb | stack      | bccf406c045d401b91ba5c7552a124ae | CREATE_COMPLETE | 2024-05-18T02:57:28Z | None         |
+--------------------------------------+------------+----------------------------------+-----------------+----------------------+--------------+
```

```sh
openstack stack show bcf618e8-7e8a-4b58-8808-40e669af88fb
```

```
+-----------------------+----------------------------------------------------------------------------------------------------------------------+
| Field                 | Value                                                                                                                |
+-----------------------+----------------------------------------------------------------------------------------------------------------------+
| id                    | bcf618e8-7e8a-4b58-8808-40e669af88fb                                                                                 |
| stack_name            | stack                                                                                                                |
| description           | Template to deploy nano instance.                                                                                    |
| creation_time         | 2024-05-18T02:57:28Z                                                                                                 |
| updated_time          | None                                                                                                                 |
| stack_status          | CREATE_COMPLETE                                                                                                      |
| stack_status_reason   | Stack CREATE completed successfully                                                                                  |
| parameters            | NetID: 53f92676-e6e0-42fb-bde9-48dd2c2506b4                                                                          |
|                       | OS::project_id: bccf406c045d401b91ba5c7552a124ae                                                                     |
|                       | OS::stack_id: bcf618e8-7e8a-4b58-8808-40e669af88fb                                                                   |
|                       | OS::stack_name: stack                                                                                                |
|                       |                                                                                                                      |
| outputs               | - description: Name of the instance.                                                                                 |
|                       |   output_key: instance_name                                                                                          |
|                       |   output_value: stack-server-7i33bimt2enk                                                                            |
|                       | - description: IP address of the instance.                                                                           |
|                       |   output_key: instance_ip                                                                                            |
|                       |   output_value: 172.16.0.174                                                                                         |
|                       |                                                                                                                      |
| links                 | - href: http://controller:8004/v1/bccf406c045d401b91ba5c7552a124ae/stacks/stack/bcf618e8-7e8a-4b58-8808-40e669af88fb |
|                       |   rel: self                                                                                                          |
|                       |                                                                                                                      |
| deletion_time         | None                                                                                                                 |
| notification_topics   | []                                                                                                                   |
| capabilities          | []                                                                                                                   |
| disable_rollback      | True                                                                                                                 |
| timeout_mins          | None                                                                                                                 |
| stack_owner           | myuser                                                                                                               |
| parent                | None                                                                                                                 |
| stack_user_project_id | 1c7a8671ead2481dbf21c811a0a1a4a9                                                                                     |
| tags                  | []                                                                                                                   |
|                       |                                                                                                                      |
+-----------------------+----------------------------------------------------------------------------------------------------------------------+
```

スタックの出力を確認する。

```sh
openstack stack output show --all stack
```

```
+---------------+--------------------------------------------------+
| Field         | Value                                            |
+---------------+--------------------------------------------------+
| instance_name | {                                                |
|               |   "output_key": "instance_name",                 |
|               |   "description": "Name of the instance.",        |
|               |   "output_value": "stack-server-7i33bimt2enk"    |
|               | }                                                |
| instance_ip   | {                                                |
|               |   "output_key": "instance_ip",                   |
|               |   "description": "IP address of the instance.",  |
|               |   "output_value": "172.16.0.174"                 |
|               | }                                                |
+---------------+--------------------------------------------------+
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
| server        | 0980cdec-f898-4a8d-8aca-b16b4897df2a | OS::Nova::Server | CREATE_COMPLETE | 2024-05-18T02:57:29Z |
+---------------+--------------------------------------+------------------+-----------------+----------------------+
```

```sh
openstack server list
```

```
+--------------------------------------+---------------------------+---------+------------------------------------------+-----------+----------+
| ID                                   | Name                      | Status  | Networks                                 | Image     | Flavor   |
+--------------------------------------+---------------------------+---------+------------------------------------------+-----------+----------+
| 0980cdec-f898-4a8d-8aca-b16b4897df2a | stack-server-7i33bimt2enk | ACTIVE  | provider=172.16.0.174                    | cirros062 | m1.milli |
+--------------------------------------+---------------------------+---------+------------------------------------------+-----------+----------+
```

```sh
openstack server show 0980cdec-f898-4a8d-8aca-b16b4897df2a
```

```
+-------------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field                               | Value                                                                                                                                                  |
+-------------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------+
| OS-DCF:diskConfig                   | MANUAL                                                                                                                                                 |
| OS-EXT-AZ:availability_zone         | nova                                                                                                                                                   |
| OS-EXT-SRV-ATTR:host                | compute.home.local                                                                                                                                     |
| OS-EXT-SRV-ATTR:hostname            | stack-server-7i33bimt2enk                                                                                                                              |
| OS-EXT-SRV-ATTR:hypervisor_hostname | compute.home.local                                                                                                                                     |
| OS-EXT-SRV-ATTR:instance_name       | instance-00000009                                                                                                                                      |
| OS-EXT-SRV-ATTR:kernel_id           |                                                                                                                                                        |
| OS-EXT-SRV-ATTR:launch_index        | 0                                                                                                                                                      |
| OS-EXT-SRV-ATTR:ramdisk_id          |                                                                                                                                                        |
| OS-EXT-SRV-ATTR:reservation_id      | r-34fz06t2                                                                                                                                             |
| OS-EXT-SRV-ATTR:root_device_name    | /dev/vda                                                                                                                                               |
| OS-EXT-SRV-ATTR:user_data           | Q29udGVudC1UeXBlOiBtdWx0aXBhcnQvbWl4ZWQ7IGJvdW5kYXJ5PSI9PT09PT09PT09PT09PT04NTY0OTUyMTUxODk1NTA4MDg2PT0iCk1JTUUtVmVyc2lvbjogMS4wCgotLT09PT09PT09PT09PT |
|                                     | 09PTg1NjQ5NTIxNTE4OTU1MDgwODY9PQpDb250ZW50LVR5cGU6IHRleHQvY2xvdWQtY29uZmlnOyBjaGFyc2V0PSJ1cy1hc2NpaSIKTUlNRS1WZXJzaW9uOiAxLjAKQ29udGVudC1UcmFuc2Zlci1F |
|                                     | bmNvZGluZzogN2JpdApDb250ZW50LURpc3Bvc2l0aW9uOiBhdHRhY2htZW50OyBmaWxlbmFtZT0iY2xvdWQtY29uZmlnIgoKCgojIENhcHR1cmUgYWxsIHN1YnByb2Nlc3Mgb3V0cHV0IGludG8gYS |
|                                     | Bsb2dmaWxlCiMgVXNlZnVsIGZvciB0cm91Ymxlc2hvb3RpbmcgY2xvdWQtaW5pdCBpc3N1ZXMKb3V0cHV0OiB7YWxsOiAnfCB0ZWUgLWEgL3Zhci9sb2cvY2xvdWQtaW5pdC1vdXRwdXQubG9nJ30K |
|                                     | Ci0tPT09PT09PT09PT09PT09ODU2NDk1MjE1MTg5NTUwODA4Nj09CkNvbnRlbnQtVHlwZTogdGV4dC9jbG91ZC1ib290aG9vazsgY2hhcnNldD0idXMtYXNjaWkiCk1JTUUtVmVyc2lvbjogMS4wCk |
|                                     | NvbnRlbnQtVHJhbnNmZXItRW5jb2Rpbmc6IDdiaXQKQ29udGVudC1EaXNwb3NpdGlvbjogYXR0YWNobWVudDsgZmlsZW5hbWU9ImJvb3Rob29rLnNoIgoKIyEvdXNyL2Jpbi9iYXNoCgojIEZJWE1F |
|                                     | KHNoYWRvd2VyKSB0aGlzIGlzIGEgd29ya2Fyb3VuZCBmb3IgY2xvdWQtaW5pdCAwLjYuMyBwcmVzZW50IGluIFVidW50dQojIDEyLjA0IExUUzoKIyBodHRwczovL2J1Z3MubGF1bmNocGFkLm5ldC |
|                                     | 9oZWF0LytidWcvMTI1NzQxMAojCiMgVGhlIG9sZCBjbG91ZC1pbml0IGRvZXNuJ3QgY3JlYXRlIHRoZSB1c2VycyBkaXJlY3RseSBzbyB0aGUgY29tbWFuZHMgdG8gZG8KIyB0aGlzIGFyZSBpbmpl |
|                                     | Y3RlZCB0aG91Z2ggbm92YV91dGlscy5weS4KIwojIE9uY2Ugd2UgZHJvcCBzdXBwb3J0IGZvciAwLjYuMywgd2UgY2FuIHNhZmVseSByZW1vdmUgdGhpcy4KCgojIGluIGNhc2UgaGVhdC1jZm50b2 |
|                                     | 9scyBoYXMgYmVlbiBpbnN0YWxsZWQgZnJvbSBwYWNrYWdlIGJ1dCBubyBzeW1saW5rcwojIGFyZSB5ZXQgaW4gL29wdC9hd3MvYmluLwpjZm4tY3JlYXRlLWF3cy1zeW1saW5rcwoKIyBEbyBub3Qg |
|                                     | cmVtb3ZlIC0gdGhlIGNsb3VkIGJvb3Rob29rIHNob3VsZCBhbHdheXMgcmV0dXJuIHN1Y2Nlc3MKZXhpdCAwCgotLT09PT09PT09PT09PT09PTg1NjQ5NTIxNTE4OTU1MDgwODY9PQpDb250ZW50LV |
|                                     | R5cGU6IHRleHQvcGFydC1oYW5kbGVyOyBjaGFyc2V0PSJ1cy1hc2NpaSIKTUlNRS1WZXJzaW9uOiAxLjAKQ29udGVudC1UcmFuc2Zlci1FbmNvZGluZzogN2JpdApDb250ZW50LURpc3Bvc2l0aW9u |
|                                     | OiBhdHRhY2htZW50OyBmaWxlbmFtZT0icGFydC1oYW5kbGVyLnB5IgoKIyBwYXJ0LWhhbmRsZXIKIwojICAgIExpY2Vuc2VkIHVuZGVyIHRoZSBBcGFjaGUgTGljZW5zZSwgVmVyc2lvbiAyLjAgKH |
|                                     | RoZSAiTGljZW5zZSIpOyB5b3UgbWF5CiMgICAgbm90IHVzZSB0aGlzIGZpbGUgZXhjZXB0IGluIGNvbXBsaWFuY2Ugd2l0aCB0aGUgTGljZW5zZS4gWW91IG1heSBvYnRhaW4KIyAgICBhIGNvcHkg |
|                                     | b2YgdGhlIExpY2Vuc2UgYXQKIwojICAgICAgICAgaHR0cDovL3d3dy5hcGFjaGUub3JnL2xpY2Vuc2VzL0xJQ0VOU0UtMi4wCiMKIyAgICBVbmxlc3MgcmVxdWlyZWQgYnkgYXBwbGljYWJsZSBsYX |
|                                     | cgb3IgYWdyZWVkIHRvIGluIHdyaXRpbmcsIHNvZnR3YXJlCiMgICAgZGlzdHJpYnV0ZWQgdW5kZXIgdGhlIExpY2Vuc2UgaXMgZGlzdHJpYnV0ZWQgb24gYW4gIkFTIElTIiBCQVNJUywgV0lUSE9V |
|                                     | VAojICAgIFdBUlJBTlRJRVMgT1IgQ09ORElUSU9OUyBPRiBBTlkgS0lORCwgZWl0aGVyIGV4cHJlc3Mgb3IgaW1wbGllZC4gU2VlIHRoZQojICAgIExpY2Vuc2UgZm9yIHRoZSBzcGVjaWZpYyBsYW |
|                                     | 5ndWFnZSBnb3Zlcm5pbmcgcGVybWlzc2lvbnMgYW5kIGxpbWl0YXRpb25zCiMgICAgdW5kZXIgdGhlIExpY2Vuc2UuCgppbXBvcnQgZGF0ZXRpbWUKaW1wb3J0IGVycm5vCmltcG9ydCBvcwppbXBv |
|                                     | cnQgc3lzCgoKZGVmIGxpc3RfdHlwZXMoKToKICAgIHJldHVybiBbInRleHQveC1jZm5pbml0ZGF0YSJdCgoKZGVmIGhhbmRsZV9wYXJ0KGRhdGEsIGN0eXBlLCBmaWxlbmFtZSwgcGF5bG9hZCk6Ci |
|                                     | AgICBpZiBjdHlwZSA9PSAiX19iZWdpbl9fIjoKICAgICAgICB0cnk6CiAgICAgICAgICAgIG9zLm1ha2VkaXJzKCcvdmFyL2xpYi9oZWF0LWNmbnRvb2xzJywgaW50KCI3MDAiLCA4KSkKICAgICAg |
|                                     | ICBleGNlcHQgT1NFcnJvcjoKICAgICAgICAgICAgZXhfdHlwZSwgZSwgdGIgPSBzeXMuZXhjX2luZm8oKQogICAgICAgICAgICBpZiBlLmVycm5vICE9IGVycm5vLkVFWElTVDoKICAgICAgICAgIC |
|                                     | AgICAgIHJhaXNlCiAgICAgICAgcmV0dXJuCgogICAgaWYgY3R5cGUgPT0gIl9fZW5kX18iOgogICAgICAgIHJldHVybgoKICAgIHRpbWVzdGFtcCA9IGRhdGV0aW1lLmRhdGV0aW1lLm5vdygpCiAg |
|                                     | ICB3aXRoIG9wZW4oJy92YXIvbG9nL3BhcnQtaGFuZGxlci5sb2cnLCAnYScpIGFzIGxvZzoKICAgICAgICBsb2cud3JpdGUoJyVzIGZpbGVuYW1lOiVzLCBjdHlwZTolc1xuJyAlICh0aW1lc3RhbX |
|                                     | AsIGZpbGVuYW1lLCBjdHlwZSkpCgogICAgaWYgY3R5cGUgPT0gJ3RleHQveC1jZm5pbml0ZGF0YSc6CiAgICAgICAgd2l0aCBvcGVuKCcvdmFyL2xpYi9oZWF0LWNmbnRvb2xzLyVzJyAlIGZpbGVu |
|                                     | YW1lLCAndycpIGFzIGY6CiAgICAgICAgICAgIGYud3JpdGUocGF5bG9hZCkKCiAgICAgICAgIyBUT0RPKHNkYWtlKSBob3BlZnVsbHkgdGVtcG9yYXJ5IHVudGlsIHVzZXJzIG1vdmUgdG8gaGVhdC |
|                                     | 1jZm50b29scy0xLjMKICAgICAgICB3aXRoIG9wZW4oJy92YXIvbGliL2Nsb3VkL2RhdGEvJXMnICUgZmlsZW5hbWUsICd3JykgYXMgZjoKICAgICAgICAgICAgZi53cml0ZShwYXlsb2FkKQoKLS09 |
|                                     | PT09PT09PT09PT09PT04NTY0OTUyMTUxODk1NTA4MDg2PT0KQ29udGVudC1UeXBlOiB0ZXh0L3gtY2ZuaW5pdGRhdGE7IGNoYXJzZXQ9InVzLWFzY2lpIgpNSU1FLVZlcnNpb246IDEuMApDb250ZW |
|                                     | 50LVRyYW5zZmVyLUVuY29kaW5nOiA3Yml0CkNvbnRlbnQtRGlzcG9zaXRpb246IGF0dGFjaG1lbnQ7IGZpbGVuYW1lPSJjZm4tdXNlcmRhdGEiCgoKLS09PT09PT09PT09PT09PT04NTY0OTUyMTUx |
|                                     | ODk1NTA4MDg2PT0KQ29udGVudC1UeXBlOiB0ZXh0L3gtc2hlbGxzY3JpcHQ7IGNoYXJzZXQ9InVzLWFzY2lpIgpNSU1FLVZlcnNpb246IDEuMApDb250ZW50LVRyYW5zZmVyLUVuY29kaW5nOiA3Ym |
|                                     | l0CkNvbnRlbnQtRGlzcG9zaXRpb246IGF0dGFjaG1lbnQ7IGZpbGVuYW1lPSJsb2d1c2VyZGF0YS5weSIKCiMhL2Jpbi9iYXNoCiMKIyAgICBMaWNlbnNlZCB1bmRlciB0aGUgQXBhY2hlIExpY2Vu |
|                                     | c2UsIFZlcnNpb24gMi4wICh0aGUgIkxpY2Vuc2UiKTsgeW91IG1heQojICAgIG5vdCB1c2UgdGhpcyBmaWxlIGV4Y2VwdCBpbiBjb21wbGlhbmNlIHdpdGggdGhlIExpY2Vuc2UuIFlvdSBtYXkgb2 |
|                                     | J0YWluCiMgICAgYSBjb3B5IG9mIHRoZSBMaWNlbnNlIGF0CiMKIyAgICAgICAgIGh0dHA6Ly93d3cuYXBhY2hlLm9yZy9saWNlbnNlcy9MSUNFTlNFLTIuMAojCiMgICAgVW5sZXNzIHJlcXVpcmVk |
|                                     | IGJ5IGFwcGxpY2FibGUgbGF3IG9yIGFncmVlZCB0byBpbiB3cml0aW5nLCBzb2Z0d2FyZQojICAgIGRpc3RyaWJ1dGVkIHVuZGVyIHRoZSBMaWNlbnNlIGlzIGRpc3RyaWJ1dGVkIG9uIGFuICJBUy |
|                                     | BJUyIgQkFTSVMsIFdJVEhPVVQKIyAgICBXQVJSQU5USUVTIE9SIENPTkRJVElPTlMgT0YgQU5ZIEtJTkQsIGVpdGhlciBleHByZXNzIG9yIGltcGxpZWQuIFNlZSB0aGUKIyAgICBMaWNlbnNlIGZv |
|                                     | ciB0aGUgc3BlY2lmaWMgbGFuZ3VhZ2UgZ292ZXJuaW5nIHBlcm1pc3Npb25zIGFuZCBsaW1pdGF0aW9ucwojICAgIHVuZGVyIHRoZSBMaWNlbnNlLgoKaW1wb3J0IGRhdGV0aW1lCmltcG9ydCBlcn |
|                                     | JubwppbXBvcnQgbG9nZ2luZwppbXBvcnQgb3MKaW1wb3J0IHN1YnByb2Nlc3MKaW1wb3J0IHN5cwoKClZBUl9QQVRIID0gJy92YXIvbGliL2hlYXQtY2ZudG9vbHMnCkxPRyA9IGxvZ2dpbmcuZ2V0 |
|                                     | TG9nZ2VyKCdoZWF0LXByb3Zpc2lvbicpCgoKZGVmIGluaXRfbG9nZ2luZygpOgogICAgTE9HLnNldExldmVsKGxvZ2dpbmcuSU5GTykKICAgIExPRy5hZGRIYW5kbGVyKGxvZ2dpbmcuU3RyZWFtSG |
|                                     | FuZGxlcigpKQogICAgZmggPSBsb2dnaW5nLkZpbGVIYW5kbGVyKCIvdmFyL2xvZy9oZWF0LXByb3Zpc2lvbi5sb2ciKQogICAgb3MuY2htb2QoZmguYmFzZUZpbGVuYW1lLCBpbnQoIjYwMCIsIDgp |
|                                     | KQogICAgTE9HLmFkZEhhbmRsZXIoZmgpCgoKZGVmIGNhbGwoYXJncyk6CgogICAgY2xhc3MgTG9nU3RyZWFtKG9iamVjdCk6CgogICAgICAgIGRlZiB3cml0ZShzZWxmLCBkYXRhKToKICAgICAgIC |
|                                     | AgICAgTE9HLmluZm8oZGF0YSkKCiAgICBMT0cuaW5mbygnJXNcbicsICcgJy5qb2luKGFyZ3MpKSAgIyBub3FhCiAgICB0cnk6CiAgICAgICAgbHMgPSBMb2dTdHJlYW0oKQogICAgICAgIHAgPSBz |
|                                     | dWJwcm9jZXNzLlBvcGVuKGFyZ3MsIHN0ZG91dD1zdWJwcm9jZXNzLlBJUEUsCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgc3RkZXJyPXN1YnByb2Nlc3MuUElQRSkKICAgICAgICBkYXRhID |
|                                     | 0gcC5jb21tdW5pY2F0ZSgpCiAgICAgICAgaWYgZGF0YToKICAgICAgICAgICAgZm9yIHggaW4gZGF0YToKICAgICAgICAgICAgICAgIGxzLndyaXRlKHgpCiAgICBleGNlcHQgT1NFcnJvcjoKICAg |
|                                     | ICAgICBleF90eXBlLCBleCwgdGIgPSBzeXMuZXhjX2luZm8oKQogICAgICAgIGlmIGV4LmVycm5vID09IGVycm5vLkVOT0VYRUM6CiAgICAgICAgICAgIExPRy5lcnJvcignVXNlcmRhdGEgZW1wdH |
|                                     | kgb3Igbm90IGV4ZWN1dGFibGU6ICVzJywgZXgpCiAgICAgICAgICAgIHJldHVybiBvcy5FWF9PSwogICAgICAgIGVsc2U6CiAgICAgICAgICAgIExPRy5lcnJvcignT1MgZXJyb3IgcnVubmluZyB1 |
|                                     | c2VyZGF0YTogJXMnLCBleCkKICAgICAgICAgICAgcmV0dXJuIG9zLkVYX09TRVJSCiAgICBleGNlcHQgRXhjZXB0aW9uOgogICAgICAgIGV4X3R5cGUsIGV4LCB0YiA9IHN5cy5leGNfaW5mbygpCi |
|                                     | AgICAgICAgTE9HLmVycm9yKCdVbmtub3duIGVycm9yIHJ1bm5pbmcgdXNlcmRhdGE6ICVzJywgZXgpCiAgICAgICAgcmV0dXJuIG9zLkVYX1NPRlRXQVJFCiAgICByZXR1cm4gcC5yZXR1cm5jb2Rl |
|                                     | CgoKZGVmIG1haW4oKToKICAgIHVzZXJkYXRhX3BhdGggPSBvcy5wYXRoLmpvaW4oVkFSX1BBVEgsICdjZm4tdXNlcmRhdGEnKQogICAgb3MuY2htb2QodXNlcmRhdGFfcGF0aCwgaW50KCI3MDAiLC |
|                                     | A4KSkKCiAgICBMT0cuaW5mbygnUHJvdmlzaW9uIGJlZ2FuOiAlcycsIGRhdGV0aW1lLmRhdGV0aW1lLm5vdygpKQogICAgcmV0dXJuY29kZSA9IGNhbGwoW3VzZXJkYXRhX3BhdGhdKQogICAgTE9H |
|                                     | LmluZm8oJ1Byb3Zpc2lvbiBkb25lOiAlcycsIGRhdGV0aW1lLmRhdGV0aW1lLm5vdygpKQogICAgaWYgcmV0dXJuY29kZToKICAgICAgICByZXR1cm4gcmV0dXJuY29kZQoKCmlmIF9fbmFtZV9fID |
|                                     | 09ICdfX21haW5fXyc6CiAgICBpbml0X2xvZ2dpbmcoKQoKICAgIGNvZGUgPSBtYWluKCkKICAgIGlmIGNvZGU6CiAgICAgICAgTE9HLmVycm9yKCdQcm92aXNpb24gZmFpbGVkIHdpdGggZXhpdCBj |
|                                     | b2RlICVzJywgY29kZSkKICAgICAgICBzeXMuZXhpdChjb2RlKQoKICAgIHByb3Zpc2lvbl9sb2cgPSBvcy5wYXRoLmpvaW4oVkFSX1BBVEgsICdwcm92aXNpb24tZmluaXNoZWQnKQogICAgIyB0b3 |
|                                     | VjaCB0aGUgZmlsZSBzbyBpdCBpcyB0aW1lc3RhbXBlZCB3aXRoIHdoZW4gZmluaXNoZWQKICAgIHdpdGggb3Blbihwcm92aXNpb25fbG9nLCAnYScpOgogICAgICAgIG9zLnV0aW1lKHByb3Zpc2lv |
|                                     | bl9sb2csIE5vbmUpCgotLT09PT09PT09PT09PT09PTg1NjQ5NTIxNTE4OTU1MDgwODY9PQpDb250ZW50LVR5cGU6IHRleHQveC1jZm5pbml0ZGF0YTsgY2hhcnNldD0idXMtYXNjaWkiCk1JTUUtVm |
|                                     | Vyc2lvbjogMS4wCkNvbnRlbnQtVHJhbnNmZXItRW5jb2Rpbmc6IDdiaXQKQ29udGVudC1EaXNwb3NpdGlvbjogYXR0YWNobWVudDsgZmlsZW5hbWU9ImNmbi1tZXRhZGF0YS1zZXJ2ZXIiCgpodHRw |
|                                     | Oi8vY29udHJvbGxlcjo4MDAwL3YxLwotLT09PT09PT09PT09PT09PTg1NjQ5NTIxNTE4OTU1MDgwODY9PQpDb250ZW50LVR5cGU6IHRleHQveC1jZm5pbml0ZGF0YTsgY2hhcnNldD0idXMtYXNjaW |
|                                     | kiCk1JTUUtVmVyc2lvbjogMS4wCkNvbnRlbnQtVHJhbnNmZXItRW5jb2Rpbmc6IDdiaXQKQ29udGVudC1EaXNwb3NpdGlvbjogYXR0YWNobWVudDsgZmlsZW5hbWU9ImNmbi1ib3RvLWNmZyIKCltC |
|                                     | b3RvXQpkZWJ1ZyA9IDAKaXNfc2VjdXJlID0gMApodHRwc192YWxpZGF0ZV9jZXJ0aWZpY2F0ZXMgPSAxCmNmbl9yZWdpb25fbmFtZSA9IGhlYXQKY2ZuX3JlZ2lvbl9lbmRwb2ludCA9IGNvbnRyb2 |
|                                     | xsZXIKLS09PT09PT09PT09PT09PT04NTY0OTUyMTUxODk1NTA4MDg2PT0tLQo=                                                                                         |
| OS-EXT-STS:power_state              | Running                                                                                                                                                |
| OS-EXT-STS:task_state               | None                                                                                                                                                   |
| OS-EXT-STS:vm_state                 | active                                                                                                                                                 |
| OS-SRV-USG:launched_at              | 2024-05-18T02:57:38.000000                                                                                                                             |
| OS-SRV-USG:terminated_at            | None                                                                                                                                                   |
| accessIPv4                          |                                                                                                                                                        |
| accessIPv6                          |                                                                                                                                                        |
| addresses                           | provider=172.16.0.174                                                                                                                                  |
| config_drive                        |                                                                                                                                                        |
| created                             | 2024-05-18T02:57:32Z                                                                                                                                   |
| description                         | None                                                                                                                                                   |
| flavor                              | description=, disk='1', ephemeral='0', , id='m1.milli', is_disabled=, is_public='True', location=, name='m1.milli', original_name='m1.milli',          |
|                                     | ram='256', rxtx_factor=, swap='0', vcpus='1'                                                                                                           |
| hostId                              | 080dcc05cbdea486877d7dddbebaa4864df1dbc3c6da09d3be8eccaa                                                                                               |
| host_status                         | UP                                                                                                                                                     |
| id                                  | 0980cdec-f898-4a8d-8aca-b16b4897df2a                                                                                                                   |
| image                               | cirros062 (6793c9b2-7cb6-4796-b477-9e22d985ea2b)                                                                                                       |
| key_name                            | mykey                                                                                                                                                  |
| locked                              | False                                                                                                                                                  |
| locked_reason                       | None                                                                                                                                                   |
| name                                | stack-server-7i33bimt2enk                                                                                                                              |
| progress                            | 0                                                                                                                                                      |
| project_id                          | bccf406c045d401b91ba5c7552a124ae                                                                                                                       |
| properties                          |                                                                                                                                                        |
| security_groups                     | name='mysecurity'                                                                                                                                      |
| server_groups                       | []                                                                                                                                                     |
| status                              | ACTIVE                                                                                                                                                 |
| tags                                |                                                                                                                                                        |
| trusted_image_certificates          | None                                                                                                                                                   |
| updated                             | 2024-05-18T02:57:38Z                                                                                                                                   |
| user_id                             | 7f3acb28d26943bab9510df3a6edf3b0                                                                                                                       |
| volumes_attached                    |                                                                                                                                                        |
+-------------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------+
```

`OS-EXT-SRV-ATTR:user_data` は以下になる。

```sh
openstack server show 0980cdec-f898-4a8d-8aca-b16b4897df2a -c OS-EXT-SRV-ATTR:user_data -f value | base64 -d
```

```
Content-Type: multipart/mixed; boundary="===============8564952151895508086=="
MIME-Version: 1.0

--===============8564952151895508086==
Content-Type: text/cloud-config; charset="us-ascii"
MIME-Version: 1.0
Content-Transfer-Encoding: 7bit
Content-Disposition: attachment; filename="cloud-config"



# Capture all subprocess output into a logfile
# Useful for troubleshooting cloud-init issues
output: {all: '| tee -a /var/log/cloud-init-output.log'}

--===============8564952151895508086==
Content-Type: text/cloud-boothook; charset="us-ascii"
MIME-Version: 1.0
Content-Transfer-Encoding: 7bit
Content-Disposition: attachment; filename="boothook.sh"

#!/usr/bin/bash

# FIXME(shadower) this is a workaround for cloud-init 0.6.3 present in Ubuntu
# 12.04 LTS:
# https://bugs.launchpad.net/heat/+bug/1257410
#
# The old cloud-init doesn't create the users directly so the commands to do
# this are injected though nova_utils.py.
#
# Once we drop support for 0.6.3, we can safely remove this.


# in case heat-cfntools has been installed from package but no symlinks
# are yet in /opt/aws/bin/
cfn-create-aws-symlinks

# Do not remove - the cloud boothook should always return success
exit 0

--===============8564952151895508086==
Content-Type: text/part-handler; charset="us-ascii"
MIME-Version: 1.0
Content-Transfer-Encoding: 7bit
Content-Disposition: attachment; filename="part-handler.py"

# part-handler
#
#    Licensed under the Apache License, Version 2.0 (the "License"); you may
#    not use this file except in compliance with the License. You may obtain
#    a copy of the License at
#
#         http://www.apache.org/licenses/LICENSE-2.0
#
#    Unless required by applicable law or agreed to in writing, software
#    distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
#    WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the
#    License for the specific language governing permissions and limitations
#    under the License.

import datetime
import errno
import os
import sys


def list_types():
    return ["text/x-cfninitdata"]


def handle_part(data, ctype, filename, payload):
    if ctype == "__begin__":
        try:
            os.makedirs('/var/lib/heat-cfntools', int("700", 8))
        except OSError:
            ex_type, e, tb = sys.exc_info()
            if e.errno != errno.EEXIST:
                raise
        return

    if ctype == "__end__":
        return

    timestamp = datetime.datetime.now()
    with open('/var/log/part-handler.log', 'a') as log:
        log.write('%s filename:%s, ctype:%s\n' % (timestamp, filename, ctype))

    if ctype == 'text/x-cfninitdata':
        with open('/var/lib/heat-cfntools/%s' % filename, 'w') as f:
            f.write(payload)

        # TODO(sdake) hopefully temporary until users move to heat-cfntools-1.3
        with open('/var/lib/cloud/data/%s' % filename, 'w') as f:
            f.write(payload)

--===============8564952151895508086==
Content-Type: text/x-cfninitdata; charset="us-ascii"
MIME-Version: 1.0
Content-Transfer-Encoding: 7bit
Content-Disposition: attachment; filename="cfn-userdata"


--===============8564952151895508086==
Content-Type: text/x-shellscript; charset="us-ascii"
MIME-Version: 1.0
Content-Transfer-Encoding: 7bit
Content-Disposition: attachment; filename="loguserdata.py"

#!/bin/bash
#
#    Licensed under the Apache License, Version 2.0 (the "License"); you may
#    not use this file except in compliance with the License. You may obtain
#    a copy of the License at
#
#         http://www.apache.org/licenses/LICENSE-2.0
#
#    Unless required by applicable law or agreed to in writing, software
#    distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
#    WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the
#    License for the specific language governing permissions and limitations
#    under the License.

import datetime
import errno
import logging
import os
import subprocess
import sys


VAR_PATH = '/var/lib/heat-cfntools'
LOG = logging.getLogger('heat-provision')


def init_logging():
    LOG.setLevel(logging.INFO)
    LOG.addHandler(logging.StreamHandler())
    fh = logging.FileHandler("/var/log/heat-provision.log")
    os.chmod(fh.baseFilename, int("600", 8))
    LOG.addHandler(fh)


def call(args):

    class LogStream(object):

        def write(self, data):
            LOG.info(data)

    LOG.info('%s\n', ' '.join(args))  # noqa
    try:
        ls = LogStream()
        p = subprocess.Popen(args, stdout=subprocess.PIPE,
                             stderr=subprocess.PIPE)
        data = p.communicate()
        if data:
            for x in data:
                ls.write(x)
    except OSError:
        ex_type, ex, tb = sys.exc_info()
        if ex.errno == errno.ENOEXEC:
            LOG.error('Userdata empty or not executable: %s', ex)
            return os.EX_OK
        else:
            LOG.error('OS error running userdata: %s', ex)
            return os.EX_OSERR
    except Exception:
        ex_type, ex, tb = sys.exc_info()
        LOG.error('Unknown error running userdata: %s', ex)
        return os.EX_SOFTWARE
    return p.returncode


def main():
    userdata_path = os.path.join(VAR_PATH, 'cfn-userdata')
    os.chmod(userdata_path, int("700", 8))

    LOG.info('Provision began: %s', datetime.datetime.now())
    returncode = call([userdata_path])
    LOG.info('Provision done: %s', datetime.datetime.now())
    if returncode:
        return returncode


if __name__ == '__main__':
    init_logging()

    code = main()
    if code:
        LOG.error('Provision failed with exit code %s', code)
        sys.exit(code)

    provision_log = os.path.join(VAR_PATH, 'provision-finished')
    # touch the file so it is timestamped with when finished
    with open(provision_log, 'a'):
        os.utime(provision_log, None)

--===============8564952151895508086==
Content-Type: text/x-cfninitdata; charset="us-ascii"
MIME-Version: 1.0
Content-Transfer-Encoding: 7bit
Content-Disposition: attachment; filename="cfn-metadata-server"

http://controller:8000/v1/
--===============8564952151895508086==
Content-Type: text/x-cfninitdata; charset="us-ascii"
MIME-Version: 1.0
Content-Transfer-Encoding: 7bit
Content-Disposition: attachment; filename="cfn-boto-cfg"

[Boto]
debug = 0
is_secure = 0
https_validate_certificates = 1
cfn_region_name = heat
cfn_region_endpoint = controller
--===============8564952151895508086==--
```
