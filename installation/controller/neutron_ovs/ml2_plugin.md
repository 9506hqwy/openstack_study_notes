# ML2 プラグインの設定 (Open vSwitch)

設定ファイルを更新する。

```sh
sed \
    -e '/^\[ml2]/,/^\[/ {
      /^type_drivers =/d
      /^#type_drivers =/atype_drivers = flat,vlan
      /^tenant_network_types =/d
      /^#tenant_network_types =/atenant_network_types =
      /^mechanism_drivers =/d
      /^#mechanism_drivers =/amechanism_drivers = openvswitch
      /^extension_drivers =/d
      /^#extension_drivers =/aextension_drivers = port_security
    }' \
    -e '/^\[ml2_type_flat]/,/^\[/ {
      /^flat_networks =/d
      /^#flat_networks =/aflat_networks = provider,mgmt
    }' \
    -i /etc/neutron/plugins/ml2/ml2_conf.ini
```
