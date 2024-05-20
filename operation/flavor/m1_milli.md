# m1.milli

## flavor の作成

1CPU, 64MiB の flavor を作成する。

| オプション                  | 説明                         |
| --------------------------- | ---------------------------- |
| --id                        | 一意な ID を指定             |
| --vcpus                     | CPU 数                       |
| --ram                       | メモリサイズ MiB             |
| --disk                      | ディスクサイズ GiB           |

```sh
openstack flavor create --id 1 --vcpus 1 --ram 256 --disk 1 m1.milli
```

```
+----------------------------+----------+
| Field                      | Value    |
+----------------------------+----------+
| OS-FLV-DISABLED:disabled   | False    |
| OS-FLV-EXT-DATA:ephemeral  | 0        |
| description                | None     |
| disk                       | 1        |
| id                         | 1        |
| name                       | m1.milli |
| os-flavor-access:is_public | True     |
| properties                 |          |
| ram                        | 256      |
| rxtx_factor                | 1.0      |
| swap                       | 0        |
| vcpus                      | 1        |
+----------------------------+----------+
```
