# 起動 (Open vSwitch)

nova を再起動する。

```sh
systemctl restart openstack-nova-api
```

マシン起動時の自動起動設定とサービスを起動する。

```sh
systemctl enable --now neutron-server
systemctl enable --now neutron-openvswitch-agent
systemctl enable --now neutron-dhcp-agent
systemctl enable --now neutron-metadata-agent
```

## flat ネットワーク用ブリッジの作成

eth0 と接続するブリッジ be-provider を作成する。

```sh
nmcli c d eth0
nmcli c delete  eth0
nmcli con add \
    type ovs-bridge \
    con-name br-provider \
    ifname br-provider
nmcli con add \
    type ovs-port \
    con-name br-provider \
    ifname br-provider \
    master br-provider
nmcli con add \
    type ovs-interface \
    con-name br-provider \
    slave-type ovs-port \
    ifname br-provider \
    master br-provider \
    ipv4.method manual \
    ipv4.address 172.16.0.11/12 \
    ipv6.method disabled
nmcli con add \
    type ovs-port \
    con-name eth0 \
    ifname eth0 \
    master br-provider
nmcli con add \
    type ethernet \
    con-name eth0 \
    ifname eth0 \
    master eth0
```
