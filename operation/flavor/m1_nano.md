# m1.nano

## flavor の作成

1CPU, 64MiB の flavor を作成する。

| オプション                  | 説明                         |
| --------------------------- | ---------------------------- |
| --id                        | 一意な ID を指定             |
| --vcpus                     | CPU 数                       |
| --ram                       | メモリサイズ MiB             |
| --disk                      | ディスクサイズ GiB           |

```sh
openstack flavor create --id 0 --vcpus 1 --ram 64 --disk 1 m1.nano
```

```
+----------------------------+---------+
| Field                      | Value   |
+----------------------------+---------+
| OS-FLV-DISABLED:disabled   | False   |
| OS-FLV-EXT-DATA:ephemeral  | 0       |
| description                | None    |
| disk                       | 1       |
| id                         | 0       |
| name                       | m1.nano |
| os-flavor-access:is_public | True    |
| properties                 |         |
| ram                        | 64      |
| rxtx_factor                | 1.0     |
| swap                       | 0       |
| vcpus                      | 1       |
+----------------------------+---------+
```
