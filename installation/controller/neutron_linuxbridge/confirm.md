# 動作確認 (Linux Bridge)

エージェントを表示する。

```sh
openstack network agent list
```

```text
+--------------------------------------+--------------------+-----------------------+-------------------+-------+-------+---------------------------+
| ID                                   | Agent Type         | Host                  | Availability Zone | Alive | State | Binary                    |
+--------------------------------------+--------------------+-----------------------+-------------------+-------+-------+---------------------------+
| 0914ecb5-a702-417d-ab43-aa27935a9267 | DHCP agent         | controller.home.local | nova              | :-)   | UP    | neutron-dhcp-agent        |
| 259b7b21-9fbd-4b67-b703-6e155038e9b7 | Linux bridge agent | controller.home.local | None              | :-)   | UP    | neutron-linuxbridge-agent |
| 74b907db-30c1-4d6a-a7d9-5e9d080e583c | Metadata agent     | controller.home.local | None              | :-)   | UP    | neutron-metadata-agent    |
+--------------------------------------+--------------------+-----------------------+-------------------+-------+-------+---------------------------+
```
