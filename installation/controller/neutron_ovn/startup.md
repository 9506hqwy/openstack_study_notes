# 起動 (Open Virtual Network)

nova を再起動する。

```sh
systemctl restart NetworkManager
systemctl restart openstack-nova-api
```

Open vSwitch を再起動する。

```sh
systemctl restart openvswitch
systemctl restart ovn-northd
```

マシン起動時の自動起動設定とサービスを起動する。

```sh
systemctl enable --now neutron-server
systemctl enable --now openvswitch
systemctl enable --now ovn-northd
```
