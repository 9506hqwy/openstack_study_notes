# OpenStack リポジトリの有効化

OpenStack Yoga パッケージをインストールするためリポジトリを有効化する。

```sh
dnf install -y centos-release-openstack-caracal
dnf config-manager --enable crb
```
