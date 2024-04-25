# OpenStack Networking (Neutron)

## インストール

Linux Bridge を使用する場合は下記をインストールする。

```sh
dnf install -y \
    openstack-neutron-linuxbridge \
    ebtables \
    ipset \
    conntrack-tools
```

Open vSwitch を使用する場合は下記をインストールする。

```sh
dnf install -y openstack-neutron-openvswitch NetworkManager-ovs
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


## メカニズムドライバの設定

Linux Bridge と Open vSwitch のどちらかを設定する。

### Linux Bridge の場合

* [](./neutron_linuxbridge/agent.md)

### Open vSwitch の場合

* [](./neutron_ovs/agent.md)

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

メカニズムドライバの種類によって手順を選択する。

* [](./neutron_linuxbridge/startup.md)
* [](./neutron_ovs/startup.md)

## 動作確認

メカニズムドライバの種類によって手順を選択する。

* [](./neutron_linuxbridge/confirm.md)
* [](./neutron_ovs/confirm.md)

## VXLAN ネットワークの設定

VXLAN ネットワークを使用する場合はメカニズムドライバの種類によって手順を選択する。

* [](./neutron_linuxbridge/vxlan.md)
* [](./neutron_ovs/vxlan.md)
