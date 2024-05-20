# Floating IP (Open vSwitch)

## 前提条件

* [](../network/ovs_router) を作成していること。
* [](../instance/ovs_vxlan.md) を作成していること。

## Floating IP の作成

```{tip}
myuser で実行
```

ルータ provider に Floating IP を作成する。

```sh
openstack floating ip create provider
```

```
+---------------------+--------------------------------------+
| Field               | Value                                |
+---------------------+--------------------------------------+
| created_at          | 2024-05-18T01:04:53Z                 |
| description         |                                      |
| dns_domain          | None                                 |
| dns_name            | None                                 |
| fixed_ip_address    | None                                 |
| floating_ip_address | 172.16.0.175                         |
| floating_network_id | 53f92676-e6e0-42fb-bde9-48dd2c2506b4 |
| id                  | cf8c7b11-f261-4555-82c3-0ec90c5eafce |
| name                | 172.16.0.175                         |
| port_details        | None                                 |
| port_id             | None                                 |
| project_id          | bccf406c045d401b91ba5c7552a124ae     |
| qos_policy_id       | None                                 |
| revision_number     | 0                                    |
| router_id           | None                                 |
| status              | DOWN                                 |
| subnet_id           | None                                 |
| tags                | []                                   |
| updated_at          | 2024-05-18T01:04:53Z                 |
+---------------------+--------------------------------------+
```

## Floating IP の割り当て

インスタンス instance02 に Floating IP を割り当てる。

```sh
openstack server add floating ip instance02 172.16.0.175
```

割り当てを確認する。

```sh
openstack server list
```

```
+--------------------------------------+------------+---------+------------------------------------------+-----------+----------+
| ID                                   | Name       | Status  | Networks                                 | Image     | Flavor   |
+--------------------------------------+------------+---------+------------------------------------------+-----------+----------+
| 2337b0eb-372c-43b8-923e-90a89337d211 | instance02 | SHUTOFF | selfservice=172.16.0.175, 192.168.101.85 | cirros062 | m1.milli |
+--------------------------------------+------------+---------+------------------------------------------+-----------+----------+
```

SSH の接続を確認する。

```sh
ssh -i demo_rsa cirros@172.16.0.175 /sbin/ip addr
```

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc pfifo_fast qlen 1000
    link/ether fa:16:3e:61:21:c6 brd ff:ff:ff:ff:ff:ff
    inet 192.168.101.85/24 brd 192.168.101.255 scope global dynamic noprefixroute eth0
       valid_lft 86391sec preferred_lft 75591sec
    inet6 fe80::f816:3eff:fe61:21c6/64 scope link
       valid_lft forever preferred_lft forever
```

## 環境の確認

イーサネットの情報を確認する。

```sh
ip netns exec qrouter-e6efd80e-9468-4787-a6f8-1648486cc1d6 ip addr show
```

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
18: qr-13e6e8ff-d7: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ether fa:16:3e:38:32:fe brd ff:ff:ff:ff:ff:ff
    inet 192.168.101.254/24 brd 192.168.101.255 scope global qr-13e6e8ff-d7
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:fe38:32fe/64 scope link
       valid_lft forever preferred_lft forever
19: qg-ca92be94-79: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ether fa:16:3e:a6:11:48 brd ff:ff:ff:ff:ff:ff
    inet 172.16.0.188/24 brd 172.16.0.255 scope global qg-ca92be94-79
       valid_lft forever preferred_lft forever
    inet 172.16.0.175/32 brd 172.16.0.175 scope global qg-ca92be94-79
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:fea6:1148/64 scope link
       valid_lft forever preferred_lft forever
```

iptables の設定を確認する。

```sh
ip netns exec qrouter-e6efd80e-9468-4787-a6f8-1648486cc1d6 iptables -n -t nat -L
```

DNAT と SNAT が設定されている。

```
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination
neutron-l3-agent-PREROUTING  0    --  0.0.0.0/0            0.0.0.0/0

Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
neutron-l3-agent-OUTPUT  0    --  0.0.0.0/0            0.0.0.0/0

Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination
neutron-l3-agent-POSTROUTING  0    --  0.0.0.0/0            0.0.0.0/0
neutron-postrouting-bottom  0    --  0.0.0.0/0            0.0.0.0/0

Chain neutron-l3-agent-OUTPUT (1 references)
target     prot opt source               destination
DNAT       0    --  0.0.0.0/0            172.16.0.175         to:192.168.101.85

Chain neutron-l3-agent-POSTROUTING (1 references)
target     prot opt source               destination
ACCEPT     0    --  0.0.0.0/0            0.0.0.0/0            ! ctstate DNAT

Chain neutron-l3-agent-PREROUTING (1 references)
target     prot opt source               destination
REDIRECT   6    --  0.0.0.0/0            169.254.169.254      tcp dpt:80 redir ports 9697
DNAT       0    --  0.0.0.0/0            172.16.0.175         to:192.168.101.85

Chain neutron-l3-agent-float-snat (1 references)
target     prot opt source               destination
SNAT       0    --  192.168.101.85       0.0.0.0/0            to:172.16.0.175 random-fully

Chain neutron-l3-agent-snat (1 references)
target     prot opt source               destination
neutron-l3-agent-float-snat  0    --  0.0.0.0/0            0.0.0.0/0
SNAT       0    --  0.0.0.0/0            0.0.0.0/0            to:172.16.0.188 random-fully
SNAT       0    --  0.0.0.0/0            0.0.0.0/0            mark match ! 0x2/0xffff ctstate DNAT to:172.16.0.188 random-fully

Chain neutron-postrouting-bottom (1 references)
target     prot opt source               destination
neutron-l3-agent-snat  0    --  0.0.0.0/0            0.0.0.0/0            /* Perform source NAT on outgoing traffic. */
```
