# FPGA

## 作成

FPGA を要求するデバイスプロファイルを作成する。

```sh
openstack accelerator device profile create \
    fpga1 \
    '[{"resources:FPGA":1, "trait:CUSTOM_FAKE_DEVICE": "required"}]'
```

```text
+-------------+-------------------------------------------------------------------+
| Field       | Value                                                             |
+-------------+-------------------------------------------------------------------+
| created_at  | 2024-05-18 16:27:45+00:00                                         |
| updated_at  | None                                                              |
| uuid        | ea92dafd-eed0-4890-8e3b-428f655e4568                              |
| name        | fpga1                                                             |
| groups      | [{'resources:FPGA': '1', 'trait:CUSTOM_FAKE_DEVICE': 'required'}] |
| description | None                                                              |
+-------------+-------------------------------------------------------------------+
```
