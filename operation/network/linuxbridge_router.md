# ルータ (Linux Bridge)

## 前提条件

* [](../network/linuxbridge_flat) を作成していること。
* [](../network/linuxbridge_vxlan) を作成していること。

## ルータの作成

```{tip}
myuser で実行
```

ルータを作成する。

```sh
openstack router create router
```

```text
+-------------------------+--------------------------------------+
| Field                   | Value                                |
+-------------------------+--------------------------------------+
| admin_state_up          | UP                                   |
| availability_zone_hints |                                      |
| availability_zones      |                                      |
| created_at              | 2024-05-12T12:01:27Z                 |
| description             |                                      |
| distributed             | False                                |
| enable_ndp_proxy        | None                                 |
| external_gateway_info   | null                                 |
| flavor_id               | None                                 |
| ha                      | False                                |
| id                      | fa9a921e-7ff6-4468-a08b-5415cebeb089 |
| name                    | router                               |
| project_id              | bccf406c045d401b91ba5c7552a124ae     |
| revision_number         | 1                                    |
| routes                  |                                      |
| status                  | ACTIVE                               |
| tags                    |                                      |
| tenant_id               | bccf406c045d401b91ba5c7552a124ae     |
| updated_at              | 2024-05-12T12:01:27Z                 |
+-------------------------+--------------------------------------+
```

## サブネットに接続

ルータ router をサブネット selfservice に接続する。

```sh
openstack router add subnet router selfservice
```

## ゲートウェイの設定

ルータ router のゲートウェイを設定する。

```sh
openstack router set router --external-gateway provider
```

## ポートの確認

ルータ router に作成されたポートを確認する。

```sh
openstack port list --router router
```

```text
+--------------------------------------+------+-------------------+--------------------------------------------------------------------------------+--------+
| ID                                   | Name | MAC Address       | Fixed IP Addresses                                                             | Status |
+--------------------------------------+------+-------------------+--------------------------------------------------------------------------------+--------+
| d8316879-dded-4124-8e6b-55036c29edfc |      | fa:16:3e:af:a9:e5 | ip_address='192.168.101.254', subnet_id='8abd1d19-1fed-412a-83d0-39e098708648' | ACTIVE |
| d8a9d7f6-47b1-450a-b21f-0ab25a8f334e |      | fa:16:3e:c5:74:4d | ip_address='172.16.0.131', subnet_id='7555697f-f6f4-4b00-af7b-e06bd9997226'    | ACTIVE |
+--------------------------------------+------+-------------------+--------------------------------------------------------------------------------+--------+
```

## 環境の確認

Controller Node でネットワーク構成を確認する。

![Linux Bridge ルータ](../../_static/image/network_vxlan_router_linuxbridge.png "Linux Bridge ルータ")

### ネットワーク名前空間

ルータを作成するとネットワーク名前空間が作成される。

```sh
ip netns
```

```text
(...)

qrouter-fa9a921e-7ff6-4468-a08b-5415cebeb089 (id: 3)
```

### デバイス

ブリッジと veth peer が作成される。

```sh
ip -d link show
```

```text
(...)

4: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq master brq83a19a08-f0 state UP mode DEFAULT group default qlen 1000
    link/ether 00:15:5d:bf:ba:55 brd ff:ff:ff:ff:ff:ff promiscuity 2  allmulti 2 minmtu 68 maxmtu 65521
    bridge_slave state forwarding priority 32 cost 2 hairpin off guard off root_block off fastleave off learning on flood on port_id 0x8001 port_no 0x1 designated_port 32769 designated_cost 0 designated_bridge 8000.0:15:5d:bf:ba:55 designated_root 8000.0:15:5d:bf:ba:55 hold_timer    0.00 message_age_timer    0.00 forward_delay_timer    0.00 topology_change_ack 0 config_pending 0 proxy_arp off proxy_arp_wifi off mcast_router 1 mcast_fast_leave off mcast_flood on bcast_flood on mcast_to_unicast off neigh_suppress off group_fwd_mask 0 group_fwd_mask_str 0x0 vlan_tunnel off isolated off locked off mab off addrgenmode none numtxqueues 64 numrxqueues 64 gso_max_size 62780 gso_max_segs 65535 tso_max_size 62780 tso_max_segs 65535 gro_max_size 65536 parentbus vmbus parentdev dffbd9a0-19dd-44c1-9b46-6dfba9829d73
9: brq83a19a08-f0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 00:15:5d:bf:ba:55 brd ff:ff:ff:ff:ff:ff promiscuity 0  allmulti 0 minmtu 68 maxmtu 65535
    bridge forward_delay 0 hello_time 200 max_age 2000 ageing_time 30000 stp_state 0 priority 32768 vlan_filtering 0 vlan_protocol 802.1Q bridge_id 8000.0:15:5d:bf:ba:55 designated_root 8000.0:15:5d:bf:ba:55 root_port 0 root_path_cost 0 topology_change 0 topology_change_detected 0 hello_timer    0.00 tcn_timer    0.00 topology_change_timer    0.00 gc_timer   22.73 vlan_default_pvid 1 vlan_stats_enabled 0 vlan_stats_per_port 0 group_fwd_mask 0 group_address 01:80:c2:00:00:00 mcast_snooping 1 no_linklocal_learn 0 mcast_vlan_snooping 0 mcast_router 1 mcast_query_use_ifaddr 0 mcast_querier 0 mcast_hash_elasticity 16 mcast_hash_max 4096 mcast_last_member_count 2 mcast_startup_query_count 2 mcast_last_member_interval 100 mcast_membership_interval 26000 mcast_querier_interval 25500 mcast_query_interval 12500 mcast_query_response_interval 1000 mcast_startup_query_interval 3125 mcast_stats_enabled 0 mcast_igmp_version 2 mcast_mld_version 1 nf_call_iptables 0 nf_call_ip6tables 0 nf_call_arptables 0 addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 62780 gso_max_segs 65535 tso_max_size 62780 tso_max_segs 65535 gro_max_size 65536
13: vxlan-267: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue master brqc08e7dcd-4a state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether ee:82:e6:12:b6:e3 brd ff:ff:ff:ff:ff:ff promiscuity 1  allmulti 1 minmtu 68 maxmtu 65535
    vxlan id 267 local 172.16.0.11 dev eth0 srcport 0 0 dstport 8472 ttl auto ageing 300 udpcsum noudp6zerocsumtx noudp6zerocsumrx
    bridge_slave state forwarding priority 32 cost 2 hairpin off guard off root_block off fastleave off learning on flood on port_id 0x8001 port_no 0x1 designated_port 32769 designated_cost 0 designated_bridge 8000.2a:f1:a8:1a:54:95 designated_root 8000.2a:f1:a8:1a:54:95 hold_timer    0.00 message_age_timer    0.00 forward_delay_timer    0.00 topology_change_ack 0 config_pending 0 proxy_arp off proxy_arp_wifi off mcast_router 1 mcast_fast_leave off mcast_flood on bcast_flood on mcast_to_unicast off neigh_suppress off group_fwd_mask 0 group_fwd_mask_str 0x0 vlan_tunnel off isolated off locked off mab off addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 62780 gso_max_segs 65535 tso_max_size 62780 tso_max_segs 65535 gro_max_size 65536
14: brqc08e7dcd-4a: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 2a:f1:a8:1a:54:95 brd ff:ff:ff:ff:ff:ff promiscuity 0  allmulti 0 minmtu 68 maxmtu 65535
    bridge forward_delay 0 hello_time 200 max_age 2000 ageing_time 30000 stp_state 0 priority 32768 vlan_filtering 0 vlan_protocol 802.1Q bridge_id 8000.2a:f1:a8:1a:54:95 designated_root 8000.2a:f1:a8:1a:54:95 root_port 0 root_path_cost 0 topology_change 0 topology_change_detected 0 hello_timer    0.00 tcn_timer    0.00 topology_change_timer    0.00 gc_timer    0.00 vlan_default_pvid 1 vlan_stats_enabled 0 vlan_stats_per_port 0 group_fwd_mask 0 group_address 01:80:c2:00:00:00 mcast_snooping 1 no_linklocal_learn 0 mcast_vlan_snooping 0 mcast_router 1 mcast_query_use_ifaddr 0 mcast_querier 0 mcast_hash_elasticity 16 mcast_hash_max 4096 mcast_last_member_count 2 mcast_startup_query_count 2 mcast_last_member_interval 100 mcast_membership_interval 26000 mcast_querier_interval 25500 mcast_query_interval 12500 mcast_query_response_interval 1000 mcast_startup_query_interval 3125 mcast_stats_enabled 0 mcast_igmp_version 2 mcast_mld_version 1 nf_call_iptables 0 nf_call_ip6tables 0 nf_call_arptables 0 addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 62780 gso_max_segs 65535 tso_max_size 62780 tso_max_segs 65535 gro_max_size 65536
15: tapd8316879-dd@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue master brqc08e7dcd-4a state UP mode DEFAULT group default qlen 1000
    link/ether 2a:f1:a8:1a:54:95 brd ff:ff:ff:ff:ff:ff link-netns qrouter-fa9a921e-7ff6-4468-a08b-5415cebeb089 promiscuity 1  allmulti 1 minmtu 68 maxmtu 65535
    veth
    bridge_slave state forwarding priority 32 cost 2 hairpin off guard off root_block off fastleave off learning on flood on port_id 0x8003 port_no 0x3 designated_port 32771 designated_cost 0 designated_bridge 8000.2a:f1:a8:1a:54:95 designated_root 8000.2a:f1:a8:1a:54:95 hold_timer    0.00 message_age_timer    0.00 forward_delay_timer    0.00 topology_change_ack 0 config_pending 0 proxy_arp off proxy_arp_wifi off mcast_router 1 mcast_fast_leave off mcast_flood on bcast_flood on mcast_to_unicast off neigh_suppress off group_fwd_mask 0 group_fwd_mask_str 0x0 vlan_tunnel off isolated off locked off mab off addrgenmode eui64 numtxqueues 2 numrxqueues 2 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
16: tapd8a9d7f6-47@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master brq83a19a08-f0 state UP mode DEFAULT group default qlen 1000
    link/ether ca:e3:55:97:56:cf brd ff:ff:ff:ff:ff:ff link-netns qrouter-fa9a921e-7ff6-4468-a08b-5415cebeb089 promiscuity 1  allmulti 1 minmtu 68 maxmtu 65535
    veth
    bridge_slave state forwarding priority 32 cost 2 hairpin off guard off root_block off fastleave off learning on flood on port_id 0x8003 port_no 0x3 designated_port 32771 designated_cost 0 designated_bridge 8000.0:15:5d:bf:ba:55 designated_root 8000.0:15:5d:bf:ba:55 hold_timer    0.00 message_age_timer    0.00 forward_delay_timer    0.00 topology_change_ack 0 config_pending 0 proxy_arp off proxy_arp_wifi off mcast_router 1 mcast_fast_leave off mcast_flood on bcast_flood on mcast_to_unicast off neigh_suppress off group_fwd_mask 0 group_fwd_mask_str 0x0 vlan_tunnel off isolated off locked off mab off addrgenmode eui64 numtxqueues 2 numrxqueues 2 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
```

ネットワーク名前空間内のデバイスを確認する。

```sh
ip netns exec qrouter-fa9a921e-7ff6-4468-a08b-5415cebeb089 ip -d link show
```

```text
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00 promiscuity 0  allmulti 0 minmtu 0 maxmtu 0 addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
2: qr-d8316879-dd@if15: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether fa:16:3e:af:a9:e5 brd ff:ff:ff:ff:ff:ff link-netnsid 0 promiscuity 0  allmulti 0 minmtu 68 maxmtu 65535
    veth addrgenmode eui64 numtxqueues 2 numrxqueues 2 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
3: qg-d8a9d7f6-47@if16: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether fa:16:3e:c5:74:4d brd ff:ff:ff:ff:ff:ff link-netnsid 0 promiscuity 0  allmulti 0 minmtu 68 maxmtu 65535
    veth addrgenmode eui64 numtxqueues 2 numrxqueues 2 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
```

tapd8316879-dd@**if2** と qr-d8316879-dd@**if15**, tapd8a9d7f6-47@**if3** と qg-d8a9d7f6-47@**if16** が接続している。

veth peer の接続先は sysfs でも確認できる。接続先の Index が取得できる。

```sh
cat /sys/class/net/tapd8316879-dd/iflink
```

```text
2
```

```sh
cat /sys/class/net/tapd8a9d7f6-47/iflink
```

```text
3
```

### イーサネット

イーサネットの情報を確認する。

```sh
ip addr show
```

```text
(...)

4: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq master brq83a19a08-f0 state UP group default qlen 1000
    link/ether 00:15:5d:bf:ba:55 brd ff:ff:ff:ff:ff:ff
9: brq83a19a08-f0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 00:15:5d:bf:ba:55 brd ff:ff:ff:ff:ff:ff
13: vxlan-267: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue master brqc08e7dcd-4a state UNKNOWN group default qlen 1000
    link/ether ee:82:e6:12:b6:e3 brd ff:ff:ff:ff:ff:ff
14: brqc08e7dcd-4a: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP group default qlen 1000
    link/ether 2a:f1:a8:1a:54:95 brd ff:ff:ff:ff:ff:ff
15: tapd8316879-dd@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue master brqc08e7dcd-4a state UP group default qlen 1000
    link/ether 2a:f1:a8:1a:54:95 brd ff:ff:ff:ff:ff:ff link-netns qrouter-fa9a921e-7ff6-4468-a08b-5415cebeb089
16: tapd8a9d7f6-47@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master brq83a19a08-f0 state UP group default qlen 1000
    link/ether ca:e3:55:97:56:cf brd ff:ff:ff:ff:ff:ff link-netns qrouter-fa9a921e-7ff6-4468-a08b-5415cebeb089
```

ネットワーク名前空間内のイーサネットの情報を確認する。

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
    inet6 fe80::f816:3eff:fec5:744d/64 scope link
       valid_lft forever preferred_lft forever
```

ルーティングを確認する。

```sh
ip netns exec qrouter-fa9a921e-7ff6-4468-a08b-5415cebeb089 ip route
```

```text
default via 172.16.0.254 dev qg-d8a9d7f6-47 proto static
172.16.0.0/24 dev qg-d8a9d7f6-47 proto kernel scope link src 172.16.0.131
192.168.101.0/24 dev qr-d8316879-dd proto kernel scope link src 192.168.101.254
```

待ち受けているポートを確認する。

```sh
ip netns exec qrouter-fa9a921e-7ff6-4468-a08b-5415cebeb089 ss -ano -4
```

メタデータのポートが待ち受けている。

```text
Netid            State             Recv-Q             Send-Q                         Local Address:Port                         Peer Address:Port            Process
tcp              LISTEN            0                  1024                                 0.0.0.0:9697                              0.0.0.0:*
```
