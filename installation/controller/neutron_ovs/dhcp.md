# DHCP の設定 (Open vSwitch)

設定ファイルを更新する。

```sh
sed \
    -e '/^\[DEFAULT]/,/^\[/ {
      /^interface_driver =/d
      /^#interface_driver =/ainterface_driver = openvswitch
      /^dhcp_driver =/d
      /^#dhcp_driver =/adhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
      /^enable_isolated_metadata =/d
      /^#enable_isolated_metadata =/aenable_isolated_metadata = true
    }' \
    -i /etc/neutron/dhcp_agent.ini
```

```{warning}
継続
```

DHCP エージェント がインスタンスの DHCP Discover を受信できないので
既定の firewalld ゾーンを trusted に設定する。

```sh
firewall-cmd --set-default-zone trusted
firewall-cmd --permanent --zone=public --change-interface=eth0
firewall-cmd --reload
```
