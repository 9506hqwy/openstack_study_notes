# ICMP

ICMP を許可するルールを作成する。

```sh
openstack security group rule create --proto icmp default
```

```
+-------------------------+--------------------------------------+
| Field                   | Value                                |
+-------------------------+--------------------------------------+
| created_at              | 2024-04-13T10:33:27Z                 |
| description             |                                      |
| direction               | ingress                              |
| ether_type              | IPv4                                 |
| id                      | f9975a36-959c-4f7a-abfa-835547b4e39c |
| name                    | None                                 |
| port_range_max          | None                                 |
| port_range_min          | None                                 |
| project_id              | f2aeffb34ff34ffb8959f1cd813655c6     |
| protocol                | icmp                                 |
| remote_address_group_id | None                                 |
| remote_group_id         | None                                 |
| remote_ip_prefix        | 0.0.0.0/0                            |
| revision_number         | 0                                    |
| security_group_id       | 87fd4685-d317-42fb-a487-28382d2c2750 |
| tags                    | []                                   |
| tenant_id               | f2aeffb34ff34ffb8959f1cd813655c6     |
| updated_at              | 2024-04-13T10:33:27Z                 |
+-------------------------+--------------------------------------+
```
