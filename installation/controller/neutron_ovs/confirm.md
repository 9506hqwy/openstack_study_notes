# 動作確認 (Open vSwitch)

エージェントを表示する。

```sh
openstack network agent list
```

```
+--------------------------------------+--------------------+-----------------------+-------------------+-------+-------+---------------------------+
| ID                                   | Agent Type         | Host                  | Availability Zone | Alive | State | Binary                    |
+--------------------------------------+--------------------+-----------------------+-------------------+-------+-------+---------------------------+
| 145d9d73-cb1c-4bbf-ad28-adb1a9f4a826 | Metadata agent     | controller.home.local | None              | :-)   | UP    | neutron-metadata-agent    |
| 5bb635cd-1e42-48fa-b2be-c5d0dce3fef4 | Open vSwitch agent | controller.home.local | None              | :-)   | UP    | neutron-openvswitch-agent |
| e843c356-67a9-418e-96db-3fa6e4210df9 | DHCP agent         | controller.home.local | nova              | :-)   | UP    | neutron-dhcp-agent        |
+--------------------------------------+--------------------+-----------------------+-------------------+-------+-------+---------------------------+
```

## 環境の確認

ネットワークの構成を確認する。

![Open vSwitch Controller Node ネットワーク](../../../_static/image/network_base_controller_ovs.png "Open vSwitch Controller Node ネットワーク")

### デバイス

デバイスを確認する。

```sh
ip -d link show
```

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00 promiscuity 0 minmtu 0 maxmtu 0 addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq master ovs-system state UP mode DEFAULT group default qlen 1000
    link/ether 00:15:5d:bf:ba:43 brd ff:ff:ff:ff:ff:ff promiscuity 1 minmtu 68 maxmtu 65521
    openvswitch_slave addrgenmode none numtxqueues 64 numrxqueues 64 gso_max_size 62780 gso_max_segs 65535 parentbus vmbus parentdev 32e6c4c2-ccf5-4432-b0ff-a854fd65c7e7
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 00:15:5d:bf:ba:45 brd ff:ff:ff:ff:ff:ff promiscuity 0 minmtu 68 maxmtu 65521 addrgenmode eui64 numtxqueues 64 numrxqueues 64 gso_max_size 62780 gso_max_segs 65535 parentbus vmbus parentdev d2418736-9a08-43b7-9980-c3f9ebbe063b
4: ovs-system: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 92:74:30:0b:f1:2c brd ff:ff:ff:ff:ff:ff promiscuity 1 minmtu 68 maxmtu 65535
    openvswitch addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535
5: br-int: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether a6:a9:0d:2e:19:4e brd ff:ff:ff:ff:ff:ff promiscuity 1 minmtu 68 maxmtu 65535
    openvswitch addrgenmode none numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535
7: br-provider: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether 00:15:5d:bf:ba:43 brd ff:ff:ff:ff:ff:ff promiscuity 1 minmtu 68 maxmtu 65535
    openvswitch addrgenmode none numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535
```

### ブリッジ

Open vSwitch の構成を確認する。

```sh
ovs-vsctl show
```

```
77a2e96a-ca65-449f-afc7-c7cbe9dff27c
    Manager "ptcp:6640:127.0.0.1"
        is_connected: true
    Bridge br-int
        Controller "tcp:127.0.0.1:6633"
            is_connected: true
        fail_mode: secure
        datapath_type: system
        Port int-br-provider
            Interface int-br-provider
                type: patch
                options: {peer=phy-br-provider}
        Port br-int
            Interface br-int
                type: internal
    Bridge br-provider
        Controller "tcp:127.0.0.1:6633"
            is_connected: true
        fail_mode: secure
        datapath_type: system
        Port phy-br-provider
            Interface phy-br-provider
                type: patch
                options: {peer=int-br-provider}
        Port br-provider
            Interface br-provider
                type: internal
        Port eth0
            Interface eth0
                type: system
    ovs_version: "3.1.4"
```
