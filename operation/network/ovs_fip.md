# Floating IP (Open vSwitch)

## 前提条件

* [](../network/ovs_router) を作成していること。
* [](../instance/ovs_vxlan.md) を作成していること。

## Floating IP の作成

ルータ provider に Floating IP を作成する。

```sh
openstack floating ip create provider
```

```
+---------------------+--------------------------------------+
| Field               | Value                                |
+---------------------+--------------------------------------+
| created_at          | 2024-05-01T11:16:30Z                 |
| description         |                                      |
| dns_domain          | None                                 |
| dns_name            | None                                 |
| fixed_ip_address    | None                                 |
| floating_ip_address | 172.17.0.95                          |
| floating_network_id | 85f372ef-f39a-42bb-a06b-9f9ed8e4e58e |
| id                  | 408f7570-2f57-4673-9cb0-c8ef8741d4d2 |
| name                | 172.17.0.95                          |
| port_details        | None                                 |
| port_id             | None                                 |
| project_id          | f2aeffb34ff34ffb8959f1cd813655c6     |
| qos_policy_id       | None                                 |
| revision_number     | 0                                    |
| router_id           | None                                 |
| status              | DOWN                                 |
| subnet_id           | None                                 |
| tags                | []                                   |
| updated_at          | 2024-05-01T11:16:30Z                 |
+---------------------+--------------------------------------+
```

## Floating IP の割り当て

インスタンス instance02 に Floating IP を割り当てる。

```sh
openstack server add floating ip instance02 172.17.0.95
```

割り当てを確認する。

```sh
openstack server list
```

```
+--------------------------------------+---------------------------+---------+-----------------------------------------+--------+---------+
| ID                                   | Name                      | Status  | Networks                                | Image  | Flavor  |
+--------------------------------------+---------------------------+---------+-----------------------------------------+--------+---------+
| 36b496b2-4976-4bf1-88c0-09f44385fd19 | instance02                | SHUTOFF | selfservice=172.17.0.95, 192.168.101.19 | cirros | m1.nano |
+--------------------------------------+---------------------------+---------+-----------------------------------------+--------+---------+
```

SSH の接続を確認する。

```sh
ssh -i demo_rsa cirros@172.17.0.95 /sbin/ip addr
```

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc pfifo_fast qlen 1000
    link/ether fa:16:3e:55:18:7a brd ff:ff:ff:ff:ff:ff
    inet 192.168.101.19/24 brd 192.168.101.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:fe55:187a/64 scope link
       valid_lft forever preferred_lft forever
```

## 環境の確認

イーサネットの情報を確認する。

```sh
ip netns exec qrouter-53a57af7-4963-4d2e-88c9-5ef0b999c507 ip addr show
```

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
15: qr-d43193bb-c2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ether fa:16:3e:53:06:1c brd ff:ff:ff:ff:ff:ff
    inet 192.168.101.1/24 brd 192.168.101.255 scope global qr-d43193bb-c2
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:fe53:61c/64 scope link
       valid_lft forever preferred_lft forever
16: qg-cfe4de6d-40: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ether fa:16:3e:9c:29:4a brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.97/12 brd 172.31.255.255 scope global qg-cfe4de6d-40
       valid_lft forever preferred_lft forever
    inet 172.17.0.95/32 brd 172.17.0.95 scope global qg-cfe4de6d-40
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:fe9c:294a/64 scope link
       valid_lft forever preferred_lft forever
```

iptables の設定を確認する。

```sh
ip netns exec qrouter-53a57af7-4963-4d2e-88c9-5ef0b999c507 iptables -n -t nat -L
```

DNAT と SNAT が設定されている。

```
(...)

Chain neutron-l3-agent-OUTPUT (1 references)
target     prot opt source               destination
DNAT       all  --  0.0.0.0/0            172.17.0.95          to:192.168.101.19

Chain neutron-l3-agent-PREROUTING (1 references)
target     prot opt source               destination
DNAT       all  --  0.0.0.0/0            172.17.0.95          to:192.168.101.19

Chain neutron-l3-agent-float-snat (1 references)
target     prot opt source               destination
SNAT       all  --  192.168.101.19       0.0.0.0/0            to:172.17.0.95 random-fully
```
