# 動作確認 (Open Virtual Network)

エージェントを表示する。

```sh
openstack network agent list
```

```
+--------------------------------------+------------------------------+-----------------------+-------------------+-------+-------+----------------+
| ID                                   | Agent Type                   | Host                  | Availability Zone | Alive | State | Binary         |
+--------------------------------------+------------------------------+-----------------------+-------------------+-------+-------+----------------+
| 3732c7d2-a358-41ae-82ca-9c350624e13d | OVN Controller Gateway agent | controller.home.local |                   | :-)   | UP    | ovn-controller |
+--------------------------------------+------------------------------+-----------------------+-------------------+-------+-------+----------------+
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

```
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
```

#### Northband データベース

ACL を確認する。

```sh
ovn-nbctl list ACL
```

```
_uuid               : 3b1768c2-3f33-4e57-a614-427811758daf
action              : drop
direction           : from-lport
external_ids        : {}
label               : 0
log                 : false
match               : "inport == @neutron_pg_drop && ip"
meter               : []
name                : []
options             : {}
priority            : 1001
severity            : []
tier                : 0

_uuid               : 3c8b2fef-ddd1-4040-8e26-ea373222b198
action              : drop
direction           : to-lport
external_ids        : {}
label               : 0
log                 : false
match               : "outport == @neutron_pg_drop && ip"
meter               : []
name                : []
options             : {}
priority            : 1001
severity            : []
tier                : 0
```

#### Southbound データベース

Controller Node が geneve で登録されていることを確認する。

```sh
ovn-sbctl show
```

```
Chassis "3732c7d2-a358-41ae-82ca-9c350624e13d"
    hostname: controller.home.local
    Encap geneve
        ip: "172.16.0.11"
        options: {csum="true"}
```

Controller Node が登録されている。

```sh
ovn-sbctl list Chassis
```

```
_uuid               : 1649a8fe-937d-44f8-861c-372fe528e9ec
encaps              : [53b36a07-1b5d-4147-a97a-853dad42b687]
external_ids        : {}
hostname            : controller.home.local
name                : "3732c7d2-a358-41ae-82ca-9c350624e13d"
nb_cfg              : 0
other_config        : {ct-commit-nat-v2="true", ct-no-masked-label="true", datapath-type=system, fdb-timestamp="true", iface-types="bareudp,erspan,geneve,gre,gtpu,internal,ip6erspan,ip6gre,lisp,patch,srv6,stt,system,tap,vxlan", is-interconn="false", ls-dpg-column="true", mac-binding-timestamp="true", ovn-bridge-mappings="provider:br-provider,mgmt:br-mgmt", ovn-chassis-mac-mappings="", ovn-cms-options=enable-chassis-as-gw, ovn-ct-lb-related="true", ovn-enable-lflow-cache="true", ovn-limit-lflow-cache="", ovn-memlimit-lflow-cache-kb="", ovn-monitor-all="false", ovn-trim-limit-lflow-cache="", ovn-trim-timeout-ms="", ovn-trim-wmark-perc-lflow-cache="", port-up-notif="true"}
transport_zones     : []
vtep_logical_switches: []
```

トンネルプロトコル geneve が有効になっている。

```sh
ovn-sbctl list Encap
```

```
_uuid               : 53b36a07-1b5d-4147-a97a-853dad42b687
chassis_name        : "3732c7d2-a358-41ae-82ca-9c350624e13d"
ip                  : "172.16.0.11"
options             : {csum="true"}
type                : geneve
```

アドレスセットを確認する。

```sh
ovn-sbctl list Address_Set
```

```
_uuid               : d0cae968-1582-4c82-8324-88900c57b6ec
addresses           : ["2a:91:79:ab:04:2d"]
name                : svc_monitor_mac

_uuid               : 840d070d-12a7-4434-8047-95526f7b0186
addresses           : []
name                : neutron_pg_drop_ip6

_uuid               : a0eda553-388f-4810-b31e-63d7edf879f8
addresses           : []
name                : neutron_pg_drop_ip4
```

#### Open vSwitch

ブリッジを確認する。

```sh
ovs-vsctl show
```

```
9869c49f-a522-47dc-a744-bf85afff8b76
    Bridge br-int
        fail_mode: secure
        datapath_type: system
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

```
system@ovs-system:
  lookups: hit:122 missed:78 lost:0
  flows: 2
  masks: hit:258 total:1 hit/pkt:1.29
  cache: hit:57 hit-rate:28.50%
  caches:
    masks-cache: size:256
  port 0: ovs-system (internal)
  port 1: br-int (internal)
  port 2: eth2
  port 3: eth3
```

ブリッジ br-int のフローのエントリを確認する。

```sh
ovs-ofctl dump-flows br-int
```

```
 cookie=0x0, duration=892.166s, table=0, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x0, duration=892.166s, table=37, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,39)
 cookie=0x0, duration=892.166s, table=38, n_packets=0, n_bytes=0, priority=10,reg10=0x1/0x1 actions=resubmit(,40)
 cookie=0x0, duration=892.166s, table=38, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,39)
 cookie=0x0, duration=892.166s, table=39, n_packets=0, n_bytes=0, priority=150,reg10=0x10/0x10 actions=resubmit(,40)
 cookie=0x0, duration=892.166s, table=39, n_packets=0, n_bytes=0, priority=150,reg10=0x2/0x3 actions=resubmit(,40)
 cookie=0x0, duration=892.166s, table=39, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,40)
 cookie=0x0, duration=892.166s, table=40, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x0, duration=892.166s, table=41, n_packets=0, n_bytes=0, priority=0 actions=load:0->NXM_NX_REG0[],load:0->NXM_NX_REG1[],load:0->NXM_NX_REG2[],load:0->NXM_NX_REG3[],load:0->NXM_NX_REG4[],load:0->NXM_NX_REG5[],load:0->NXM_NX_REG6[],load:0->NXM_NX_REG7[],load:0->NXM_NX_REG8[],load:0->NXM_NX_REG9[],resubmit(,42)
 cookie=0x0, duration=892.166s, table=64, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,65)
 cookie=0x0, duration=892.166s, table=65, n_packets=0, n_bytes=0, priority=0 actions=drop
```

ブリッジ br-provider のフローのエントリを確認する。

```sh
ovs-ofctl dump-flows br-provider
```

```
 cookie=0x0, duration=893.723s, table=0, n_packets=126, n_bytes=22172, priority=0 actions=NORMAL
```

ブリッジ br-mgmt のフローのエントリを確認する。

```sh
ovs-ofctl dump-flows br-mgmt
```

```
 cookie=0x0, duration=901.568s, table=0, n_packets=130, n_bytes=22932, priority=0 actions=NORMAL
```

トンネルは作成されない。

```sh
ovs-appctl ofproto/list-tunnels
```
