# SSH

SSH を許可するルールを作成する。

```sh
openstack security group rule create --proto tcp --dst-port 22 default
```

```
+-------------------------+--------------------------------------+
| Field                   | Value                                |
+-------------------------+--------------------------------------+
| created_at              | 2024-04-13T10:33:41Z                 |
| description             |                                      |
| direction               | ingress                              |
| ether_type              | IPv4                                 |
| id                      | 7d523668-89d7-4cb2-b035-e4dde8ec5be5 |
| name                    | None                                 |
| port_range_max          | 22                                   |
| port_range_min          | 22                                   |
| project_id              | f2aeffb34ff34ffb8959f1cd813655c6     |
| protocol                | tcp                                  |
| remote_address_group_id | None                                 |
| remote_group_id         | None                                 |
| remote_ip_prefix        | 0.0.0.0/0                            |
| revision_number         | 0                                    |
| security_group_id       | 87fd4685-d317-42fb-a487-28382d2c2750 |
| tags                    | []                                   |
| tenant_id               | f2aeffb34ff34ffb8959f1cd813655c6     |
| updated_at              | 2024-04-13T10:33:41Z                 |
+-------------------------+--------------------------------------+
```
