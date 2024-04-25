# 起動 (Linux Bridge)

nova を再起動する。

```sh
systemctl restart openstack-nova-api
```

マシン起動時の自動起動設定とサービスを起動する。

```sh
systemctl enable --now neutron-server
systemctl enable --now neutron-linuxbridge-agent
systemctl enable --now neutron-dhcp-agent
systemctl enable --now neutron-metadata-agent
```
