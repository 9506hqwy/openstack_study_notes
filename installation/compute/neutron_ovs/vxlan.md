# VXLAN ネットワーク (Open vSwitch)

## Open vSwitch の設定

設定ファイルに物理ネットワークのために使用する IP アドレスを指定する。

```sh
sed \
    -e '/^\[agent]/,/^\[/ {
      /^tunnel_types =/d
      /^#tunnel_types =/atunnel_types = vxlan
    }' \
    -e '/^\[ovs]/,/^\[/ {
      /^local_ip =/d
      /^#local_ip =/alocal_ip = 172.16.0.31
    }' \
    -i /etc/neutron/plugins/ml2/openvswitch_agent.ini
```

## ネットワークの設定

ファイアウィールを開ける。

```sh
firewall-cmd --permanent --zone=public --add-port=4789/udp
firewall-cmd --reload
```

## 起動

neutron を再起動する。

```sh
systemctl restart neutron-openvswitch-agent
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
| 1e13255e-0f7c-4002-8d52-f53db8839bae | L3 agent           | controller.home.local | nova              | :-)   | UP    | neutron-l3-agent          |
| 20918645-7ee8-429c-a9ff-ba942695e4a4 | Open vSwitch agent | compute.home.local    | None              | :-)   | UP    | neutron-openvswitch-agent |
| 5bb635cd-1e42-48fa-b2be-c5d0dce3fef4 | Open vSwitch agent | controller.home.local | None              | :-)   | UP    | neutron-openvswitch-agent |
| e843c356-67a9-418e-96db-3fa6e4210df9 | DHCP agent         | controller.home.local | nova              | :-)   | UP    | neutron-dhcp-agent        |
+--------------------------------------+--------------------+-----------------------+-------------------+-------+-------+---------------------------+
```
