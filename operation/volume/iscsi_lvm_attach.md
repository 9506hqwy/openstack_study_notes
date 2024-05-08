# ボリュームのアタッチ (iSCSI/LVM)

## インスタンスにアタッチ

インスタンス instance00 にボリューム volume1 をアタッチする。

```sh
openstack server add volume instance00 volume1
```

```
+-----------------------+--------------------------------------+
| Field                 | Value                                |
+-----------------------+--------------------------------------+
| ID                    | dc770907-5aec-4d8f-922d-060051f342cf |
| Server ID             | 5aa817f3-b106-42b0-95d6-f2a8d1b6bea0 |
| Volume ID             | dc770907-5aec-4d8f-922d-060051f342cf |
| Device                | /dev/vdb                             |
| Tag                   | None                                 |
| Delete On Termination | False                                |
+-----------------------+--------------------------------------+
```

## アタッチの確認

アタッチを確認する。


```sh
openstack volume attachment list
```

```
+--------------------------------------+--------------------------------------+--------------------------------------+----------+
| ID                                   | Volume ID                            | Server ID                            | Status   |
+--------------------------------------+--------------------------------------+--------------------------------------+----------+
| e41840f0-8ca4-4a7c-8a5d-41c6de48a49c | dc770907-5aec-4d8f-922d-060051f342cf | 5aa817f3-b106-42b0-95d6-f2a8d1b6bea0 | attached |
+--------------------------------------+--------------------------------------+--------------------------------------+----------+
```

```sh
openstack volume attachment show e41840f0-8ca4-4a7c-8a5d-41c6de48a49c
```

```
+-------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field       | Value                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
+-------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| ID          | e41840f0-8ca4-4a7c-8a5d-41c6de48a49c                                                                                                                                                                                                                                                                                                                                                                                                                              |
| Volume ID   | dc770907-5aec-4d8f-922d-060051f342cf                                                                                                                                                                                                                                                                                                                                                                                                                              |
| Instance ID | 5aa817f3-b106-42b0-95d6-f2a8d1b6bea0                                                                                                                                                                                                                                                                                                                                                                                                                              |
| Status      | attached                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| Attach Mode | rw                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| Attached At | 2024-04-29T02:10:27.000000                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| Detached At |                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| Properties  | access_mode='rw', attachment_id='e41840f0-8ca4-4a7c-8a5d-41c6de48a49c', auth_method='CHAP', auth_password='V7fuDgMS33tJtfqD', auth_username='c36SLcG5x7WKBJMGqjnr', cacheable='False', driver_volume_type='iscsi', encrypted='False', qos_specs=, target_discovered='False', target_iqn='iqn.2010-10.org.openstack:volume-dc770907-5aec-4d8f-922d-060051f342cf', target_lun='0', target_portal='10.0.0.11:3260', volume_id='dc770907-5aec-4d8f-922d-060051f342cf' |
+-------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```

## iSCSI ターゲットの確認

Controller Node で iSCSI ターゲットを確認する。

```sh
targetcli ls
```

```
o- / ......................................................................................................................... [...]
  o- backstores .............................................................................................................. [...]
  | o- block .................................................................................................. [Storage Objects: 1]
  | | o- iqn.2010-10.org.openstack:volume-dc770907-5aec-4d8f-922d-060051f342cf  [/dev/cinder-volumes/volume-dc770907-5aec-4d8f-922d-060051f342cf (1.0GiB) write-thru activated]
  | |   o- alua ................................................................................................... [ALUA Groups: 1]
  | |     o- default_tg_pt_gp ....................................................................... [ALUA state: Active/optimized]
  | o- fileio ................................................................................................. [Storage Objects: 0]
  | o- pscsi .................................................................................................. [Storage Objects: 0]
  | o- ramdisk ................................................................................................ [Storage Objects: 0]
  o- iscsi ............................................................................................................ [Targets: 1]
  | o- iqn.2010-10.org.openstack:volume-dc770907-5aec-4d8f-922d-060051f342cf ............................................. [TPGs: 1]
  |   o- tpg1 .......................................................................................... [no-gen-acls, auth per-acl]
  |     o- acls .......................................................................................................... [ACLs: 1]
  |     | o- iqn.1994-05.com.redhat:db5b82fc6c31 ...................................................... [1-way auth, Mapped LUNs: 1]
  |     |   o- mapped_lun0 ................. [lun0 block/iqn.2010-10.org.openstack:volume-dc770907-5aec-4d8f-922d-060051f342cf (rw)]
  |     o- luns .......................................................................................................... [LUNs: 1]
  |     | o- lun0  [block/iqn.2010-10.org.openstack:volume-dc770907-5aec-4d8f-922d-060051f342cf (/dev/cinder-volumes/volume-dc770907-5aec-4d8f-922d-060051f342cf) (default_tg_pt_gp)]
  |     o- portals .................................................................................................... [Portals: 1]
  |       o- 10.0.0.11:3260 ................................................................................................... [OK]
  o- loopback ......................................................................................................... [Targets: 0]
```

iSCSI ACL を確認する。

```sh
targetcli /iscsi/iqn.2010-10.org.openstack:volume-dc770907-5aec-4d8f-922d-060051f342cf/tpg1/acls/iqn.1994-05.com.redhat:db5b82fc6c31 info
```

```
chap_password: V7fuDgMS33tJtfqD
chap_userid: c36SLcG5x7WKBJMGqjnr
wwns:
iqn.1994-05.com.redhat:db5b82fc6c31
```

LUN を確認する。

```sh
targetcli /iscsi/iqn.2010-10.org.openstack:volume-dc770907-5aec-4d8f-922d-060051f342cf/tpg1/luns/lun0 info
```

```
alias: 64329a4e73
alua_tg_pt_gp_name: default_tg_pt_gp
index: 0
storage_object: /backstores/block/iqn.2010-10.org.openstack:volume-dc770907-5aec-4d8f-922d-060051f342cf
```

iSCSI ポータルを確認する。

```sh
targetcli /iscsi/iqn.2010-10.org.openstack:volume-dc770907-5aec-4d8f-922d-060051f342cf/tpg1/portals/10.0.0.11:3260 info
```

```
ip_address: 10.0.0.11
iser: False
offload: False
port: 3260
```

## iSCSI イニシエータの確認

Compute Node で iSCSI イニシエータを確認する。

```sh
iscsiadm --mode node
```

```
10.0.0.11:3260,4294967295 iqn.2010-10.org.openstack:volume-dc770907-5aec-4d8f-922d-060051f342cf
```

デバイスを確認する。

```sh
lsblk -S
```

```
NAME HCTL       TYPE VENDOR   MODEL             REV TRAN
sdb  4:0:0:0    disk LIO-ORG  IBLOCK           4.0  iscsi
```

デバイスの情報を確認する。

```sh
udevadm info /dev/sdb
```

```
P: /devices/platform/host4/session1/target4:0:0/4:0:0:0/block/sdb
N: sdb
S: disk/by-id/scsi-1LIO-ORG_IBLOCK:5b9485a9-d431-4afc-92a5-e3b1222dbae9
S: disk/by-id/scsi-360014055b9485a9d4314afc92a5e3b12
S: disk/by-id/scsi-SLIO-ORG_IBLOCK_5b9485a9-d431-4afc-92a5-e3b1222dbae9
S: disk/by-id/wwn-0x60014055b9485a9d4314afc92a5e3b12
S: disk/by-path/ip-10.0.0.11:3260-iscsi-iqn.2010-10.org.openstack:volume-dc770907-5aec-4d8f-922d-060051f342cf-lun-0
E: DEVLINKS=/dev/disk/by-id/scsi-360014055b9485a9d4314afc92a5e3b12 /dev/disk/by-id/scsi-SLIO-ORG_IBLOCK_5b9485a9-d431-4afc-92a5-e3b1222dbae9 /dev/disk/by-id/wwn-0x60014055b9485a9d4314afc92a5e3b12 /dev/disk/by-id/scsi-1LIO-ORG_IBLOCK:5b9485a9-d431-4afc-92a5-e3b1222dbae9 /dev/disk/by-path/ip-10.0.0.11:3260-iscsi-iqn.2010-10.org.openstack:volume-dc770907-5aec-4d8f-922d-060051f342cf-lun-0
E: DEVNAME=/dev/sdb
E: DEVPATH=/devices/platform/host4/session1/target4:0:0/4:0:0:0/block/sdb
E: DEVTYPE=disk
E: ID_BUS=scsi
E: ID_MODEL=IBLOCK
E: ID_MODEL_ENC=IBLOCK\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
E: ID_PATH=ip-10.0.0.11:3260-iscsi-iqn.2010-10.org.openstack:volume-dc770907-5aec-4d8f-922d-060051f342cf-lun-0
E: ID_PATH_TAG=ip-10_0_0_11_3260-iscsi-iqn_2010-10_org_openstack_volume-dc770907-5aec-4d8f-922d-060051f342cf-lun-0
E: ID_REVISION=4.0
E: ID_SCSI=1
E: ID_SCSI_INQUIRY=1
E: ID_SCSI_SERIAL=5b9485a9-d431-4afc-92a5-e3b1222dbae9
E: ID_SERIAL=360014055b9485a9d4314afc92a5e3b12
E: ID_SERIAL_SHORT=60014055b9485a9d4314afc92a5e3b12
E: ID_TARGET_PORT=0
E: ID_TYPE=disk
E: ID_VENDOR=LIO-ORG
E: ID_VENDOR_ENC=LIO-ORG\x20
E: ID_WWN=0x60014055b9485a9d
E: ID_WWN_VENDOR_EXTENSION=0x4314afc92a5e3b12
E: ID_WWN_WITH_EXTENSION=0x60014055b9485a9d4314afc92a5e3b12
E: MAJOR=8
E: MINOR=16
E: SCSI_IDENT_LUN_LOGICAL_UNIT_GROUP=0x0
E: SCSI_IDENT_LUN_NAA_REGEXT=60014055b9485a9d4314afc92a5e3b12
E: SCSI_IDENT_LUN_T10=LIO-ORG_IBLOCK:5b9485a9-d431-4afc-92a5-e3b1222dbae9
E: SCSI_IDENT_PORT_NAME=iqn.2010-10.org.openstack:volume-dc770907-5aec-4d8f-922d-060051f342cf,t,0x0001
E: SCSI_IDENT_PORT_RELATIVE=1
E: SCSI_IDENT_PORT_TARGET_PORT_GROUP=0x0
E: SCSI_IDENT_SERIAL=5b9485a9-d431-4afc-92a5-e3b1222dbae9
E: SCSI_IDENT_TARGET_NAME=iqn.2010-10.org.openstack:volume-dc770907-5aec-4d8f-922d-060051f342cf
E: SCSI_MODEL=IBLOCK
E: SCSI_MODEL_ENC=IBLOCK\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
E: SCSI_REVISION=4.0
E: SCSI_TPGS=3
E: SCSI_TYPE=disk
E: SCSI_VENDOR=LIO-ORG
E: SCSI_VENDOR_ENC=LIO-ORG\x20
E: SUBSYSTEM=block
E: TAGS=:systemd:
E: USEC_INITIALIZED=12945609200
```

## インスタンスの確認

ディスクを確認する。

```sh
virsh dumpxml instance-00000026 | sed -n -e '/<disk/,/<\/disk>/{ p }'
```

```xml
<disk type='file' device='disk'>
  <driver name='qemu' type='qcow2' cache='none'/>
  <source file='/var/lib/nova/instances/5aa817f3-b106-42b0-95d6-f2a8d1b6bea0/disk'/>
  <target dev='vda' bus='virtio'/>
  <address type='pci' domain='0x0000' bus='0x00' slot='0x04' function='0x0'/>
</disk>
<disk type='block' device='disk'>
  <driver name='qemu' type='raw' cache='none' io='native'/>
  <source dev='/dev/sdb'/>
  <target dev='vdb' bus='virtio'/>
  <serial>dc770907-5aec-4d8f-922d-060051f342cf</serial>
  <address type='pci' domain='0x0000' bus='0x00' slot='0x07' function='0x0'/>
</disk>
```

インスタンスを起動して動作確認する。

```sh
ssh -i demo_rsa cirros@172.17.0.153 /usr/bin/sudo /sbin/fdisk -l /dev/vdb
```

```
Disk /dev/vdb: 1 GiB, 1073741824 bytes, 2097152 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

