# Neutron (VXLAN ネットワーク)

## Linux Bridge の設定

設定ファイルに物理ネットワークに使用する IP アドレスを指定する。

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

```
+--------------------------------------+--------------------+-----------------------+-------------------+-------+-------+---------------------------+
| ID                                   | Agent Type         | Host                  | Availability Zone | Alive | State | Binary                    |
+--------------------------------------+--------------------+-----------------------+-------------------+-------+-------+---------------------------+
| 145d9d73-cb1c-4bbf-ad28-adb1a9f4a826 | Metadata agent     | controller.home.local | None              | :-)   | UP    | neutron-metadata-agent    |
| d8153174-094a-4c91-9650-8bce3042bad3 | Linux bridge agent | compute.home.local    | None              | :-)   | UP    | neutron-linuxbridge-agent |
| e2dc94e8-29ad-4722-88ed-06abacce7bfd | L3 agent           | controller.home.local | nova              | :-)   | UP    | neutron-l3-agent          |
| e843c356-67a9-418e-96db-3fa6e4210df9 | DHCP agent         | controller.home.local | nova              | :-)   | UP    | neutron-dhcp-agent        |
| f075f5df-7f2f-4482-881f-5f463387be89 | Linux bridge agent | controller.home.local | None              | :-)   | UP    | neutron-linuxbridge-agent |
+--------------------------------------+--------------------+-----------------------+-------------------+-------+-------+---------------------------+
```
