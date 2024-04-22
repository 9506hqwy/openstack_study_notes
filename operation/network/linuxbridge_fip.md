# Floating IP (Linux Bridge)

## 前提条件

* [](../network/linuxbridge_router) を作成していること。
* [](../instance/linuxbridge_vxlan.md) を作成していること。

## Floating IP の作成

ルータ provider に Floating IP を作成する。

```sh
openstack floating ip create provider
```

```
+---------------------+--------------------------------------+
| Field               | Value                                |
+---------------------+--------------------------------------+
| created_at          | 2024-04-16T11:33:23Z                 |
| description         |                                      |
| dns_domain          | None                                 |
| dns_name            | None                                 |
| fixed_ip_address    | None                                 |
| floating_ip_address | 172.17.0.86                          |
| floating_network_id | ca4e2bc3-fe44-48d0-8096-20e6a85d6510 |
| id                  | 42e961ba-6c8b-419d-9cdf-4b04da8eac05 |
| name                | 172.17.0.86                          |
| port_details        | None                                 |
| port_id             | None                                 |
| project_id          | f2aeffb34ff34ffb8959f1cd813655c6     |
| qos_policy_id       | None                                 |
| revision_number     | 0                                    |
| router_id           | None                                 |
| status              | DOWN                                 |
| subnet_id           | None                                 |
| tags                | []                                   |
| updated_at          | 2024-04-16T11:33:23Z                 |
+---------------------+--------------------------------------+
```

## Floating IP の割り当て

インスタンス instance02 に Floating IP を割り当てる。

```sh
openstack server add floating ip instance02 172.17.0.86
```

割り当てを確認する。

```sh
openstack server list
```

```
+--------------------------------------+------------+---------+-----------------------------------------+--------+---------+
| ID                                   | Name       | Status  | Networks                                | Image  | Flavor  |
+--------------------------------------+------------+---------+-----------------------------------------+--------+---------+
| 1bd994c2-a0f6-4f14-953a-ebd0de6b4adf | instance02 | ACTIVE  | selfservice=172.17.0.86, 192.168.101.77 | cirros | m1.nano |
+--------------------------------------+------------+---------+-----------------------------------------+--------+---------+
```

SSH の接続を確認する。

```sh
ssh -i demo_rsa cirros@172.17.0.86 /sbin/ip addr
```

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc pfifo_fast qlen 1000
    link/ether fa:16:3e:57:cd:11 brd ff:ff:ff:ff:ff:ff
    inet 192.168.101.77/24 brd 192.168.101.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:fe57:cd11/64 scope link
       valid_lft forever preferred_lft forever
```

## 環境の確認

イーサネットの情報を確認する。

```sh
ip netns exec qrouter-d0e3870f-c70a-4165-af86-2e34ec001967 ip addr show
```

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: qr-d32eb4b3-78@if13: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP group default qlen 1000
    link/ether fa:16:3e:a7:f3:e2 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.101.1/24 brd 192.168.101.255 scope global qr-d32eb4b3-78
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:fea7:f3e2/64 scope link
       valid_lft forever preferred_lft forever
3: qg-6a1873e6-21@if14: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether fa:16:3e:58:15:56 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.135/12 brd 172.31.255.255 scope global qg-6a1873e6-21
       valid_lft forever preferred_lft forever
    inet 172.17.0.86/32 brd 172.17.0.86 scope global qg-6a1873e6-21
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:fe58:1556/64 scope link
       valid_lft forever preferred_lft forever
```

iptables の設定を確認する。

```sh
ip netns exec qrouter-d0e3870f-c70a-4165-af86-2e34ec001967 iptables -n -t nat -L
```

DNAT と SNAT が設定されている。

```
(...)

Chain neutron-l3-agent-OUTPUT (1 references)
target     prot opt source               destination
DNAT       all  --  0.0.0.0/0            172.17.0.86          to:192.168.101.77

Chain neutron-l3-agent-PREROUTING (1 references)
target     prot opt source               destination
DNAT       all  --  0.0.0.0/0            172.17.0.86          to:192.168.101.77

Chain neutron-l3-agent-float-snat (1 references)
target     prot opt source               destination
SNAT       all  --  192.168.101.77       0.0.0.0/0            to:172.17.0.86 random-fully
```

