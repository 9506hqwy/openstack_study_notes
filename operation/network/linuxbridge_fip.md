# Floating IP (Linux Bridge)

## 前提条件

* [](../network/linuxbridge_router) を作成していること。
* [](../instance/linuxbridge_vxlan.md) を作成していること。

## Floating IP の作成

```{tip}
myuser で実行
```

ルータ provider に Floating IP を作成する。

```sh
openstack floating ip create provider
```

```text
+---------------------+--------------------------------------+
| Field               | Value                                |
+---------------------+--------------------------------------+
| created_at          | 2024-05-12T13:44:32Z                 |
| description         |                                      |
| dns_domain          | None                                 |
| dns_name            | None                                 |
| fixed_ip_address    | None                                 |
| floating_ip_address | 172.16.0.168                         |
| floating_network_id | 83a19a08-f066-465f-a5e1-23b4fc66e5ac |
| id                  | d0d7222b-39ca-4d7d-8351-f166fe5cb558 |
| name                | 172.16.0.168                         |
| port_details        | None                                 |
| port_id             | None                                 |
| project_id          | bccf406c045d401b91ba5c7552a124ae     |
| qos_policy_id       | None                                 |
| revision_number     | 0                                    |
| router_id           | None                                 |
| status              | DOWN                                 |
| subnet_id           | None                                 |
| tags                | []                                   |
| updated_at          | 2024-05-12T13:44:32Z                 |
+---------------------+--------------------------------------+
```

## Floating IP の割り当て

インスタンス instance02 に Floating IP を割り当てる。

```sh
openstack server add floating ip instance02 172.16.0.168
```

割り当てを確認する。

```sh
openstack server list
```

```text
+--------------------------------------+------------+---------+------------------------------------------+-----------+----------+
| ID                                   | Name       | Status  | Networks                                 | Image     | Flavor   |
+--------------------------------------+------------+---------+------------------------------------------+-----------+----------+
| df970ec9-8410-48a6-9f04-d4621c967fce | instance02 | SHUTOFF | selfservice=172.16.0.168, 192.168.101.88 | cirros062 | m1.milli |
+--------------------------------------+------------+---------+------------------------------------------+-----------+----------+
```

SSH の接続を確認する。

```sh
ssh -i demo_rsa cirros@172.16.0.168 /sbin/ip addr
```

```text
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc pfifo_fast qlen 1000
    link/ether fa:16:3e:be:71:21 brd ff:ff:ff:ff:ff:ff
    inet 192.168.101.88/24 brd 192.168.101.255 scope global dynamic noprefixroute eth0
       valid_lft 86369sec preferred_lft 75569sec
    inet6 fe80::f816:3eff:febe:7121/64 scope link
       valid_lft forever preferred_lft forever
```

## 環境の確認

イーサネットの情報を確認する。

```sh
ip netns exec qrouter-fa9a921e-7ff6-4468-a08b-5415cebeb089 ip addr show
```

```text
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: qr-d8316879-dd@if15: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP group default qlen 1000
    link/ether fa:16:3e:af:a9:e5 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.101.254/24 brd 192.168.101.255 scope global qr-d8316879-dd
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:feaf:a9e5/64 scope link
       valid_lft forever preferred_lft forever
3: qg-d8a9d7f6-47@if16: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether fa:16:3e:c5:74:4d brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.16.0.131/24 brd 172.16.0.255 scope global qg-d8a9d7f6-47
       valid_lft forever preferred_lft forever
    inet 172.16.0.168/32 brd 172.16.0.168 scope global qg-d8a9d7f6-47
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:fec5:744d/64 scope link
       valid_lft forever preferred_lft forever
```

iptables の設定を確認する。

```sh
ip netns exec qrouter-fa9a921e-7ff6-4468-a08b-5415cebeb089 iptables -n -t nat -L
```

DNAT と SNAT が設定されている。

```text
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
DNAT       0    --  0.0.0.0/0            172.16.0.168         to:192.168.101.88

Chain neutron-l3-agent-POSTROUTING (1 references)
target     prot opt source               destination
ACCEPT     0    --  0.0.0.0/0            0.0.0.0/0            ! ctstate DNAT

Chain neutron-l3-agent-PREROUTING (1 references)
target     prot opt source               destination
REDIRECT   6    --  0.0.0.0/0            169.254.169.254      tcp dpt:80 redir ports 9697
DNAT       0    --  0.0.0.0/0            172.16.0.168         to:192.168.101.88

Chain neutron-l3-agent-float-snat (1 references)
target     prot opt source               destination
SNAT       0    --  192.168.101.88       0.0.0.0/0            to:172.16.0.168 random-fully

Chain neutron-l3-agent-snat (1 references)
target     prot opt source               destination
neutron-l3-agent-float-snat  0    --  0.0.0.0/0            0.0.0.0/0
SNAT       0    --  0.0.0.0/0            0.0.0.0/0            to:172.16.0.131 random-fully
SNAT       0    --  0.0.0.0/0            0.0.0.0/0            mark match ! 0x2/0xffff ctstate DNAT to:172.16.0.131 random-fully

Chain neutron-postrouting-bottom (1 references)
target     prot opt source               destination
neutron-l3-agent-snat  0    --  0.0.0.0/0            0.0.0.0/0            /* Perform source NAT on outgoing traffic. */
```
