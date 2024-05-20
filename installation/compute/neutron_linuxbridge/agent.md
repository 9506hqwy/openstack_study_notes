# Linux Bridge Agent の設定 (Linux Bridge)

設定ファイルを更新する。

```sh
sed \
    -e '/^\[linux_bridge]/,/^\[/ {
      /^physical_interface_mappings =/d
      /^#physical_interface_mappings =/aphysical_interface_mappings = provider:eth2,mgmt:eth3
    }' \
    -e '/^\[vxlan]/,/^\[/ {
      /^enable_vxlan =/d
      /^#enable_vxlan =/aenable_vxlan = false
    }' \
    -e '/^\[securitygroup]/,/^\[/ {
      /^enable_security_group =/d
      /^#enable_security_group =/aenable_security_group = true
      /^firewall_driver =/d
      /^#firewall_driver =/afirewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
    }' \
    -i /etc/neutron/plugins/ml2/linuxbridge_agent.ini
```

br_netfilter モジュールを有効化する。

```sh
cat > /etc/modules-load.d/nuetron.conf <<EOF
br_netfilter
EOF

systemctl restart systemd-modules-load
```
