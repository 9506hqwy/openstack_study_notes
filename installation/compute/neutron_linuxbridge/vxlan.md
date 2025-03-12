# VXLAN ネットワーク (Linux Bridge)

## Linux Bridge の設定

設定ファイルに物理ネットワークのために使用する IP アドレスを指定する。

```sh
sed \
    -e '/^\[vxlan]/,/^\[/ {
      /^enable_vxlan =/d
      /^#enable_vxlan =/aenable_vxlan = true
      /^local_ip =/d
      /^#local_ip =/alocal_ip = 172.16.0.31
      /^l2_population =/d
      /^#l2_population =/al2_population = true
    }' \
    -i /etc/neutron/plugins/ml2/linuxbridge_agent.ini
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
systemctl restart neutron-linuxbridge-agent
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
| 0914ecb5-a702-417d-ab43-aa27935a9267 | DHCP agent         | controller.home.local | nova              | :-)   | UP    | neutron-dhcp-agent        |
| 259b7b21-9fbd-4b67-b703-6e155038e9b7 | Linux bridge agent | controller.home.local | None              | :-)   | UP    | neutron-linuxbridge-agent |
| 69317793-1277-4b32-8fdf-7b0336d09a32 | L3 agent           | controller.home.local | nova              | :-)   | UP    | neutron-l3-agent          |
| 74b907db-30c1-4d6a-a7d9-5e9d080e583c | Metadata agent     | controller.home.local | None              | :-)   | UP    | neutron-metadata-agent    |
| 899f52f1-5c0f-43da-b330-da74402c035b | Linux bridge agent | compute.home.local    | None              | :-)   | UP    | neutron-linuxbridge-agent |
+--------------------------------------+--------------------+-----------------------+-------------------+-------+-------+---------------------------+
```
