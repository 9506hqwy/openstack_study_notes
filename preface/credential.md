# 認証情報

構築に使用する認証情報を記載する。

認証情報は下記のコマンドで生成する。

```sh
openssl rand -hex 10
```

## MariaDB

| ユーザ名  | パスワード           |
| --------- | -------------------- |
| root      | 59c6b803ff3fd0e68dcd |
| keystone  | f22df81b56cb7c686990 |
| glance    | 5ba24787139e1a1a86b8 |
| placement | 251ccf2f93a6bd6ef9ee |
| nova      | 830e052c9ff3fab2efa4 |
| neutron   | 7d58a6544954d3272e44 |

## RabbitMQ

| ユーザ名  | パスワード           |
| --------- | ----------           |
| openstack | 3c17215fe69ba1dad320 |

## OpenStack

| ユーザ名  | パスワード           |
| --------- | -------------------- |
| admin     | e0f7bb1a2f0571b09c33 |
| myuser    | 77f17eb865932cb5d1af |
| glance    | 7a69c4834de4f3ed2cfa |
| placement | b4b6936f87c338567d3c |
| nova      | 5c5cfbce3214db530456 |
| neutron   | 76283d854fd24b78f90b |

## Neutron

|             |                      |
| ----------- | -------------------- |
| シークレット | 44cb41ccbed49e089ab4 |
