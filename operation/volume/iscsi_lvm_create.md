# ボリュームの作成 (iSCSI/LVM)

## ボリュームの作成

容量 1 GiB のボリュームを作成する。

```sh
openstack volume create --size 1 volume1
```

```
+---------------------+--------------------------------------+
| Field               | Value                                |
+---------------------+--------------------------------------+
| attachments         | []                                   |
| availability_zone   | nova                                 |
| bootable            | false                                |
| consistencygroup_id | None                                 |
| created_at          | 2024-04-29T00:53:49.791949           |
| description         | None                                 |
| encrypted           | False                                |
| id                  | dc770907-5aec-4d8f-922d-060051f342cf |
| multiattach         | False                                |
| name                | volume1                              |
| properties          |                                      |
| replication_status  | None                                 |
| size                | 1                                    |
| snapshot_id         | None                                 |
| source_volid        | None                                 |
| status              | creating                             |
| type                | __DEFAULT__                          |
| updated_at          | None                                 |
| user_id             | 71b5948c75f24c0f841dbf1c4eb4c4a7     |
+---------------------+--------------------------------------+
```

## ボリュームの確認

ボリュームが作成されていることを確認する。

```sh
openstack volume list
```

```
+--------------------------------------+---------+-----------+------+-------------+
| ID                                   | Name    | Status    | Size | Attached to |
+--------------------------------------+---------+-----------+------+-------------+
| dc770907-5aec-4d8f-922d-060051f342cf | volume1 | available |    1 |             |
+--------------------------------------+---------+-----------+------+-------------+
```

Controller Node で LVM を確認する。

```sh
lvdisplay cinder-volumes
```

```
  --- Logical volume ---
  LV Name                cinder-volumes-pool
  VG Name                cinder-volumes
  LV UUID                bbJ72l-8qun-1KZ6-1DYc-wx0T-2VP8-ES7u1f
  LV Write Access        read/write (activated read only)
  LV Creation host, time controller.home.local, 2024-04-29 09:47:32 +0900
  LV Pool metadata       cinder-volumes-pool_tmeta
  LV Pool data           cinder-volumes-pool_tdata
  LV Status              available
  # open                 0
  LV Size                60.80 GiB
  Allocated pool data    0.00%
  Allocated metadata     10.44%
  Current LE             15565
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:6

  --- Logical volume ---
  LV Path                /dev/cinder-volumes/volume-dc770907-5aec-4d8f-922d-060051f342cf
  LV Name                volume-dc770907-5aec-4d8f-922d-060051f342cf
  VG Name                cinder-volumes
  LV UUID                ZAum6P-7x4f-Rvwg-ACWG-X8rm-u3EH-TULiU2
  LV Write Access        read/write
  LV Creation host, time controller.home.local, 2024-04-29 09:53:51 +0900
  LV Pool name           cinder-volumes-pool
  LV Status              available
  # open                 0
  LV Size                1.00 GiB
  Mapped size            0.00%
  Current LE             256
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:7
```

## iSCSI ターゲットの確認

iSCSI ターゲットは変更ない。

```sh
targetcli ls
```

```
o- / ......................................................................................................................... [...]
  o- backstores .............................................................................................................. [...]
  | o- block .................................................................................................. [Storage Objects: 0]
  | o- fileio ................................................................................................. [Storage Objects: 0]
  | o- pscsi .................................................................................................. [Storage Objects: 0]
  | o- ramdisk ................................................................................................ [Storage Objects: 0]
  o- iscsi ............................................................................................................ [Targets: 0]
  o- loopback ......................................................................................................... [Targets: 0]
```
