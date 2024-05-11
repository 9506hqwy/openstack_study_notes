# b1.nano

## flavor の作成

ベアメタルプロビジョニングで使用する flavor を作成する。

| オプション                  | 説明                         |
| --------------------------- | ---------------------------- |
| --vcpus                     | CPU 数                       |
| --ram                       | メモリサイズ MiB             |
| --disk                      | ディスクサイズ GiB           |

```sh
openstack flavor create \
    --ram 1024 \
    --vcpus 1 \
    --disk 10 \
    b1.nano
```

```
+----------------------------+--------------------------------------+
| Field                      | Value                                |
+----------------------------+--------------------------------------+
| OS-FLV-DISABLED:disabled   | False                                |
| OS-FLV-EXT-DATA:ephemeral  | 0                                    |
| description                | None                                 |
| disk                       | 10                                   |
| id                         | 3ccda944-adcb-478c-b94a-21cd4d321905 |
| name                       | b1.nano                              |
| os-flavor-access:is_public | True                                 |
| properties                 |                                      |
| ram                        | 1024                                 |
| rxtx_factor                | 1.0                                  |
| swap                       |                                      |
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
