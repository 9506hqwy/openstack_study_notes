# FPGA

## 作成

FPGA を要求するデバイスプロファイルを作成する。

```sh
openstack accelerator device profile create \
    fpga1 \
    '[{"resources:FPGA":1, "trait:CUSTOM_FAKE_DEVICE": "required"}]'
```

```
+-------------+-------------------------------------------------------------------+
| Field       | Value                                                             |
+-------------+-------------------------------------------------------------------+
| created_at  | 2024-05-06 03:51:38+00:00                                         |
| updated_at  | None                                                              |
| uuid        | 7e2d3f5e-8065-4cd2-a593-4cfba8e540c0                              |
| name        | fpga1                                                             |
| groups      | [{'resources:FPGA': '1', 'trait:CUSTOM_FAKE_DEVICE': 'required'}] |
| description | None                                                              |
+-------------+-------------------------------------------------------------------+
```
