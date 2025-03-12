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

```text
+-------------+-------------------------------------------------+
| Field       | Value                                           |
+-------------+-------------------------------------------------+
| created_at  | None                                            |
| fingerprint | d2:b8:9e:71:0f:86:dc:ca:ac:a1:c6:c8:4d:45:16:8c |
| id          | mykey                                           |
| is_deleted  | None                                            |
| name        | mykey                                           |
| type        | ssh                                             |
| user_id     | 7f3acb28d26943bab9510df3a6edf3b0                |
+-------------+-------------------------------------------------+
```

## SSH 公開鍵の確認

```sh
openstack keypair list
```

```text
+-------+-------------------------------------------------+------+
| Name  | Fingerprint                                     | Type |
+-------+-------------------------------------------------+------+
| mykey | d2:b8:9e:71:0f:86:dc:ca:ac:a1:c6:c8:4d:45:16:8c | ssh  |
+-------+-------------------------------------------------+------+
```
