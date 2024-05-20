# 動作確認 (Open vSwitch)

Controller Node でエージェントを表示する。

```sh
openstack network agent list
```

```
+--------------------------------------+--------------------+-----------------------+-------------------+-------+-------+---------------------------+
| ID                                   | Agent Type         | Host                  | Availability Zone | Alive | State | Binary                    |
+--------------------------------------+--------------------+-----------------------+-------------------+-------+-------+---------------------------+
| 1fbd5909-3c41-4356-a042-24dc5befc8a0 | DHCP agent         | controller.home.local | nova              | :-)   | UP    | neutron-dhcp-agent        |
| 2a57065a-d890-450f-bae0-2f456898d009 | Open vSwitch agent | compute.home.local    | None              | :-)   | UP    | neutron-openvswitch-agent |
| 5368ea0a-53d6-4f3c-946b-e01467f08929 | Metadata agent     | controller.home.local | None              | :-)   | UP    | neutron-metadata-agent    |
| ec88144f-cccb-48f3-a94d-4899f4cf0743 | Open vSwitch agent | controller.home.local | None              | :-)   | UP    | neutron-openvswitch-agent |
+--------------------------------------+--------------------+-----------------------+-------------------+-------+-------+---------------------------+
```

## 環境の確認

ネットワークの構成を確認する。

![Open vSwitch Compute Node ネットワーク](../../../_static/image/network_base_compute_ovs.png "Open vSwitch Compute Node ネットワーク")

### デバイス

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
    link/ether a2:2b:ac:26:3b:91 brd ff:ff:ff:ff:ff:ff promiscuity 1  allmulti 0 minmtu 68 maxmtu 65535
    openvswitch addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 65536 tso_max_segs 65535 gro_max_size 65536
7: br-int: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether 4a:73:de:e4:cd:4f brd ff:ff:ff:ff:ff:ff promiscuity 1  allmulti 0 minmtu 68 maxmtu 65535
    openvswitch addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 65536 tso_max_segs 65535 gro_max_size 65536
8: br-provider: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether 00:15:5d:bf:ba:55 brd ff:ff:ff:ff:ff:ff promiscuity 1  allmulti 0 minmtu 68 maxmtu 65535
    openvswitch addrgenmode none numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 65536 tso_max_segs 65535 gro_max_size 65536
9: br-mgmt: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether 00:15:5d:bf:ba:56 brd ff:ff:ff:ff:ff:ff promiscuity 1  allmulti 0 minmtu 68 maxmtu 65535
    openvswitch addrgenmode none numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 65536 tso_max_segs 65535 gro_max_size 65536
```

### ブリッジ

Open vSwitch の構成を確認する。

```sh
ovs-vsctl show
```

```
698493db-95ef-4c31-b419-56b3d4096f2b
    Manager "ptcp:6640:127.0.0.1"
        is_connected: true
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
        Port int-br-mgmt
            Interface int-br-mgmt
                type: patch
                options: {peer=phy-br-mgmt}
        Port int-br-provider
            Interface int-br-provider
                type: patch
                options: {peer=phy-br-provider}
    Bridge br-provider
        Controller "tcp:127.0.0.1:6633"
            is_connected: true
        fail_mode: secure
        datapath_type: system
        Port phy-br-provider
            Interface phy-br-provider
                type: patch
                options: {peer=int-br-provider}
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
  lookups: hit:6 missed:2 lost:0
  flows: 0
  masks: hit:6 total:0 hit/pkt:0.75
  cache: hit:4 hit-rate:50.00%
  caches:
    masks-cache: size:256
  port 0: ovs-system (internal)
  port 1: br-int (internal)
  port 2: eth2
  port 3: br-provider (internal)
  port 4: eth3
  port 5: br-mgmt (internal)
```

ブリッジ br-provider のフローのエントリを確認する。

```sh
ovs-ofctl dump-flows br-provider
```

```
 cookie=0x76728ec05de82ede, duration=191.827s, table=0, n_packets=0, n_bytes=0, priority=2,in_port="phy-br-provider" actions=drop
 cookie=0x76728ec05de82ede, duration=191.833s, table=0, n_packets=4, n_bytes=868, priority=0 actions=NORMAL
```

ブリッジ br-mgmt のフローのエントリを確認する。

```sh
ovs-ofctl dump-flows br-mgmt
```

```
 cookie=0xbccbc2cb2fecfe9d, duration=200.342s, table=0, n_packets=0, n_bytes=0, priority=2,in_port="phy-br-mgmt" actions=drop
 cookie=0xbccbc2cb2fecfe9d, duration=200.386s, table=0, n_packets=4, n_bytes=868, priority=0 actions=NORMAL
```

ブリッジ br-int のフローのエントリを確認する。

```sh
ovs-ofctl dump-flows br-int
```

```
 cookie=0xb582a404f2ab1285, duration=212.534s, table=0, n_packets=0, n_bytes=0, priority=65535,dl_vlan=4095 actions=drop
 cookie=0xb582a404f2ab1285, duration=212.514s, table=0, n_packets=6, n_bytes=1302, priority=2,in_port="int-br-provider" actions=drop
 cookie=0xb582a404f2ab1285, duration=211.424s, table=0, n_packets=6, n_bytes=1302, priority=2,in_port="int-br-mgmt" actions=drop
 cookie=0xb582a404f2ab1285, duration=212.548s, table=0, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,58)
 cookie=0xb582a404f2ab1285, duration=212.551s, table=23, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0xb582a404f2ab1285, duration=212.536s, table=24, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0xb582a404f2ab1285, duration=212.529s, table=30, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,58)
 cookie=0xb582a404f2ab1285, duration=212.526s, table=31, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,58)
 cookie=0xb582a404f2ab1285, duration=212.544s, table=58, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,60)
 cookie=0xb582a404f2ab1285, duration=212.539s, table=60, n_packets=0, n_bytes=0, priority=3 actions=NORMAL
 cookie=0xb582a404f2ab1285, duration=212.531s, table=62, n_packets=0, n_bytes=0, priority=3 actions=NORMAL
 cookie=0xb582a404f2ab1285, duration=208.735s, table=71, n_packets=0, n_bytes=0, priority=110,ct_state=+trk actions=ct_clear,resubmit(,71)
 cookie=0xb582a404f2ab1285, duration=208.839s, table=71, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0xb582a404f2ab1285, duration=208.817s, table=72, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0xb582a404f2ab1285, duration=208.797s, table=73, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0xb582a404f2ab1285, duration=208.776s, table=81, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0xb582a404f2ab1285, duration=208.756s, table=82, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0xb582a404f2ab1285, duration=208.694s, table=91, n_packets=0, n_bytes=0, priority=1 actions=resubmit(,94)
 cookie=0xb582a404f2ab1285, duration=208.673s, table=92, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0xb582a404f2ab1285, duration=208.653s, table=93, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0xb582a404f2ab1285, duration=208.715s, table=94, n_packets=0, n_bytes=0, priority=1 actions=NORMAL
```
