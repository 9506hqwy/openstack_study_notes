# ML2 プラグインの設定 (Open Virtual Network)

設定ファイルを更新する。

```sh
sed \
    -e '/^\[ml2]/,/^\[/ {
      /^type_drivers =/d
      /^#type_drivers =/atype_drivers = flat,vlan,geneve
      /^tenant_network_types =/d
      /^#tenant_network_types =/atenant_network_types = geneve
      /^mechanism_drivers =/d
      /^#mechanism_drivers =/amechanism_drivers = ovn
      /^extension_drivers =/d
      /^#extension_drivers =/aextension_drivers = port_security
      /^overlay_ip_version =/d
      /^#overlay_ip_version =/aoverlay_ip_version = 4
    }' \
    -e '/^\[ml2_type_flat]/,/^\[/ {
      /^flat_networks =/d
      /^#flat_networks =/aflat_networks = provider,mgmt
    }' \
    -e '/^\[ovn]/,/^\[/ {
      /^ovn_nb_connection =/d
      /^#ovn_nb_connection =/aovn_nb_connection = tcp:10.0.0.11:6641
      /^ovn_sb_connection =/d
      /^#ovn_sb_connection =/aovn_sb_connection = tcp:10.0.0.11:6642
      /^ovn_l3_scheduler =/d
      /^#ovn_l3_scheduler =/aovn_l3_scheduler = leastloaded
      /^ovn_metadata_enabled =/d
      /^#ovn_metadata_enabled =/aovn_metadata_enabled = true
    }' \
    -e '/^\[ml2_type_geneve]/,/^\[/ {
      /^vni_ranges =/d
      /^#vni_ranges =/avni_ranges = 300:399
      /^max_header_size =/d
      /^#max_header_size =/amax_header_size = 38
    }' \
    -e '/^\[securitygrou]/,/^\[/ {
      /^enable_security_group =/d
      /^#enable_security_group =/aenable_security_group = true
    }' \
    -i /etc/neutron/plugins/ml2/ml2_conf.ini
```
