# Open vSwitch の設定

```{warning}
unix socket を使用するとエラーが発生する。

Unable to open stream to unix:/var/run/openvswitch/db.sock to retrieve schema: Permission denied
```

ovsdb に TCP で接続するため起動時のオプションを設定する。

```sh
sed \
    -e s/^OPTIONS=\"\"/OPTIONS=\"--ovsdb-server-options='--remote=ptcp:6640:127.0.0.1'\"/ \
    -i /etc/sysconfig/openvswitch
```

Open vSwitch を起動する。

```sh
systemctl start openvswitch
```

Controller Node の Southbound データベースに接続する。

```sh
ovs-vsctl set open . external-ids:ovn-remote=tcp:10.0.0.11:6642
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

トンネルの接続先に Controller Node を指定する。

```sh
ovs-vsctl set open . external-ids:ovn-encap-ip=172.16.0.31
```
