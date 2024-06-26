# Open vSwitch の設定

```{warning}
unix socket を使用するとエラーが発生する。

Unable to open stream to unix:/var/run/openvswitch/db.sock to retrieve schema: Permission denied
```

ovsdb に外部から接続するため起動時のオプションを設定する。

```sh
sed \
    -e s/^OPTIONS=\"\"/OPTIONS=\"--ovsdb-server-options='--remote=ptcp:6640:0.0.0.0'\"/ \
    -i /etc/sysconfig/openvswitch
```

Open vSwitch を起動する。

```sh
systemctl start openvswitch
systemctl start ovn-northd
```

Northbound データベースおよび Southbound データベースに外部から接続するため待ち受ける。

```sh
ovn-nbctl set-connection ptcp:6641:0.0.0.0 -- set connection . inactivity_probe=60000
ovn-sbctl set-connection ptcp:6642:0.0.0.0 -- set connection . inactivity_probe=60000
```

ファイアウォールを開ける。

```sh
firewall-cmd --permanent --zone=internal --add-port=6640/tcp
firewall-cmd --permanent --zone=internal --add-port=6641/tcp
firewall-cmd --permanent --zone=internal --add-port=6642/tcp
firewall-cmd --reload
```

## Gateway Chassis の設定

Gateway Node として Controller Node を使用する。

Southbound データベースに接続する。

```sh
ovs-vsctl set open . external-ids:ovn-remote=tcp:127.0.0.1:6642
```

トンネルプロトコルに geneve を指定する。

```sh
ovs-vsctl set open . external-ids:ovn-encap-type=geneve
```

geneve プロトコル用のファイアウォールを開ける。

```sh
firewall-cmd --permanent --zone=public --add-port=6081/udp
firewall-cmd --reload
```

トンネルに使用する IP アドレスを指定する。

```sh
ovs-vsctl set open . external-ids:ovn-encap-ip=172.16.0.11
```

Controller Node を OVS のゲートウェイとして設定する。

```sh
ovs-vsctl set open . external-ids:ovn-cms-options=enable-chassis-as-gw
```
