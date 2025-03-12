# 動作確認 (Open Virtual Network)

エージェントを表示する。

```sh
openstack network agent list
```

```text
+--------------------------------------+------------------------------+-----------------------+-------------------+-------+-------+----------------------------+
| ID                                   | Agent Type                   | Host                  | Availability Zone | Alive | State | Binary                     |
+--------------------------------------+------------------------------+-----------------------+-------------------+-------+-------+----------------------------+
| 3732c7d2-a358-41ae-82ca-9c350624e13d | OVN Controller Gateway agent | controller.home.local |                   | :-)   | UP    | ovn-controller             |
| 32a2f4d1-1918-4bcf-beee-d6094e1a1d4c | OVN Controller agent         | compute.home.local    |                   | :-)   | UP    | ovn-controller             |
| eeedb10b-768a-5e9f-8fdf-739b042ab389 | OVN Metadata agent           | compute.home.local    |                   | :-)   | UP    | neutron-ovn-metadata-agent |
+--------------------------------------+------------------------------+-----------------------+-------------------+-------+-------+----------------------------+
```

## 環境の確認

ネットワークの構成を確認する。

### Controller Node

#### ネットワーク名前空間

ネットワーク名前空間は作成されない。

#### デバイス

デバイスを確認する。

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
    link/ether 46:47:6e:56:e9:12 brd ff:ff:ff:ff:ff:ff promiscuity 1  allmulti 0 minmtu 68 maxmtu 65535
    openvswitch addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 65536 tso_max_segs 65535 gro_max_size 65536
7: br-int: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether 16:5d:99:54:ca:d3 brd ff:ff:ff:ff:ff:ff promiscuity 1  allmulti 0 minmtu 68 maxmtu 65535
    openvswitch addrgenmode none numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 65536 tso_max_segs 65535 gro_max_size 65536
8: genev_sys_6081: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 65000 qdisc noqueue master ovs-system state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether 0e:e3:c7:9a:24:99 brd ff:ff:ff:ff:ff:ff promiscuity 1  allmulti 0 minmtu 68 maxmtu 65465
    geneve external id 0 ttl auto dstport 6081 udp6zerocsumrx
    openvswitch_slave addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 65536 tso_max_segs 65535 gro_max_size 65536
```

#### Northband データベース

変化はない。

#### Southbound データベース

Compute Node が geneve で登録されていることを確認する。

```sh
ovn-sbctl show
```

```text
Chassis "3732c7d2-a358-41ae-82ca-9c350624e13d"
    hostname: controller.home.local
    Encap geneve
        ip: "172.16.0.11"
        options: {csum="true"}
Chassis "32a2f4d1-1918-4bcf-beee-d6094e1a1d4c"
    hostname: compute.home.local
    Encap geneve
        ip: "172.16.0.31"
        options: {csum="true"}
```

Compute Node が登録されている。

```sh
ovn-sbctl list Chassis
```

```text
_uuid               : 1649a8fe-937d-44f8-861c-372fe528e9ec
encaps              : [53b36a07-1b5d-4147-a97a-853dad42b687]
external_ids        : {}
hostname            : controller.home.local
name                : "3732c7d2-a358-41ae-82ca-9c350624e13d"
nb_cfg              : 0
other_config        : {ct-commit-nat-v2="true", ct-no-masked-label="true", datapath-type=system, fdb-timestamp="true", iface-types="bareudp,erspan,geneve,gre,gtpu,internal,ip6erspan,ip6gre,lisp,patch,srv6,stt,system,tap,vxlan", is-interconn="false", ls-dpg-column="true", mac-binding-timestamp="true", ovn-bridge-mappings="provider:br-provider,mgmt:br-mgmt", ovn-chassis-mac-mappings="", ovn-cms-options=enable-chassis-as-gw, ovn-ct-lb-related="true", ovn-enable-lflow-cache="true", ovn-limit-lflow-cache="", ovn-memlimit-lflow-cache-kb="", ovn-monitor-all="false", ovn-trim-limit-lflow-cache="", ovn-trim-timeout-ms="", ovn-trim-wmark-perc-lflow-cache="", port-up-notif="true"}
transport_zones     : []
vtep_logical_switches: []

_uuid               : 1cf792ab-7614-464f-8fae-ad635fe6ebd6
encaps              : [cadca5ae-c845-4390-8b5f-c14e7c161617]
external_ids        : {}
hostname            : compute.home.local
name                : "32a2f4d1-1918-4bcf-beee-d6094e1a1d4c"
nb_cfg              : 0
other_config        : {ct-commit-nat-v2="true", ct-no-masked-label="true", datapath-type=system, fdb-timestamp="true", iface-types="bareudp,erspan,geneve,gre,gtpu,internal,ip6erspan,ip6gre,lisp,patch,srv6,stt,system,tap,vxlan", is-interconn="false", ls-dpg-column="true", mac-binding-timestamp="true", ovn-bridge-mappings="provider:br-provider,mgmt:br-mgmt", ovn-chassis-mac-mappings="", ovn-cms-options="", ovn-ct-lb-related="true", ovn-enable-lflow-cache="true", ovn-limit-lflow-cache="", ovn-memlimit-lflow-cache-kb="", ovn-monitor-all="false", ovn-trim-limit-lflow-cache="", ovn-trim-timeout-ms="", ovn-trim-wmark-perc-lflow-cache="", port-up-notif="true"}
transport_zones     : []
vtep_logical_switches: []
```

トンネルプロトコル geneve が有効になっている。

```sh
ovn-sbctl list Encap
```

```text
_uuid               : cadca5ae-c845-4390-8b5f-c14e7c161617
chassis_name        : "32a2f4d1-1918-4bcf-beee-d6094e1a1d4c"
ip                  : "172.16.0.31"
options             : {csum="true"}
type                : geneve

_uuid               : 53b36a07-1b5d-4147-a97a-853dad42b687
chassis_name        : "3732c7d2-a358-41ae-82ca-9c350624e13d"
ip                  : "172.16.0.11"
options             : {csum="true"}
type                : geneve
```

#### Open vSwitch

ブリッジを確認する。

```sh
ovs-vsctl show
```

```text
9869c49f-a522-47dc-a744-bf85afff8b76
    Bridge br-int
        fail_mode: secure
        datapath_type: system
        Port ovn-32a2f4-0
            Interface ovn-32a2f4-0
                type: geneve
                options: {csum="true", key=flow, remote_ip="172.16.0.31"}
        Port br-int
            Interface br-int
                type: internal
    Bridge br-provider
        Port eth2
            Interface eth2
                type: system
    Bridge br-mgmt
        Port eth3
            Interface eth3
                type: system
    ovs_version: "3.3.1"
```

データパスを確認する。

```sh
ovs-dpctl show
```

```text
system@ovs-system:
  lookups: hit:333 missed:198 lost:0
  flows: 0
  masks: hit:683 total:0 hit/pkt:1.29
  cache: hit:156 hit-rate:29.38%
  caches:
    masks-cache: size:256
  port 0: ovs-system (internal)
  port 1: br-int (internal)
  port 2: eth2
  port 3: eth3
  port 4: genev_sys_6081 (geneve: packet_type=ptap)
```

ブリッジ br-int のフローのエントリを確認する。

```sh
ovs-ofctl dump-flows br-int
```

```text
 cookie=0x0, duration=540.637s, table=0, n_packets=0, n_bytes=0, priority=120,icmp6,in_port="ovn-32a2f4-0",icmp_type=2,icmp_code=0 actions=move:NXM_NX_TUN_ID[0..23]->OXM_OF_METADATA[0..23],move:NXM_NX_TUN_METADATA0[16..30]->NXM_NX_REG14[0..14],move:NXM_NX_TUN_METADATA0[0..15]->NXM_NX_REG15[0..15],resubmit(,40),load:0x1->NXM_NX_REG10[16],resubmit(,8)
 cookie=0x0, duration=540.637s, table=0, n_packets=0, n_bytes=0, priority=120,icmp,in_port="ovn-32a2f4-0",icmp_type=3,icmp_code=4 actions=move:NXM_NX_TUN_ID[0..23]->OXM_OF_METADATA[0..23],move:NXM_NX_TUN_METADATA0[16..30]->NXM_NX_REG14[0..14],move:NXM_NX_TUN_METADATA0[0..15]->NXM_NX_REG15[0..15],resubmit(,40),load:0x1->NXM_NX_REG10[16],resubmit(,8)
 cookie=0x0, duration=540.637s, table=0, n_packets=0, n_bytes=0, priority=100,in_port="ovn-32a2f4-0" actions=move:NXM_NX_TUN_ID[0..23]->OXM_OF_METADATA[0..23],move:NXM_NX_TUN_METADATA0[16..30]->NXM_NX_REG14[0..14],move:NXM_NX_TUN_METADATA0[0..15]->NXM_NX_REG15[0..15],resubmit(,40)
 cookie=0x0, duration=2021.420s, table=0, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x0, duration=2021.420s, table=37, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,39)
 cookie=0x0, duration=2021.420s, table=38, n_packets=0, n_bytes=0, priority=10,reg10=0x1/0x1 actions=resubmit(,40)
 cookie=0x0, duration=2021.420s, table=38, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,39)
 cookie=0x0, duration=2021.420s, table=39, n_packets=0, n_bytes=0, priority=150,reg10=0x10/0x10 actions=resubmit(,40)
 cookie=0x0, duration=2021.420s, table=39, n_packets=0, n_bytes=0, priority=150,reg10=0x2/0x3 actions=resubmit(,40)
 cookie=0x0, duration=2021.420s, table=39, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,40)
 cookie=0x0, duration=2021.420s, table=40, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x0, duration=2021.420s, table=41, n_packets=0, n_bytes=0, priority=0 actions=load:0->NXM_NX_REG0[],load:0->NXM_NX_REG1[],load:0->NXM_NX_REG2[],load:0->NXM_NX_REG3[],load:0->NXM_NX_REG4[],load:0->NXM_NX_REG5[],load:0->NXM_NX_REG6[],load:0->NXM_NX_REG7[],load:0->NXM_NX_REG8[],load:0->NXM_NX_REG9[],resubmit(,42)
 cookie=0x0, duration=2021.420s, table=64, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,65)
 cookie=0x0, duration=2021.420s, table=65, n_packets=0, n_bytes=0, priority=0 actions=drop
```

ブリッジ br-provider のフローのエントリを確認する。

```sh
ovs-ofctl dump-flows br-provider
```

```text
 cookie=0x0, duration=2026.655s, table=0, n_packets=261, n_bytes=45072, priority=0 actions=NORMAL
```

ブリッジ br-mgmt のフローのエントリを確認する。

```sh
ovs-ofctl dump-flows br-mgmt
```

```text
 cookie=0x0, duration=2030.498s, table=0, n_packets=270, n_bytes=46038, priority=0 actions=NORMAL
```

トンネルを確認する。

```sh
ovs-appctl ofproto/list-tunnels
```

```text
port 4: ovn-32a2f4-0 (geneve: ::->172.16.0.31, key=flow, legacy_l2, dp port=4, ttl=64, csum=true)
```

### Compute Node

#### ネットワーク名前空間

ネットワーク名前空間は作成されない。

#### デバイス

デバイスを確認する。

```sh
ip -d link show
```

```text
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00 promiscuity 0  allmulti 0 minmtu 0 maxmtu 0 addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 524280 tso_max_segs 65535 gro_max_size 65536
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 00:15:5d:bf:ba:50 brd ff:ff:ff:ff:ff:ff promiscuity 0  allmulti 0 minmtu 68 maxmtu 65521 addrgenmode none numtxqueues 64 numrxqueues 64 gso_max_size 62780 gso_max_segs 65535 tso_max_size 62780 tso_max_segs 65535 gro_max_size 65536 parentbus vmbus parentdev f4fd36cf-d20d-4c23-83bb-e1b5cc9fbfbc
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 00:15:5d:bf:ba:57 brd ff:ff:ff:ff:ff:ff promiscuity 0  allmulti 0 minmtu 68 maxmtu 65521 addrgenmode none numtxqueues 64 numrxqueues 64 gso_max_size 62780 gso_max_segs 65535 tso_max_size 62780 tso_max_segs 65535 gro_max_size 65536 parentbus vmbus parentdev 1e1fc9b9-f159-4473-9410-cf7db6750e26
4: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq master ovs-system state UP mode DEFAULT group default qlen 1000
    link/ether 00:15:5d:bf:ba:58 brd ff:ff:ff:ff:ff:ff promiscuity 1  allmulti 0 minmtu 68 maxmtu 65521
    openvswitch_slave addrgenmode none numtxqueues 64 numrxqueues 64 gso_max_size 62780 gso_max_segs 65535 tso_max_size 62780 tso_max_segs 65535 gro_max_size 65536 parentbus vmbus parentdev fb193242-70ac-437d-9008-4382b02d2a70
5: eth3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq master ovs-system state UP mode DEFAULT group default qlen 1000
    link/ether 00:15:5d:bf:ba:59 brd ff:ff:ff:ff:ff:ff promiscuity 1  allmulti 0 minmtu 68 maxmtu 65521
    openvswitch_slave addrgenmode none numtxqueues 64 numrxqueues 64 gso_max_size 62780 gso_max_segs 65535 tso_max_size 62780 tso_max_segs 65535 gro_max_size 65536 parentbus vmbus parentdev 6de0f76b-b7bc-45ba-9087-d8bee9131e1c
6: ovs-system: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether c6:04:f7:00:92:53 brd ff:ff:ff:ff:ff:ff promiscuity 1  allmulti 0 minmtu 68 maxmtu 65535
    openvswitch addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 65536 tso_max_segs 65535 gro_max_size 65536
7: br-int: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 9a:20:83:87:33:1b brd ff:ff:ff:ff:ff:ff promiscuity 1  allmulti 0 minmtu 68 maxmtu 65535
    openvswitch addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 65536 tso_max_segs 65535 gro_max_size 65536
8: genev_sys_6081: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 65000 qdisc noqueue master ovs-system state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether 86:4d:50:1d:ff:45 brd ff:ff:ff:ff:ff:ff promiscuity 1  allmulti 0 minmtu 68 maxmtu 65465
    geneve external id 0 ttl auto dstport 6081 udp6zerocsumrx
    openvswitch_slave addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 65536 tso_max_segs 65535 gro_max_size 65536
```

#### Open vSwitch

ブリッジの構成を確認する。

```sh
ovs-vsctl show
```

```text
46423187-0689-4323-afe2-2ad6d689596b
    Bridge br-mgmt
        Port eth3
            Interface eth3
                type: system
    Bridge br-int
        fail_mode: secure
        datapath_type: system
        Port ovn-3732c7-0
            Interface ovn-3732c7-0
                type: geneve
                options: {csum="true", key=flow, remote_ip="172.16.0.11"}
        Port br-int
            Interface br-int
                type: internal
    Bridge br-provider
        Port eth2
            Interface eth2
                type: system
    ovs_version: "3.3.1"
```

データパスを確認する。

```sh
ovs-dpctl show
```

```text
system@ovs-system:
  lookups: hit:61 missed:20 lost:0
  flows: 0
  masks: hit:72 total:0 hit/pkt:0.89
  cache: hit:41 hit-rate:50.62%
  caches:
    masks-cache: size:256
  port 0: ovs-system (internal)
  port 1: br-int (internal)
  port 2: genev_sys_6081 (geneve: packet_type=ptap)
  port 3: eth2
  port 4: eth3
```

ブリッジ br-int のフローのエントリを確認する。

```sh
ovs-ofctl dump-flows br-int
```

```text
 cookie=0x0, duration=716.660s, table=0, n_packets=0, n_bytes=0, priority=120,icmp6,in_port="ovn-3732c7-0",icmp_type=2,icmp_code=0 actions=move:NXM_NX_TUN_ID[0..23]->OXM_OF_METADATA[0..23],move:NXM_NX_TUN_METADATA0[16..30]->NXM_NX_REG14[0..14],move:NXM_NX_TUN_METADATA0[0..15]->NXM_NX_REG15[0..15],resubmit(,40),load:0x1->NXM_NX_REG10[16],resubmit(,8)
 cookie=0x0, duration=716.660s, table=0, n_packets=0, n_bytes=0, priority=120,icmp,in_port="ovn-3732c7-0",icmp_type=3,icmp_code=4 actions=move:NXM_NX_TUN_ID[0..23]->OXM_OF_METADATA[0..23],move:NXM_NX_TUN_METADATA0[16..30]->NXM_NX_REG14[0..14],move:NXM_NX_TUN_METADATA0[0..15]->NXM_NX_REG15[0..15],resubmit(,40),load:0x1->NXM_NX_REG10[16],resubmit(,8)
 cookie=0x0, duration=716.660s, table=0, n_packets=0, n_bytes=0, priority=100,in_port="ovn-3732c7-0" actions=move:NXM_NX_TUN_ID[0..23]->OXM_OF_METADATA[0..23],move:NXM_NX_TUN_METADATA0[16..30]->NXM_NX_REG14[0..14],move:NXM_NX_TUN_METADATA0[0..15]->NXM_NX_REG15[0..15],resubmit(,40)
 cookie=0x0, duration=716.660s, table=0, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x0, duration=716.660s, table=37, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,39)
 cookie=0x0, duration=716.660s, table=38, n_packets=0, n_bytes=0, priority=10,reg10=0x1/0x1 actions=resubmit(,40)
 cookie=0x0, duration=716.660s, table=38, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,39)
 cookie=0x0, duration=716.660s, table=39, n_packets=0, n_bytes=0, priority=150,reg10=0x10/0x10 actions=resubmit(,40)
 cookie=0x0, duration=716.660s, table=39, n_packets=0, n_bytes=0, priority=150,reg10=0x2/0x3 actions=resubmit(,40)
 cookie=0x0, duration=716.660s, table=39, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,40)
 cookie=0x0, duration=716.660s, table=40, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x0, duration=716.660s, table=41, n_packets=0, n_bytes=0, priority=0 actions=load:0->NXM_NX_REG0[],load:0->NXM_NX_REG1[],load:0->NXM_NX_REG2[],load:0->NXM_NX_REG3[],load:0->NXM_NX_REG4[],load:0->NXM_NX_REG5[],load:0->NXM_NX_REG6[],load:0->NXM_NX_REG7[],load:0->NXM_NX_REG8[],load:0->NXM_NX_REG9[],resubmit(,42)
 cookie=0x0, duration=716.660s, table=64, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,65)
 cookie=0x0, duration=716.660s, table=65, n_packets=0, n_bytes=0, priority=0 actions=drop
```

ブリッジ br-provider のフローのエントリを確認する。

```sh
ovs-ofctl dump-flows br-provider
```

```text
 cookie=0x0, duration=726.859s, table=0, n_packets=36, n_bytes=6062, priority=0 actions=NORMAL
```

ブリッジ br-mgmt のフローのエントリを確認する。

```sh
ovs-ofctl dump-flows br-mgmt
```

```text
 cookie=0x0, duration=729.875s, table=0, n_packets=49, n_bytes=7944, priority=0 actions=NORMAL
```

トンネルを確認する。

```sh
ovs-appctl ofproto/list-tunnels
```

```text
port 2: ovn-3732c7-0 (geneve: ::->172.16.0.11, key=flow, legacy_l2, dp port=2, ttl=64, csum=true)
```
