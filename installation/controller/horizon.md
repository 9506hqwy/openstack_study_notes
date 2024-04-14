# OpenStack Dashboard (Horizon)

## インストール

horizon をインストールする。

```sh
dnf install -y openstack-dashboard
```

## Horizon の設定

設定ファイルを更新する。

```sh
sed \
    -e "s/^\(OPENSTACK_HOST = \).*/\1'controller'/" \
    -e "s/^\(ALLOWED_HOSTS = \).*/\1\['*'\]/" \
    -e "/^#SESSION_ENGINE =/aSESSION_ENGINE = 'django.contrib.sessions.backends.cache'" \
    -e "/^#CACHES =/,/^#}/s/^#\(.*\)/\1/" \
    -e "s|http://%s/identity/v3|http://%s:5000/identity/v3|" \
    -i /etc/openstack-dashboard/local_settings

cat >> /etc/openstack-dashboard/local_settings <<EOF

# domain をサポート
OPENSTACK_KEYSTONE_MULTIDOMAIN_SUPPORT = True

# 既定の domain を指定
OPENSTACK_KEYSTONE_DEFAULT_DOMAIN = "Default"

# ドキュメントルートを設定する。
WEBROOT = '/dashboard/'
EOF
```

## HTTP サーバの設定

設定ファイルを更新する。

```sh
sed \
    -e '1iWSGIApplicationGroup %{GLOBAL}' \
    -e 's|wsgi/django.wsgi|wsgi.py|' \
    -e 's|openstack_dashboard/wsgi>|openstack_dashboard>|' \
    -i /etc/httpd/conf.d/openstack-dashboard.conf
```

ファイアウォールを開ける。

```sh
firewall-cmd --permanent --zone=internal --add-service=http
firewall-cmd --reload
```

## 起動

HTTP サーバを再起動する。

```sh
systemctl restart httpd
```
