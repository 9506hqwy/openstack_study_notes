# flat ネットワーク (Open vSwitch)

Open vSwitch を利用した flat ネットワークを作成する。

## 外部ネットワークの作成

eth0 に繋がる外部ネットワークに flat ネットワークを作成する。

| オプション                  | 説明                         |
| --------------------------- | ---------------------------- |
| --share                     | プロジェクトで共有           |
| -external                   | OpenStack 外部のネットワーク |
| --provider-physical-network | */etc/neutron/plugins/ml2/ml2_conf.ini* の flat_networks に指定した値 |
| --provider-physical-network | flat                         |

```sh
openstack network create \
    --share \
    --external \
    --provider-physical-network provider \
    --provider-network-type flat \
    provider
```

```
+---------------------------+--------------------------------------+
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| admin_state_up            | UP                                   |
| availability_zone_hints   |                                      |
| availability_zones        |                                      |
| created_at                | 2024-04-21T07:02:29Z                 |
| description               |                                      |
| dns_domain                | None                                 |
| id                        | 85f372ef-f39a-42bb-a06b-9f9ed8e4e58e |
| ipv4_address_scope        | None                                 |
| ipv6_address_scope        | None                                 |
| is_default                | False                                |
| is_vlan_transparent       | None                                 |
| mtu                       | 1500                                 |
| name                      | provider                             |
| port_security_enabled     | True                                 |
| project_id                | 1e3ac7ae10e24515a0956beaa1d8073c     |
| provider:network_type     | flat                                 |
| provider:physical_network | provider                             |
| provider:segmentation_id  | None                                 |
| qos_policy_id             | None                                 |
| revision_number           | 1                                    |
| router:external           | External                             |
| segments                  | None                                 |
| shared                    | True                                 |
| status                    | ACTIVE                               |
| subnets                   |                                      |
| tags                      |                                      |
| updated_at                | 2024-04-21T07:02:29Z                 |
+---------------------------+--------------------------------------+
```

## サブネットの作成

サブネットを作成する。


| オプション                  | 説明                         |
| --------------------------- | ---------------------------- |
| --network                   | ネットワーク                 |
| --allocation-pool           | IP アドレス範囲              |
| --gateway                   | ゲートウェイ IP アドレス     |
| -subnet-range               | サブネットの CIDR            |

```sh
openstack subnet create \
    --network provider \
    --allocation-pool start=172.17.0.0,end=172.17.0.255 \
    --gateway 172.16.0.1 \
    --subnet-range 172.16.0.0/12 \
    provider
```

```
+----------------------+--------------------------------------+
| Field                | Value                                |
+----------------------+--------------------------------------+
| allocation_pools     | 172.17.0.0-172.17.0.255              |
| cidr                 | 172.16.0.0/12                        |
| created_at           | 2024-04-21T07:04:04Z                 |
| description          |                                      |
| dns_nameservers      |                                      |
| dns_publish_fixed_ip | None                                 |
| enable_dhcp          | True                                 |
| gateway_ip           | 172.16.0.1                           |
| host_routes          |                                      |
| id                   | ef7c7472-0b1d-497f-bc86-889a5c6ae52c |
| ip_version           | 4                                    |
| ipv6_address_mode    | None                                 |
| ipv6_ra_mode         | None                                 |
| name                 | provider                             |
| network_id           | 85f372ef-f39a-42bb-a06b-9f9ed8e4e58e |
| project_id           | 1e3ac7ae10e24515a0956beaa1d8073c     |
| revision_number      | 0                                    |
| segment_id           | None                                 |
| service_types        |                                      |
| subnetpool_id        | None                                 |
| tags                 |                                      |
| updated_at           | 2024-04-21T07:04:04Z                 |
+----------------------+--------------------------------------+
```

DHCP サーバのポートの作成を確認する。

```sh
openstack port list
```

```
+--------------------------------------+------+-------------------+---------------------------------------------------------------------------+--------+
| ID                                   | Name | MAC Address       | Fixed IP Addresses                                                        | Status |
+--------------------------------------+------+-------------------+---------------------------------------------------------------------------+--------+
| 43bf8dcc-083a-4c61-8d5f-bd6c7e763e5b |      | fa:16:3e:4a:1e:19 | ip_address='172.17.0.0', subnet_id='ef7c7472-0b1d-497f-bc86-889a5c6ae52c' | ACTIVE |
+--------------------------------------+------+-------------------+---------------------------------------------------------------------------+--------+
```

```sh
openstack port show 43bf8dcc-083a-4c61-8d5f-bd6c7e763e5b
```

```
+-------------------------+---------------------------------------------------------------------------------------------------------------------------------------------+
| Field                   | Value                                                                                                                                       |
+-------------------------+---------------------------------------------------------------------------------------------------------------------------------------------+
| admin_state_up          | UP                                                                                                                                          |
| allowed_address_pairs   |                                                                                                                                             |
| binding_host_id         | controller.home.local                                                                                                                       |
| binding_profile         |                                                                                                                                             |
| binding_vif_details     | bound_drivers.0='openvswitch', bridge_name='br-int', connectivity='l2', datapath_type='system', ovs_hybrid_plug='False', port_filter='True' |
| binding_vif_type        | ovs                                                                                                                                         |
| binding_vnic_type       | normal                                                                                                                                      |
| created_at              | 2024-04-21T07:04:05Z                                                                                                                        |
| data_plane_status       | None                                                                                                                                        |
| description             |                                                                                                                                             |
| device_id               | dhcpd3377d3c-a0d1-5d71-9947-f17125c357bb-85f372ef-f39a-42bb-a06b-9f9ed8e4e58e                                                               |
| device_owner            | network:dhcp                                                                                                                                |
| device_profile          | None                                                                                                                                        |
| dns_assignment          | None                                                                                                                                        |
| dns_domain              | None                                                                                                                                        |
| dns_name                | None                                                                                                                                        |
| extra_dhcp_opts         |                                                                                                                                             |
| fixed_ips               | ip_address='172.17.0.0', subnet_id='ef7c7472-0b1d-497f-bc86-889a5c6ae52c'                                                                   |
| id                      | 43bf8dcc-083a-4c61-8d5f-bd6c7e763e5b                                                                                                        |
| ip_allocation           | None                                                                                                                                        |
| mac_address             | fa:16:3e:4a:1e:19                                                                                                                           |
| name                    |                                                                                                                                             |
| network_id              | 85f372ef-f39a-42bb-a06b-9f9ed8e4e58e                                                                                                        |
| numa_affinity_policy    | None                                                                                                                                        |
| port_security_enabled   | False                                                                                                                                       |
| project_id              | 1e3ac7ae10e24515a0956beaa1d8073c                                                                                                            |
| propagate_uplink_status | None                                                                                                                                        |
| qos_network_policy_id   | None                                                                                                                                        |
| qos_policy_id           | None                                                                                                                                        |
| resource_request        | None                                                                                                                                        |
| revision_number         | 3                                                                                                                                           |
| security_group_ids      |                                                                                                                                             |
| status                  | ACTIVE                                                                                                                                      |
| tags                    |                                                                                                                                             |
| trunk_details           | None                                                                                                                                        |
| updated_at              | 2024-04-21T07:04:14Z                                                                                                                        |
+-------------------------+---------------------------------------------------------------------------------------------------------------------------------------------+
```

## 環境の確認

Controller Node でネットワーク構成を確認する。

![Open vSwitch flat ネットワーク](../../_static/image/network_flat_dhcpagent_ovs.png "Open vSwitch flat ネットワーク")

### ネットワーク名前空間

サブネットを作成するとネットワーク名前空間が作成される。

```sh
ip netns
```

```
qdhcp-85f372ef-f39a-42bb-a06b-9f9ed8e4e58e (id: 0)
```

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
    openvswitch_slave addrgenmode eui64 numtxqueues 64 numrxqueues 64 gso_max_size 62780 gso_max_segs 65535 parentbus vmbus parentdev 32e6c4c2-ccf5-4432-b0ff-a854fd65c7e7
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 00:15:5d:bf:ba:45 brd ff:ff:ff:ff:ff:ff promiscuity 0 minmtu 68 maxmtu 65521 addrgenmode eui64 numtxqueues 64 numrxqueues 64 gso_max_size 62780 gso_max_segs 65535 parentbus vmbus parentdev d2418736-9a08-43b7-9980-c3f9ebbe063b
4: ovs-system: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 6a:8a:ae:41:17:ed brd ff:ff:ff:ff:ff:ff promiscuity 1 minmtu 68 maxmtu 65535
    openvswitch addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535
5: br-int: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether a6:a9:0d:2e:19:4e brd ff:ff:ff:ff:ff:ff promiscuity 1 minmtu 68 maxmtu 65535
    openvswitch addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535
6: br-provider: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 00:15:5d:bf:ba:43 brd ff:ff:ff:ff:ff:ff promiscuity 1 minmtu 68 maxmtu 65535
    openvswitch addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535
```

ネットワーク名前空間内のデバイスを確認する。

```sh
ip netns exec qdhcp-85f372ef-f39a-42bb-a06b-9f9ed8e4e58e ip -d link show
```

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00 promiscuity 0 minmtu 0 maxmtu 0 addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535
7: tap43bf8dcc-08: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether fa:16:3e:4a:1e:19 brd ff:ff:ff:ff:ff:ff promiscuity 1 minmtu 68 maxmtu 65535
    openvswitch addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535
```

ブリッジを確認する。

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
        Port tap43bf8dcc-08
            tag: 1
            Interface tap43bf8dcc-08
                type: internal
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
        Port eth0
            Interface eth0
        Port phy-br-provider
            Interface phy-br-provider
                type: patch
                options: {peer=int-br-provider}
        Port br-provider
            Interface br-provider
                type: internal
    ovs_version: "3.1.4"
```

### イーサネット

ネットワーク名前空間内のイーサネットの情報を確認する。
169.254.169.254 は Metadata agent が使用する。

```sh
ip netns exec qdhcp-85f372ef-f39a-42bb-a06b-9f9ed8e4e58e ip addr show
```

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
7: tap43bf8dcc-08: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ether fa:16:3e:4a:1e:19 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.0/12 brd 172.31.255.255 scope global tap43bf8dcc-08
       valid_lft forever preferred_lft forever
    inet 169.254.169.254/32 brd 169.254.169.254 scope global tap43bf8dcc-08
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:fe4a:1e19/64 scope link
       valid_lft forever preferred_lft forever
```

ルーティングを確認する。

```sh
ip netns exec qdhcp-85f372ef-f39a-42bb-a06b-9f9ed8e4e58e ip route
```

```
default via 172.16.0.1 dev tap43bf8dcc-08 proto static
172.16.0.0/12 dev tap43bf8dcc-08 proto kernel scope link src 172.17.0.0
```

待ち受けているポートを確認する。

```sh
ip netns exec qdhcp-85f372ef-f39a-42bb-a06b-9f9ed8e4e58e ss -ano -4
```

```
Netid              State               Recv-Q              Send-Q                             Local Address:Port                           Peer Address:Port             Process
udp                UNCONN              0                   0                                      127.0.0.1:53                                  0.0.0.0:*
udp                UNCONN              0                   0                                     172.17.0.0:53                                  0.0.0.0:*
udp                UNCONN              0                   0                                169.254.169.254:53                                  0.0.0.0:*
udp                UNCONN              0                   0                                        0.0.0.0:67                                  0.0.0.0:*
tcp                LISTEN              0                   32                                     127.0.0.1:53                                  0.0.0.0:*
tcp                LISTEN              0                   32                                    172.17.0.0:53                                  0.0.0.0:*
tcp                LISTEN              0                   32                               169.254.169.254:53                                  0.0.0.0:*
tcp                LISTEN              0                   128                              169.254.169.254:80                                  0.0.0.0:*
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
    --pid-file=/var/lib/neutron/dhcp/85f372ef-f39a-42bb-a06b-9f9ed8e4e58e/pid \
    --dhcp-hostsfile=/var/lib/neutron/dhcp/85f372ef-f39a-42bb-a06b-9f9ed8e4e58e/host \
    --addn-hosts=/var/lib/neutron/dhcp/85f372ef-f39a-42bb-a06b-9f9ed8e4e58e/addn_hosts \
    --dhcp-optsfile=/var/lib/neutron/dhcp/85f372ef-f39a-42bb-a06b-9f9ed8e4e58e/opts \
    --dhcp-leasefile=/var/lib/neutron/dhcp/85f372ef-f39a-42bb-a06b-9f9ed8e4e58e/leases \
    --dhcp-match=set:ipxe,175 \
    --dhcp-userclass=set:ipxe6,iPXE \
    --local-service \
    --bind-dynamic \
    --dhcp-range=set:subnet-ef7c7472-0b1d-497f-bc86-889a5c6ae52c,172.16.0.0,static,255.240.0.0,86400s \
    --dhcp-option-force=option:mtu,1500 \
    --dhcp-lease-max=1048576 \
    --conf-file=/dev/null \
    --domain=openstacklocal
```

使用しているインターフェイスを確認する。

```sh
cat /var/lib/neutron/dhcp/85f372ef-f39a-42bb-a06b-9f9ed8e4e58e/interface
```

```
tap43bf8dcc-08
```
