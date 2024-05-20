# メモリ

## 対象の確認

監視対象を確認する。

```sh
openstack server list
```

```
+--------------------------------------+------------+--------+-----------------------+-----------+----------+
| ID                                   | Name       | Status | Networks              | Image     | Flavor   |
+--------------------------------------+------------+--------+-----------------------+-----------+----------+
| a8fbb935-7f2b-49a5-983a-1f6e78a1e0ed | instance00 | ACTIVE | provider=172.16.0.169 | cirros062 | m1.milli |
+--------------------------------------+------------+--------+-----------------------+-----------+----------+
```

## アラームの作成

```{tip}
myuser で実行
```

メモリ使用率のアラームを作成する。

```sh
openstack alarm create  \
    --name mem_ave \
    --type gnocchi_resources_threshold \
    --description 'Memory Usage' \
    --metric memory.usage \
    --threshold 30.0 \
    --comparison-operator gt \
    --aggregation-method mean \
    --granularity 300 \
    --evaluation-periods 1 \
    --resource-type instance \
    --resource-id a8fbb935-7f2b-49a5-983a-1f6e78a1e0ed
```

```
+---------------------------+--------------------------------------+
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| aggregation_method        | mean                                 |
| alarm_actions             | []                                   |
| alarm_id                  | b67294e3-3006-4a9e-a366-3e8011145d3c |
| comparison_operator       | gt                                   |
| description               | Memory Usage                         |
| enabled                   | True                                 |
| evaluate_timestamp        | 2024-05-19T00:26:50.201043           |
| evaluation_periods        | 1                                    |
| granularity               | 300                                  |
| insufficient_data_actions | []                                   |
| metric                    | memory.usage                         |
| name                      | mem_ave                              |
| ok_actions                | []                                   |
| project_id                | bccf406c045d401b91ba5c7552a124ae     |
| repeat_actions            | False                                |
| resource_id               | a8fbb935-7f2b-49a5-983a-1f6e78a1e0ed |
| resource_type             | instance                             |
| severity                  | low                                  |
| state                     | insufficient data                    |
| state_reason              | Not evaluated yet                    |
| state_timestamp           | 2024-05-19T00:26:50.152841           |
| threshold                 | 30.0                                 |
| time_constraints          | []                                   |
| timestamp                 | 2024-05-19T00:26:50.152841           |
| type                      | gnocchi_resources_threshold          |
| user_id                   | 7f3acb28d26943bab9510df3a6edf3b0     |
+---------------------------+--------------------------------------+
```

アラームを確認する。

```sh
openstack alarm list
```

```
+--------------------------------------+-----------------------------+---------+-------------------+----------+---------+
| alarm_id                             | type                        | name    | state             | severity | enabled |
+--------------------------------------+-----------------------------+---------+-------------------+----------+---------+
| b67294e3-3006-4a9e-a366-3e8011145d3c | gnocchi_resources_threshold | mem_ave | insufficient data | low      | True    |
+--------------------------------------+-----------------------------+---------+-------------------+----------+---------+
```

## アラームの評価

閾値を超えると `state` が `alarm` になる。

```sh
openstack alarm list
```

```
+--------------------------------------+-----------------------------+---------+-------------------+----------+---------+
| alarm_id                             | type                        | name    | state             | severity | enabled |
+--------------------------------------+-----------------------------+---------+-------------------+----------+---------+
| b67294e3-3006-4a9e-a366-3e8011145d3c | gnocchi_resources_threshold | mem_ave | alarm             | low      | True    |
+--------------------------------------+-----------------------------+---------+-------------------+----------+---------+
```
