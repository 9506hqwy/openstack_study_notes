# a1.nano

## flavor の作成

FPGA を要求する flavor を作成する。

| オプション | 説明               |
| ---------- | ------------------ |
| --vcpus    | CPU 数             |
| --ram      | メモリサイズ MiB   |
| --disk     | ディスクサイズ GiB |

```sh
openstack flavor create \
    --ram 64 \
    --vcpus 1 \
    --disk 1 \
    a1.nano
```

```text
+----------------------------+--------------------------------------+
| Field                      | Value                                |
+----------------------------+--------------------------------------+
| OS-FLV-DISABLED:disabled   | False                                |
| OS-FLV-EXT-DATA:ephemeral  | 0                                    |
| description                | None                                 |
| disk                       | 1                                    |
| id                         | 9ac967af-d538-4bd6-acd0-f4eb77fb6e00 |
| name                       | a1.nano                              |
| os-flavor-access:is_public | True                                 |
| properties                 |                                      |
| ram                        | 64                                   |
| rxtx_factor                | 1.0                                  |
| swap                       |                                      |
| vcpus                      | 1                                    |
+----------------------------+--------------------------------------+
```

## デバイスプロファイルの設定

デバイスプロファイル fpga1 を設定する。

```sh
openstack flavor set --property 'accel:device_profile=fpga1' a1.nano
```
