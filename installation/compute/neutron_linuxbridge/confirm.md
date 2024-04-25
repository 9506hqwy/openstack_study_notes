# 動作確認 (Linux Bridge)

Controller Node でエージェントを表示する。

```sh
openstack network agent list
```

```
+--------------------------------------+--------------------+-----------------------+-------------------+-------+-------+---------------------------+
| ID                                   | Agent Type         | Host                  | Availability Zone | Alive | State | Binary                    |
+--------------------------------------+--------------------+-----------------------+-------------------+-------+-------+---------------------------+
| 145d9d73-cb1c-4bbf-ad28-adb1a9f4a826 | Metadata agent     | controller.home.local | None              | :-)   | UP    | neutron-metadata-agent    |
| d8153174-094a-4c91-9650-8bce3042bad3 | Linux bridge agent | compute.home.local    | None              | :-)   | UP    | neutron-linuxbridge-agent |
| e843c356-67a9-418e-96db-3fa6e4210df9 | DHCP agent         | controller.home.local | nova              | :-)   | UP    | neutron-dhcp-agent        |
| f075f5df-7f2f-4482-881f-5f463387be89 | Linux bridge agent | controller.home.local | None              | :-)   | UP    | neutron-linuxbridge-agent |
+--------------------------------------+--------------------+-----------------------+-------------------+-------+-------+---------------------------+
```
