# ICMP

ICMP を許可するルールを作成する。

```sh
openstack security group rule create --proto icmp mysecurity
```

```text
+-------------------------+--------------------------------------+
| Field                   | Value                                |
+-------------------------+--------------------------------------+
| belongs_to_default_sg   | False                                |
| created_at              | 2024-05-10T15:06:59Z                 |
| description             |                                      |
| direction               | ingress                              |
| ether_type              | IPv4                                 |
| id                      | 1bd83061-6436-4eae-91a9-7d345d275547 |
| name                    | None                                 |
| normalized_cidr         | 0.0.0.0/0                            |
| port_range_max          | None                                 |
| port_range_min          | None                                 |
| project_id              | bccf406c045d401b91ba5c7552a124ae     |
| protocol                | icmp                                 |
| remote_address_group_id | None                                 |
| remote_group_id         | None                                 |
| remote_ip_prefix        | 0.0.0.0/0                            |
| revision_number         | 0                                    |
| security_group_id       | 7a3d9999-6c97-42da-a755-f7c6a435049e |
| tags                    | []                                   |
| updated_at              | 2024-05-10T15:06:59Z                 |
+-------------------------+--------------------------------------+
```
