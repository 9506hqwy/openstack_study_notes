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

```text
+--------------------------------------+--------------------+-----------------------+-------------------+-------+-------+---------------------------+
| ID                                   | Agent Type         | Host                  | Availability Zone | Alive | State | Binary                    |
+--------------------------------------+--------------------+-----------------------+-------------------+-------+-------+---------------------------+
| 1fbd5909-3c41-4356-a042-24dc5befc8a0 | DHCP agent         | controller.home.local | nova              | :-)   | UP    | neutron-dhcp-agent        |
| 2a57065a-d890-450f-bae0-2f456898d009 | Open vSwitch agent | compute.home.local    | None              | :-)   | UP    | neutron-openvswitch-agent |
| 5368ea0a-53d6-4f3c-946b-e01467f08929 | Metadata agent     | controller.home.local | None              | :-)   | UP    | neutron-metadata-agent    |
| 73bea1cf-d980-4aaf-8e9f-9d4781342c86 | L3 agent           | controller.home.local | nova              | :-)   | UP    | neutron-l3-agent          |
| ec88144f-cccb-48f3-a94d-4899f4cf0743 | Open vSwitch agent | controller.home.local | None              | :-)   | UP    | neutron-openvswitch-agent |
+--------------------------------------+--------------------+-----------------------+-------------------+-------+-------+---------------------------+
```
