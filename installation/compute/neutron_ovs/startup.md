# 起動 (Open vSwitch)

nova を再起動する。

```sh
systemctl restart NetworkManager
systemctl restart openstack-nova-compute
```

マシン起動時の自動起動設定とサービスを起動する。

```sh
systemctl enable --now neutron-openvswitch-agent
```

## flat ネットワーク用ブリッジの作成

eth2 と接続するブリッジ be-provider を作成する。

```sh
nmcli c d eth2
nmcli c delete  eth2
nmcli con add \
    type ovs-bridge \
    con-name br-provider \
    ifname br-provider
nmcli con add \
    type ovs-port \
    con-name eth2 \
    ifname eth2 \
    master br-provider
nmcli con add \
    type ethernet \
    con-name eth2 \
    ifname eth2 \
    master eth2
```

eth3 と接続するブリッジ be-mgmt を作成する。

```sh
nmcli c d eth3
nmcli c delete  eth3
nmcli con add \
    type ovs-bridge \
    con-name br-mgmt \
    ifname br-mgmt
nmcli con add \
    type ovs-port \
    con-name eth3 \
    ifname eth3 \
    master br-mgmt
nmcli con add \
    type ethernet \
    con-name eth3 \
    ifname eth3 \
    master eth3
```
