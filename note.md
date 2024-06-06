# 注記

## CPU モードをサポートしていない

x86_64 なら host-model, aarch64 なら host-passthrough をサポートしていない場合は
インスタンス作成でエラーが発生する。

Compute Node で下記のコマンドでサポートしている CPU モードを確認する。

```sh
virsh domcapabilities | sed -n -e '/<cpu>/,/<\/cpu>/ { p }'
```

*/etc/nova/nova.conf* の cpu_mode および cpu_models を指定する。

サービスを再起動する。

```sh
systemctl restart openstack-nova-compute
```

参考: [libvirt: use 'host-passthrough' as default on AArch64](https://opendev.org/openstack/nova/commit/8bc7b950b7c0a3c80cdd120fe4df97c14848c344)

## ゲストOSが起動しない

aarch64 アーキテクチャで QEMU を使用してインスタンスを作成すると下記のエラーでゲスト OS が起動しない。

```
Guest has not initialized the display (yet).
```

## SELinux の設定

SELinux で拒否される操作は */var/log/audit/audit.log* から denied されたログを確認する。

```
type=AVC msg=audit(1713051947.886:721): avc:  denied  { setattr } for  pid=9593 comm="dnsmasq" name="dhcp_dns_log" dev="dm-0" ino=70321107 scontext=system_u:system_r:dnsmasq_t:s0 tcontext=system_u:object_r:neutron_log_t:s0 tclass=file permissive=1
```

操作を許可する。

```sh
ausearch -c dnsmasq | audit2allow
```

参考: [SELinux の使用](https://access.redhat.com/documentation/ja-jp/red_hat_enterprise_linux/8/html-single/using_selinux/index#fixing-an-analyzed-selinux-denial_troubleshooting-problems-related-to-selinux)

## vxlan ネットワークでインスタンスに IP アドレスが割り当たらない

Linux Bridge を使用した場合 DHCP offer をインスタンスが受信しないため、
Controller Node / Compute Node でファイアウォールを開ける。

```sh
firewall-cmd --permanent --zone=public --add-service=dhcp
firewall-cmd --permanent --zone=public --add-port=68/udp
firewall-cmd --reload
```

これでも疎通ができない場合はファイアウォールを無効にする。

```sh
systemctl stop firewalld
```

## ironic staging driver の修正

[_set_boot_device](https://github.com/openstack-archive/ironic-staging-drivers/blob/stable/ocata/ironic_staging_drivers/libvirt/power.py#L320)
で型エラーが発生するためソースコードを修正する。

引数に `encoding` を追加する。

```python
tostring(..., encoding='unicode')
```

## nova-compute が起動しない

Compute Node のホスト名を変更すると下記のエラーで openstack-nova-compute サービスが起動しない。

```
nova.exception.InvalidConfiguration: My hypervisor has existing instances, but I appear to be a new service in this database. Possible database configuration error, refusing to start!
```

```
nova.exception.InvalidConfiguration: My compute node 6c53c76c-39d1-45dc-a5f5-293e91abd3e4 has hypervisor_hostname compute.home.local but virt driver reports it should be localhost.localdomain. Possible rename detected, refusing to start!
```
