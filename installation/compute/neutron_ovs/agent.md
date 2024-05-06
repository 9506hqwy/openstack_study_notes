# Open vSwitch Agent の設定 (Open vSwitch)

設定ファイルを更新する。

```sh
sed \
    -e '/^\[ovs]/,/^\[/ {
      /^bridge_mappings =/d
      /^#bridge_mappings =/abridge_mappings = provider:br-provider,mgmt:br-mgmt
    }' \
    -e '/^\[securitygroup]/,/^\[/ {
      /^enable_security_group =/d
      /^#enable_security_group =/aenable_security_group = true
      /^firewall_driver =/d
      /^#firewall_driver =/afirewall_driver = openvswitch
    }' \
    -i /etc/neutron/plugins/ml2/openvswitch_agent.ini
```
