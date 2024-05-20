# ボリュームの作成 (iSCSI/LVM)

## ボリュームの作成

```{tip}
myuser で実行
```

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
| created_at          | 2024-05-18T04:10:27.660659           |
| description         | None                                 |
| encrypted           | False                                |
| id                  | fb4470ac-e504-4959-939c-7f4b22fd07b4 |
| migration_status    | None                                 |
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
| user_id             | 7f3acb28d26943bab9510df3a6edf3b0     |
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
| fb4470ac-e504-4959-939c-7f4b22fd07b4 | volume1 | available |    1 |             |
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
  LV UUID                ube6om-ypYq-8ZKE-Fojr-Lg0L-XE5J-NEoOXj
  LV Write Access        read/write (activated read only)
  LV Creation host, time controller.home.local, 2024-05-18 13:08:43 +0900
  LV Pool metadata       cinder-volumes-pool_tmeta
  LV Pool data           cinder-volumes-pool_tdata
  LV Status              available
  # open                 0
  LV Size                30.40 GiB
  Allocated pool data    0.00%
  Allocated metadata     10.49%
  Current LE             7783
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:6

  --- Logical volume ---
  LV Path                /dev/cinder-volumes/volume-fb4470ac-e504-4959-939c-7f4b22fd07b4
  LV Name                volume-fb4470ac-e504-4959-939c-7f4b22fd07b4
  VG Name                cinder-volumes
  LV UUID                xddSZt-2eiB-BinZ-uMNG-XwZP-eRnW-VsBeeV
  LV Write Access        read/write
  LV Creation host, time controller.home.local, 2024-05-18 13:10:29 +0900
  LV Pool name           cinder-volumes-pool
  LV Status              available
  # open                 0
  LV Size                1.00 GiB
  Mapped size            0.00%
  Current LE             256
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
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
