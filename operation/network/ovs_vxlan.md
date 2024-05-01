# vxlan ネットワーク (Open vSwitch)

Open vSwitch を利用した vxlan ネットワークを作成する。

## 前提条件

* Controller Node で [](../../installation/controller/neutron_ovs/vxlan) を設定していること。
* Compute Node で [](../../installation/compute/neutron_ovs/vxlan) を設定していること。

## セルフサービスネットワークの作成

管理者以外のユーザで、
eth0 に繋がるセルフサービスネットワークとして vxlan ネットワークを作成する。

```sh
openstack network create selfservice
```

```
+---------------------------+--------------------------------------+
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| admin_state_up            | UP                                   |
| availability_zone_hints   |                                      |
| availability_zones        |                                      |
| created_at                | 2024-04-23T10:45:21Z                 |
| description               |                                      |
| dns_domain                | None                                 |
| id                        | 9eb0ca8c-b4d4-455b-9f5e-fc3641a0ebd1 |
| ipv4_address_scope        | None                                 |
| ipv6_address_scope        | None                                 |
| is_default                | False                                |
| is_vlan_transparent       | None                                 |
| mtu                       | 1450                                 |
| name                      | selfservice                          |
| port_security_enabled     | True                                 |
| project_id                | f2aeffb34ff34ffb8959f1cd813655c6     |
| provider:network_type     | None                                 |
| provider:physical_network | None                                 |
| provider:segmentation_id  | None                                 |
| qos_policy_id             | None                                 |
| revision_number           | 1                                    |
| router:external           | Internal                             |
| segments                  | None                                 |
| shared                    | False                                |
| status                    | ACTIVE                               |
| subnets                   |                                      |
| tags                      |                                      |
| updated_at                | 2024-04-23T10:45:21Z                 |
+---------------------------+--------------------------------------+
```

## サブネットの作成

サブネットを作成する。

```sh
openstack subnet create \
    --network selfservice \
    --gateway 192.168.101.1 \
    --subnet-range 192.168.101.0/24 \
    selfservice
```

```
+----------------------+--------------------------------------+
| Field                | Value                                |
+----------------------+--------------------------------------+
| allocation_pools     | 192.168.101.2-192.168.101.254        |
| cidr                 | 192.168.101.0/24                     |
| created_at           | 2024-04-23T11:58:16Z                 |
| description          |                                      |
| dns_nameservers      |                                      |
| dns_publish_fixed_ip | None                                 |
| enable_dhcp          | True                                 |
| gateway_ip           | 192.168.101.1                        |
| host_routes          |                                      |
| id                   | f09a54e3-3b18-4dfa-9dda-423ed0641889 |
| ip_version           | 4                                    |
| ipv6_address_mode    | None                                 |
| ipv6_ra_mode         | None                                 |
| name                 | selfservice                          |
| network_id           | 9eb0ca8c-b4d4-455b-9f5e-fc3641a0ebd1 |
| project_id           | f2aeffb34ff34ffb8959f1cd813655c6     |
| revision_number      | 0                                    |
| segment_id           | None                                 |
| service_types        |                                      |
| subnetpool_id        | None                                 |
| tags                 |                                      |
| updated_at           | 2024-04-23T11:58:16Z                 |
+----------------------+--------------------------------------+
```

DHCP サーバのポートの作成を確認する。

```sh
openstack port list
```

```
+--------------------------------------+------+-------------------+-------------------------------------------------------------------------------+--------+
| ID                                   | Name | MAC Address       | Fixed IP Addresses                                                            | Status |
+--------------------------------------+------+-------------------+-------------------------------------------------------------------------------+--------+
| 86f066d2-2b37-4d29-8623-abe0d45248bd |      | fa:16:3e:b4:b1:36 | ip_address='192.168.101.2', subnet_id='f09a54e3-3b18-4dfa-9dda-423ed0641889'  | ACTIVE |
+--------------------------------------+------+-------------------+-------------------------------------------------------------------------------+--------+
```

```sh
openstack port show 86f066d2-2b37-4d29-8623-abe0d45248bd
```

```
+-------------------------+-------------------------------------------------------------------------------+
| Field                   | Value                                                                         |
+-------------------------+-------------------------------------------------------------------------------+
| admin_state_up          | UP                                                                            |
| allowed_address_pairs   |                                                                               |
| binding_host_id         | None                                                                          |
| binding_profile         | None                                                                          |
| binding_vif_details     | None                                                                          |
| binding_vif_type        | None                                                                          |
| binding_vnic_type       | normal                                                                        |
| created_at              | 2024-04-23T11:58:17Z                                                          |
| data_plane_status       | None                                                                          |
| description             |                                                                               |
| device_id               | dhcpd3377d3c-a0d1-5d71-9947-f17125c357bb-9eb0ca8c-b4d4-455b-9f5e-fc3641a0ebd1 |
| device_owner            | network:dhcp                                                                  |
| device_profile          | None                                                                          |
| dns_assignment          | None                                                                          |
| dns_domain              | None                                                                          |
| dns_name                | None                                                                          |
| extra_dhcp_opts         |                                                                               |
| fixed_ips               | ip_address='192.168.101.2', subnet_id='f09a54e3-3b18-4dfa-9dda-423ed0641889'  |
| id                      | 86f066d2-2b37-4d29-8623-abe0d45248bd                                          |
| ip_allocation           | None                                                                          |
| mac_address             | fa:16:3e:b4:b1:36                                                             |
| name                    |                                                                               |
| network_id              | 9eb0ca8c-b4d4-455b-9f5e-fc3641a0ebd1                                          |
| numa_affinity_policy    | None                                                                          |
| port_security_enabled   | False                                                                         |
| project_id              | f2aeffb34ff34ffb8959f1cd813655c6                                              |
| propagate_uplink_status | None                                                                          |
| qos_network_policy_id   | None                                                                          |
| qos_policy_id           | None                                                                          |
| resource_request        | None                                                                          |
| revision_number         | 3                                                                             |
| security_group_ids      |                                                                               |
| status                  | ACTIVE                                                                        |
| tags                    |                                                                               |
| trunk_details           | None                                                                          |
| updated_at              | 2024-04-23T11:58:19Z                                                          |
+-------------------------+-------------------------------------------------------------------------------+
```

## 環境の確認

Controller Node でネットワーク構成を確認する。

![Open vSwitch vxlan ネットワーク](../../_static/image/network_vxlan_dhcpagent_ovs.png "Open vSwitch vxlan ネットワーク")

### ネットワーク名前空間

サブネットを作成するとネットワーク名前空間が作成される。

```sh
ip netns
```

```
(...)

qdhcp-9eb0ca8c-b4d4-455b-9f5e-fc3641a0ebd1 (id: 2)
```

### デバイス

ブリッジが作成される。

```sh
ip -d link show
```

```
(...)

14: br-tun: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 32:99:d7:17:c4:42 brd ff:ff:ff:ff:ff:ff promiscuity 1 minmtu 68 maxmtu 65535
    openvswitch addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535
16: vxlan_sys_4789: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 65000 qdisc noqueue master ovs-system state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether ae:ce:ab:31:34:5a brd ff:ff:ff:ff:ff:ff promiscuity 1 minmtu 68 maxmtu 65535
    vxlan external id 0 srcport 0 0 dstport 4789 nolearning ttl auto ageing 300 udpcsum noudp6zerocsumtx udp6zerocsumrx
    openvswitch_slave addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535
```

ネットワーク名前空間内のデバイスを確認する。

```sh
ip netns exec qdhcp-9eb0ca8c-b4d4-455b-9f5e-fc3641a0ebd1 ip -d link show
```

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00 promiscuity 0 minmtu 0 maxmtu 0 addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535
15: tap86f066d2-2b: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether fa:16:3e:b4:b1:36 brd ff:ff:ff:ff:ff:ff promiscuity 1 minmtu 68 maxmtu 65535
    openvswitch addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535
```

データパス(と同じ名前のブリッジ)に接続されたデバイスを確認する。

```sh
ip link show master ovs-system
```

```
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq master ovs-system state UP mode DEFAULT group default qlen 1000
    link/ether 00:15:5d:bf:ba:43 brd ff:ff:ff:ff:ff:ff
16: vxlan_sys_4789: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 65000 qdisc noqueue master ovs-system state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether ae:ce:ab:31:34:5a brd ff:ff:ff:ff:ff:ff
```

### Open vSwitch

ブリッジを確認する。

VNI は `in_key`, `out_key` の設定に従って flow で決まる。

```sh
ovs-vsctl show
```

```
77a2e96a-ca65-449f-afc7-c7cbe9dff27c
    Manager "ptcp:6640:127.0.0.1"
        is_connected: true
    Bridge br-tun
        Controller "tcp:127.0.0.1:6633"
            is_connected: true
        fail_mode: secure
        datapath_type: system
        Port br-tun
            Interface br-tun
                type: internal
        Port patch-int
            Interface patch-int
                type: patch
                options: {peer=patch-tun}
        Port vxlan-ac10001f
            Interface vxlan-ac10001f
                type: vxlan
                options: {df_default="true", egress_pkt_mark="0", in_key=flow, local_ip="172.16.0.11", out_key=flow, remote_ip="172.16.0.31"}
    Bridge br-int
        Controller "tcp:127.0.0.1:6633"
            is_connected: true
        fail_mode: secure
        datapath_type: system
        Port tap86f066d2-2b
            tag: 3
            Interface tap86f066d2-2b
                type: internal
        Port int-br-provider
            Interface int-br-provider
                type: patch
                options: {peer=phy-br-provider}
        Port tap43bf8dcc-08
            tag: 1
            Interface tap43bf8dcc-08
                type: internal
        Port tapeb4f2e2e-d9
            tag: 2
            Interface tapeb4f2e2e-d9
                type: internal
        Port br-int
            Interface br-int
                type: internal
        Port patch-tun
            Interface patch-tun
                type: patch
                options: {peer=patch-int}
    Bridge br-provider
        Controller "tcp:127.0.0.1:6633"
            is_connected: true
        fail_mode: secure
        datapath_type: system
        Port phy-br-provider
            Interface phy-br-provider
                type: patch
                options: {peer=int-br-provider}
        Port eth0
            Interface eth0
                type: system
        Port br-provider
            Interface br-provider
                type: internal
    ovs_version: "3.1.4"
```

フローを確認する。

```sh
ovs-ofctl show br-tun
```

```
OFPT_FEATURES_REPLY (xid=0x2): dpid:00003299d717c442
n_tables:254, n_buffers:0
capabilities: FLOW_STATS TABLE_STATS PORT_STATS QUEUE_STATS ARP_MATCH_IP
actions: output enqueue set_vlan_vid set_vlan_pcp strip_vlan mod_dl_src mod_dl_dst mod_nw_src mod_nw_dst mod_nw_tos mod_tp_src mod_tp_dst
 1(patch-int): addr:fe:85:cd:0c:06:43
     config:     0
     state:      0
     speed: 0 Mbps now, 0 Mbps max
 2(vxlan-ac10001f): addr:76:a0:ec:68:e7:de
     config:     0
     state:      0
     speed: 0 Mbps now, 0 Mbps max
 LOCAL(br-tun): addr:32:99:d7:17:c4:42
     config:     PORT_DOWN
     state:      LINK_DOWN
     speed: 0 Mbps now, 0 Mbps max
OFPT_GET_CONFIG_REPLY (xid=0x4): frags=normal miss_send_len=0
```

```sh
ovs-ofctl show br-provider
```

```
(...)
```

```sh
ovs-ofctl show br-int
```

```
OFPT_FEATURES_REPLY (xid=0x2): dpid:0000a6a90d2e194e
n_tables:254, n_buffers:0
capabilities: FLOW_STATS TABLE_STATS PORT_STATS QUEUE_STATS ARP_MATCH_IP
actions: output enqueue set_vlan_vid set_vlan_pcp strip_vlan mod_dl_src mod_dl_dst mod_nw_src mod_nw_dst mod_nw_tos mod_tp_src mod_tp_dst
 1(int-br-provider): addr:06:06:b7:07:7e:62
     config:     0
     state:      0
     speed: 0 Mbps now, 0 Mbps max
 2(tapeb4f2e2e-d9): addr:fa:16:3e:8a:1b:88
     config:     0
     state:      0
     speed: 0 Mbps now, 0 Mbps max
 5(tap43bf8dcc-08): addr:fa:16:3e:4a:1e:19
     config:     0
     state:      0
     speed: 0 Mbps now, 0 Mbps max
 9(patch-tun): addr:06:e0:ca:62:60:b3
     config:     0
     state:      0
     speed: 0 Mbps now, 0 Mbps max
 10(tap86f066d2-2b): addr:fa:16:3e:b4:b1:36
     config:     0
     state:      0
     speed: 0 Mbps now, 0 Mbps max
 LOCAL(br-int): addr:a6:a9:0d:2e:19:4e
     config:     0
     state:      0
     speed: 0 Mbps now, 0 Mbps max
OFPT_GET_CONFIG_REPLY (xid=0x4): frags=normal miss_send_len=0
```

フローのエントリを確認する。

tag 3 のパケットは VNI 203 (0xcb) で vxlan-ac10001f に流れる。

```sh
ovs-ofctl dump-flows br-tun
```

```
 cookie=0x7751792768bcbae5, duration=708.992s, table=0, n_packets=48, n_bytes=6880, priority=1,in_port="patch-int" actions=resubmit(,2)
 cookie=0x7751792768bcbae5, duration=578.235s, table=0, n_packets=0, n_bytes=0, priority=1,in_port="vxlan-ac10001f" actions=resubmit(,4)
 cookie=0x7751792768bcbae5, duration=708.991s, table=0, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x7751792768bcbae5, duration=708.990s, table=2, n_packets=0, n_bytes=0, priority=0,dl_dst=00:00:00:00:00:00/01:00:00:00:00:00 actions=resubmit(,20)
 cookie=0x7751792768bcbae5, duration=708.988s, table=2, n_packets=48, n_bytes=6880, priority=0,dl_dst=01:00:00:00:00:00/01:00:00:00:00:00 actions=resubmit(,22)
 cookie=0x7751792768bcbae5, duration=708.988s, table=3, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x7751792768bcbae5, duration=659.563s, table=4, n_packets=0, n_bytes=0, priority=1,tun_id=0xcb actions=mod_vlan_vid:3,resubmit(,10)
 cookie=0x7751792768bcbae5, duration=708.987s, table=4, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x7751792768bcbae5, duration=708.986s, table=6, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x7751792768bcbae5, duration=708.985s, table=10, n_packets=0, n_bytes=0, priority=1 actions=learn(table=20,hard_timeout=300,priority=1,cookie=0x7751792768bcbae5,NXM_OF_VLAN_TCI[0..11],NXM_OF_ETH_DST[]=NXM_OF_ETH_SRC[],load:0->NXM_OF_VLAN_TCI[],load:NXM_NX_TUN_ID[]->NXM_NX_TUN_ID[],output:OXM_OF_IN_PORT[]),output:"patch-int"
 cookie=0x7751792768bcbae5, duration=708.984s, table=20, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,22)
 cookie=0x7751792768bcbae5, duration=578.233s, table=22, n_packets=3, n_bytes=210, priority=1,dl_vlan=3 actions=strip_vlan,load:0xcb->NXM_NX_TUN_ID[],output:"vxlan-ac10001f"
 cookie=0x7751792768bcbae5, duration=708.984s, table=22, n_packets=45, n_bytes=6670, priority=0 actions=drop
```

```sh
ovs-ofctl dump-flows br-provider
```

```
 cookie=0x1f66204c052f51e3, duration=753.471s, table=0, n_packets=7, n_bytes=490, priority=4,in_port="phy-br-provider",dl_vlan=2 actions=mod_vlan_vid:100,NORMAL
 cookie=0x1f66204c052f51e3, duration=753.464s, table=0, n_packets=8, n_bytes=560, priority=4,in_port="phy-br-provider",dl_vlan=1 actions=strip_vlan,NORMAL
 cookie=0x1f66204c052f51e3, duration=756.258s, table=0, n_packets=12, n_bytes=916, priority=2,in_port="phy-br-provider" actions=drop
 cookie=0x1f66204c052f51e3, duration=756.261s, table=0, n_packets=329, n_bytes=60664, priority=0 actions=NORMAL
```

```sh
ovs-ofctl dump-flows br-int
```

```
 cookie=0x7cfdaeb53fbd8607, duration=769.095s, table=0, n_packets=0, n_bytes=0, priority=65535,dl_vlan=4095 actions=drop
 cookie=0x7cfdaeb53fbd8607, duration=766.295s, table=0, n_packets=0, n_bytes=0, priority=3,in_port="int-br-provider",dl_vlan=100 actions=mod_vlan_vid:2,resubmit(,59)
 cookie=0x7cfdaeb53fbd8607, duration=766.289s, table=0, n_packets=307, n_bytes=59142, priority=3,in_port="int-br-provider",vlan_tci=0x0000/0x1fff actions=mod_vlan_vid:1,resubmit(,59)
 cookie=0x7cfdaeb53fbd8607, duration=769.085s, table=0, n_packets=18, n_bytes=1820, priority=2,in_port="int-br-provider" actions=drop
 cookie=0x7cfdaeb53fbd8607, duration=769.098s, table=0, n_packets=83, n_bytes=6386, priority=0 actions=resubmit(,59)
 cookie=0x7cfdaeb53fbd8607, duration=769.099s, table=23, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x7cfdaeb53fbd8607, duration=769.096s, table=24, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x7cfdaeb53fbd8607, duration=769.094s, table=30, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,59)
 cookie=0x7cfdaeb53fbd8607, duration=769.093s, table=31, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,59)
 cookie=0x7cfdaeb53fbd8607, duration=769.097s, table=59, n_packets=390, n_bytes=65528, priority=0 actions=resubmit(,60)
 cookie=0x7cfdaeb53fbd8607, duration=766.253s, table=60, n_packets=7, n_bytes=490, priority=100,in_port="tapeb4f2e2e-d9" actions=load:0x2->NXM_NX_REG5[],load:0x2->NXM_NX_REG6[],resubmit(,73)
 cookie=0x7cfdaeb53fbd8607, duration=766.253s, table=60, n_packets=8, n_bytes=560, priority=100,in_port="tap43bf8dcc-08" actions=load:0x5->NXM_NX_REG5[],load:0x1->NXM_NX_REG6[],resubmit(,73)
 cookie=0x7cfdaeb53fbd8607, duration=716.648s, table=60, n_packets=12, n_bytes=916, priority=100,in_port="tap86f066d2-2b" actions=load:0xa->NXM_NX_REG5[],load:0x3->NXM_NX_REG6[],resubmit(,73)
 cookie=0x7cfdaeb53fbd8607, duration=769.096s, table=60, n_packets=363, n_bytes=63562, priority=3 actions=NORMAL
 cookie=0x7cfdaeb53fbd8607, duration=769.094s, table=62, n_packets=0, n_bytes=0, priority=3 actions=NORMAL
 cookie=0x7cfdaeb53fbd8607, duration=767.638s, table=71, n_packets=0, n_bytes=0, priority=110,ct_state=+trk actions=ct_clear,resubmit(,71)
 cookie=0x7cfdaeb53fbd8607, duration=767.688s, table=71, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x7cfdaeb53fbd8607, duration=767.678s, table=72, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x7cfdaeb53fbd8607, duration=766.253s, table=73, n_packets=7, n_bytes=490, priority=80,reg5=0x2 actions=resubmit(,94)
 cookie=0x7cfdaeb53fbd8607, duration=766.253s, table=73, n_packets=8, n_bytes=560, priority=80,reg5=0x5 actions=resubmit(,94)
 cookie=0x7cfdaeb53fbd8607, duration=716.648s, table=73, n_packets=12, n_bytes=916, priority=80,reg5=0xa actions=resubmit(,94)
 cookie=0x7cfdaeb53fbd8607, duration=767.669s, table=73, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x7cfdaeb53fbd8607, duration=767.659s, table=81, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x7cfdaeb53fbd8607, duration=767.650s, table=82, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x7cfdaeb53fbd8607, duration=767.619s, table=91, n_packets=0, n_bytes=0, priority=1 actions=resubmit(,94)
 cookie=0x7cfdaeb53fbd8607, duration=767.610s, table=92, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x7cfdaeb53fbd8607, duration=767.599s, table=93, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x7cfdaeb53fbd8607, duration=767.628s, table=94, n_packets=13, n_bytes=986, priority=1 actions=NORMAL
```

データパスを確認する。

```sh
ovs-dpctl show
```

```
system@ovs-system:
  lookups: hit:2091 missed:403 lost:0
  flows: 0
  masks: hit:3967 total:0 hit/pkt:1.59
  cache: hit:673 hit-rate:26.98%
  caches:
    masks-cache: size:256
  port 0: ovs-system (internal)
  port 1: vxlan_sys_4789 (vxlan: packet_type=ptap)
  port 2: br-tun (internal)
  port 3: tap43bf8dcc-08 (internal)
  port 4: tapeb4f2e2e-d9 (internal)
  port 5: br-int (internal)
  port 6: tap86f066d2-2b (internal)
  port 7: eth0
  port 8: br-provider (internal)
```

トンネルを確認する。

```sh
ovs-appctl ofproto/list-tunnels
```

```
port 1: vxlan-ac10001f (vxlan: 172.16.0.11->172.16.0.31, key=flow, legacy_l2, dp port=1, ttl=64)
```

Compute Node 側でパケットキャプチャして VNI の設定を確認する。

```sh
tcpdump -n -e -i eth0
```

```
(...)

12:17:49.205780 00:15:5d:bf:ba:43 > 00:15:5d:bf:ba:42, ethertype IPv4 (0x0800), length 148: 172.16.0.11.44957 > 172.16.0.31.vxlan: VXLAN, flags [I] (0x08), vni 203
```

VXLAN デバイス vxlan_sys_4789 をキャプチャしても VNI はない。

### イーサネット

ネットワーク名前空間内のイーサネットの情報を確認する。
169.254.169.254 は Metadata agent が使用する。

```sh
ip netns exec qdhcp-9eb0ca8c-b4d4-455b-9f5e-fc3641a0ebd1 ip addr show
```

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
15: tap86f066d2-2b: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ether fa:16:3e:b4:b1:36 brd ff:ff:ff:ff:ff:ff
    inet 192.168.101.2/24 brd 192.168.101.255 scope global tap86f066d2-2b
       valid_lft forever preferred_lft forever
    inet 169.254.169.254/32 brd 169.254.169.254 scope global tap86f066d2-2b
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:feb4:b136/64 scope link
       valid_lft forever preferred_lft forever
```

ルーティングを確認する。

```sh
ip netns exec qdhcp-9eb0ca8c-b4d4-455b-9f5e-fc3641a0ebd1 ip route
```

```
default via 192.168.101.1 dev tap86f066d2-2b proto static
192.168.101.0/24 dev tap86f066d2-2b proto kernel scope link src 192.168.101.2
```

待ち受けているポートを確認する。

```sh
ip netns exec qdhcp-9eb0ca8c-b4d4-455b-9f5e-fc3641a0ebd1 ss -ano -4
```

```
Netid              State               Recv-Q              Send-Q                             Local Address:Port                             Peer Address:Port              Process
udp                UNCONN              0                   0                                      127.0.0.1:53                                    0.0.0.0:*
udp                UNCONN              0                   0                                  192.168.101.2:53                                    0.0.0.0:*
udp                UNCONN              0                   0                                169.254.169.254:53                                    0.0.0.0:*
udp                UNCONN              0                   0                                        0.0.0.0:67                                    0.0.0.0:*
tcp                LISTEN              0                   128                              169.254.169.254:80                                    0.0.0.0:*
tcp                LISTEN              0                   32                                     127.0.0.1:53                                    0.0.0.0:*
tcp                LISTEN              0                   32                                 192.168.101.2:53                                    0.0.0.0:*
tcp                LISTEN              0                   32                               169.254.169.254:53                                    0.0.0.0:*
```

### DHCP agent

dnsmasq のプロセスを確認する。

```sh
ps ax | grep dnsmasq
```

以下が動作していることが確認できる。

```
dnsmasq \
    --no-hosts \
    --no-resolv \
    --pid-file=/var/lib/neutron/dhcp/9eb0ca8c-b4d4-455b-9f5e-fc3641a0ebd1/pid \
    --dhcp-hostsfile=/var/lib/neutron/dhcp/9eb0ca8c-b4d4-455b-9f5e-fc3641a0ebd1/host \
    --addn-hosts=/var/lib/neutron/dhcp/9eb0ca8c-b4d4-455b-9f5e-fc3641a0ebd1/addn_hosts \
    --dhcp-optsfile=/var/lib/neutron/dhcp/9eb0ca8c-b4d4-455b-9f5e-fc3641a0ebd1/opts \
    --dhcp-leasefile=/var/lib/neutron/dhcp/9eb0ca8c-b4d4-455b-9f5e-fc3641a0ebd1/leases \
    --dhcp-match=set:ipxe,175 \
    --dhcp-userclass=set:ipxe6,iPXE \
    --local-service \
    --bind-dynamic \
    --dhcp-range=set:subnet-f09a54e3-3b18-4dfa-9dda-423ed0641889,192.168.101.0,static,255.255.255.0,86400s \
    --dhcp-option-force=option:mtu,1450 \
    --dhcp-lease-max=256 \
    --conf-file=/dev/null \
    --domain=openstacklocal
```

使用しているインターフェイスを確認する。

```sh
cat /var/lib/neutron/dhcp/9eb0ca8c-b4d4-455b-9f5e-fc3641a0ebd1/interface
```

```
tap86f066d2-2b
```
