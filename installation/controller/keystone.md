#  OpenStack Identity (Keystone)

## データベースの作成

データベースに root でログインする。

```sh
mysql -u root -p59c6b803ff3fd0e68dcd
```

データベース keystone を作成する。

```sh
CREATE DATABASE keystone;
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY 'f22df81b56cb7c686990';
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY 'f22df81b56cb7c686990';
```

## インストール

keystone と HTTP サーバをインストールする。

```sh
dnf install -y openstack-keystone httpd python3-mod_wsgi
```

## Keystone の設定

設定ファイルを更新する。

```sh
sed \
    -e '/^\[cache]/,/^\[/ {
      /^memcache_servers =/d
      /^#memcache_servers =/amemcache_servers = controller:11211
    }' \
    -e '/^\[database]/,/^\[/ {
      /^connection =/d
      /^#connection =/aconnection = mysql+pymysql://keystone:f22df81b56cb7c686990@controller/keystone
    }' \
    -e '/^\[token]/,/^\[/ {
      /^provider =/d
      /^#provider =/aprovider = fernet
    }' \
    -i /etc/keystone/keystone.conf
```

## データベースの構築

データベースを構築する。

```sh
su -s /bin/sh -c "keystone-manage db_sync" keystone
```

## トークンプロバイダの設定

fernet キーリポジトリを初期化する。

```sh
keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
keystone-manage credential_setup --keystone-user keystone --keystone-group keystone
```

## エンドポイントの作成

Keystone のエンドポイントを作成する。

```{tip}
カレントディレクトリに keystone.conf があると /etc/keystone/keystone.conf より優先される
```

```sh
keystone-manage bootstrap \
    --bootstrap-password e0f7bb1a2f0571b09c33 \
    --bootstrap-admin-url http://controller:5000/v3/ \
    --bootstrap-internal-url http://controller:5000/v3/ \
    --bootstrap-public-url http://controller:5000/v3/ \
    --bootstrap-region-id RegionOne
```

ファイアウォールを開ける。

```sh
firewall-cmd --permanent --zone=internal --add-port=5000/tcp
firewall-cmd --reload
```

## HTTP サーバの設定

HTTP サーバの設定ファイルを更新する。

```sh
sed -e '/^#ServerName/aServerName controller' -i /etc/httpd/conf/httpd.conf
ln -s /usr/share/keystone/wsgi-keystone.conf /etc/httpd/conf.d/
```

## 起動

マシン起動時の自動起動設定とサービスを起動する。

```sh
systemctl enable --now httpd
```

## 環境設定ファイルの作成

ユーザ admin で OpenStack を操作するための環境設定ファイルを作成して読み込む。

```sh
cat > ~/admin-openrc <<EOF
export OS_USERNAME=admin
export OS_PASSWORD=e0f7bb1a2f0571b09c33
export OS_PROJECT_NAME=admin
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_DOMAIN_NAME=Default
export OS_AUTH_URL=http://controller:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
EOF

source ~/admin-openrc
```

## 初期構築

OpenStack コンポーネント用のプロジェクトを作成する。

```sh
openstack project create --domain default --description "Service Project" service
```

```
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Service Project                  |
| domain_id   | default                          |
| enabled     | True                             |
| id          | 6b004b6cbcf74e94a5f41f6cd4ed1d67 |
| is_domain   | False                            |
| name        | service                          |
| options     | {}                               |
| parent_id   | default                          |
| tags        | []                               |
+-------------+----------------------------------+
```

管理者以外の使用するプロジェクトを作成する。

```sh
openstack project create --domain default --description "Demo Project" myproject
```

```
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Demo Project                     |
| domain_id   | default                          |
| enabled     | True                             |
| id          | 20f45fd8b06b4115ba061c3c1e321234 |
| is_domain   | False                            |
| name        | myproject                        |
| options     | {}                               |
| parent_id   | default                          |
| tags        | []                               |
+-------------+----------------------------------+
```

管理者以外のユーザを作成する。

```sh
openstack user create --domain default --password 77f17eb865932cb5d1af myuser
```

```
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 88de68c6ed0a477494f5607ed23f0bbe |
| name                | myuser                           |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```

ロールを作成する。

```sh
openstack role create myrole
```

```
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | None                             |
| domain_id   | None                             |
| id          | 8af9bc09ca0e44e582bf5196c2015d30 |
| name        | myrole                           |
| options     | {}                               |
+-------------+----------------------------------+
```

プロジェクト myproject にロール myrole 権限でユーザ myuser を追加する。

```sh
openstack role add --project myproject --user myuser myrole
```

ユーザ myuser で OpenStack を操作するための環境設定ファイルを作成する。

```sh
cat > ~/demo-openrc <<EOF
export OS_USERNAME=myuser
export OS_PASSWORD=77f17eb865932cb5d1af
export OS_PROJECT_NAME=myproject
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_DOMAIN_NAME=Default
export OS_AUTH_URL=http://controller:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
EOF
```

## 動作確認

ユーザ admin で認証トークンを要求する。

```sh
unset OS_AUTH_URL
unset OS_PASSWORD
openstack \
    --os-auth-url http://controller:5000/v3 \
    --os-project-domain-name Default \
    --os-user-domain-name Default \
    --os-project-name admin \
    --os-username admin \
    token \
    issue
```

```
Password:
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field      | Value                                                                                                                                                                                   |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| expires    | 2024-04-09T12:32:19+0000                                                                                                                                                                |
| id         | gAAAAABmFSdDPdAWDmXUy2gwuEaXCDhSBe_B4KvK3Mf-cdG4OXdkpyw1LgtMu25D9LxjUJjMbzs7AkR6gCOuy2SZamJZRoC-gJllsQwMUqhoequIsOiPLQ1kUwNxyUCySaekERY0wO1FoJxMl12Vok2b2a31c9b6CPTDl4MWlKnOC-BX38Q3EOg |
| project_id | cb2be036057d4225af1a33e3afb9f891                                                                                                                                                        |
| user_id    | ef1ad879bf7d4f449de4c3eeaa7089cf                                                                                                                                                        |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```

ユーザ myuser で認証トークンを要求する。

```sh
unset OS_AUTH_URL
unset OS_PASSWORD
openstack \
    --os-auth-url http://controller:5000/v3 \
    --os-project-domain-name Default \
    --os-user-domain-name Default \
    --os-project-name myproject \
    --os-username myuser \
    token \
    issue
```

```
Password:
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field      | Value                                                                                                                                                                                   |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| expires    | 2024-04-09T12:32:52+0000                                                                                                                                                                |
| id         | gAAAAABmFSdkYR_hSN5iNArmYeO1uuzC1_OOqVx_tGRxWD8KaWp0jY6dEtSdKyesiGGmZnJn7K1xT168QP0WpRLdaI_9xtCjhCM18vm-Frmts6dKHO88lewQdKt91jBilf7woTaPHGOsICsteV0W6JkSdXcePyny2_Hnw0LoKa7VYDYHIArRmrQ |
| project_id | 20f45fd8b06b4115ba061c3c1e321234                                                                                                                                                        |
| user_id    | 88de68c6ed0a477494f5607ed23f0bbe                                                                                                                                                        |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```
