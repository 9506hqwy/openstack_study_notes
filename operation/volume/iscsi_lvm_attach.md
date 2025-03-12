# ボリュームのアタッチ (iSCSI/LVM)

## インスタンスにアタッチ

```{tip}
myuser で実行
```

インスタンス instance02 にボリューム volume1 をアタッチする。

```sh
openstack server add volume instance02 volume1
```

```text
+-----------------------+--------------------------------------+
| Field                 | Value                                |
+-----------------------+--------------------------------------+
| ID                    | fb4470ac-e504-4959-939c-7f4b22fd07b4 |
| Server ID             | 2337b0eb-372c-43b8-923e-90a89337d211 |
| Volume ID             | fb4470ac-e504-4959-939c-7f4b22fd07b4 |
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

```text
+--------------------------------------+--------------------------------------+--------------------------------------+----------+
| ID                                   | Volume ID                            | Server ID                            | Status   |
+--------------------------------------+--------------------------------------+--------------------------------------+----------+
| 6bb646a5-af8b-4966-8035-fb0f1426097e | fb4470ac-e504-4959-939c-7f4b22fd07b4 | 2337b0eb-372c-43b8-923e-90a89337d211 | attached |
+--------------------------------------+--------------------------------------+--------------------------------------+----------+
```

```sh
openstack volume attachment show 6bb646a5-af8b-4966-8035-fb0f1426097e
```

```text
+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field       | Value                                                                                                                                                          |
+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------+
| ID          | 6bb646a5-af8b-4966-8035-fb0f1426097e                                                                                                                           |
| Volume ID   | fb4470ac-e504-4959-939c-7f4b22fd07b4                                                                                                                           |
| Instance ID | 2337b0eb-372c-43b8-923e-90a89337d211                                                                                                                           |
| Status      | attached                                                                                                                                                       |
| Attach Mode | rw                                                                                                                                                             |
| Attached At | 2024-05-18T05:03:59.000000                                                                                                                                     |
| Detached At |                                                                                                                                                                |
| Properties  | access_mode='rw', attachment_id='6bb646a5-af8b-4966-8035-fb0f1426097e', auth_method='CHAP', auth_password='2vCAQz2fcVeAjX9X',                                  |
|             | auth_username='5KDJu7yNCnns6np2hp9h', cacheable='False', driver_volume_type='iscsi', encrypted='False', qos_specs=, target_discovered='False',                 |
|             | target_iqn='iqn.2010-10.org.openstack:volume-fb4470ac-e504-4959-939c-7f4b22fd07b4', target_lun='0', target_portal='10.0.0.11:3260',                            |
|             | volume_id='fb4470ac-e504-4959-939c-7f4b22fd07b4'                                                                                                               |
+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------+
```

## iSCSI ターゲットの確認

Controller Node で iSCSI ターゲットを確認する。

```sh
targetcli ls
```

```text
o- / ......................................................................................................................... [...]
  o- backstores .............................................................................................................. [...]
  | o- block .................................................................................................. [Storage Objects: 1]
  | | o- iqn.2010-10.org.openstack:volume-fb4470ac-e504-4959-939c-7f4b22fd07b4  [/dev/cinder-volumes/volume-fb4470ac-e504-4959-939c-7f4b22fd07b4 (1.0GiB) write-thru activated]
  | |   o- alua ................................................................................................... [ALUA Groups: 1]
  | |     o- default_tg_pt_gp ....................................................................... [ALUA state: Active/optimized]
  | o- fileio ................................................................................................. [Storage Objects: 0]
  | o- pscsi .................................................................................................. [Storage Objects: 0]
  | o- ramdisk ................................................................................................ [Storage Objects: 0]
  o- iscsi ............................................................................................................ [Targets: 1]
  | o- iqn.2010-10.org.openstack:volume-fb4470ac-e504-4959-939c-7f4b22fd07b4 ............................................. [TPGs: 1]
  |   o- tpg1 .......................................................................................... [no-gen-acls, auth per-acl]
  |     o- acls .......................................................................................................... [ACLs: 1]
  |     | o- iqn.1994-05.com.redhat:fe91975b2a7 ....................................................... [1-way auth, Mapped LUNs: 1]
  |     |   o- mapped_lun0 ................. [lun0 block/iqn.2010-10.org.openstack:volume-fb4470ac-e504-4959-939c-7f4b22fd07b4 (rw)]
  |     o- luns .......................................................................................................... [LUNs: 1]
  |     | o- lun0  [block/iqn.2010-10.org.openstack:volume-fb4470ac-e504-4959-939c-7f4b22fd07b4 (/dev/cinder-volumes/volume-fb4470ac-e504-4959-939c-7f4b22fd07b4) (default_tg_pt_gp)]
  |     o- portals .................................................................................................... [Portals: 1]
  |       o- 10.0.0.11:3260 ................................................................................................... [OK]
  o- loopback ......................................................................................................... [Targets: 0]
```

iSCSI ACL を確認する。

```sh
targetcli /iscsi/iqn.2010-10.org.openstack:volume-fb4470ac-e504-4959-939c-7f4b22fd07b4/tpg1/acls/iqn.1994-05.com.redhat:fe91975b2a7 info
```

```text
chap_password: 2vCAQz2fcVeAjX9X
chap_userid: 5KDJu7yNCnns6np2hp9h
wwns:
iqn.1994-05.com.redhat:fe91975b2a7
```

LUN を確認する。

```sh
targetcli /iscsi/iqn.2010-10.org.openstack:volume-fb4470ac-e504-4959-939c-7f4b22fd07b4/tpg1/luns/lun0 info
```

```text
alias: 67e53965ff
alua_tg_pt_gp_name: default_tg_pt_gp
index: 0
storage_object: /backstores/block/iqn.2010-10.org.openstack:volume-fb4470ac-e504-4959-939c-7f4b22fd07b4
```

iSCSI ポータルを確認する。

```sh
targetcli /iscsi/iqn.2010-10.org.openstack:volume-fb4470ac-e504-4959-939c-7f4b22fd07b4/tpg1/portals/10.0.0.11:3260 info
```

```text
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

```text
10.0.0.11:3260,4294967295 iqn.2010-10.org.openstack:volume-fb4470ac-e504-4959-939c-7f4b22fd07b4
```

デバイスを確認する。

```sh
lsblk -S
```

```text
NAME HCTL       TYPE VENDOR   MODEL         REV SERIAL                               TRAN
sda  0:0:0:0    disk Msft     Virtual Disk 1.0  60022480b76b290be6de03390c018a08
sdb  4:0:0:0    disk LIO-ORG  IBLOCK       4.0  378fc26a-fa7b-4f4d-ac6a-a0d23e7ff16a iscsi
sr0  3:0:0:0    rom  Msft     Virtual CD   1.0  Virtual_CD                           ata
```

デバイスの情報を確認する。

```sh
udevadm info /dev/sdb
```

```text
P: /devices/platform/host4/session1/target4:0:0/4:0:0:0/block/sdb
M: sdb
U: block
T: disk
D: b 8:16
N: sdb
L: 0
S: disk/by-path/ip-10.0.0.11:3260-iscsi-iqn.2010-10.org.openstack:volume-fb4470ac-e504-4959-939c-7f4b22fd07b4-lun-0
S: disk/by-id/scsi-SLIO-ORG_IBLOCK_378fc26a-fa7b-4f4d-ac6a-a0d23e7ff16a
S: disk/by-id/wwn-0x6001405378fc26afa7b4f4dac6aa0d23
S: disk/by-diskseq/6
S: disk/by-id/scsi-36001405378fc26afa7b4f4dac6aa0d23
S: disk/by-id/scsi-1LIO-ORG_IBLOCK:378fc26a-fa7b-4f4d-ac6a-a0d23e7ff16a
Q: 6
E: DEVPATH=/devices/platform/host4/session1/target4:0:0/4:0:0:0/block/sdb
E: DEVNAME=/dev/sdb
E: DEVTYPE=disk
E: DISKSEQ=6
E: MAJOR=8
E: MINOR=16
E: SUBSYSTEM=block
E: USEC_INITIALIZED=21167437869
E: ID_SCSI=1
E: ID_VENDOR=LIO-ORG
E: ID_VENDOR_ENC=LIO-ORG\x20
E: ID_MODEL=IBLOCK
E: ID_MODEL_ENC=IBLOCK\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
E: ID_REVISION=4.0
E: ID_TYPE=disk
E: ID_SERIAL=36001405378fc26afa7b4f4dac6aa0d23
E: ID_SERIAL_SHORT=6001405378fc26afa7b4f4dac6aa0d23
E: ID_WWN=0x6001405378fc26af
E: ID_WWN_VENDOR_EXTENSION=0xa7b4f4dac6aa0d23
E: ID_WWN_WITH_EXTENSION=0x6001405378fc26afa7b4f4dac6aa0d23
E: ID_TARGET_PORT=0
E: ID_SCSI_SERIAL=378fc26a-fa7b-4f4d-ac6a-a0d23e7ff16a
E: ID_BUS=scsi
E: ID_PATH=ip-10.0.0.11:3260-iscsi-iqn.2010-10.org.openstack:volume-fb4470ac-e504-4959-939c-7f4b22fd07b4-lun-0
E: ID_PATH_TAG=ip-10_0_0_11_3260-iscsi-iqn_2010-10_org_openstack_volume-fb4470ac-e504-4959-939c-7f4b22fd07b4-lun-0
E: SCSI_TPGS=3
E: SCSI_TYPE=disk
E: SCSI_VENDOR=LIO-ORG
E: SCSI_VENDOR_ENC=LIO-ORG\x20
E: SCSI_MODEL=IBLOCK
E: SCSI_MODEL_ENC=IBLOCK\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
E: SCSI_REVISION=4.0
E: ID_SCSI_INQUIRY=1
E: SCSI_IDENT_SERIAL=378fc26a-fa7b-4f4d-ac6a-a0d23e7ff16a
E: SCSI_IDENT_LUN_NAA_REGEXT=6001405378fc26afa7b4f4dac6aa0d23
E: SCSI_IDENT_LUN_T10=LIO-ORG_IBLOCK:378fc26a-fa7b-4f4d-ac6a-a0d23e7ff16a
E: SCSI_IDENT_PORT_RELATIVE=1
E: SCSI_IDENT_PORT_TARGET_PORT_GROUP=0x0
E: SCSI_IDENT_LUN_LOGICAL_UNIT_GROUP=0x0
E: SCSI_IDENT_PORT_NAME=iqn.2010-10.org.openstack:volume-fb4470ac-e504-4959-939c-7f4b22fd07b4,t,0x0001
E: SCSI_IDENT_TARGET_NAME=iqn.2010-10.org.openstack:volume-fb4470ac-e504-4959-939c-7f4b22fd07b4
E: DEVLINKS=/dev/disk/by-path/ip-10.0.0.11:3260-iscsi-iqn.2010-10.org.openstack:volume-fb4470ac-e504-4959-939c-7f4b22fd07b4-lun-0 /dev/disk/by-id/scsi-SLIO-ORG_IBLOCK_378fc2>
E: TAGS=:systemd:
E: CURRENT_TAGS=:systemd:
```

## インスタンスの確認

ディスクを確認する。

```sh
virsh dumpxml instance-00000006 | sed -n -e '/<disk/,/<\/disk>/{ p }'
```

```xml
<disk type='file' device='disk'>
  <driver name='qemu' type='qcow2' cache='none'/>
  <source file='/var/lib/nova/instances/2337b0eb-372c-43b8-923e-90a89337d211/disk'/>
  <target dev='vda' bus='virtio'/>
  <address type='pci' domain='0x0000' bus='0x00' slot='0x04' function='0x0'/>
</disk>
<disk type='block' device='disk'>
  <driver name='qemu' type='raw' cache='none' io='native'/>
  <source dev='/dev/sdb'/>
  <target dev='vdb' bus='virtio'/>
  <serial>fb4470ac-e504-4959-939c-7f4b22fd07b4</serial>
  <alias name='ua-fb4470ac-e504-4959-939c-7f4b22fd07b4'/>
  <address type='pci' domain='0x0000' bus='0x00' slot='0x07' function='0x0'/>
</disk>
```

インスタンスを起動して動作確認する。

```sh
ssh -i demo_rsa cirros@172.16.0.175 /usr/bin/sudo /sbin/fdisk -l /dev/vdb
```

```text
Disk /dev/vdb: 1 GiB, 1073741824 bytes, 2097152 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```
