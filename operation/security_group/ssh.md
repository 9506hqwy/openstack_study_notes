# SSH

SSH を許可するルールを作成する。

```sh
openstack security group rule create --proto tcp --dst-port 22 mysecurity
```

```text
+-------------------------+--------------------------------------+
| Field                   | Value                                |
+-------------------------+--------------------------------------+
| belongs_to_default_sg   | False                                |
| created_at              | 2024-05-10T15:07:14Z                 |
| description             |                                      |
| direction               | ingress                              |
| ether_type              | IPv4                                 |
| id                      | 39b1a482-c246-4a0d-bc7d-21d2dd3ea29d |
| name                    | None                                 |
| normalized_cidr         | 0.0.0.0/0                            |
| port_range_max          | 22                                   |
| port_range_min          | 22                                   |
| project_id              | bccf406c045d401b91ba5c7552a124ae     |
| protocol                | tcp                                  |
| remote_address_group_id | None                                 |
| remote_group_id         | None                                 |
| remote_ip_prefix        | 0.0.0.0/0                            |
| revision_number         | 0                                    |
| security_group_id       | 7a3d9999-6c97-42da-a755-f7c6a435049e |
| tags                    | []                                   |
| updated_at              | 2024-05-10T15:07:14Z                 |
+-------------------------+--------------------------------------+
```
