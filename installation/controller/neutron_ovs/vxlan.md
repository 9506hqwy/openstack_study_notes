# VXLAN ネットワーク (Open vSwitch)

## Neutron の設定

設定ファイルにサービスプラグインで使用するプラグインを指定する。

```sh
sed \
    -e '/^\[DEFAULT]/,/^\[/ {
      /^service_plugins =/d
      /^#service_plugins =/aservice_plugins = router
    }' \
    -i /etc/neutron/neutron.conf
```

## ML2 プラグインの設定

設定ファイルに使用する VXLAN 範囲を指定する。

```sh
sed \
    -e '/^\[ml2]/,/^\[/ {
      /^type_drivers =/d
      /^#type_drivers =/atype_drivers = flat,vlan,vxlan
      /^tenant_network_types =/d
      /^#tenant_network_types =/atenant_network_types = vxlan
      /^mechanism_drivers =/d
      /^#mechanism_drivers =/amechanism_drivers = openvswitch,l2population
    }' \
    -e '/^\[ml2_type_vxlan]/,/^\[/ {
      /^vni_ranges =/d
      /^#vni_ranges =/avni_ranges = 200:299
    }' \
    -i /etc/neutron/plugins/ml2/ml2_conf.ini
```

## Open vSwitch の設定

設定ファイルに物理ネットワークのために使用する IP アドレスを指定する。

```sh
sed \
    -e '/^\[agent]/,/^\[/ {
      /^tunnel_types =/d
      /^#tunnel_types =/atunnel_types = vxlan
    }' \
    -e '/^\[ovs]/,/^\[/ {
      /^local_ip =/d
      /^#local_ip =/alocal_ip = 172.16.0.11
    }' \
    -i /etc/neutron/plugins/ml2/openvswitch_agent.ini
```

## L3 エージェントの設定

```sh
sed \
    -e '/^\[DEFAULT]/,/^\[/ {
      /^interface_driver =/d
      /^#interface_driver =/ainterface_driver = openvswitch
    }' \
    -i /etc/neutron/l3_agent.ini
```

## ネットワークの設定

ファイアウィールを開ける。

```sh
firewall-cmd --permanent --zone=public --add-port=4789/udp
firewall-cmd --reload
```

## 起動

neutron を再起動する。

```sh
systemctl restart neutron-server neutron-openvswitch-agent
```

マシン起動時の自動起動設定とサービスを起動する。

```sh
systemctl enable --now neutron-l3-agent
```
