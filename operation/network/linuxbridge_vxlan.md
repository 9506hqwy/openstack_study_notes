# vxlan ネットワーク (Linux Bridge)

Linux Bridge を利用した vxlan ネットワークを作成する。

## 前提条件

* Controller Node で [](../../installation/controller/neutron_linuxbridge/vxlan) を設定していること。
* Compute Node で [](../../installation/compute/neutron_linuxbridge/vxlan) を設定していること。

## セルフサービスネットワークの作成

```{tip}
myuser で実行
```

eth0 に繋がるセルフサービスネットワークとして vxlan ネットワークを作成する。

```sh
openstack network create selfservice
```

```text
+---------------------------+--------------------------------------+
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| admin_state_up            | UP                                   |
| availability_zone_hints   |                                      |
| availability_zones        |                                      |
| created_at                | 2024-05-12T10:18:01Z                 |
| description               |                                      |
| dns_domain                | None                                 |
| id                        | c08e7dcd-4acb-4cd0-ad08-ebd3b48467ee |
| ipv4_address_scope        | None                                 |
| ipv6_address_scope        | None                                 |
| is_default                | False                                |
| is_vlan_transparent       | None                                 |
| mtu                       | 1450                                 |
| name                      | selfservice                          |
| port_security_enabled     | True                                 |
| project_id                | be94f4411bd74f249f5e25f642209b82     |
| provider:network_type     | vxlan                                |
| provider:physical_network | None                                 |
| provider:segmentation_id  | 267                                  |
| qos_policy_id             | None                                 |
| revision_number           | 1                                    |
| router:external           | Internal                             |
| segments                  | None                                 |
| shared                    | False                                |
| status                    | ACTIVE                               |
| subnets                   |                                      |
| tags                      |                                      |
| tenant_id                 | be94f4411bd74f249f5e25f642209b82     |
| updated_at                | 2024-05-12T10:18:01Z                 |
+---------------------------+--------------------------------------+
```

## サブネットの作成

サブネットを作成する。

```sh
openstack subnet create \
    --network selfservice \
    --gateway 192.168.101.254 \
    --subnet-range 192.168.101.0/24 \
    selfservice
```

```text
+----------------------+--------------------------------------+
| Field                | Value                                |
+----------------------+--------------------------------------+
| allocation_pools     | 192.168.101.1-192.168.101.253        |
| cidr                 | 192.168.101.0/24                     |
| created_at           | 2024-05-12T10:19:46Z                 |
| description          |                                      |
| dns_nameservers      |                                      |
| dns_publish_fixed_ip | None                                 |
| enable_dhcp          | True                                 |
| gateway_ip           | 192.168.101.254                      |
| host_routes          |                                      |
| id                   | 8abd1d19-1fed-412a-83d0-39e098708648 |
| ip_version           | 4                                    |
| ipv6_address_mode    | None                                 |
| ipv6_ra_mode         | None                                 |
| name                 | selfservice                          |
| network_id           | c08e7dcd-4acb-4cd0-ad08-ebd3b48467ee |
| project_id           | be94f4411bd74f249f5e25f642209b82     |
| revision_number      | 0                                    |
| segment_id           | None                                 |
| service_types        |                                      |
| subnetpool_id        | None                                 |
| tags                 |                                      |
| updated_at           | 2024-05-12T10:19:46Z                 |
+----------------------+--------------------------------------+
```

DHCP サーバのポートの作成を確認する。

```sh
openstack port list --network selfservice
```

```text
+--------------------------------------+------+-------------------+------------------------------------------------------------------------------+--------+
| ID                                   | Name | MAC Address       | Fixed IP Addresses                                                           | Status |
+--------------------------------------+------+-------------------+------------------------------------------------------------------------------+--------+
| e0bba9cc-3e09-4c9d-ac97-22ece432c096 |      | fa:16:3e:fb:fe:e2 | ip_address='192.168.101.1', subnet_id='8abd1d19-1fed-412a-83d0-39e098708648' | ACTIVE |
+--------------------------------------+------+-------------------+------------------------------------------------------------------------------+--------+
```

```sh
openstack port show e0bba9cc-3e09-4c9d-ac97-22ece432c096
```

```text
+-------------------------+-------------------------------------------------------------------------------+
| Field                   | Value                                                                         |
+-------------------------+-------------------------------------------------------------------------------+
| admin_state_up          | UP                                                                            |
| allowed_address_pairs   |                                                                               |
| binding_host_id         | controller.home.local                                                         |
| binding_profile         |                                                                               |
| binding_vif_details     | bound_drivers.0='linuxbridge', connectivity='l2', port_filter='True'          |
| binding_vif_type        | bridge                                                                        |
| binding_vnic_type       | normal                                                                        |
| created_at              | 2024-05-12T10:19:46Z                                                          |
| data_plane_status       | None                                                                          |
| description             |                                                                               |
| device_id               | dhcpd3377d3c-a0d1-5d71-9947-f17125c357bb-c08e7dcd-4acb-4cd0-ad08-ebd3b48467ee |
| device_owner            | network:dhcp                                                                  |
| device_profile          | None                                                                          |
| dns_assignment          | None                                                                          |
| dns_domain              | None                                                                          |
| dns_name                | None                                                                          |
| extra_dhcp_opts         |                                                                               |
| fixed_ips               | ip_address='192.168.101.1', subnet_id='8abd1d19-1fed-412a-83d0-39e098708648'  |
| hardware_offload_type   | None                                                                          |
| hints                   |                                                                               |
| id                      | e0bba9cc-3e09-4c9d-ac97-22ece432c096                                          |
| ip_allocation           | None                                                                          |
| mac_address             | fa:16:3e:fb:fe:e2                                                             |
| name                    |                                                                               |
| network_id              | c08e7dcd-4acb-4cd0-ad08-ebd3b48467ee                                          |
| numa_affinity_policy    | None                                                                          |
| port_security_enabled   | False                                                                         |
| project_id              | be94f4411bd74f249f5e25f642209b82                                              |
| propagate_uplink_status | None                                                                          |
| resource_request        | None                                                                          |
| revision_number         | 4                                                                             |
| qos_network_policy_id   | None                                                                          |
| qos_policy_id           | None                                                                          |
| security_group_ids      |                                                                               |
| status                  | ACTIVE                                                                        |
| tags                    |                                                                               |
| trunk_details           | None                                                                          |
| updated_at              | 2024-05-12T10:19:48Z                                                          |
+-------------------------+-------------------------------------------------------------------------------+
```

## 環境の確認

Controller Node でネットワーク構成を確認する。

![Linux Bridge vxlan ネットワーク](../../_static/image/network_vxlan_dhcpagent_linuxbridge.png "Linux Bridge vxlan ネットワーク")

### ネットワーク名前空間

サブネットを作成するとネットワーク名前空間が作成される。

```sh
ip netns
```

```text
(...)

qdhcp-c08e7dcd-4acb-4cd0-ad08-ebd3b48467ee (id: 2)
```

### デバイス

ブリッジと veth peer が作成される。

```sh
ip -d link show
```

```text
(...)

12: tape0bba9cc-3e@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue master brqc08e7dcd-4a state UP mode DEFAULT group default qlen 1000
    link/ether 96:f9:76:88:67:e2 brd ff:ff:ff:ff:ff:ff link-netns qdhcp-c08e7dcd-4acb-4cd0-ad08-ebd3b48467ee promiscuity 1  allmulti 1 minmtu 68 maxmtu 65535
    veth
    bridge_slave state forwarding priority 32 cost 2 hairpin off guard off root_block off fastleave off learning on flood on port_id 0x8002 port_no 0x2 designated_port 32770 designated_cost 0 designated_bridge 8000.96:f9:76:88:67:e2 designated_root 8000.96:f9:76:88:67:e2 hold_timer    0.00 message_age_timer    0.00 forward_delay_timer    0.00 topology_change_ack 0 config_pending 0 proxy_arp off proxy_arp_wifi off mcast_router 1 mcast_fast_leave off mcast_flood on bcast_flood on mcast_to_unicast off neigh_suppress off group_fwd_mask 0 group_fwd_mask_str 0x0 vlan_tunnel off isolated off locked off mab off addrgenmode eui64 numtxqueues 2 numrxqueues 2 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
13: vxlan-267: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue master brqc08e7dcd-4a state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether ee:82:e6:12:b6:e3 brd ff:ff:ff:ff:ff:ff promiscuity 1  allmulti 1 minmtu 68 maxmtu 65535
    vxlan id 267 local 172.16.0.11 dev eth0 srcport 0 0 dstport 8472 ttl auto ageing 300 udpcsum noudp6zerocsumtx noudp6zerocsumrx
    bridge_slave state forwarding priority 32 cost 2 hairpin off guard off root_block off fastleave off learning on flood on port_id 0x8001 port_no 0x1 designated_port 32769 designated_cost 0 designated_bridge 8000.96:f9:76:88:67:e2 designated_root 8000.96:f9:76:88:67:e2 hold_timer    0.00 message_age_timer    0.00 forward_delay_timer    0.00 topology_change_ack 0 config_pending 0 proxy_arp off proxy_arp_wifi off mcast_router 1 mcast_fast_leave off mcast_flood on bcast_flood on mcast_to_unicast off neigh_suppress off group_fwd_mask 0 group_fwd_mask_str 0x0 vlan_tunnel off isolated off locked off mab off addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 62780 gso_max_segs 65535 tso_max_size 62780 tso_max_segs 65535 gro_max_size 65536
14: brqc08e7dcd-4a: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 96:f9:76:88:67:e2 brd ff:ff:ff:ff:ff:ff promiscuity 0  allmulti 0 minmtu 68 maxmtu 65535
    bridge forward_delay 0 hello_time 200 max_age 2000 ageing_time 30000 stp_state 0 priority 32768 vlan_filtering 0 vlan_protocol 802.1Q bridge_id 8000.96:f9:76:88:67:e2 designated_root 8000.96:f9:76:88:67:e2 root_port 0 root_path_cost 0 topology_change 0 topology_change_detected 0 hello_timer    0.00 tcn_timer    0.00 topology_change_timer    0.00 gc_timer  176.58 vlan_default_pvid 1 vlan_stats_enabled 0 vlan_stats_per_port 0 group_fwd_mask 0 group_address 01:80:c2:00:00:00 mcast_snooping 1 no_linklocal_learn 0 mcast_vlan_snooping 0 mcast_router 1 mcast_query_use_ifaddr 0 mcast_querier 0 mcast_hash_elasticity 16 mcast_hash_max 4096 mcast_last_member_count 2 mcast_startup_query_count 2 mcast_last_member_interval 100 mcast_membership_interval 26000 mcast_querier_interval 25500 mcast_query_interval 12500 mcast_query_response_interval 1000 mcast_startup_query_interval 3125 mcast_stats_enabled 0 mcast_igmp_version 2 mcast_mld_version 1 nf_call_iptables 0 nf_call_ip6tables 0 nf_call_arptables 0 addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 62780 gso_max_segs 65535 tso_max_size 62780 tso_max_segs 65535 gro_max_size 65536
```

ネットワーク名前空間内のデバイスを確認する。

```sh
ip netns exec qdhcp-c08e7dcd-4acb-4cd0-ad08-ebd3b48467ee ip -d link show
```

```text
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00 promiscuity 0  allmulti 0 minmtu 0 maxmtu 0 addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
2: ns-e0bba9cc-3e@if12: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether fa:16:3e:fb:fe:e2 brd ff:ff:ff:ff:ff:ff link-netnsid 0 promiscuity 0  allmulti 0 minmtu 68 maxmtu 65535
    veth addrgenmode eui64 numtxqueues 2 numrxqueues 2 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
```

tape0bba9cc-3e@**if2** と ns-e0bba9cc-3e@**if12** が接続している。

veth peer の接続先は sysfs でも確認できる。接続先の Index が取得できる。

```sh
cat /sys/class/net/tape0bba9cc-3e/iflink
```

```text
2
```

### イーサネット

イーサネットの情報を確認する。

```sh
ip addr show
```

```text
(...)

12: tape0bba9cc-3e@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue master brqc08e7dcd-4a state UP group default qlen 1000
    link/ether 96:f9:76:88:67:e2 brd ff:ff:ff:ff:ff:ff link-netns qdhcp-c08e7dcd-4acb-4cd0-ad08-ebd3b48467ee
13: vxlan-267: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue master brqc08e7dcd-4a state UNKNOWN group default qlen 1000
    link/ether ee:82:e6:12:b6:e3 brd ff:ff:ff:ff:ff:ff
14: brqc08e7dcd-4a: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP group default qlen 1000
    link/ether 96:f9:76:88:67:e2 brd ff:ff:ff:ff:ff:ff
```

ネットワーク名前空間内のイーサネットの情報を確認する。
169.254.169.254 は Metadata agent が使用する。

```sh
ip netns exec qdhcp-c08e7dcd-4acb-4cd0-ad08-ebd3b48467ee ip addr show
```

```text
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ns-e0bba9cc-3e@if12: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP group default qlen 1000
    link/ether fa:16:3e:fb:fe:e2 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.101.1/24 brd 192.168.101.255 scope global ns-e0bba9cc-3e
       valid_lft forever preferred_lft forever
    inet 169.254.169.254/32 brd 169.254.169.254 scope global ns-e0bba9cc-3e
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:fefb:fee2/64 scope link
       valid_lft forever preferred_lft forever
```

ルーティングを確認する。

```sh
ip netns exec qdhcp-c08e7dcd-4acb-4cd0-ad08-ebd3b48467ee ip route show
```

```text
default via 192.168.101.254 dev ns-e0bba9cc-3e proto static
192.168.101.0/24 dev ns-e0bba9cc-3e proto kernel scope link src 192.168.101.1
```

待ち受けているポートを確認する。

```sh
ip netns exec qdhcp-c08e7dcd-4acb-4cd0-ad08-ebd3b48467ee ss -ano -4
```

```text
Netid            State             Recv-Q            Send-Q                          Local Address:Port                         Peer Address:Port            Process
udp              UNCONN            0                 0                                   127.0.0.1:53                                0.0.0.0:*
udp              UNCONN            0                 0                               192.168.101.1:53                                0.0.0.0:*
udp              UNCONN            0                 0                             169.254.169.254:53                                0.0.0.0:*
udp              UNCONN            0                 0                                     0.0.0.0:67                                0.0.0.0:*
tcp              LISTEN            0                 1024                          169.254.169.254:80                                0.0.0.0:*
tcp              LISTEN            0                 32                            169.254.169.254:53                                0.0.0.0:*
tcp              LISTEN            0                 32                              192.168.101.1:53                                0.0.0.0:*
tcp              LISTEN            0                 32                                  127.0.0.1:53                                0.0.0.0:*
```

### DHCP agent

dnsmasq のプロセスを確認する。

```sh
ps ax | grep dnsmasq
```

以下が動作していることが確認できる。

```sh
dnsmasq \
    --no-hosts \
    --no-resolv \
    --pid-file=/var/lib/neutron/dhcp/c08e7dcd-4acb-4cd0-ad08-ebd3b48467ee/pid \
    --dhcp-hostsfile=/var/lib/neutron/dhcp/c08e7dcd-4acb-4cd0-ad08-ebd3b48467ee/host \
    --addn-hosts=/var/lib/neutron/dhcp/c08e7dcd-4acb-4cd0-ad08-ebd3b48467ee/addn_hosts \
    --dhcp-optsfile=/var/lib/neutron/dhcp/c08e7dcd-4acb-4cd0-ad08-ebd3b48467ee/opts \
    --dhcp-leasefile=/var/lib/neutron/dhcp/c08e7dcd-4acb-4cd0-ad08-ebd3b48467ee/leases \
    --dhcp-match=set:ipxe,175 \
    --dhcp-userclass=set:ipxe6,iPXE \
    --local-service \
    --bind-dynamic \
    --dhcp-range=set:subnet-8abd1d19-1fed-412a-83d0-39e098708648,192.168.101.0,static,255.255.255.0,86400s \
    --dhcp-option-force=option:mtu,1450 \
    --dhcp-lease-max=256 \
    --conf-file=/dev/null \
    --domain=openstacklocal
```

使用しているインターフェイスを確認する。

```sh
cat /var/lib/neutron/dhcp/c08e7dcd-4acb-4cd0-ad08-ebd3b48467ee/interface
```

```text
ns-e0bba9cc-3e
```
