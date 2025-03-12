# ルータ (Open vSwitch)

## 前提条件

* [](../network/ovs_flat) を作成していること。
* [](../network/ovs_vxlan) を作成していること。

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
| created_at              | 2024-05-18T00:35:54Z                 |
| description             |                                      |
| distributed             | False                                |
| enable_ndp_proxy        | None                                 |
| external_gateway_info   | null                                 |
| flavor_id               | None                                 |
| ha                      | False                                |
| id                      | e6efd80e-9468-4787-a6f8-1648486cc1d6 |
| name                    | router                               |
| project_id              | bccf406c045d401b91ba5c7552a124ae     |
| revision_number         | 1                                    |
| routes                  |                                      |
| status                  | ACTIVE                               |
| tags                    |                                      |
| tenant_id               | bccf406c045d401b91ba5c7552a124ae     |
| updated_at              | 2024-05-18T00:35:54Z                 |
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
| 13e6e8ff-d73e-40d4-8742-f6244a88e6e0 |      | fa:16:3e:38:32:fe | ip_address='192.168.101.254', subnet_id='765053a1-4543-4732-a9ac-47065f945e9f' | ACTIVE |
| ca92be94-798e-45f4-9b7f-c96e0074cade |      | fa:16:3e:a6:11:48 | ip_address='172.16.0.188', subnet_id='02c232a9-e10c-42d6-8087-4515b46449d4'    | ACTIVE |
+--------------------------------------+------+-------------------+--------------------------------------------------------------------------------+--------+
```

## 環境の確認

Controller Node でネットワーク構成を確認する。

![Open vSwitch ルータ](../../_static/image/network_vxlan_router_ovs.png "Open vSwitch ルータ")

### ネットワーク名前空間

ルータを作成するとネットワーク名前空間が作成される。

```sh
ip netns
```

```text
(...)

qrouter-e6efd80e-9468-4787-a6f8-1648486cc1d6 (id: 3)
```

### デバイス

デバイスに変更はない。

```sh
ip -d link show
```

```text
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00 promiscuity 0  allmulti 0 minmtu 0 maxmtu 0 addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 00:15:5d:bf:ba:4f brd ff:ff:ff:ff:ff:ff promiscuity 0  allmulti 0 minmtu 68 maxmtu 65521 addrgenmode none numtxqueues 64 numrxqueues 64 gso_max_size 62780 gso_max_segs 65535 tso_max_size 62780 tso_max_segs 65535 gro_max_size 65536 parentbus vmbus parentdev b7c073a0-7837-4a9f-94e7-eba43ef222ef
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 00:15:5d:bf:ba:54 brd ff:ff:ff:ff:ff:ff promiscuity 0  allmulti 0 minmtu 68 maxmtu 65521 addrgenmode none numtxqueues 64 numrxqueues 64 gso_max_size 62780 gso_max_segs 65535 tso_max_size 62780 tso_max_segs 65535 gro_max_size 65536 parentbus vmbus parentdev 0f15ccb6-3ab3-45ce-b737-a73ecf5a6339
4: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq master ovs-system state UP mode DEFAULT group default qlen 1000
    link/ether 00:15:5d:bf:ba:55 brd ff:ff:ff:ff:ff:ff promiscuity 1  allmulti 0 minmtu 68 maxmtu 65521
    openvswitch_slave addrgenmode none numtxqueues 64 numrxqueues 64 gso_max_size 62780 gso_max_segs 65535 tso_max_size 62780 tso_max_segs 65535 gro_max_size 65536 parentbus vmbus parentdev dffbd9a0-19dd-44c1-9b46-6dfba9829d73
5: eth3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq master ovs-system state UP mode DEFAULT group default qlen 1000
    link/ether 00:15:5d:bf:ba:56 brd ff:ff:ff:ff:ff:ff promiscuity 1  allmulti 0 minmtu 68 maxmtu 65521
    openvswitch_slave addrgenmode none numtxqueues 64 numrxqueues 64 gso_max_size 62780 gso_max_segs 65535 tso_max_size 62780 tso_max_segs 65535 gro_max_size 65536 parentbus vmbus parentdev 31e9f926-7af1-481e-bf58-cbca38bc3cba
6: ovs-system: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether d6:70:ef:61:e9:e5 brd ff:ff:ff:ff:ff:ff promiscuity 1  allmulti 0 minmtu 68 maxmtu 65535
    openvswitch addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 65536 tso_max_segs 65535 gro_max_size 65536
7: br-tun: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether 56:a8:05:61:4b:43 brd ff:ff:ff:ff:ff:ff promiscuity 1  allmulti 0 minmtu 68 maxmtu 65535
    openvswitch addrgenmode none numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 65536 tso_max_segs 65535 gro_max_size 65536
8: vxlan_sys_4789: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 65000 qdisc noqueue master ovs-system state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether 0e:6d:d0:0f:f8:39 brd ff:ff:ff:ff:ff:ff promiscuity 1  allmulti 0 minmtu 68 maxmtu 65535
    vxlan external id 0 srcport 0 0 dstport 4789 nolearning ttl auto ageing 300 udpcsum noudp6zerocsumtx udp6zerocsumrx
    openvswitch_slave addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 65536 tso_max_segs 65535 gro_max_size 65536
9: br-provider: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether 00:15:5d:bf:ba:55 brd ff:ff:ff:ff:ff:ff promiscuity 1  allmulti 0 minmtu 68 maxmtu 65535
    openvswitch addrgenmode none numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 65536 tso_max_segs 65535 gro_max_size 65536
10: br-mgmt: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether 00:15:5d:bf:ba:56 brd ff:ff:ff:ff:ff:ff promiscuity 1  allmulti 0 minmtu 68 maxmtu 65535
    openvswitch addrgenmode none numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 65536 tso_max_segs 65535 gro_max_size 65536
14: br-int: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether 4a:73:de:e4:cd:4f brd ff:ff:ff:ff:ff:ff promiscuity 1  allmulti 0 minmtu 68 maxmtu 65535
    openvswitch addrgenmode none numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 65536 tso_max_segs 65535 gro_max_size 65536
```

ネットワーク名前空間内のデバイスを確認する。

```sh
ip netns exec qrouter-e6efd80e-9468-4787-a6f8-1648486cc1d6 ip -d link show
```

```text
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00 promiscuity 0  allmulti 0 minmtu 0 maxmtu 0 addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
18: qr-13e6e8ff-d7: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether fa:16:3e:38:32:fe brd ff:ff:ff:ff:ff:ff promiscuity 1  allmulti 0 minmtu 68 maxmtu 65535
    openvswitch addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 65536 tso_max_segs 65535 gro_max_size 65536
19: qg-ca92be94-79: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether fa:16:3e:a6:11:48 brd ff:ff:ff:ff:ff:ff promiscuity 1  allmulti 0 minmtu 68 maxmtu 65535
    openvswitch addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 65536 tso_max_segs 65535 gro_max_size 65536
```

### Open vSwitch

ブリッジを確認する。

```sh
ovs-vsctl show
```

```text
2a1ab795-d59f-4a33-a5a1-1fb4c942dce4
    Manager "ptcp:6640:127.0.0.1"
        is_connected: true
    Bridge br-tun
        Controller "tcp:127.0.0.1:6633"
            is_connected: true
        fail_mode: secure
        datapath_type: system
        Port vxlan-ac10001f
            Interface vxlan-ac10001f
                type: vxlan
                options: {df_default="true", egress_pkt_mark="0", in_key=flow, local_ip="172.16.0.11", out_key=flow, remote_ip="172.16.0.31"}
        Port patch-int
            Interface patch-int
                type: patch
                options: {peer=patch-tun}
        Port br-tun
            Interface br-tun
                type: internal
    Bridge br-provider
        Controller "tcp:127.0.0.1:6633"
            is_connected: true
        fail_mode: secure
        datapath_type: system
        Port eth2
            Interface eth2
                type: system
        Port phy-br-provider
            Interface phy-br-provider
                type: patch
                options: {peer=int-br-provider}
    Bridge br-mgmt
        Controller "tcp:127.0.0.1:6633"
            is_connected: true
        fail_mode: secure
        datapath_type: system
        Port phy-br-mgmt
            Interface phy-br-mgmt
                type: patch
                options: {peer=int-br-mgmt}
        Port eth3
            Interface eth3
                type: system
    Bridge br-int
        Controller "tcp:127.0.0.1:6633"
            is_connected: true
        fail_mode: secure
        datapath_type: system
        Port br-int
            Interface br-int
                type: internal
        Port tap183c56bb-b2
            tag: 3
            Interface tap183c56bb-b2
                type: internal
        Port qr-13e6e8ff-d7
            tag: 2
            Interface qr-13e6e8ff-d7
                type: internal
        Port tap2dac2cf0-61
            tag: 2
            Interface tap2dac2cf0-61
                type: internal
        Port tap52b1b85b-8e
            tag: 1
            Interface tap52b1b85b-8e
                type: internal
        Port patch-tun
            Interface patch-tun
                type: patch
                options: {peer=patch-int}
        Port int-br-provider
            Interface int-br-provider
                type: patch
                options: {peer=phy-br-provider}
        Port int-br-mgmt
            Interface int-br-mgmt
                type: patch
                options: {peer=phy-br-mgmt}
        Port qg-ca92be94-79
            tag: 3
            Interface qg-ca92be94-79
                type: internal
    ovs_version: "3.3.1"
```

データパスを確認する。

```sh
ovs-dpctl show
```

```text
system@ovs-system:
  lookups: hit:386 missed:178 lost:0
  flows: 0
  masks: hit:475 total:0 hit/pkt:0.84
  cache: hit:247 hit-rate:43.79%
  caches:
    masks-cache: size:256
  port 0: ovs-system (internal)
  port 1: br-tun (internal)
  port 2: vxlan_sys_4789 (vxlan: packet_type=ptap)
  port 3: br-provider (internal)
  port 4: eth3
  port 5: eth2
  port 6: br-mgmt (internal)
  port 7: tap183c56bb-b2 (internal)
  port 8: tap52b1b85b-8e (internal)
  port 9: tap2dac2cf0-61 (internal)
  port 10: br-int (internal)
  port 11: qr-13e6e8ff-d7 (internal)
  port 12: qg-ca92be94-79 (internal)
```

ブリッジ br-provider のフローのエントリを確認する。

```sh
ovs-ofctl dump-flows br-provider
```

```text
 cookie=0x887c3572d5e57691, duration=5307.666s, table=0, n_packets=8, n_bytes=560, priority=4,in_port="phy-br-provider",dl_vlan=1 actions=mod_vlan_vid:100,NORMAL
 cookie=0x887c3572d5e57691, duration=5307.642s, table=0, n_packets=26, n_bytes=1748, priority=4,in_port="phy-br-provider",dl_vlan=3 actions=strip_vlan,NORMAL
 cookie=0x887c3572d5e57691, duration=5426.715s, table=0, n_packets=22, n_bytes=1440, priority=2,in_port="phy-br-provider" actions=drop
 cookie=0x887c3572d5e57691, duration=5426.718s, table=0, n_packets=220, n_bytes=41868, priority=0 actions=NORMAL
```

ブリッジ br-mgmt のフローのエントリを確認する。

```sh
ovs-ofctl dump-flows br-mgmt
```

```text
 cookie=0xd7598ea6a4179ea2, duration=5439.649s, table=0, n_packets=268, n_bytes=44576, priority=2,in_port="phy-br-mgmt" actions=drop
 cookie=0xd7598ea6a4179ea2, duration=5439.652s, table=0, n_packets=246, n_bytes=44820, priority=0 actions=NORMAL
```

ブリッジ br-int のフローのエントリを確認する。

```sh
ovs-ofctl dump-flows br-int
```

```text
 cookie=0x68a8ad4f4c63c160, duration=5453.081s, table=0, n_packets=0, n_bytes=0, priority=65535,dl_vlan=4095 actions=drop
 cookie=0x68a8ad4f4c63c160, duration=5334.020s, table=0, n_packets=0, n_bytes=0, priority=3,in_port="int-br-provider",dl_vlan=100 actions=mod_vlan_vid:1,resubmit(,58)
 cookie=0x68a8ad4f4c63c160, duration=5333.997s, table=0, n_packets=212, n_bytes=40828, priority=3,in_port="int-br-provider",vlan_tci=0x0000/0x1fff actions=mod_vlan_vid:3,resubmit(,58)
 cookie=0x68a8ad4f4c63c160, duration=5453.071s, table=0, n_packets=8, n_bytes=1040, priority=2,in_port="int-br-provider" actions=drop
 cookie=0x68a8ad4f4c63c160, duration=5453.061s, table=0, n_packets=246, n_bytes=44820, priority=2,in_port="int-br-mgmt" actions=drop
 cookie=0x68a8ad4f4c63c160, duration=5453.085s, table=0, n_packets=84, n_bytes=6024, priority=0 actions=resubmit(,58)
 cookie=0x68a8ad4f4c63c160, duration=5453.086s, table=23, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x68a8ad4f4c63c160, duration=5453.082s, table=24, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x68a8ad4f4c63c160, duration=5453.079s, table=30, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,58)
 cookie=0x68a8ad4f4c63c160, duration=5453.078s, table=31, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,58)
 cookie=0x68a8ad4f4c63c160, duration=5453.084s, table=58, n_packets=296, n_bytes=46852, priority=0 actions=resubmit(,60)
 cookie=0x68a8ad4f4c63c160, duration=5332.488s, table=60, n_packets=8, n_bytes=560, priority=100,in_port="tap52b1b85b-8e" actions=load:0x4->NXM_NX_REG5[],load:0x1->NXM_NX_REG6[],resubmit(,73)
 cookie=0x68a8ad4f4c63c160, duration=5332.488s, table=60, n_packets=8, n_bytes=560, priority=100,in_port="tap2dac2cf0-61" actions=load:0x9->NXM_NX_REG5[],load:0x2->NXM_NX_REG6[],resubmit(,73)
 cookie=0x68a8ad4f4c63c160, duration=5332.488s, table=60, n_packets=8, n_bytes=560, priority=100,in_port="tap183c56bb-b2" actions=load:0x3->NXM_NX_REG5[],load:0x3->NXM_NX_REG6[],resubmit(,73)
 cookie=0x68a8ad4f4c63c160, duration=423.456s, table=60, n_packets=14, n_bytes=880, priority=100,in_port="qr-13e6e8ff-d7" actions=load:0xa->NXM_NX_REG5[],load:0x2->NXM_NX_REG6[],resubmit(,73)
 cookie=0x68a8ad4f4c63c160, duration=399.415s, table=60, n_packets=17, n_bytes=1098, priority=100,in_port="qg-ca92be94-79" actions=load:0xb->NXM_NX_REG5[],load:0x3->NXM_NX_REG6[],resubmit(,73)
 cookie=0x68a8ad4f4c63c160, duration=5453.083s, table=60, n_packets=241, n_bytes=43194, priority=3 actions=NORMAL
 cookie=0x68a8ad4f4c63c160, duration=5453.080s, table=62, n_packets=0, n_bytes=0, priority=3 actions=NORMAL
 cookie=0x68a8ad4f4c63c160, duration=5335.775s, table=71, n_packets=0, n_bytes=0, priority=110,ct_state=+trk actions=ct_clear,resubmit(,71)
 cookie=0x68a8ad4f4c63c160, duration=5335.871s, table=71, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x68a8ad4f4c63c160, duration=5335.830s, table=72, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x68a8ad4f4c63c160, duration=5332.488s, table=73, n_packets=8, n_bytes=560, priority=80,reg5=0x4 actions=resubmit(,94)
 cookie=0x68a8ad4f4c63c160, duration=5332.488s, table=73, n_packets=8, n_bytes=560, priority=80,reg5=0x9 actions=resubmit(,94)
 cookie=0x68a8ad4f4c63c160, duration=5332.488s, table=73, n_packets=8, n_bytes=560, priority=80,reg5=0x3 actions=resubmit(,94)
 cookie=0x68a8ad4f4c63c160, duration=423.456s, table=73, n_packets=14, n_bytes=880, priority=80,reg5=0xa actions=resubmit(,94)
 cookie=0x68a8ad4f4c63c160, duration=399.415s, table=73, n_packets=17, n_bytes=1098, priority=80,reg5=0xb actions=resubmit(,94)
 cookie=0x68a8ad4f4c63c160, duration=5335.810s, table=73, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x68a8ad4f4c63c160, duration=5335.797s, table=81, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x68a8ad4f4c63c160, duration=5335.786s, table=82, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x68a8ad4f4c63c160, duration=5335.755s, table=91, n_packets=0, n_bytes=0, priority=1 actions=resubmit(,94)
 cookie=0x68a8ad4f4c63c160, duration=5335.745s, table=92, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x68a8ad4f4c63c160, duration=5335.734s, table=93, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x68a8ad4f4c63c160, duration=5335.765s, table=94, n_packets=55, n_bytes=3658, priority=1 actions=NORMAL
```

ブリッジ br-tun のフローのエントリを確認する。

```sh
ovs-ofctl dump-flows br-tun
```

```text
 cookie=0x20ed499125a369ae, duration=5484.669s, table=0, n_packets=272, n_bytes=45448, priority=1,in_port="patch-int" actions=resubmit(,2)
 cookie=0x20ed499125a369ae, duration=5366.117s, table=0, n_packets=0, n_bytes=0, priority=1,in_port="vxlan-ac10001f" actions=resubmit(,4)
 cookie=0x20ed499125a369ae, duration=5484.667s, table=0, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x20ed499125a369ae, duration=5484.666s, table=2, n_packets=0, n_bytes=0, priority=0,dl_dst=00:00:00:00:00:00/01:00:00:00:00:00 actions=resubmit(,20)
 cookie=0x20ed499125a369ae, duration=5484.665s, table=2, n_packets=272, n_bytes=45448, priority=0,dl_dst=01:00:00:00:00:00/01:00:00:00:00:00 actions=resubmit(,22)
 cookie=0x20ed499125a369ae, duration=5484.664s, table=3, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x20ed499125a369ae, duration=5365.628s, table=4, n_packets=0, n_bytes=0, priority=1,tun_id=0xef actions=mod_vlan_vid:2,resubmit(,10)
 cookie=0x20ed499125a369ae, duration=5484.663s, table=4, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x20ed499125a369ae, duration=5484.663s, table=6, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x20ed499125a369ae, duration=5484.662s, table=10, n_packets=0, n_bytes=0, priority=1 actions=learn(table=20,hard_timeout=300,priority=1,cookie=0x20ed499125a369ae,NXM_OF_VLAN_TCI[0..11],NXM_OF_ETH_DST[]=NXM_OF_ETH_SRC[],load:0->NXM_OF_VLAN_TCI[],load:NXM_NX_TUN_ID[]->NXM_NX_TUN_ID[],output:OXM_OF_IN_PORT[]),output:"patch-int"
 cookie=0x20ed499125a369ae, duration=5484.661s, table=20, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,22)
 cookie=0x20ed499125a369ae, duration=5365.629s, table=22, n_packets=22, n_bytes=1440, priority=1,dl_vlan=2 actions=strip_vlan,load:0xef->NXM_NX_TUN_ID[],output:"vxlan-ac10001f"
 cookie=0x20ed499125a369ae, duration=5484.660s, table=22, n_packets=250, n_bytes=44008, priority=0 actions=drop
```

```sh
ovs-appctl ofproto/list-tunnels
```

```text
port 2: vxlan-ac10001f (vxlan: 172.16.0.11->172.16.0.31, key=flow, legacy_l2, dp port=2, ttl=64)
```

### イーサネット

イーサネットの情報を確認する。

```sh
ip addr show
```

```text
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:15:5d:bf:ba:4f brd ff:ff:ff:ff:ff:ff
    inet 172.16.0.11/24 brd 172.16.0.255 scope global noprefixroute eth0
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:15:5d:bf:ba:54 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.11/24 brd 10.0.0.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
4: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq master ovs-system state UP group default qlen 1000
    link/ether 00:15:5d:bf:ba:55 brd ff:ff:ff:ff:ff:ff
5: eth3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq master ovs-system state UP group default qlen 1000
    link/ether 00:15:5d:bf:ba:56 brd ff:ff:ff:ff:ff:ff
6: ovs-system: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether d6:70:ef:61:e9:e5 brd ff:ff:ff:ff:ff:ff
7: br-tun: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ether 56:a8:05:61:4b:43 brd ff:ff:ff:ff:ff:ff
8: vxlan_sys_4789: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 65000 qdisc noqueue master ovs-system state UNKNOWN group default qlen 1000
    link/ether 0e:6d:d0:0f:f8:39 brd ff:ff:ff:ff:ff:ff
9: br-provider: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ether 00:15:5d:bf:ba:55 brd ff:ff:ff:ff:ff:ff
10: br-mgmt: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ether 00:15:5d:bf:ba:56 brd ff:ff:ff:ff:ff:ff
14: br-int: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ether 4a:73:de:e4:cd:4f brd ff:ff:ff:ff:ff:ff
```

ネットワーク名前空間内のイーサネットの情報を確認する。

```sh
ip netns exec qrouter-e6efd80e-9468-4787-a6f8-1648486cc1d6 ip addr show
```

```text
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
    inet6 fe80::f816:3eff:fea6:1148/64 scope link
       valid_lft forever preferred_lft forever
```

ルーティングを確認する。

```sh
ip netns exec qrouter-e6efd80e-9468-4787-a6f8-1648486cc1d6 ip route show
```

```text
default via 172.16.0.254 dev qg-ca92be94-79 proto static
172.16.0.0/24 dev qg-ca92be94-79 proto kernel scope link src 172.16.0.188
192.168.101.0/24 dev qr-13e6e8ff-d7 proto kernel scope link src 192.168.101.254
```

待ち受けているポートを確認する。

```sh
ip netns exec qrouter-e6efd80e-9468-4787-a6f8-1648486cc1d6 ss -ano -4
```

メタデータのポートが待ち受けている。

```text
Netid              State               Recv-Q              Send-Q                            Local Address:Port                             Peer Address:Port              Process
tcp                LISTEN              0                   1024                                    0.0.0.0:9697                                  0.0.0.0:*
```
