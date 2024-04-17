# インスタンスの作成 (flat/Linux Bridge)

flat ネットワーク(Linux Bridge)に接続するインスタンスを作成する。

## 前提条件

* [](../network/linuxbridge_flat) を作成していること。
* flavor [](../flavor/m1_nano) を作成していること。
* イメージ [](../../installation/controller/glance) でイメージを作成していること。
* セキュリティグループのルール [](../security_group/icmp) を作成していること。
* セキュリティグループのルール [](../security_group/ssh) を作成していること。

## インスタンスの作成

インスタンス instance00 を作成する。

```sh
openstack server create \
    --flavor m1.nano \
    --image cirros \
    --nic net-id=ca4e2bc3-fe44-48d0-8096-20e6a85d6510 \
    --security-group default \
    --key-name mykey \
    instance00
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
| adminPass                   | MmVAeLh6eFWC                                  |
| config_drive                |                                               |
| created                     | 2024-04-13T15:47:41Z                          |
| flavor                      | m1.nano (0)                                   |
| hostId                      |                                               |
| id                          | 24106df7-6f37-455b-8329-ca2a115aa241          |
| image                       | cirros (e83903c4-7fa8-42a7-b693-f5034bc33603) |
| key_name                    | mykey                                         |
| name                        | instance00                                    |
| progress                    | 0                                             |
| project_id                  | f2aeffb34ff34ffb8959f1cd813655c6              |
| properties                  |                                               |
| security_groups             | name='87fd4685-d317-42fb-a487-28382d2c2750'   |
| status                      | BUILD                                         |
| updated                     | 2024-04-13T15:47:42Z                          |
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
+--------------------------------------+------------+--------+----------------------+--------+---------+
| ID                                   | Name       | Status | Networks             | Image  | Flavor  |
+--------------------------------------+------------+--------+----------------------+--------+---------+
| 24106df7-6f37-455b-8329-ca2a115aa241 | instance00 | ACTIVE | provider=172.17.0.70 | cirros | m1.nano |
+--------------------------------------+------------+--------+----------------------+--------+---------+
```

SSH で接続できるか確認する。

```sh
ssh -i demo_rsa cirros@172.17.0.70 /sbin/ip addr
```

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast qlen 1000
    link/ether fa:16:3e:69:de:60 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.70/12 brd 172.31.255.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:fe69:de60/64 scope link
       valid_lft forever preferred_lft forever
```

## 環境の確認

### dnsmasq

DHCP で IP アドレスが払い出されている。

```sh
cat /var/lib/neutron/dhcp/ca4e2bc3-fe44-48d0-8096-20e6a85d6510/leases
```

```
1713142565 fa:16:3e:69:de:60 172.17.0.70 host-172-17-0-70 01:fa:16:3e:69:de:60
```

DHCP に MAC アドレスと IP アドレスの関連が追加される。

```sh
cat /var/lib/neutron/dhcp/ca4e2bc3-fe44-48d0-8096-20e6a85d6510/host
```

```
fa:16:3e:69:de:60,host-172-17-0-70.openstacklocal,172.17.0.70
```

DNS のエントリが追加される。

```sh
cat /var/lib/neutron/dhcp/ca4e2bc3-fe44-48d0-8096-20e6a85d6510/addn_hosts
```

```
172.17.0.70     host-172-17-0-70.openstacklocal host-172-17-0-70
```

### インスタンス

Compute Node で確認する。

```sh
virsh list
```

```
 Id   名前                状態
----------------------------------
 10   instance-0000000a   実行中
```

ネットワークインターフェイスの設定を確認する。

```sh
virsh dumpxml 10 | sed -n -e '/<interface/,/<\/interface>/ { p }'
```

```xml
<interface type='bridge'>
  <mac address='fa:16:3e:69:de:60'/>
  <source bridge='brqca4e2bc3-fe'/>
  <target dev='tap044a295e-ea'/>
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
nmcli device status
```

```
DEVICE          TYPE      STATE            CONNECTION
eth0            ethernet  接続済み         eth0
brqca4e2bc3-fe  bridge    接続済み (外部)  brqca4e2bc3-fe
eth1            ethernet  接続済み         eth1
tap044a295e-ea  tun       接続済み (外部)  tap044a295e-ea
lo              loopback  管理無し         --
```

ブリッジを確認する。

```sh
bridge link show
```

```
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 master brqca4e2bc3-fe state forwarding priority 32 cost 2
14: tap044a295e-ea: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 master brqca4e2bc3-fe state forwarding priority 32 cost 100
```

インスタンスの eth0 は TAP デバイス tap044a295e-ea を経由してブリッジ brqca4e2bc3-fe に接続する。
