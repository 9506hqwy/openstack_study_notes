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

eth1 と接続するブリッジ be-mgmt を作成する。

```sh
nmcli c d eth1
nmcli c delete  eth1
nmcli con add \
    type ovs-bridge \
    con-name br-mgmt \
    ifname br-mgmt
nmcli con add \
    type ovs-port \
    con-name br-mgmt \
    ifname br-mgmt \
    master br-mgmt
nmcli con add \
    type ovs-interface \
    con-name br-mgmt \
    slave-type ovs-port \
    ifname br-mgmt \
    master br-mgmt \
    ipv4.method manual \
    ipv4.dns 10.0.0.1 \
    ipv4.address 10.0.0.11/24 \
    ipv4.gateway 10.0.0.1 \
    ipv6.method disabled
nmcli con add \
    type ovs-port \
    con-name eth1 \
    ifname eth1 \
    master br-mgmt
nmcli con add \
    type ethernet \
    con-name eth1 \
    ifname eth1 \
    master eth1
```
