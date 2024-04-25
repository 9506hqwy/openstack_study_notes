# 起動 (Linux Bridge)

nova を再起動する。

```sh
systemctl restart openstack-nova-compute
```

マシン起動時の自動起動設定とサービスを起動する。

```sh
systemctl enable --now neutron-linuxbridge-agent
```
