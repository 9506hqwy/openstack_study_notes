# Neutron (VLAN ネットワーク)

## Neutron の設定

設定ファイルに使用する VLAN 範囲を指定する。

```sh
sed \
    -e '/^\[ml2_type_vlan]/,/^\[/ {
      /^network_vlan_ranges =/d
      /^#network_vlan_ranges =/anetwork_vlan_ranges = provider:100:199
    }' \
    -i /etc/neutron/plugins/ml2/ml2_conf.ini
```

## 起動

neutron を再起動する。

```sh
systemctl restart neutron-server
```
