# a1.milli

## flavor の作成

FPGA を要求する flavor を作成する。

| オプション | 説明               |
| ---------- | ------------------ |
| --vcpus    | CPU 数             |
| --ram      | メモリサイズ MiB   |
| --disk     | ディスクサイズ GiB |

```sh
openstack flavor create \
    --ram 256 \
    --vcpus 1 \
    --disk 1 \
    a1.milli
```

```text
+----------------------------+--------------------------------------+
| Field                      | Value                                |
+----------------------------+--------------------------------------+
| OS-FLV-DISABLED:disabled   | False                                |
| OS-FLV-EXT-DATA:ephemeral  | 0                                    |
| description                | None                                 |
| disk                       | 1                                    |
| id                         | 82979736-dea6-4e0f-a155-bcfbda0afd49 |
| name                       | a1.milli                             |
| os-flavor-access:is_public | True                                 |
| properties                 |                                      |
| ram                        | 256                                  |
| rxtx_factor                | 1.0                                  |
| swap                       | 0                                    |
| vcpus                      | 1                                    |
+----------------------------+--------------------------------------+
```

## デバイスプロファイルの設定

デバイスプロファイル fpga1 を設定する。

```sh
openstack flavor set --property 'accel:device_profile=fpga1' a1.milli
```
