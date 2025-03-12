# b1.nano

## flavor の作成

ベアメタルプロビジョニングで使用する flavor を作成する。

```sh
openstack flavor create b1.nano
```

```text
+----------------------------+--------------------------------------+
| Field                      | Value                                |
+----------------------------+--------------------------------------+
| OS-FLV-DISABLED:disabled   | False                                |
| OS-FLV-EXT-DATA:ephemeral  | 0                                    |
| description                | None                                 |
| disk                       | 0                                    |
| id                         | 7a123317-6326-4517-8d57-d42e09dda099 |
| name                       | b1.nano                              |
| os-flavor-access:is_public | True                                 |
| properties                 |                                      |
| ram                        | 256                                  |
| rxtx_factor                | 1.0                                  |
| swap                       | 0                                    |
| vcpus                      | 1                                    |
+----------------------------+--------------------------------------+
```

## リソースクラスの設定

`resource_class` が `baremetal` と一致するノードを対象とする。

```sh
openstack flavor set --property resources:CUSTOM_BAREMETAL=1 b1.nano
openstack flavor set --property resources:VCPU=0 b1.nano
openstack flavor set --property resources:MEMORY_MB=0 b1.nano
openstack flavor set --property resources:DISK_GB=0 b1.nano
```
