# インスタンスの作成 (vxlan/Open vSwitch)

vxlan ネットワーク(Open vSwitch)に接続するインスタンスを作成する。

## 前提条件

* [](../network/ovs_vxlan) を作成していること。
* flavor [](../flavor/m1_nano) を作成していること。
* イメージ [](../../installation/controller/glance) でイメージを作成していること。
* セキュリティグループのルール [](../security_group/icmp) を作成していること。
* セキュリティグループのルール [](../security_group/ssh) を作成していること。

## インスタンスの作成

インスタンス instance02 を作成する。

```sh
openstack server create \
    --flavor m1.nano \
    --image cirros \
    --nic net-id=9eb0ca8c-b4d4-455b-9f5e-fc3641a0ebd1 \
    --security-group default \
    --key-name mykey \
    instance02
```

```
+-----------------------------+-----------------------------------------------+
| Field                       | Value                                         |
+-----------------------------+-----------------------------------------------+
| OS-DCF:diskConfig           | MANUAL                                        |
| OS-EXT-AZ:availability_zone |                                               |
| OS-EXT-STS:power_state      | NOSTATE                                       |
| OS-EXT-STS:task_state       | scheduling                                    |
| OS-EXT-STS:vm_state         | building                                      |
| OS-SRV-USG:launched_at      | None                                          |
| OS-SRV-USG:terminated_at    | None                                          |
| accessIPv4                  |                                               |
| accessIPv6                  |                                               |
| addresses                   |                                               |
| adminPass                   | K7FKqBfuSz4A                                  |
| config_drive                |                                               |
| created                     | 2024-04-23T12:28:27Z                          |
| flavor                      | m1.nano (0)                                   |
| hostId                      |                                               |
| id                          | 36b496b2-4976-4bf1-88c0-09f44385fd19          |
| image                       | cirros (e83903c4-7fa8-42a7-b693-f5034bc33603) |
| key_name                    | mykey                                         |
| name                        | instance02                                    |
| progress                    | 0                                             |
| project_id                  | f2aeffb34ff34ffb8959f1cd813655c6              |
| properties                  |                                               |
| security_groups             | name='87fd4685-d317-42fb-a487-28382d2c2750'   |
| status                      | BUILD                                         |
| updated                     | 2024-04-23T12:28:27Z                          |
| user_id                     | 71b5948c75f24c0f841dbf1c4eb4c4a7              |
| volumes_attached            |                                               |
+-----------------------------+-----------------------------------------------+
```

## インスタンスの確認

インスタンスが ACTIVE になったことを確認する。

```sh
openstack server list
```

```
+--------------------------------------+------------+---------+-----------------------------+--------+---------+
| ID                                   | Name       | Status  | Networks                    | Image  | Flavor  |
+--------------------------------------+------------+---------+-----------------------------+--------+---------+
| 36b496b2-4976-4bf1-88c0-09f44385fd19 | instance02 | ACTIVE  | selfservice=192.168.101.19  | cirros | m1.nano |
+--------------------------------------+------------+---------+-----------------------------+--------+---------+
```

## 環境の確認

### dnsmasq

DHCP で IP アドレスが払い出されている。

```sh
cat /var/lib/neutron/dhcp/9eb0ca8c-b4d4-455b-9f5e-fc3641a0ebd1/leases
```

```
1713961748 fa:16:3e:55:18:7a 192.168.101.19 host-192-168-101-19 01:fa:16:3e:55:18:7a
```

DHCP に MAC アドレスと IP アドレスの関連が追加される。

```sh
cat /var/lib/neutron/dhcp/9eb0ca8c-b4d4-455b-9f5e-fc3641a0ebd1/host
```

```
fa:16:3e:55:18:7a,host-192-168-101-19.openstacklocal,192.168.101.19
```

DNS のエントリが追加される。

```sh
cat /var/lib/neutron/dhcp/9eb0ca8c-b4d4-455b-9f5e-fc3641a0ebd1/addn_hosts
```

```
192.168.101.19  host-192-168-101-19.openstacklocal host-192-168-101-19
```

### インスタンス

Compute Node で確認する。

```sh
virsh list
```

```
 Id   名前                状態
----------------------------------
 1    instance-00000029   実行中
```

ネットワークインターフェイスの設定を確認する。
```sh
virsh dumpxml 1 | sed -n -e '/<interface/,/<\/interface>/ { p }'
```

```xml
<interface type='ethernet'>
  <mac address='fa:16:3e:55:18:7a'/>
  <target dev='tap8811e933-5c'/>
  <model type='virtio'/>
  <driver name='qemu'/>
  <mtu size='1450'/>
  <alias name='net0'/>
  <address type='pci' domain='0x0000' bus='0x00' slot='0x03' function='0x0'/>
</interface>
```

### ネットワーク

Compute Node でネットワーク構成を確認する。

![Open vSwitch vxlan ネットワーク](../../_static/image/network_vxlan_instance_ovs.png "Open vSwitch vxlan ネットワーク")

#### ネットワーク名前空間

ネットワーク名前空間は作成されない。

#### デバイス

TAP デバイスが追加される。

```sh
ip -d link show
```

```
(...)

7: br-tun: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether a6:9e:8e:08:7b:4a brd ff:ff:ff:ff:ff:ff promiscuity 1 minmtu 68 maxmtu 65535
    openvswitch addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535
8: vxlan_sys_4789: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 65000 qdisc noqueue master ovs-system state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether 6a:7f:82:d7:0d:95 brd ff:ff:ff:ff:ff:ff promiscuity 1 minmtu 68 maxmtu 65535
    vxlan external id 0 srcport 0 0 dstport 4789 nolearning ttl auto ageing 300 udpcsum noudp6zerocsumtx udp6zerocsumrx
    openvswitch_slave addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535
9: tap8811e933-5c: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue master ovs-system state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether fe:16:3e:55:18:7a brd ff:ff:ff:ff:ff:ff promiscuity 1 minmtu 68 maxmtu 65521
    tun type tap pi off vnet_hdr on persist off
    openvswitch_slave addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535
```

#### Open vSwitch

ブリッジを確認する。

VNI は `in_key`, `out_key` の設定に従って flow で決まる。

```sh
ovs-vsctl show
```

```
9ab7209e-d2af-4403-9ddb-416bc283b632
    Manager "ptcp:6640:127.0.0.1"
        is_connected: true
    Bridge br-tun
        Controller "tcp:127.0.0.1:6633"
            is_connected: true
        fail_mode: secure
        datapath_type: system
        Port patch-int
            Interface patch-int
                type: patch
                options: {peer=patch-tun}
        Port vxlan-ac10000b
            Interface vxlan-ac10000b
                type: vxlan
                options: {df_default="true", egress_pkt_mark="0", in_key=flow, local_ip="172.16.0.31", out_key=flow, remote_ip="172.16.0.11"}
        Port br-tun
            Interface br-tun
                type: internal
    Bridge br-provider
        Controller "tcp:127.0.0.1:6633"
            is_connected: true
        fail_mode: secure
        datapath_type: system
        Port eth0
            Interface eth0
                type: system
        Port phy-br-provider
            Interface phy-br-provider
                type: patch
                options: {peer=int-br-provider}
        Port br-provider
            Interface br-provider
                type: internal
    Bridge br-int
        Controller "tcp:127.0.0.1:6633"
            is_connected: true
        fail_mode: secure
        datapath_type: system
        Port patch-tun
            Interface patch-tun
                type: patch
                options: {peer=patch-int}
        Port int-br-provider
            Interface int-br-provider
                type: patch
                options: {peer=phy-br-provider}
        Port tap8811e933-5c
            tag: 3
            Interface tap8811e933-5c
        Port br-int
            Interface br-int
                type: internal
    ovs_version: "3.1.4"
```

フローを確認する。

```sh
ovs-ofctl show br-tun
```

```
OFPT_FEATURES_REPLY (xid=0x2): dpid:0000a69e8e087b4a
n_tables:254, n_buffers:0
capabilities: FLOW_STATS TABLE_STATS PORT_STATS QUEUE_STATS ARP_MATCH_IP
actions: output enqueue set_vlan_vid set_vlan_pcp strip_vlan mod_dl_src mod_dl_dst mod_nw_src mod_nw_dst mod_nw_tos mod_tp_src mod_tp_dst
 1(patch-int): addr:92:72:37:95:20:eb
     config:     0
     state:      0
     speed: 0 Mbps now, 0 Mbps max
 2(vxlan-ac10000b): addr:02:6d:a8:b9:a5:f7
     config:     0
     state:      0
     speed: 0 Mbps now, 0 Mbps max
 LOCAL(br-tun): addr:a6:9e:8e:08:7b:4a
     config:     PORT_DOWN
     state:      LINK_DOWN
     speed: 0 Mbps now, 0 Mbps max
OFPT_GET_CONFIG_REPLY (xid=0x4): frags=normal miss_send_len=0
```

```sh
ovs-ofctl show br-provider
```

```
OFPT_FEATURES_REPLY (xid=0x2): dpid:000000155dbfba42
n_tables:254, n_buffers:0
capabilities: FLOW_STATS TABLE_STATS PORT_STATS QUEUE_STATS ARP_MATCH_IP
actions: output enqueue set_vlan_vid set_vlan_pcp strip_vlan mod_dl_src mod_dl_dst mod_nw_src mod_nw_dst mod_nw_tos mod_tp_src mod_tp_dst
 2(phy-br-provider): addr:2a:3b:a9:0c:4c:b7
     config:     0
     state:      0
     speed: 0 Mbps now, 0 Mbps max
 3(eth0): addr:00:15:5d:bf:ba:42
     config:     0
     state:      0
     current:    10GB-FD
     speed: 10000 Mbps now, 0 Mbps max
 LOCAL(br-provider): addr:00:15:5d:bf:ba:42
     config:     0
     state:      0
     speed: 0 Mbps now, 0 Mbps max
OFPT_GET_CONFIG_REPLY (xid=0x4): frags=normal miss_send_len=0
```

```sh
ovs-ofctl show br-int
```

```
OFPT_FEATURES_REPLY (xid=0x2): dpid:0000d68b3895ac45
n_tables:254, n_buffers:0
capabilities: FLOW_STATS TABLE_STATS PORT_STATS QUEUE_STATS ARP_MATCH_IP
actions: output enqueue set_vlan_vid set_vlan_pcp strip_vlan mod_dl_src mod_dl_dst mod_nw_src mod_nw_dst mod_nw_tos mod_tp_src mod_tp_dst
 1(int-br-provider): addr:56:24:1d:c4:bd:57
     config:     0
     state:      0
     speed: 0 Mbps now, 0 Mbps max
 2(patch-tun): addr:92:af:c7:29:a9:90
     config:     0
     state:      0
     speed: 0 Mbps now, 0 Mbps max
 3(tap8811e933-5c): addr:fe:16:3e:55:18:7a
     config:     0
     state:      0
     current:    10MB-FD COPPER
     speed: 10 Mbps now, 0 Mbps max
 LOCAL(br-int): addr:d6:8b:38:95:ac:45
     config:     0
     state:      0
     speed: 0 Mbps now, 0 Mbps max
OFPT_GET_CONFIG_REPLY (xid=0x4): frags=normal miss_send_len=0
```

フローのエントリを確認する。

tag 3 のパケットは VNI 203 (0xcb) で vxlan-ac10000b に流れる。

```sh
ovs-ofctl dump-flows br-tun
```

```
 cookie=0x81e0f7737031ced9, duration=6068.201s, table=0, n_packets=391, n_bytes=59854, priority=1,in_port="patch-int" actions=resubmit(,2)
 cookie=0x81e0f7737031ced9, duration=6066.336s, table=0, n_packets=73, n_bytes=8563, priority=1,in_port="vxlan-ac10000b" actions=resubmit(,4)
 cookie=0x81e0f7737031ced9, duration=6068.200s, table=0, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x81e0f7737031ced9, duration=6068.199s, table=2, n_packets=113, n_bytes=10062, priority=0,dl_dst=00:00:00:00:00:00/01:00:00:00:00:00 actions=resubmit(,20)
 cookie=0x81e0f7737031ced9, duration=6068.198s, table=2, n_packets=278, n_bytes=49792, priority=0,dl_dst=01:00:00:00:00:00/01:00:00:00:00:00 actions=resubmit(,22)
 cookie=0x81e0f7737031ced9, duration=6068.197s, table=3, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x81e0f7737031ced9, duration=4329.134s, table=4, n_packets=69, n_bytes=8283, priority=1,tun_id=0xcb actions=mod_vlan_vid:3,resubmit(,10)
 cookie=0x81e0f7737031ced9, duration=6068.196s, table=4, n_packets=4, n_bytes=280, priority=0 actions=drop
 cookie=0x81e0f7737031ced9, duration=6068.196s, table=6, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x81e0f7737031ced9, duration=6068.195s, table=10, n_packets=69, n_bytes=8283, priority=1 actions=learn(table=20,hard_timeout=300,priority=1,cookie=0x81e0f7737031ced9,NXM_OF_VLAN_TCI[0..11],NXM_OF_ETH_DST[]=NXM_OF_ETH_SRC[],load:0->NXM_OF_VLAN_TCI[],load:NXM_NX_TUN_ID[]->NXM_NX_TUN_ID[],output:OXM_OF_IN_PORT[]),output:"patch-int"
 cookie=0x81e0f7737031ced9, duration=6068.194s, table=20, n_packets=5, n_bytes=716, priority=0 actions=resubmit(,22)
 cookie=0x81e0f7737031ced9, duration=4329.145s, table=22, n_packets=15, n_bytes=1316, priority=1,dl_vlan=3 actions=strip_vlan,load:0xcb->NXM_NX_TUN_ID[],output:"vxlan-ac10000b"
 cookie=0x81e0f7737031ced9, duration=6068.193s, table=22, n_packets=268, n_bytes=49192, priority=0 actions=drop
```

```sh
ovs-ofctl dump-flows br-provider
```

```
 cookie=0xd49325e788c834f8, duration=6080.989s, table=0, n_packets=0, n_bytes=0, priority=4,in_port="phy-br-provider",dl_vlan=1 actions=strip_vlan,NORMAL
 cookie=0xd49325e788c834f8, duration=6080.983s, table=0, n_packets=0, n_bytes=0, priority=4,in_port="phy-br-provider",dl_vlan=2 actions=mod_vlan_vid:100,NORMAL
 cookie=0xd49325e788c834f8, duration=6084.272s, table=0, n_packets=125, n_bytes=10802, priority=2,in_port="phy-br-provider" actions=drop
 cookie=0xd49325e788c834f8, duration=6084.275s, table=0, n_packets=764, n_bytes=133255, priority=0 actions=NORMAL
```

```sh
ovs-ofctl dump-flows br-int
```

```
(...)

 cookie=0x574b6b102b172bf4, duration=4371.518s, table=60, n_packets=128, n_bytes=11100, priority=100,in_port="tap8811e933-5c" actions=load:0x3->NXM_NX_REG5[],load:0x3->NXM_NX_REG6[],resubmit(,71)
 cookie=0x574b6b102b172bf4, duration=4371.518s, table=60, n_packets=67, n_bytes=8143, priority=90,dl_vlan=3,dl_dst=fa:16:3e:55:18:7a actions=load:0x3->NXM_NX_REG5[],load:0x3->NXM_NX_REG6[],strip_vlan,resubmit(,81)
 cookie=0x574b6b102b172bf4, duration=4371.518s, table=71, n_packets=11, n_bytes=462, priority=95,arp,reg5=0x3,in_port="tap8811e933-5c",dl_src=fa:16:3e:55:18:7a,arp_spa=192.168.101.19 actions=resubmit(,94)
 cookie=0x574b6b102b172bf4, duration=4371.518s, table=71, n_packets=107, n_bytes=9304, priority=65,ip,reg5=0x3,in_port="tap8811e933-5c",dl_src=fa:16:3e:55:18:7a,nw_src=192.168.101.19 actions=ct(table=72,zone=NXM_NX_REG6[0..15])
 cookie=0x574b6b102b172bf4, duration=4371.518s, table=71, n_packets=0, n_bytes=0, priority=95,icmp6,reg5=0x3,in_port="tap8811e933-5c",dl_src=fa:16:3e:55:18:7a,ipv6_src=fe80::f816:3eff:fe55:187a,icmp_type=130 actions=resubmit(,94)
 cookie=0x574b6b102b172bf4, duration=4371.518s, table=71, n_packets=3, n_bytes=210, priority=95,icmp6,reg5=0x3,in_port="tap8811e933-5c",dl_src=fa:16:3e:55:18:7a,ipv6_src=fe80::f816:3eff:fe55:187a,icmp_type=133 actions=resubmit(,94)
 cookie=0x574b6b102b172bf4, duration=4371.518s, table=71, n_packets=0, n_bytes=0, priority=95,icmp6,reg5=0x3,in_port="tap8811e933-5c",dl_src=fa:16:3e:55:18:7a,ipv6_src=fe80::f816:3eff:fe55:187a,icmp_type=135 actions=resubmit(,94)
 cookie=0x574b6b102b172bf4, duration=4371.517s, table=71, n_packets=0, n_bytes=0, priority=95,icmp6,reg5=0x3,in_port="tap8811e933-5c",icmp_type=136,nd_target=fe80::f816:3eff:fe55:187a actions=resubmit(,94)
 cookie=0x574b6b102b172bf4, duration=4371.517s, table=71, n_packets=0, n_bytes=0, priority=80,udp,reg5=0x3,in_port="tap8811e933-5c",dl_src=fa:16:3e:55:18:7a,nw_src=192.168.101.19,tp_src=68,tp_dst=67 actions=resubmit(,73)
 cookie=0x574b6b102b172bf4, duration=4371.517s, table=71, n_packets=2, n_bytes=686, priority=80,udp,reg5=0x3,in_port="tap8811e933-5c",dl_src=fa:16:3e:55:18:7a,nw_src=0.0.0.0,tp_src=68,tp_dst=67 actions=resubmit(,73)
 cookie=0x574b6b102b172bf4, duration=4371.517s, table=71, n_packets=0, n_bytes=0, priority=80,udp6,reg5=0x3,in_port="tap8811e933-5c",dl_src=fa:16:3e:55:18:7a,ipv6_src=fe80::f816:3eff:fe55:187a,tp_src=546,tp_dst=547 actions=resubmit(,73)
 cookie=0x574b6b102b172bf4, duration=4371.517s, table=71, n_packets=0, n_bytes=0, priority=70,udp,reg5=0x3,in_port="tap8811e933-5c",tp_src=67,tp_dst=68 actions=resubmit(,93)
 cookie=0x574b6b102b172bf4, duration=4371.517s, table=71, n_packets=0, n_bytes=0, priority=70,udp6,reg5=0x3,in_port="tap8811e933-5c",tp_src=547,tp_dst=546 actions=resubmit(,93)
 cookie=0x574b6b102b172bf4, duration=4371.517s, table=71, n_packets=0, n_bytes=0, priority=70,icmp6,reg5=0x3,in_port="tap8811e933-5c",icmp_type=134 actions=resubmit(,93)
 cookie=0x574b6b102b172bf4, duration=4371.517s, table=71, n_packets=2, n_bytes=180, priority=65,ipv6,reg5=0x3,in_port="tap8811e933-5c",dl_src=fa:16:3e:55:18:7a,ipv6_src=fe80::f816:3eff:fe55:187a actions=ct(table=72,zone=NXM_NX_REG6[0..15])
 cookie=0x574b6b102b172bf4, duration=4371.517s, table=71, n_packets=3, n_bytes=258, priority=10,reg5=0x3,in_port="tap8811e933-5c" actions=ct_clear,resubmit(,93)
 cookie=0x574b6b102b172bf4, duration=4371.517s, table=73, n_packets=0, n_bytes=0, priority=100,reg6=0x3,dl_dst=fa:16:3e:55:18:7a actions=load:0x3->NXM_NX_REG5[],resubmit(,81)
 cookie=0x574b6b102b172bf4, duration=4371.516s, table=81, n_packets=2, n_bytes=84, priority=100,arp,reg5=0x3 actions=output:"tap8811e933-5c"
 cookie=0x574b6b102b172bf4, duration=4371.516s, table=81, n_packets=0, n_bytes=0, priority=100,icmp6,reg5=0x3,icmp_type=130 actions=output:"tap8811e933-5c"
 cookie=0x574b6b102b172bf4, duration=4371.516s, table=81, n_packets=0, n_bytes=0, priority=100,icmp6,reg5=0x3,icmp_type=135 actions=output:"tap8811e933-5c"
 cookie=0x574b6b102b172bf4, duration=4371.516s, table=81, n_packets=0, n_bytes=0, priority=100,icmp6,reg5=0x3,icmp_type=136 actions=output:"tap8811e933-5c"
 cookie=0x574b6b102b172bf4, duration=4371.516s, table=81, n_packets=0, n_bytes=0, priority=100,icmp6,reg5=0x3,icmp_type=134 actions=output:"tap8811e933-5c"
 cookie=0x574b6b102b172bf4, duration=4371.516s, table=81, n_packets=2, n_bytes=761, priority=95,udp,reg5=0x3,tp_src=67,tp_dst=68 actions=output:"tap8811e933-5c"
 cookie=0x574b6b102b172bf4, duration=4371.516s, table=81, n_packets=0, n_bytes=0, priority=95,udp6,reg5=0x3,tp_src=547,tp_dst=546 actions=output:"tap8811e933-5c"
 cookie=0x574b6b102b172bf4, duration=4371.515s, table=82, n_packets=0, n_bytes=0, priority=77,ct_state=+est-rel-rpl,tcp,reg5=0x3,tp_dst=22 actions=output:"tap8811e933-5c"
 cookie=0x574b6b102b172bf4, duration=4371.514s, table=82, n_packets=0, n_bytes=0, priority=77,ct_state=+new-est,tcp,reg5=0x3,tp_dst=22 actions=ct(commit,zone=NXM_NX_REG6[0..15]),output:"tap8811e933-5c",resubmit(,92)
 cookie=0x574b6b102b172bf4, duration=4371.514s, table=82, n_packets=0, n_bytes=0, priority=75,ct_state=+est-rel-rpl,icmp,reg5=0x3 actions=output:"tap8811e933-5c"
 cookie=0x574b6b102b172bf4, duration=4371.514s, table=82, n_packets=0, n_bytes=0, priority=75,ct_state=+new-est,icmp,reg5=0x3 actions=ct(commit,zone=NXM_NX_REG6[0..15]),output:"tap8811e933-5c",resubmit(,92)
 cookie=0x574b6b102b172bf4, duration=4371.514s, table=82, n_packets=0, n_bytes=0, priority=70,conj_id=16,ct_state=+est-rel-rpl,ip,reg5=0x3 actions=load:0x10->NXM_NX_REG7[],output:"tap8811e933-5c"
 cookie=0x574b6b102b172bf4, duration=4371.514s, table=82, n_packets=0, n_bytes=0, priority=70,conj_id=24,ct_state=+est-rel-rpl,ipv6,reg5=0x3 actions=load:0x18->NXM_NX_REG7[],output:"tap8811e933-5c"
 cookie=0x574b6b102b172bf4, duration=4371.514s, table=82, n_packets=0, n_bytes=0, priority=70,conj_id=17,ct_state=+new-est,ip,reg5=0x3 actions=load:0x11->NXM_NX_REG7[],ct(commit,zone=NXM_NX_REG6[0..15]),output:"tap8811e933-5c",resubmit(,92)
 cookie=0x574b6b102b172bf4, duration=4371.513s, table=82, n_packets=0, n_bytes=0, priority=70,conj_id=25,ct_state=+new-est,ipv6,reg5=0x3 actions=load:0x19->NXM_NX_REG7[],ct(commit,zone=NXM_NX_REG6[0..15]),output:"tap8811e933-5c",resubmit(,92)
 cookie=0x574b6b102b172bf4, duration=4371.516s, table=82, n_packets=63, n_bytes=7298, priority=50,ct_state=+est-rel+rpl,ct_zone=3,ct_mark=0,reg5=0x3 actions=output:"tap8811e933-5c"
 cookie=0x574b6b102b172bf4, duration=4371.516s, table=82, n_packets=0, n_bytes=0, priority=50,ct_state=-new-est+rel-inv,ct_zone=3,ct_mark=0,reg5=0x3 actions=output:"tap8811e933-5c"
 ```

データパスを確認する。

```sh
ovs-dpctl show
```

```
system@ovs-system:
  lookups: hit:454 missed:219 lost:0
  flows: 0
  masks: hit:558 total:0 hit/pkt:0.83
  cache: hit:296 hit-rate:43.98%
  caches:
    masks-cache: size:256
  port 0: ovs-system (internal)
  port 1: br-int (internal)
  port 2: br-tun (internal)
  port 3: vxlan_sys_4789 (vxlan: packet_type=ptap)
  port 4: eth0
  port 5: br-provider (internal)
  port 6: tap8811e933-5c
```

トンネルを確認する。

```sh
ovs-appctl ofproto/list-tunnels
```

```
port 3: vxlan-ac10000b (vxlan: 172.16.0.31->172.16.0.11, key=flow, legacy_l2, dp port=3, ttl=64)
```
