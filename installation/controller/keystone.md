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

API エンドポイントを作成する。

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
| id          | 39dd96b7af3a420096b892fabd45900a |
| is_domain   | False                            |
| name        | service                          |
| options     | {}                               |
| parent_id   | default                          |
| tags        | []                               |
+-------------+----------------------------------+
```

管理者以外が使用するプロジェクトを作成する。

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
| id          | f2aeffb34ff34ffb8959f1cd813655c6 |
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
| id                  | 71b5948c75f24c0f841dbf1c4eb4c4a7 |
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
| id          | a4fac38ae4de4a56a5f6ece8cc36bc4c |
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
| expires    | 2024-04-13T06:27:19+0000                                                                                                                                                                |
| id         | gAAAAABmGhe3jJ1bwtAmwkLkLokKujDlQ-pWNGf1QxllvA89HqfxUOio1UNyIR3WSpiKchzdTfEhQ7QgmMvWXB2OQ7R5dQztLL8Nnq_D5NQn4g38YDhfzNsBvxb5L7ieA5blnfDvYu-YsvpSmHxN92l328HsXG4iksZoC7QNHpxsB5gPnfLiAkY |
| project_id | 1e3ac7ae10e24515a0956beaa1d8073c                                                                                                                                                        |
| user_id    | d0a08833034240889e27ff1a8c007be0                                                                                                                                                        |
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
| expires    | 2024-04-13T06:27:42+0000                                                                                                                                                                |
| id         | gAAAAABmGhfOWk2-K98vK7IRZONrxpSPB_6KJWs7oZAGSMvwIIxzWkdxl-26THOfFFjkLaHz1rA9qC-R8FS9dSmcbXsEGchMunDUwnkoTvY5MHqOf9-hLVzSwZRzqCMj1Nwf95u89ou95iZ5IxcAyN5hJ0BsHusFLqXX8zEEe-QnHck8RGNwzRM |
| project_id | f2aeffb34ff34ffb8959f1cd813655c6                                                                                                                                                        |
| user_id    | 71b5948c75f24c0f841dbf1c4eb4c4a7                                                                                                                                                        |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```
