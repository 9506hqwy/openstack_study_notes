# メモリ

## 対象の確認

監視対象を確認する。

```sh
openstack server list
```

```
+--------------------------------------+---------------------------+---------+-----------------------------------------+--------+---------+
| ID                                   | Name                      | Status  | Networks                                | Image  | Flavor  |
+--------------------------------------+---------------------------+---------+-----------------------------------------+--------+---------+
| 36b496b2-4976-4bf1-88c0-09f44385fd19 | instance02                | ACTIVE  | selfservice=172.17.0.95, 192.168.101.19 | cirros | m1.nano |
+--------------------------------------+---------------------------+---------+-----------------------------------------+--------+---------+
```

## アラームの作成

メモリ使用率のアラームを作成する。

```sh
openstack alarm create  \
    --name mem_ave \
    --type gnocchi_resources_threshold \
    --description 'Memory Usage' \
    --metric memory \
    --threshold 60.0 \
    --comparison-operator gt \
    --aggregation-method mean \
    --granularity 300 \
    --evaluation-periods 1 \
    --resource-type instance \
    --resource-id 36b496b2-4976-4bf1-88c0-09f44385fd19
```

```
+---------------------------+--------------------------------------+
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| aggregation_method        | mean                                 |
| alarm_actions             | []                                   |
| alarm_id                  | 2475246c-70d5-4858-aa82-b93b9c7685ee |
| comparison_operator       | gt                                   |
| description               | Memory Usage                         |
| enabled                   | True                                 |
| evaluate_timestamp        | 2024-05-04T03:57:23.412646           |
| evaluation_periods        | 1                                    |
| granularity               | 300                                  |
| insufficient_data_actions | []                                   |
| metric                    | memory                               |
| name                      | mem_ave                              |
| ok_actions                | []                                   |
| project_id                | f2aeffb34ff34ffb8959f1cd813655c6     |
| repeat_actions            | False                                |
| resource_id               | 36b496b2-4976-4bf1-88c0-09f44385fd19 |
| resource_type             | instance                             |
| severity                  | low                                  |
| state                     | insufficient data                    |
| state_reason              | Not evaluated yet                    |
| state_timestamp           | 2024-05-04T03:57:23.389488           |
| threshold                 | 60.0                                 |
| time_constraints          | []                                   |
| timestamp                 | 2024-05-04T03:57:23.389488           |
| type                      | gnocchi_resources_threshold          |
| user_id                   | 71b5948c75f24c0f841dbf1c4eb4c4a7     |
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
| 2475246c-70d5-4858-aa82-b93b9c7685ee | gnocchi_resources_threshold | mem_ave | insufficient data | low      | True    |
+--------------------------------------+-----------------------------+---------+-------------------+----------+---------+
```

## アラームの評価

閾値を超えると `state` が `alarm` になる。

```sh
openstack alarm list
```

```
+--------------------------------------+-----------------------------+---------+-------+----------+---------+
| alarm_id                             | type                        | name    | state | severity | enabled |
+--------------------------------------+-----------------------------+---------+-------+----------+---------+
| 2475246c-70d5-4858-aa82-b93b9c7685ee | gnocchi_resources_threshold | mem_ave | alarm | low      | True    |
+--------------------------------------+-----------------------------+---------+-------+----------+---------+
```
