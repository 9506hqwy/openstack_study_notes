# Identity (Keystone)

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
export IRONIC_API_VERSION=1.78
export OS_BAREMETAL_API_VERSION=1.78
export CYBORG_API_VERSION=2
export OS_ACCELERATOR_API_VERSION=2
EOF

source ~/admin-openrc
```

## 初期構築

OpenStack コンポーネント用のプロジェクトを作成する。

```sh
openstack project create --domain default --description "Service Project" service
```

```text
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Service Project                  |
| domain_id   | default                          |
| enabled     | True                             |
| id          | bb6848524614439b9b0f89f69ca98c37 |
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

```text
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Demo Project                     |
| domain_id   | default                          |
| enabled     | True                             |
| id          | bccf406c045d401b91ba5c7552a124ae |
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

```text
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 7f3acb28d26943bab9510df3a6edf3b0 |
| name                | myuser                           |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```

ロールを作成する。

```sh
openstack role create myrole
```

```text
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | None                             |
| domain_id   | None                             |
| id          | 994fea031e034d85980415fa73785877 |
| name        | myrole                           |
| options     | {}                               |
+-------------+----------------------------------+
```

プロジェクト myproject でユーザ myuser にロール myrole 権限を追加する。

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
export OS_VOLUME_API_VERSION=3.27
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
    --os-password e0f7bb1a2f0571b09c33 \
    token \
    issue
```

```text
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field      | Value                                                                                                                                                                                   |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| expires    | 2024-05-08T12:33:36+0000                                                                                                                                                                |
| id         | gAAAAABmO2MQCLKpjtH9ecSNR5oxRVDL1za3pFtuz-Y1LvJLBW-9UTVJcqb5MzeTb_wedMFEDPHPzVsjZW5lcsjq3FzMp7XmLGKuiDuk1Jpj5Ac9jkBIvHT9f9nXgRuCITNV-74OrewRhpcvRCGIeWBQE6MEKvn8KOODuQxrPibIx1LQTmDwByY |
| project_id | be94f4411bd74f249f5e25f642209b82                                                                                                                                                        |
| user_id    | 34d4ed4faeac41b6bc6a2116fb2d0f47                                                                                                                                                        |
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
    --os-password 77f17eb865932cb5d1af \
    token \
    issue
```

```text
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field      | Value                                                                                                                                                                                   |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| expires    | 2024-05-08T12:34:30+0000                                                                                                                                                                |
| id         | gAAAAABmO2NGjhfSHcgtYogXR_89MjCS9OGJi58pw4pMInCnjph6NW24BLZuuo5AnTgvQL7Yr6DrYR1R_LugkD0wxFIKT98tvECYiz7aSfAfcFg719l_uUd5tSoh0RAC3zduOQuJOZXOo6geGMhMiNFUODbAyKQoyc8FDN0mstXap87yKrJiE3A |
| project_id | bccf406c045d401b91ba5c7552a124ae                                                                                                                                                        |
| user_id    | 7f3acb28d26943bab9510df3a6edf3b0                                                                                                                                                        |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```
