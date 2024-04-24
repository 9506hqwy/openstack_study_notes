# ロードバランサ (Linux Bridge)

## 前提条件

* [](../network/linuxbridge_router) を作成していること。
* [](../instance/linuxbridge_vxlan.md) を 2 つ作成していること。

## ロードバランサの作成

ロードバランサのサービスを提供するサブネット lb-mgmt-subnet を指定してロードバランサを作成する。

サブネット lb-mgmt-subnet はセルフサービス vxlan ネットワーク 192.168.102.0/24 とする。

```sh
openstack loadbalancer create \
    --name lb \
    --vip-subnet-id lb-mgmt-subnet
```

```
+---------------------+--------------------------------------+
| Field               | Value                                |
+---------------------+--------------------------------------+
| admin_state_up      | True                                 |
| availability_zone   | None                                 |
| created_at          | 2024-04-20T05:01:18                  |
| description         |                                      |
| flavor_id           | None                                 |
| id                  | b9bf4356-fcdb-432d-b838-156b68a398ad |
| listeners           |                                      |
| name                | lb                                   |
| operating_status    | OFFLINE                              |
| pools               |                                      |
| project_id          | 39dd96b7af3a420096b892fabd45900a     |
| provider            | amphora                              |
| provisioning_status | PENDING_CREATE                       |
| updated_at          | None                                 |
| vip_address         | 192.168.102.170                      |
| vip_network_id      | 3bc6794c-936e-47e4-a5f9-093c699883e2 |
| vip_port_id         | 9d5f3364-f25f-4ef6-95c8-8ed35c51a316 |
| vip_qos_policy_id   | None                                 |
| vip_subnet_id       | e8843a57-f8e4-4c06-b4fe-6b8f996fbde8 |
| tags                |                                      |
+---------------------+--------------------------------------+
```

イメージ amphora からインスタンスが作成される。

```sh
openstack server list
```

```
+--------------------------------------+----------------------------------------------+--------+-----------------+---------------------+---------+
| ID                                   | Name                                         | Status | Networks        | Image               | Flavor  |
+--------------------------------------+----------------------------------------------+--------+-----------------+---------------------+---------+
| 9579ab86-8b57-4dd0-b7c8-d4048dee39fe | amphora-3cb1055b-a092-4dd5-87ef-59e729e7a6d4 | ACTIVE | mgmt=10.0.0.218 | amphora-x64-haproxy | amphora |
+--------------------------------------+----------------------------------------------+--------+-----------------+---------------------+---------+
```

しばらく時間が経過するとロードバランサのサービスが機能する。

```sh
curl -E /etc/octavia/certs/private/client.cert-and-key.pem -ksS https://10.0.0.218:9443
```

```json
{"api_version":"1.0"}
```

*/etc/octavia/octavia.conf* に指定したネットワークと `openstack loadbalancer create` に指定したネットワークに
接続された構成となることを確認する。

```sh
openstack server list
```

```
+--------------------------------------+----------------------------------------------+--------+---------------------------------------------+---------------------+---------+
| ID                                   | Name                                         | Status | Networks                                    | Image               | Flavor  |
+--------------------------------------+----------------------------------------------+--------+---------------------------------------------+---------------------+---------+
| 9579ab86-8b57-4dd0-b7c8-d4048dee39fe | amphora-3cb1055b-a092-4dd5-87ef-59e729e7a6d4 | ACTIVE | lb-mgmt-net=192.168.102.81; mgmt=10.0.0.218 | amphora-x64-haproxy | amphora |
+--------------------------------------+----------------------------------------------+--------+---------------------------------------------+---------------------+---------+
```

ロードバランサが作成されたことを確認する。

```sh
openstack loadbalancer list
```

```
+--------------------------------------+------+----------------------------------+-----------------+---------------------+------------------+----------+
| id                                   | name | project_id                       | vip_address     | provisioning_status | operating_status | provider |
+--------------------------------------+------+----------------------------------+-----------------+---------------------+------------------+----------+
| b9bf4356-fcdb-432d-b838-156b68a398ad | lb   | 39dd96b7af3a420096b892fabd45900a | 192.168.102.170 | ACTIVE              | OFFLINE          | amphora  |
+--------------------------------------+------+----------------------------------+-----------------+---------------------+------------------+----------+
```

## 環境の確認

Compute Node でネットワーク構成を確認する。

![Linux Bridge Load Balancer](../../_static/image/network_vxlan_lb_linuxbridge.png "Linux Bridge Load Balancer")

### インスタンス

amphora のインスタンスを確認する。

```sh
virsh list
```

```
 Id   名前                状態
----------------------------------
 12   instance-00000022   実行中
```

ネットワークインターフェイスの設定を確認する。

```sh
virsh dumpxml 12 | sed -n -e '/<interface/,/<\/interface>/ { p }'
```

```xml
<interface type='bridge'>
  <mac address='fa:16:3e:58:be:f8'/>
  <source bridge='brq2549b734-56'/>
  <target dev='tap84fa10ad-d5'/>
  <model type='virtio'/>
  <driver name='qemu'/>
  <mtu size='1500'/>
  <alias name='net0'/>
  <address type='pci' domain='0x0000' bus='0x00' slot='0x03' function='0x0'/>
</interface>
<interface type='bridge'>
  <mac address='fa:16:3e:6f:b4:54'/>
  <source bridge='brq3bc6794c-93'/>
  <target dev='tapde453235-21'/>
  <model type='virtio'/>
  <driver name='qemu'/>
  <mtu size='1450'/>
  <alias name='net1'/>
  <address type='pci' domain='0x0000' bus='0x00' slot='0x07' function='0x0'/>
</interface>
```

### ネットワーク

#### ネットワーク名前空間

amphora インスタンスのネットワーク名前空間を確認する。

```sh
ssh -i ./demo_rsa cloud-user@10.0.0.218 ip netns
```

```
amphora-haproxy (id: 0)
```

#### デバイス

Compute Node のデバイスを確認する。

```sh
ip -d link show
```

```
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq master brq2549b734-56 state UP mode DEFAULT group default qlen 1000
    link/ether 00:15:5d:bf:ba:46 brd ff:ff:ff:ff:ff:ff promiscuity 1 minmtu 68 maxmtu 65521
    bridge_slave state forwarding priority 32 cost 2 hairpin off guard off root_block off fastleave off learning on flood on port_id 0x8002 port_no 0x2 designated_port 32770 designated_cost 0 designated_bridge 8000.0:15:5d:bf:ba:46 designated_root 8000.0:15:5d:bf:ba:46 hold_timer    0.00 message_age_timer    0.00 forward_delay_timer    0.00 topology_change_ack 0 config_pending 0 proxy_arp off proxy_arp_wifi off mcast_router 1 mcast_fast_leave off mcast_flood on bcast_flood on mcast_to_unicast off neigh_suppress off group_fwd_mask 0 group_fwd_mask_str 0x0 vlan_tunnel off isolated off locked off addrgenmode eui64 numtxqueues 64 numrxqueues 64 gso_max_size 62780 gso_max_segs 65535 parentbus vmbus parentdev 97ecc413-a192-4d83-b344-02846e18823d
8: brq3bc6794c-93: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 22:8a:7f:7a:33:56 brd ff:ff:ff:ff:ff:ff promiscuity 0 minmtu 68 maxmtu 65535
    bridge forward_delay 0 hello_time 200 max_age 2000 ageing_time 30000 stp_state 0 priority 32768 vlan_filtering 0 vlan_protocol 802.1Q bridge_id 8000.22:8a:7f:7a:33:56 designated_root 8000.22:8a:7f:7a:33:56 root_port 0 root_path_cost 0 topology_change 0 topology_change_detected 0 hello_timer    0.00 tcn_timer    0.00 topology_change_timer    0.00 gc_timer   38.83 vlan_default_pvid 1 vlan_stats_enabled 0 vlan_stats_per_port 0 group_fwd_mask 0 group_address 01:80:c2:00:00:00 mcast_snooping 0 no_linklocal_learn 0 mcast_vlan_snooping 0 mcast_router 1 mcast_query_use_ifaddr 0 mcast_querier 0 mcast_hash_elasticity 16 mcast_hash_max 4096 mcast_last_member_count 2 mcast_startup_query_count 2 mcast_last_member_interval 100 mcast_membership_interval 26000 mcast_querier_interval 25500 mcast_query_interval 12500 mcast_query_response_interval 1000 mcast_startup_query_interval 3125 mcast_stats_enabled 0 mcast_igmp_version 2 mcast_mld_version 1 nf_call_iptables 0 nf_call_ip6tables 0 nf_call_arptables 0 addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 62780 gso_max_segs 65535
10: vxlan-247: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue master brq3bc6794c-93 state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether 22:8a:7f:7a:33:56 brd ff:ff:ff:ff:ff:ff promiscuity 1 minmtu 68 maxmtu 65535
    vxlan id 247 local 172.16.0.31 dev eth0 srcport 0 0 dstport 8472 ttl auto ageing 300 udpcsum noudp6zerocsumtx noudp6zerocsumrx
    bridge_slave state forwarding priority 32 cost 2 hairpin off guard off root_block off fastleave off learning on flood on port_id 0x8002 port_no 0x2 designated_port 32770 designated_cost 0 designated_bridge 8000.22:8a:7f:7a:33:56 designated_root 8000.22:8a:7f:7a:33:56 hold_timer    0.00 message_age_timer    0.00 forward_delay_timer    0.00 topology_change_ack 0 config_pending 0 proxy_arp off proxy_arp_wifi off mcast_router 1 mcast_fast_leave off mcast_flood on bcast_flood on mcast_to_unicast off neigh_suppress off group_fwd_mask 0 group_fwd_mask_str 0x0 vlan_tunnel off isolated off locked off addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 62780 gso_max_segs 65535
23: brq2549b734-56: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 00:15:5d:bf:ba:46 brd ff:ff:ff:ff:ff:ff promiscuity 0 minmtu 68 maxmtu 65535
    bridge forward_delay 0 hello_time 200 max_age 2000 ageing_time 30000 stp_state 0 priority 32768 vlan_filtering 0 vlan_protocol 802.1Q bridge_id 8000.0:15:5d:bf:ba:46 designated_root 8000.0:15:5d:bf:ba:46 root_port 0 root_path_cost 0 topology_change 0 topology_change_detected 0 hello_timer    0.00 tcn_timer    0.00 topology_change_timer    0.00 gc_timer  177.12 vlan_default_pvid 1 vlan_stats_enabled 0 vlan_stats_per_port 0 group_fwd_mask 0 group_address 01:80:c2:00:00:00 mcast_snooping 0 no_linklocal_learn 0 mcast_vlan_snooping 0 mcast_router 1 mcast_query_use_ifaddr 0 mcast_querier 0 mcast_hash_elasticity 16 mcast_hash_max 4096 mcast_last_member_count 2 mcast_startup_query_count 2 mcast_last_member_interval 100 mcast_membership_interval 26000 mcast_querier_interval 25500 mcast_query_interval 12500 mcast_query_response_interval 1000 mcast_startup_query_interval 3125 mcast_stats_enabled 0 mcast_igmp_version 2 mcast_mld_version 1 nf_call_iptables 0 nf_call_ip6tables 0 nf_call_arptables 0 addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 62780 gso_max_segs 65535
27: tap84fa10ad-d5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master brq2549b734-56 state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether fe:16:3e:58:be:f8 brd ff:ff:ff:ff:ff:ff promiscuity 1 minmtu 68 maxmtu 65521
    tun type tap pi off vnet_hdr on persist off
    bridge_slave state forwarding priority 32 cost 100 hairpin off guard off root_block off fastleave off learning on flood on port_id 0x8001 port_no 0x1 designated_port 32769 designated_cost 0 designated_bridge 8000.0:15:5d:bf:ba:46 designated_root 8000.0:15:5d:bf:ba:46 hold_timer    0.00 message_age_timer    0.00 forward_delay_timer    0.00 topology_change_ack 0 config_pending 0 proxy_arp off proxy_arp_wifi off mcast_router 1 mcast_fast_leave off mcast_flood on bcast_flood on mcast_to_unicast off neigh_suppress off group_fwd_mask 0 group_fwd_mask_str 0x0 vlan_tunnel off isolated off locked off addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535
28: tapde453235-21: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue master brq3bc6794c-93 state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether fe:16:3e:6f:b4:54 brd ff:ff:ff:ff:ff:ff promiscuity 1 minmtu 68 maxmtu 65521
    tun type tap pi off vnet_hdr on persist off
    bridge_slave state forwarding priority 32 cost 100 hairpin off guard off root_block off fastleave off learning on flood on port_id 0x8001 port_no 0x1 designated_port 32769 designated_cost 0 designated_bridge 8000.22:8a:7f:7a:33:56 designated_root 8000.22:8a:7f:7a:33:56 hold_timer    0.00 message_age_timer    0.00 forward_delay_timer    0.00 topology_change_ack 0 config_pending 0 proxy_arp off proxy_arp_wifi off mcast_router 1 mcast_fast_leave off mcast_flood on bcast_flood on mcast_to_unicast off neigh_suppress off group_fwd_mask 0 group_fwd_mask_str 0x0 vlan_tunnel off isolated off locked off addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535
```

amphora インスタンスのデバイスを確認する。

```sh
ssh -i ./demo_rsa cloud-user@10.0.0.218 ip -d link show
```

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00 promiscuity 0  allmulti 0 minmtu 0 maxmtu 0 addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether fa:16:3e:58:be:f8 brd ff:ff:ff:ff:ff:ff promiscuity 0  allmulti 0 minmtu 68 maxmtu 1500 addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 65536 tso_max_segs 65535 gro_max_size 65536 parentbus virtio parentdev virtio1
    altname enp0s3
```

ネットワーク名前空間内のデバイスを確認する。

```sh
ssh -i ./demo_rsa cloud-user@10.0.0.218 sudo ip netns exec amphora-haproxy ip -d link show
```

```
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00 promiscuity 0  allmulti 0 minmtu 0 maxmtu 0 addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether fa:16:3e:6f:b4:54 brd ff:ff:ff:ff:ff:ff promiscuity 0  allmulti 0 minmtu 68 maxmtu 1450 addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 65536 tso_max_segs 65535 gro_max_size 65536 parentbus virtio parentdev virtio5
    altname enp0s7
```

#### イーサネット

amphora インスタンスのイーサネットの情報を確認する。

```sh
ssh -i ./demo_rsa cloud-user@10.0.0.218 ip addr show
```

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether fa:16:3e:58:be:f8 brd ff:ff:ff:ff:ff:ff
    altname enp0s3
    inet 10.0.0.218/24 brd 10.0.0.255 scope global dynamic noprefixroute ens3
       valid_lft 86129sec preferred_lft 86129sec
    inet6 fe80::f816:3eff:fe58:bef8/64 scope link
       valid_lft forever preferred_lft forever
```

ネットワーク名前空間内のイーサネットの情報を確認する。

```sh
ssh -i ./demo_rsa cloud-user@10.0.0.218 sudo ip netns exec amphora-haproxy ip addr show
```

```
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc fq_codel state UP group default qlen 1000
    link/ether fa:16:3e:6f:b4:54 brd ff:ff:ff:ff:ff:ff
    altname enp0s7
    inet 192.168.102.81/24 scope global eth1
       valid_lft forever preferred_lft forever
    inet 192.168.102.170/32 scope global eth1
       valid_lft forever preferred_lft forever
```

## リスナの作成

ロードバランサに SSH で待ち受けるリスナを作成する。

```sh
openstack loadbalancer listener create \
    --name lb-ssh \
    --protocol TCP \
    --protocol-port 22 \
    b9bf4356-fcdb-432d-b838-156b68a398ad
```

```
+-----------------------------+--------------------------------------+
| Field                       | Value                                |
+-----------------------------+--------------------------------------+
| admin_state_up              | True                                 |
| connection_limit            | -1                                   |
| created_at                  | 2024-04-20T05:27:29                  |
| default_pool_id             | None                                 |
| default_tls_container_ref   | None                                 |
| description                 |                                      |
| id                          | b38c1145-a5cf-44be-bc47-3040d217cf34 |
| insert_headers              | None                                 |
| l7policies                  |                                      |
| loadbalancers               | b9bf4356-fcdb-432d-b838-156b68a398ad |
| name                        | lb-ssh                               |
| operating_status            | OFFLINE                              |
| project_id                  | 39dd96b7af3a420096b892fabd45900a     |
| protocol                    | TCP                                  |
| protocol_port               | 22                                   |
| provisioning_status         | PENDING_CREATE                       |
| sni_container_refs          | []                                   |
| timeout_client_data         | 50000                                |
| timeout_member_connect      | 5000                                 |
| timeout_member_data         | 50000                                |
| timeout_tcp_inspect         | 0                                    |
| updated_at                  | None                                 |
| client_ca_tls_container_ref | None                                 |
| client_authentication       | NONE                                 |
| client_crl_container_ref    | None                                 |
| allowed_cidrs               | None                                 |
| tls_ciphers                 | None                                 |
| tls_versions                | None                                 |
| alpn_protocols              | None                                 |
| tags                        |                                      |
+-----------------------------+--------------------------------------+
```

## プールの作成

リスナにプールを作成する。

```sh
openstack loadbalancer pool create \
    --name lb-ssh-pool \
    --lb-algorithm ROUND_ROBIN \
    --listener lb-ssh \
    --protocol TCP
```

```
+----------------------+--------------------------------------+
| Field                | Value                                |
+----------------------+--------------------------------------+
| admin_state_up       | True                                 |
| created_at           | 2024-04-20T05:29:08                  |
| description          |                                      |
| healthmonitor_id     |                                      |
| id                   | 8f5ae380-0d85-4302-8f1c-78ad6843ef2b |
| lb_algorithm         | ROUND_ROBIN                          |
| listeners            | b38c1145-a5cf-44be-bc47-3040d217cf34 |
| loadbalancers        | b9bf4356-fcdb-432d-b838-156b68a398ad |
| members              |                                      |
| name                 | lb-ssh-pool                          |
| operating_status     | OFFLINE                              |
| project_id           | 39dd96b7af3a420096b892fabd45900a     |
| protocol             | TCP                                  |
| provisioning_status  | PENDING_CREATE                       |
| session_persistence  | None                                 |
| updated_at           | None                                 |
| tls_container_ref    | None                                 |
| ca_tls_container_ref | None                                 |
| crl_container_ref    | None                                 |
| tls_enabled          | False                                |
| tls_ciphers          | None                                 |
| tls_versions         | None                                 |
| tags                 |                                      |
| alpn_protocols       | None                                 |
+----------------------+--------------------------------------+
```

## プールにメンバを追加

プールにインスタンスの IP を登録する。

```sh
openstack server list
```

```
+--------------------------------------+----------------------------------------------+--------+---------------------------------------------+---------------------+---------+
| ID                                   | Name                                         | Status | Networks                                    | Image               | Flavor  |
+--------------------------------------+----------------------------------------------+--------+---------------------------------------------+---------------------+---------+
| 0d3bb33a-649a-4821-bd89-af69e55c3bab | instance05                                   | ACTIVE | lb-mgmt-net=192.168.102.127                 | cirros              | m1.nano |
| 0f4e7d68-9519-43da-97a3-3a618799451c | instance04                                   | ACTIVE | lb-mgmt-net=192.168.102.183                 | cirros              | m1.nano |
+--------------------------------------+----------------------------------------------+--------+---------------------------------------------+---------------------+---------+
```

メンバを追加する。

```sh
openstack loadbalancer member create \
    --subnet-id lb-mgmt-subnet \
    --address 192.168.102.183 \
    --protocol-port 22 \
    lb-ssh-pool
```

```
+---------------------+--------------------------------------+
| Field               | Value                                |
+---------------------+--------------------------------------+
| address             | 192.168.102.183                      |
| admin_state_up      | True                                 |
| created_at          | 2024-04-20T05:46:55                  |
| id                  | 2ea4728c-c793-4dfa-a7f5-310e80b04c33 |
| name                |                                      |
| operating_status    | NO_MONITOR                           |
| project_id          | 39dd96b7af3a420096b892fabd45900a     |
| protocol_port       | 22                                   |
| provisioning_status | PENDING_CREATE                       |
| subnet_id           | e8843a57-f8e4-4c06-b4fe-6b8f996fbde8 |
| updated_at          | None                                 |
| weight              | 1                                    |
| monitor_port        | None                                 |
| monitor_address     | None                                 |
| backup              | False                                |
| tags                |                                      |
+---------------------+--------------------------------------+
```

```sh
openstack loadbalancer member create \
    --subnet-id lb-mgmt-subnet \
    --address 192.168.102.127 \
    --protocol-port 22 \
    lb-ssh-pool
```

```
+---------------------+--------------------------------------+
| Field               | Value                                |
+---------------------+--------------------------------------+
| address             | 192.168.102.127                      |
| admin_state_up      | True                                 |
| created_at          | 2024-04-20T05:47:41                  |
| id                  | 6bf6d513-2631-485e-b48b-024c44a69098 |
| name                |                                      |
| operating_status    | NO_MONITOR                           |
| project_id          | 39dd96b7af3a420096b892fabd45900a     |
| protocol_port       | 22                                   |
| provisioning_status | PENDING_CREATE                       |
| subnet_id           | e8843a57-f8e4-4c06-b4fe-6b8f996fbde8 |
| updated_at          | None                                 |
| weight              | 1                                    |
| monitor_port        | None                                 |
| monitor_address     | None                                 |
| backup              | False                                |
| tags                |                                      |
+---------------------+--------------------------------------+
```

メンバの追加を確認する。

```sh
openstack loadbalancer member list lb-ssh-pool
```

```
+--------------------------------------+------+----------------------------------+---------------------+-----------------+---------------+------------------+--------+
| id                                   | name | project_id                       | provisioning_status | address         | protocol_port | operating_status | weight |
+--------------------------------------+------+----------------------------------+---------------------+-----------------+---------------+------------------+--------+
| 2ea4728c-c793-4dfa-a7f5-310e80b04c33 |      | 39dd96b7af3a420096b892fabd45900a | ACTIVE              | 192.168.102.183 |            22 | NO_MONITOR       |      1 |
| 6bf6d513-2631-485e-b48b-024c44a69098 |      | 39dd96b7af3a420096b892fabd45900a | ACTIVE              | 192.168.102.127 |            22 | NO_MONITOR       |      1 |
+--------------------------------------+------+----------------------------------+---------------------+-----------------+---------------+------------------+--------+
```

## Floating IP の作成

ロードバランサを外部ネットワークから接続するため Floating IP を作成する。

```sh
openstack floating ip create provider
```

```
+---------------------+--------------------------------------+
| Field               | Value                                |
+---------------------+--------------------------------------+
| created_at          | 2024-04-20T05:49:57Z                 |
| description         |                                      |
| dns_domain          | None                                 |
| dns_name            | None                                 |
| fixed_ip_address    | None                                 |
| floating_ip_address | 172.17.0.113                         |
| floating_network_id | ca4e2bc3-fe44-48d0-8096-20e6a85d6510 |
| id                  | 7f6e5047-ff0d-4a51-a648-29550f61ce6d |
| name                | 172.17.0.113                         |
| port_details        | None                                 |
| port_id             | None                                 |
| project_id          | 39dd96b7af3a420096b892fabd45900a     |
| qos_policy_id       | None                                 |
| revision_number     | 0                                    |
| router_id           | None                                 |
| status              | DOWN                                 |
| subnet_id           | None                                 |
| tags                | []                                   |
| updated_at          | 2024-04-20T05:49:57Z                 |
+---------------------+--------------------------------------+
```

ロードバランサのポートを確認して Floating IP を割り当てる。

```sh
openstack loadbalancer show b9bf4356-fcdb-432d-b838-156b68a398ad -c vip_port_id -f value
```

```
9d5f3364-f25f-4ef6-95c8-8ed35c51a316
```

```sh
openstack floating ip set \
    --port 9d5f3364-f25f-4ef6-95c8-8ed35c51a316 \
    172.17.0.113
```

## 動作確認

ラウンドロビンすることを確認する。

1回目。

```sh
ssh -i ./demo_rsa cirros@172.17.0.113 /sbin/ip addr show
```

```
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc pfifo_fast qlen 1000
    link/ether fa:16:3e:6a:39:2f brd ff:ff:ff:ff:ff:ff
    inet 192.168.102.127/24 brd 192.168.102.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:fe6a:392f/64 scope link
       valid_lft forever preferred_lft forever
```

2回目。

```sh
ssh -i ./demo_rsa cirros@172.17.0.113 /sbin/ip addr show
```

```
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc pfifo_fast qlen 1000
    link/ether fa:16:3e:f3:4c:3e brd ff:ff:ff:ff:ff:ff
    inet 192.168.102.183/24 brd 192.168.102.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:fef3:4c3e/64 scope link
       valid_lft forever preferred_lft forever
```

HAProxy の設定が追加されることを確認する。

```sh
ssh -i ./demo_rsa cloud-user@10.0.0.218 sudo cat /var/lib/octavia/b9bf4356-fcdb-432d-b838-156b68a398ad/haproxy.cfg
```

```
backend 8f5ae380-0d85-4302-8f1c-78ad6843ef2b:b38c1145-a5cf-44be-bc47-3040d217cf34
    mode tcp
    balance roundrobin
    fullconn 50000
    option allbackups
    timeout connect 5000
    timeout server 50000
    server 6bf6d513-2631-485e-b48b-024c44a69098 192.168.102.127:22 weight 1
    server 2ea4728c-c793-4dfa-a7f5-310e80b04c33 192.168.102.183:22 weight 1
```
