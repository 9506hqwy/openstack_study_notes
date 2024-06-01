# 動作確認 (Open Virtual Network)

エージェントを表示する。

```sh
openstack network agent list
```

```{note}
OVN Metadata agent はインスタンスが起動するまで存在しない？再確認。
```

```
+--------------------------------------+----------------------+--------------------+-------------------+-------+-------+----------------------------+
| ID                                   | Agent Type           | Host               | Availability Zone | Alive | State | Binary                     |
+--------------------------------------+----------------------+--------------------+-------------------+-------+-------+----------------------------+
| 69090316-351c-449a-a1e5-87382599cd5c | OVN Controller agent | compute.home.local |                   | :-)   | UP    | ovn-controller             |
| 7878ae05-b57f-537b-b550-38574fac0bbb | OVN Metadata agent   | compute.home.local |                   | :-)   | UP    | neutron-ovn-metadata-agent |
+--------------------------------------+----------------------+--------------------+-------------------+-------+-------+----------------------------+
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
4: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 00:15:5d:bf:ba:55 brd ff:ff:ff:ff:ff:ff promiscuity 0  allmulti 0 minmtu 68 maxmtu 65521 addrgenmode none numtxqueues 64 numrxqueues 64 gso_max_size 62780 gso_max_segs 65535 tso_max_size 62780 tso_max_segs 65535 gro_max_size 65536 parentbus vmbus parentdev dffbd9a0-19dd-44c1-9b46-6dfba9829d73
5: eth3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 00:15:5d:bf:ba:56 brd ff:ff:ff:ff:ff:ff promiscuity 0  allmulti 0 minmtu 68 maxmtu 65521 addrgenmode none numtxqueues 64 numrxqueues 64 gso_max_size 62780 gso_max_segs 65535 tso_max_size 62780 tso_max_segs 65535 gro_max_size 65536 parentbus vmbus parentdev 31e9f926-7af1-481e-bf58-cbca38bc3cba
```

#### Northband データベース

論理スイッチはない。

```sh
ovn-nbctl list Logical_Switch
```

#### Southbound データベース

Compute Node と geneve で接続されていることを確認する。

```sh
ovn-sbctl show
```

```
Chassis "69090316-351c-449a-a1e5-87382599cd5c"
    hostname: compute.home.local
    Encap geneve
        ip: "172.16.0.31"
        options: {csum="true"}
```

Compute Node が登録されている。

```sh
ovn-sbctl list Chassis
```

```
_uuid               : b9687b99-c292-4283-94d3-563919d242f7
encaps              : [acb76f7b-b321-4cd2-a120-dc477b04d4f1]
external_ids        : {}
hostname            : compute.home.local
name                : "69090316-351c-449a-a1e5-87382599cd5c"
nb_cfg              : 0
other_config        : {ct-commit-nat-v2="true", ct-no-masked-label="true", datapath-type=system, fdb-timestamp="true", iface-types="bareudp,erspan,geneve,gre,gtpu,internal,ip6erspan,ip6gre,lisp,patch,srv6,stt,system,tap,vxlan", is-interconn="false", ls-dpg-column="true", mac-binding-timestamp="true", ovn-bridge-mappings="mgmt:br-mgmt", ovn-chassis-mac-mappings="", ovn-cms-options="", ovn-ct-lb-related="true", ovn-enable-lflow-cache="true", ovn-limit-lflow-cache="", ovn-memlimit-lflow-cache-kb="", ovn-monitor-all="false", ovn-trim-limit-lflow-cache="", ovn-trim-timeout-ms="", ovn-trim-wmark-perc-lflow-cache="", port-up-notif="true"}
transport_zones     : []
vtep_logical_switches: []
```

トンネルプロトコル geneve が有効になっている。

```sh
ovn-sbctl list Encap
```

```
_uuid               : acb76f7b-b321-4cd2-a120-dc477b04d4f1
chassis_name        : "69090316-351c-449a-a1e5-87382599cd5c"
ip                  : "172.16.0.31"
options             : {csum="true"}
type                : geneve
```

#### Open vSwitch

Open vSwitch にブリッジは作成されない。

```sh
ovs-vsctl show
```

```
f6d63fd3-3bf4-45eb-afc6-603984ff8667
    ovs_version: "3.3.1"
```

### Compute Node

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
    link/ether 00:15:5d:bf:ba:50 brd ff:ff:ff:ff:ff:ff promiscuity 0  allmulti 0 minmtu 68 maxmtu 65521 addrgenmode none numtxqueues 64 numrxqueues 64 gso_max_size 62780 gso_max_segs 65535 tso_max_size 62780 tso_max_segs 65535 gro_max_size 65536 parentbus vmbus parentdev f4fd36cf-d20d-4c23-83bb-e1b5cc9fbfbc
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 00:15:5d:bf:ba:57 brd ff:ff:ff:ff:ff:ff promiscuity 0  allmulti 0 minmtu 68 maxmtu 65521 addrgenmode none numtxqueues 64 numrxqueues 64 gso_max_size 62780 gso_max_segs 65535 tso_max_size 62780 tso_max_segs 65535 gro_max_size 65536 parentbus vmbus parentdev 1e1fc9b9-f159-4473-9410-cf7db6750e26
4: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 00:15:5d:bf:ba:58 brd ff:ff:ff:ff:ff:ff promiscuity 0  allmulti 0 minmtu 68 maxmtu 65521 addrgenmode none numtxqueues 64 numrxqueues 64 gso_max_size 62780 gso_max_segs 65535 tso_max_size 62780 tso_max_segs 65535 gro_max_size 65536 parentbus vmbus parentdev fb193242-70ac-437d-9008-4382b02d2a70
5: eth3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 00:15:5d:bf:ba:59 brd ff:ff:ff:ff:ff:ff promiscuity 0  allmulti 0 minmtu 68 maxmtu 65521 addrgenmode none numtxqueues 64 numrxqueues 64 gso_max_size 62780 gso_max_segs 65535 tso_max_size 62780 tso_max_segs 65535 gro_max_size 65536 parentbus vmbus parentdev 6de0f76b-b7bc-45ba-9087-d8bee9131e1c
6: ovs-system: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether a2:c9:30:29:50:37 brd ff:ff:ff:ff:ff:ff promiscuity 1  allmulti 0 minmtu 68 maxmtu 65535
    openvswitch addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 65536 tso_max_segs 65535 gro_max_size 65536
7: br-int: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether 9e:df:11:29:6c:ab brd ff:ff:ff:ff:ff:ff promiscuity 1  allmulti 0 minmtu 68 maxmtu 65535
    openvswitch addrgenmode none numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 65536 tso_max_segs 65535 gro_max_size 65536
```

#### Open vSwitch

ブリッジの構成を確認する。

```sh
ovs-vsctl show
```

```
210309a0-2cc9-4a7f-96af-960f78c1c32d
    Bridge br-int
        fail_mode: secure
        datapath_type: system
        Port br-int
            Interface br-int
                type: internal
    Bridge br-mgmt
        Port eth3
            Interface eth3
                type: system
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

```
system@ovs-system:
  lookups: hit:79 missed:53 lost:0
  flows: 0
  masks: hit:178 total:0 hit/pkt:1.35
  cache: hit:49 hit-rate:37.12%
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
 cookie=0x0, duration=1001.887s, table=0, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x0, duration=1001.888s, table=37, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,39)
 cookie=0x0, duration=1001.888s, table=38, n_packets=0, n_bytes=0, priority=10,reg10=0x1/0x1 actions=resubmit(,40)
 cookie=0x0, duration=1001.887s, table=38, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,39)
 cookie=0x0, duration=1001.888s, table=39, n_packets=0, n_bytes=0, priority=150,reg10=0x2/0x3 actions=resubmit(,40)
 cookie=0x0, duration=1001.887s, table=39, n_packets=0, n_bytes=0, priority=150,reg10=0x10/0x10 actions=resubmit(,40)
 cookie=0x0, duration=1001.887s, table=39, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,40)
 cookie=0x0, duration=1001.888s, table=40, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x0, duration=1001.888s, table=41, n_packets=0, n_bytes=0, priority=0 actions=load:0->NXM_NX_REG0[],load:0->NXM_NX_REG1[],load:0->NXM_NX_REG2[],load:0->NXM_NX_REG3[],load:0->NXM_NX_REG4[],load:0->NXM_NX_REG5[],load:0->NXM_NX_REG6[],load:0->NXM_NX_REG7[],load:0->NXM_NX_REG8[],load:0->NXM_NX_REG9[],resubmit(,42)
 cookie=0x0, duration=1001.888s, table=64, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,65)
 cookie=0x0, duration=1001.888s, table=65, n_packets=0, n_bytes=0, priority=0 actions=drop
```

ブリッジ br-provider のフローのエントリを確認する。

```sh
ovs-ofctl dump-flows br-provider
```

```
 cookie=0x0, duration=385.809s, table=0, n_packets=72, n_bytes=12848, priority=0 actions=NORMAL
```

ブリッジ br-mgmt のフローのエントリを確認する。

```sh
ovs-ofctl dump-flows br-mgmt
```

```
 cookie=0x0, duration=359.971s, table=0, n_packets=68, n_bytes=11980, priority=0 actions=NORMAL
```
