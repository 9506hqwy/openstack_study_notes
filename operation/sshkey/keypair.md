# keypair

## SSH 鍵の作成

SSH 鍵を作成する。

```sh
ssh-keygen -q -N "" -f demo_rsa
```

## SSH 公開鍵の登録

```sh
openstack keypair create --public-key demo_rsa.pub mykey
```

```
+-------------+-------------------------------------------------+
| Field       | Value                                           |
+-------------+-------------------------------------------------+
| created_at  | None                                            |
| fingerprint | e3:10:52:4d:ef:91:51:d9:a6:8b:31:f5:6d:f5:51:ed |
| id          | mykey                                           |
| is_deleted  | None                                            |
| name        | mykey                                           |
| type        | ssh                                             |
| user_id     | 71b5948c75f24c0f841dbf1c4eb4c4a7                |
+-------------+-------------------------------------------------+
```

## SSH 公開鍵の確認

```sh
openstack keypair list
```

```
+-------+-------------------------------------------------+------+
| Name  | Fingerprint                                     | Type |
+-------+-------------------------------------------------+------+
| mykey | e3:10:52:4d:ef:91:51:d9:a6:8b:31:f5:6d:f5:51:ed | ssh  |
+-------+-------------------------------------------------+------+
```