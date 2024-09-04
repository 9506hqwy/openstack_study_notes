# Libvirt

## インストール

libvirt をインストールする。

```sh
dnf install -y \
    libvirt \
    virt-install \
    cyrus-sasl-md5 \
    cyrus-sasl-scram \
    cockpit \
    cockpit-machines
```

## 設定

TCP で接続するため設定ファイルを更新する。

```sh
sed \
    -e '/^listen_tls =/d' \
    -e '/^#listen_tls =/alisten_tls = 0' \
    -e '/^listen_tcp =/d' \
    -e '/^#listen_tcp =/alisten_tcp = 1' \
    -i /etc/libvirt/libvirtd.conf
```

SASL のメカニズムを digest-md5 に変更する。

```{note}
scram-sha-256 では認証が成功しなかった。
```

```sh
sed \
    -e 's/^mech_list: gssapi$/#mech_list: gssapi/' \
    -e 's/^#mech_list: scram-sha-256$/mech_list: digest-md5/' \
    -e 's|^#sasldb_path: /etc/libvirt/passwd.db$|sasldb_path: /etc/libvirt/passwd.db|' \
    -i /etc/sasl2/libvirt.conf
```

ファイアウォールを開ける。

```sh
firewall-cmd --permanent --zone=internal --add-service=cockpit
firewall-cmd --permanent --zone=internal --add-service=libvirt
firewall-cmd --permanent --zone=internal --add-service=vnc-server
firewall-cmd --reload
```

## 起動

マシン起動時の自動起動設定とサービスを起動する。

```sh
systemctl enable --now libvirtd-tcp.socket
systemctl enable --now cockpit.socket
```

## ユーザの登録

SASL のユーザを登録する。

```sh
saslpasswd2 -a libvirt openstack
```

```
Password:
Again (for verification):
```

ユーザの登録を確認する。

```sh
sasldblistusers2 -f /etc/libvirt/passwd.db
```

```
openstack@baremetal.home.local: userPassword
```

接続を確認する。

```sh
virsh -c qemu+tcp://127.0.0.1/system list
```

```
Please enter your authentication name: openstack
Please enter your password:
 Id   名前   状態
-------------------
```

## 仮想ブリッジの作成

eth0, eth1 にそれぞれ仮想ブリッジを作成する。

```sh
nmcli connection down eth0
nmcli connection delete eth0
nmcli connection add \
    type bridge \
    con-name br0 \
    ifname br0 \
    ipv6.method disabled \
    ipv4.method manual \
    ipv4.addresses 172.16.0.51/24 \
    ipv4.gateway 172.16.0.254 \
    connection.autoconnect yes
nmcli con add \
    type ethernet \
    con-name eth0 \
    ifname eth0 \
    master br0
nmcli connection up br0

nmcli connection down eth1
nmcli connection delete eth1
nmcli connection add \
    type bridge \
    con-name br1 \
    ifname br1 \
    ipv6.method disabled \
    ipv4.method manual \
    ipv4.dns 10.0.0.254 \
    ipv4.dns-search home.local \
    ipv4.addresses 10.0.0.51/24 \
    connection.autoconnect yes
nmcli con add \
    type ethernet \
    con-name eth1 \
    ifname eth1 \
    master br1
nmcli connection up br1

firewall-cmd --permanent --zone=public --change-interface=eth0
firewall-cmd --permanent --zone=internal --change-interface=eth1
firewall-cmd --reload
```

## ストレージプールの作成

プール default を作成する。

```sh
virsh pool-define-as default dir --target /var/lib/libvirt/images
virsh pool-start default
virsh pool-autostart default
```

## 仮想ベアメタルノードの作成

仮想ベアメタルノードとして使用するインスタンスを作成する。

`--os-variant` の値は `osinfo-query os` コマンドの実行結果から取得する。

### PXE

```sh
virsh vol-create-as default node01.qcow2 10GiB --format qcow2
virt-install \
    --name node01 \
    --vcpu 1 \
    --memory 1024 \
    --os-variant cirros0.5.2 \
    --disk /var/lib/libvirt/images/node01.qcow2,bus=virtio \
    --network bridge=br1,model=virtio \
    --graphics vnc,listen=0.0.0.0 \
    --virt-type qemu \
    --pxe \
    --noautoconsole
```

起動を確認した後に停止する。

```sh
virsh destroy 1
```

### ISO

```sh
virsh vol-create-as default node01.qcow2 16GiB --format qcow2
virt-install \
    --name node01 \
    --vcpu 2 \
    --cpu host-passthrough \
    --memory 4096 \
    --os-variant centos-stream9 \
    --disk /var/lib/libvirt/images/node01.qcow2,bus=virtio \
    --network network=default,model=virtio \
    --graphics vnc,listen=0.0.0.0 \
    --virt-type kvm \
    --cdrom /mnt/CentOS-Stream-9-latest-aarch64-dvd1.iso \
    --noautoconsole
```

コンソールでインストールした後に停止する。

```sh
virsh destroy 1
```

### kickstart

```sh
virsh vol-create-as default node01.qcow2 16GiB --format qcow2
virt-install \
    --name node01 \
    --vcpu 2 \
    --cpu host-passthrough \
    --memory 4096 \
    --os-variant centos-stream9 \
    --disk /var/lib/libvirt/images/node01.qcow2,bus=virtio \
    --network network=default,model=virtio \
    --graphics vnc,listen=0.0.0.0 \
    --virt-type kvm \
    --location /mnt/CentOS-Stream-9-latest-aarch64-dvd1.iso \
    --initrd-inject /root/ks.cfg \
    --extra-args="inst.ks=file:/ks.cfg" \
    --noautoconsole
```

インストールが完了した後に停止する。

```sh
virsh destroy 1
```

*/root/ks.cfg* は以下を用意する。
ファイルの内容については [パート IV. キックスタートの参照](https://docs.redhat.com/ja/documentation/red_hat_enterprise_linux/9/html/performing_an_advanced_rhel_9_installation/kickstart_references) を参照する。

```
# Use CDROM installation media
cdrom

# Use text installation
text

# Select install disk
ignoredisk --only-use=vda

# Do not use X
skipx

# System language
lang ja_JP.UTF-8

# Keyboard layouts
keyboard --xlayouts='jp'

# System timezone
timezone Asia/Tokyo --utc

# Network
network --hostname=localhost.localdomain
network --bootproto=dhcp --device=enp1s0 --activate

# Partition layout
clearpart --drives=vda --all
zerombr
autopart --type=plain --fstype=xfs

# Root password
rootpw --plaintext --allow-ssh centos9

# Reboot after installation
reboot

# Enable kdump
%addon com_redhat_kdump --enable --reserve-mb='auto'
%end

# Install packages
%packages
@^minimal-environment
qemu-guest-agent
%end
```
