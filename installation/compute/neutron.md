# OpenStack Networking (Neutron)

## インストール

Linux Bridge ML2 プラグインをインストールする。

```sh
dnf install -y \
    openstack-neutron-linuxbridge \
    ebtables \
    ipset \
    conntrack-tools
```

## Neutron の設定

設定ファイルを更新する。

```sh
sed \
    -e '/^\[DEFAULT]/,/^\[/ {
      /^state_path=/d
      /^#state_path=/astate_path=/var/lib/neutron
      /^transport_url =/d
      /^#transport_url =/atransport_url = rabbit://openstack:3c17215fe69ba1dad320@controller:5672/
      /^auth_strategy =/d
      /^#auth_strategy =/aauth_strategy = keystone
    }' \
    -e '/^\[keystone_authtoken]/,/^\[/ {
      /^www_authenticate_uri =/d
      /^#www_authenticate_uri =/awww_authenticate_uri = http://controller:5000
      /^memcached_servers =/d
      /^#memcached_servers =/amemcached_servers = controller:11211
      /^auth_type =/d
      /^#auth_type =/aauth_type = password
      /^auth_url =/d
      /^\[keystone_authtoken]/aauth_url = http://controller:5000
      /^project_domain_name =/d
      /^\[keystone_authtoken]/aproject_domain_name = Default
      /^project_name =/d
      /^\[keystone_authtoken]/aproject_name = service
      /^user_domain_name =/d
      /^\[keystone_authtoken]/auser_domain_name = Default
      /^username =/d
      /^\[keystone_authtoken]/ausername = neutron
      /^password =/d
      /^\[keystone_authtoken]/apassword = 76283d854fd24b78f90b
    }' \
    -e '/^\[oslo_concurrency]/,/^\[/ {
      /^lock_path =/d
      /^# lock_path =/alock_path = $state_path/tmp
    }' \
    -i /etc/neutron/neutron.conf
```

## Linux Bridge の設定

設定ファイルを更新する。

```sh
sed \
    -e '/^\[linux_bridge]/,/^\[/ {
      /^physical_interface_mappings =/d
      /^#physical_interface_mappings =/aphysical_interface_mappings = provider:eth0
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

## ファイアウォールの設定

```{warning}
継続
```

インスタンスが DHCP エージェントの DHCP Offer を受信できないので
既定の firewalld ゾーンを trusted に設定する。

```sh
firewall-cmd --set-default-zone trusted
firewall-cmd --permanent --zone=public --change-interface=eth0
firewall-cmd --reload
```

## Nova の設定

設定ファイルを更新する。

```sh
sed \
    -e '/^\[DEFAULT]/,/^\[/ {
      /^transport_url =/d
      /^#transport_url =/atransport_url = rabbit://openstack:3c17215fe69ba1dad320@controller:5672/
      /^vif_plugging_is_fatal=/d
      /^#vif_plugging_is_fatal=/avif_plugging_is_fatal=true
    }' \
    -e '/^\[neutron]/,/^\[/ {
      /^auth_type=/d
      /^#auth_type=/aauth_type=password
      /^auth_url=/d
      /^#auth_url=/aauth_url=http://controller:5000
      /^project_domain_name=/d
      /^#project_domain_name=/aproject_domain_name=Default
      /^project_name=/d
      /^#project_name=/aproject_name=service
      /^user_domain_name=/d
      /^#user_domain_name=/auser_domain_name=Default
      /^username=/d
      /^#username=/ausername=neutron
      /^password=/d
      /^#password=/apassword=76283d854fd24b78f90b
      /^region_name=/d
      /^#region_name=/aregion_name=RegionOne
    }' \
    -i /etc/nova/nova.conf
```

## SELinux の設定

ポリシーを変更する。

```sh
setsebool -P os_neutron_dac_override on
```

[Bug 1850973](https://bugzilla.redhat.com/show_bug.cgi?id=1850973) を参照。

## 起動

nova を再起動する。

```sh
systemctl restart openstack-nova-compute
```

マシン起動時の自動起動設定とサービスを起動する。

```sh
systemctl enable --now neutron-linuxbridge-agent
```

## 動作確認

Controller Node でエージェントを表示する。

```sh
openstack network agent list
```

```
+--------------------------------------+--------------------+-----------------------+-------------------+-------+-------+---------------------------+
| ID                                   | Agent Type         | Host                  | Availability Zone | Alive | State | Binary                    |
+--------------------------------------+--------------------+-----------------------+-------------------+-------+-------+---------------------------+
| 145d9d73-cb1c-4bbf-ad28-adb1a9f4a826 | Metadata agent     | controller.home.local | None              | :-)   | UP    | neutron-metadata-agent    |
| d8153174-094a-4c91-9650-8bce3042bad3 | Linux bridge agent | compute.home.local    | None              | :-)   | UP    | neutron-linuxbridge-agent |
| e843c356-67a9-418e-96db-3fa6e4210df9 | DHCP agent         | controller.home.local | nova              | :-)   | UP    | neutron-dhcp-agent        |
| f075f5df-7f2f-4482-881f-5f463387be89 | Linux bridge agent | controller.home.local | None              | :-)   | UP    | neutron-linuxbridge-agent |
+--------------------------------------+--------------------+-----------------------+-------------------+-------+-------+---------------------------+
```

## VXLAN ネットワークの設定

VXLAN ネットワークを使用する場合は [](./neutron_vxlan.md) を参照して設定する。
