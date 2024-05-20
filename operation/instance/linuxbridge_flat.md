# インスタンスの作成 (flat/Linux Bridge)

flat ネットワーク(Linux Bridge)に接続するインスタンスを作成する。

## 前提条件

* [](../network/linuxbridge_flat) を作成していること。
* flavor [](../flavor/m1_milli) を作成していること。
* イメージ [](../../installation/controller/glance) でイメージを作成していること。
* [](../sshkey/keypair.md) を作成していること。
* セキュリティグループのルール [](../security_group/icmp) を作成していること。
* セキュリティグループのルール [](../security_group/ssh) を作成していること。

## インスタンスの作成

```{tip}
myuser で実行
```

```{note}
イメージの取得(`openstack image list`)で HTTP403 のエラーが発生するため、
プロジェクト myproject でユーザ myuser にロール admin 権限を追加する。

> openstack role add --user myuser --project myproject admin
```

```{warning}
セキュリティグループに default を指定するとエラーが発生するため、
セキュリティグループ mysecurity を指定する。

> More than one SecurityGroup exists with the name 'default'.
```

インスタンス instance00 を作成する。

```sh
openstack server create \
    --flavor m1.milli \
    --image cirros062 \
    --nic net-id=83a19a08-f066-465f-a5e1-23b4fc66e5ac \
    --security-group mysecurity \
    --key-name mykey \
    instance00
```

```
+--------------------------------------+--------------------------------------------------+
| Field                                | Value                                            |
+--------------------------------------+--------------------------------------------------+
| OS-DCF:diskConfig                    | MANUAL                                           |
| OS-EXT-AZ:availability_zone          |                                                  |
| OS-EXT-SRV-ATTR:host                 | None                                             |
| OS-EXT-SRV-ATTR:hypervisor_hostname  | None                                             |
| OS-EXT-SRV-ATTR:instance_name        |                                                  |
| OS-EXT-STS:power_state               | NOSTATE                                          |
| OS-EXT-STS:task_state                | scheduling                                       |
| OS-EXT-STS:vm_state                  | building                                         |
| OS-SRV-USG:launched_at               | None                                             |
| OS-SRV-USG:terminated_at             | None                                             |
| accessIPv4                           |                                                  |
| accessIPv6                           |                                                  |
| addresses                            |                                                  |
| adminPass                            | Y3SYvjTsivUX                                     |
| config_drive                         |                                                  |
| created                              | 2024-05-10T16:04:55Z                             |
| flavor                               | m1.milli (1)                                     |
| hostId                               |                                                  |
| id                                   | 6714991a-2c2c-4835-ac83-e936f16bbca3             |
| image                                | cirros062 (18b72eca-3cf5-40fb-b4fa-441938b13964) |
| key_name                             | mykey                                            |
| name                                 | instance00                                       |
| os-extended-volumes:volumes_attached | []                                               |
| progress                             | 0                                                |
| project_id                           | bccf406c045d401b91ba5c7552a124ae                 |
| properties                           |                                                  |
| security_groups                      | name='7a3d9999-6c97-42da-a755-f7c6a435049e'      |
| status                               | BUILD                                            |
| updated                              | 2024-05-10T16:04:55Z                             |
| user_id                              | 7f3acb28d26943bab9510df3a6edf3b0                 |
+--------------------------------------+--------------------------------------------------+
```

## インスタンスの確認

インスタンスが ACTIVE になったことを確認する。

```sh
openstack server list
```

```
+--------------------------------------+------------+--------+-----------------------+-----------+----------+
| ID                                   | Name       | Status | Networks              | Image     | Flavor   |
+--------------------------------------+------------+--------+-----------------------+-----------+----------+
| 6714991a-2c2c-4835-ac83-e936f16bbca3 | instance00 | ACTIVE | provider=172.16.0.163 | cirros062 | m1.milli |
+--------------------------------------+------------+--------+-----------------------+-----------+----------+
```

SSH で接続できるか確認する。

```sh
ssh -i demo_rsa cirros@172.16.0.163 ip addr show
```

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast qlen 1000
    link/ether fa:16:3e:6c:5b:9d brd ff:ff:ff:ff:ff:ff
    inet 172.16.0.163/24 brd 172.16.0.255 scope global dynamic noprefixroute eth0
       valid_lft 86305sec preferred_lft 75505sec
    inet6 fe80::f816:3eff:fe6c:5b9d/64 scope link
       valid_lft forever preferred_lft forever
```

## 環境の確認

### dnsmasq

DHCP で IP アドレスが払い出されている。

```sh
cat /var/lib/neutron/dhcp/83a19a08-f066-465f-a5e1-23b4fc66e5ac/leases
```

```
1715443531 fa:16:3e:6c:5b:9d 172.16.0.163 host-172-16-0-163 01:fa:16:3e:6c:5b:9d
```

DHCP に MAC アドレスと IP アドレスの関連が追加される。

```sh
cat /var/lib/neutron/dhcp/83a19a08-f066-465f-a5e1-23b4fc66e5ac/host
```

```
fa:16:3e:6c:5b:9d,host-172-16-0-163.openstacklocal,172.16.0.163
```

DNS のエントリが追加される。

```sh
cat /var/lib/neutron/dhcp/83a19a08-f066-465f-a5e1-23b4fc66e5ac/addn_hosts
```

```
172.16.0.163    host-172-16-0-163.openstacklocal host-172-16-0-163
```

### インスタンス

Compute Node で確認する。

```sh
virsh list
```

```
 Id   名前                状態
----------------------------------
 7    instance-00000009   実行中
```

ネットワークインターフェイスの設定を確認する。

```sh
virsh dumpxml 7 | sed -n -e '/<interface/,/<\/interface>/ { p }'
```

```xml
<interface type='bridge'>
  <mac address='fa:16:3e:6c:5b:9d'/>
  <source bridge='brq83a19a08-f0'/>
  <target dev='tap3d11c4ac-9c'/>
  <model type='virtio'/>
  <driver name='qemu'/>
  <mtu size='1500'/>
  <alias name='net0'/>
  <address type='pci' domain='0x0000' bus='0x00' slot='0x03' function='0x0'/>
</interface>
```

### ネットワーク

Compute Node でネットワーク構成を確認する。

![Linux Bridge flat ネットワーク](../../_static/image/network_flat_instance_linuxbridge.png "Linux Bridge flat ネットワーク")

#### ネットワーク名前空間

ネットワーク名前空間は作成されない。

#### デバイス

ブリッジと TAP デバイスが追加される。

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
4: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq master brq83a19a08-f0 state UP mode DEFAULT group default qlen 1000
    link/ether 00:15:5d:bf:ba:58 brd ff:ff:ff:ff:ff:ff promiscuity 1  allmulti 1 minmtu 68 maxmtu 65521
    bridge_slave state forwarding priority 32 cost 2 hairpin off guard off root_block off fastleave off learning on flood on port_id 0x8002 port_no 0x2 designated_port 32770 designated_cost 0 designated_bridge 8000.0:15:5d:bf:ba:58 designated_root 8000.0:15:5d:bf:ba:58 hold_timer    0.00 message_age_timer    0.00 forward_delay_timer    0.00 topology_change_ack 0 config_pending 0 proxy_arp off proxy_arp_wifi off mcast_router 1 mcast_fast_leave off mcast_flood on bcast_flood on mcast_to_unicast off neigh_suppress off group_fwd_mask 0 group_fwd_mask_str 0x0 vlan_tunnel off isolated off locked off mab off addrgenmode none numtxqueues 64 numrxqueues 64 gso_max_size 62780 gso_max_segs 65535 tso_max_size 62780 tso_max_segs 65535 gro_max_size 65536 parentbus vmbus parentdev fb193242-70ac-437d-9008-4382b02d2a70
5: eth3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 00:15:5d:bf:ba:59 brd ff:ff:ff:ff:ff:ff promiscuity 0  allmulti 0 minmtu 68 maxmtu 65521 addrgenmode none numtxqueues 64 numrxqueues 64 gso_max_size 62780 gso_max_segs 65535 tso_max_size 62780 tso_max_segs 65535 gro_max_size 65536 parentbus vmbus parentdev 6de0f76b-b7bc-45ba-9087-d8bee9131e1c
6: brq83a19a08-f0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 00:15:5d:bf:ba:58 brd ff:ff:ff:ff:ff:ff promiscuity 0  allmulti 0 minmtu 68 maxmtu 65535
    bridge forward_delay 0 hello_time 200 max_age 2000 ageing_time 30000 stp_state 0 priority 32768 vlan_filtering 0 vlan_protocol 802.1Q bridge_id 8000.0:15:5d:bf:ba:58 designated_root 8000.0:15:5d:bf:ba:58 root_port 0 root_path_cost 0 topology_change 0 topology_change_detected 0 hello_timer    0.00 tcn_timer    0.00 topology_change_timer    0.00 gc_timer  204.56 vlan_default_pvid 1 vlan_stats_enabled 0 vlan_stats_per_port 0 group_fwd_mask 0 group_address 01:80:c2:00:00:00 mcast_snooping 0 no_linklocal_learn 0 mcast_vlan_snooping 0 mcast_router 1 mcast_query_use_ifaddr 0 mcast_querier 0 mcast_hash_elasticity 16 mcast_hash_max 4096 mcast_last_member_count 2 mcast_startup_query_count 2 mcast_last_member_interval 100 mcast_membership_interval 26000 mcast_querier_interval 25500 mcast_query_interval 12500 mcast_query_response_interval 1000 mcast_startup_query_interval 3125 mcast_stats_enabled 0 mcast_igmp_version 2 mcast_mld_version 1 nf_call_iptables 0 nf_call_ip6tables 0 nf_call_arptables 0 addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 62780 gso_max_segs 65535 tso_max_size 62780 tso_max_segs 65535 gro_max_size 65536
13: tap3d11c4ac-9c: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master brq83a19a08-f0 state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether fe:16:3e:6c:5b:9d brd ff:ff:ff:ff:ff:ff promiscuity 1  allmulti 1 minmtu 68 maxmtu 65521
    tun type tap pi off vnet_hdr on persist off
    bridge_slave state forwarding priority 32 cost 100 hairpin off guard off root_block off fastleave off learning on flood on port_id 0x8001 port_no 0x1 designated_port 32769 designated_cost 0 designated_bridge 8000.0:15:5d:bf:ba:58 designated_root 8000.0:15:5d:bf:ba:58 hold_timer    0.00 message_age_timer    0.00 forward_delay_timer    0.00 topology_change_ack 0 config_pending 0 proxy_arp off proxy_arp_wifi off mcast_router 1 mcast_fast_leave off mcast_flood on bcast_flood on mcast_to_unicast off neigh_suppress off group_fwd_mask 0 group_fwd_mask_str 0x0 vlan_tunnel off isolated off locked off mab off addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 65536 tso_max_segs 65535 gro_max_size 65536
```

#### イーサネット

インスタンスのルーティングを確認する。

```sh
ssh -i demo_rsa cirros@172.16.0.163 ip route show
```

```
default via 172.16.0.254 dev eth0  src 172.16.0.163  metric 1002
169.254.169.254 via 172.16.0.100 dev eth0  src 172.16.0.163  metric 1002
172.16.0.0/24 dev eth0 scope link  src 172.16.0.163  metric 1002
```

インスタンスの名前解決設定を確認する。

```sh
ssh -i demo_rsa cirros@172.16.0.163 cat /etc/resolv.conf
```

```
# Generated by dhcpcd from eth0.dhcp
# /etc/resolv.conf.head can replace this line
domain openstacklocal
nameserver 172.16.0.100
# /etc/resolv.conf.tail can replace this line
```

### セキュリティグループ

セキュリティグループのルールは iptables のテーブル(neutron-linuxbri-i3d11c4ac-9)に設定される。

```sh
iptables -L
```

```
Chain INPUT (policy ACCEPT)
target     prot opt source               destination
neutron-linuxbri-INPUT  all  --  anywhere             anywhere

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination
neutron-filter-top  all  --  anywhere             anywhere
neutron-linuxbri-FORWARD  all  --  anywhere             anywhere

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
neutron-filter-top  all  --  anywhere             anywhere
neutron-linuxbri-OUTPUT  all  --  anywhere             anywhere

Chain neutron-filter-top (2 references)
target     prot opt source               destination
neutron-linuxbri-local  all  --  anywhere             anywhere

Chain neutron-linuxbri-FORWARD (1 references)
target     prot opt source               destination
neutron-linuxbri-sg-chain  all  --  anywhere             anywhere             PHYSDEV match --physdev-out tap3d11c4ac-9c --physdev-is-bridged /* Direct traffic from the VM interface to the security group chain. */
neutron-linuxbri-sg-chain  all  --  anywhere             anywhere             PHYSDEV match --physdev-in tap3d11c4ac-9c --physdev-is-bridged /* Direct traffic from the VM interface to the security group chain. */

Chain neutron-linuxbri-INPUT (1 references)
target     prot opt source               destination
neutron-linuxbri-o3d11c4ac-9  all  --  anywhere             anywhere             PHYSDEV match --physdev-in tap3d11c4ac-9c --physdev-is-bridged /* Direct incoming traffic from VM to the security group chain. */

Chain neutron-linuxbri-OUTPUT (1 references)
target     prot opt source               destination

Chain neutron-linuxbri-i3d11c4ac-9 (1 references)
target     prot opt source               destination
RETURN     all  --  anywhere             anywhere             state RELATED,ESTABLISHED /* Direct packets associated with a known session to the RETURN chain. */
RETURN     udp  --  anywhere             172.16.0.163         udp spt:bootps dpt:bootpc
RETURN     udp  --  anywhere             255.255.255.255      udp spt:bootps dpt:bootpc
RETURN     icmp --  anywhere             anywhere
RETURN     tcp  --  anywhere             anywhere             tcp dpt:ssh
DROP       all  --  anywhere             anywhere             state INVALID /* Drop packets that appear related to an existing connection (e.g. TCP ACK/FIN) but do not have an entry in conntrack. */
neutron-linuxbri-sg-fallback  all  --  anywhere             anywhere             /* Send unmatched traffic to the fallback chain. */

Chain neutron-linuxbri-local (1 references)
target     prot opt source               destination

Chain neutron-linuxbri-o3d11c4ac-9 (2 references)
target     prot opt source               destination
RETURN     udp  --  0.0.0.0              255.255.255.255      udp spt:bootpc dpt:bootps /* Allow DHCP client traffic. */
neutron-linuxbri-s3d11c4ac-9  all  --  anywhere             anywhere
RETURN     udp  --  anywhere             anywhere             udp spt:bootpc dpt:bootps /* Allow DHCP client traffic. */
DROP       udp  --  anywhere             anywhere             udp spt:bootps dpt:bootpc /* Prevent DHCP Spoofing by VM. */
RETURN     all  --  anywhere             anywhere             state RELATED,ESTABLISHED /* Direct packets associated with a known session to the RETURN chain. */
RETURN     all  --  anywhere             anywhere
DROP       all  --  anywhere             anywhere             state INVALID /* Drop packets that appear related to an existing connection (e.g. TCP ACK/FIN) but do not have an entry in conntrack. */
neutron-linuxbri-sg-fallback  all  --  anywhere             anywhere             /* Send unmatched traffic to the fallback chain. */

Chain neutron-linuxbri-s3d11c4ac-9 (1 references)
target     prot opt source               destination
RETURN     all  --  172.16.0.163         anywhere             MAC fa:16:3e:6c:5b:9d /* Allow traffic from defined IP/MAC pairs. */
DROP       all  --  anywhere             anywhere             /* Drop traffic without an IP/MAC allow rule. */

Chain neutron-linuxbri-sg-chain (2 references)
target     prot opt source               destination
neutron-linuxbri-i3d11c4ac-9  all  --  anywhere             anywhere             PHYSDEV match --physdev-out tap3d11c4ac-9c --physdev-is-bridged /* Jump to the VM specific chain. */
neutron-linuxbri-o3d11c4ac-9  all  --  anywhere             anywhere             PHYSDEV match --physdev-in tap3d11c4ac-9c --physdev-is-bridged /* Jump to the VM specific chain. */
ACCEPT     all  --  anywhere             anywhere

Chain neutron-linuxbri-sg-fallback (2 references)
target     prot opt source               destination
DROP       all  --  anywhere             anywhere             /* Default drop rule for unmatched traffic. */
```
