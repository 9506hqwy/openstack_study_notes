# OpenStack Load Balancing Service (Octavia)

## データベースの作成

データベースに root でログインする。

```sh
mysql -u root -p59c6b803ff3fd0e68dcd
```

データベース octavia を作成する。

```sh
CREATE DATABASE octavia;
GRANT ALL PRIVILEGES ON octavia.* TO 'octavia'@'localhost' IDENTIFIED BY '37cf176d919e31c90e43';
GRANT ALL PRIVILEGES ON octavia.* TO 'octavia'@'%' IDENTIFIED BY '37cf176d919e31c90e43';
```

## ユーザの作成

ユーザ octavia を作成する。

```sh
openstack user create --domain default --password 1672b62e6cd2c1682425 octavia
```

```
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | a431e053e30549958b1e1012cc5d67b8 |
| name                | octavia                          |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```

プロジェクト service にロール admin 権限でユーザ octavia を追加する。

```sh
openstack role add --project service --user octavia admin
```

## サービスの作成

load-balancer サービスを作成する。

```sh
openstack service create --name octavia --description "OpenStack Octavia" load-balancer
```

```
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Octavia                |
| enabled     | True                             |
| id          | 6995bf70a67b4eec89273b32f5d0d726 |
| name        | octavia                          |
| type        | load-balancer                    |
+-------------+----------------------------------+
```

## エンドポイントの作成

API エンドポイントを作成する。

```sh
openstack endpoint create --region RegionOne load-balancer public http://controller:9876
```

```
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | a997d4911e6944df83dd5a912bc48363 |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 6995bf70a67b4eec89273b32f5d0d726 |
| service_name | octavia                          |
| service_type | load-balancer                    |
| url          | http://controller:9876           |
+--------------+----------------------------------+
```

```sh
openstack endpoint create --region RegionOne load-balancer internal http://controller:9876
```

```
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 2fa3ae5ea44f4cdd9dc85989324c1cc4 |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 6995bf70a67b4eec89273b32f5d0d726 |
| service_name | octavia                          |
| service_type | load-balancer                    |
| url          | http://controller:9876           |
+--------------+----------------------------------+
```

```sh
openstack endpoint create --region RegionOne load-balancer admin http://controller:9876
```

```
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 7fe875144ec546b7a60dc8e7d07cce1a |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 6995bf70a67b4eec89273b32f5d0d726 |
| service_name | octavia                          |
| service_type | load-balancer                    |
| url          | http://controller:9876           |
+--------------+----------------------------------+
```

ファイアウォールを開ける。

```sh
firewall-cmd --permanent --zone=internal --add-port=5555/udp
firewall-cmd --permanent --zone=internal --add-port=9876/tcp
firewall-cmd --reload
```

ユーザ octavia で OpenStack を操作するための環境設定ファイルを作成して読み込む。

```sh
cat << EOF >> ~/octavia-openrc
export OS_USERNAME=octavia
export OS_PASSWORD=1672b62e6cd2c1682425
export OS_PROJECT_NAME=service
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_AUTH_URL=http://controller:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
EOF

source ~/octavia-openrc
```

## Amphora イメージの作成

### ディスクイメージの作成

```{tip}
CentOS のパッケージではイメージの生成が成功しなかったため最新のソースコードを利用
```

CentOS ベースの amphora のディスクイメージを作成する。

```sh
dnf install -y git
git clone --depth 1 https://github.com/openstack/octavia.git

cd octavia
python3 -m venv disk_image_create
source ./disk_image_create/bin/activate

cd diskimage-create/
pip install -r requirements.txt

./diskimage-create.sh -a amd64 -i centos-minimal
deactivate
```

```
(...)

Successfully built the amphora image using amphora-agent from the master branch.
Amphora image size: /root/octavia/diskimage-create/amphora-x64-haproxy.qcow2 549275648
```

### イメージの登録

ディスクイメージを glance に登録する。

```sh
openstack image create \
    --disk-format qcow2 \
    --container-format bare \
    --private \
    --tag amphora  \
    --file /root/octavia/diskimage-create/amphora-x64-haproxy.qcow2 \
    amphora-x64-haproxy
```

```
+------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field            | Value                                                                                                                                                   |
+------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------+
| container_format | bare                                                                                                                                                    |
| created_at       | 2024-04-17T15:38:22Z                                                                                                                                    |
| disk_format      | qcow2                                                                                                                                                   |
| file             | /v2/images/cb6ac2cc-5dc6-4735-9735-c8f4ccd0fa5e/file                                                                                                    |
| id               | cb6ac2cc-5dc6-4735-9735-c8f4ccd0fa5e                                                                                                                    |
| min_disk         | 0                                                                                                                                                       |
| min_ram          | 0                                                                                                                                                       |
| name             | amphora-x64-haproxy                                                                                                                                     |
| owner            | 39dd96b7af3a420096b892fabd45900a                                                                                                                        |
| properties       | os_hidden='False', owner_specified.openstack.md5='', owner_specified.openstack.object='images/amphora-x64-haproxy', owner_specified.openstack.sha256='' |
| protected        | False                                                                                                                                                   |
| schema           | /v2/schemas/image                                                                                                                                       |
| status           | queued                                                                                                                                                  |
| tags             | amphora                                                                                                                                                 |
| updated_at       | 2024-04-17T15:38:22Z                                                                                                                                    |
| visibility       | private                                                                                                                                                 |
+------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------+
```

### flavor の作成

amphora 用の flavor を作成する。

```sh
openstack flavor create \
    --id 200 \
    --vcpus 1 \
    --ram 1024 \
    --disk 2 \
    --private \
    amphora
```

```
+----------------------------+---------+
| Field                      | Value   |
+----------------------------+---------+
| OS-FLV-DISABLED:disabled   | False   |
| OS-FLV-EXT-DATA:ephemeral  | 0       |
| description                | None    |
| disk                       | 2       |
| id                         | 200     |
| name                       | amphora |
| os-flavor-access:is_public | False   |
| properties                 |         |
| ram                        | 1024    |
| rxtx_factor                | 1.0     |
| swap                       |         |
| vcpus                      | 1       |
+----------------------------+---------+
```

### セキュリティグループの作成

amphora の管理用のセキュリティグループを作成する。

```sh
openstack security group create lb-mgmt-sec-grp
```

```
+-----------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field           | Value                                                                                                                                                                                                                      |
+-----------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| created_at      | 2024-04-18T11:07:00Z                                                                                                                                                                                                       |
| description     | lb-mgmt-sec-grp                                                                                                                                                                                                            |
| id              | 560176ef-81fa-4deb-8699-75aa8304d47e                                                                                                                                                                                       |
| name            | lb-mgmt-sec-grp                                                                                                                                                                                                            |
| project_id      | 39dd96b7af3a420096b892fabd45900a                                                                                                                                                                                           |
| revision_number | 1                                                                                                                                                                                                                          |
| rules           | created_at='2024-04-18T11:07:00Z', direction='egress', ethertype='IPv6', id='341b7471-f28b-48f4-855a-e2b7f021c61e', standard_attr_id='60', tenant_id='39dd96b7af3a420096b892fabd45900a', updated_at='2024-04-18T11:07:00Z' |
|                 | created_at='2024-04-18T11:07:00Z', direction='egress', ethertype='IPv4', id='6472eac6-aea6-4eef-92fe-a8bf2997ae71', standard_attr_id='59', tenant_id='39dd96b7af3a420096b892fabd45900a', updated_at='2024-04-18T11:07:00Z' |
| stateful        | True                                                                                                                                                                                                                       |
| tags            | []                                                                                                                                                                                                                         |
| updated_at      | 2024-04-18T11:07:00Z                                                                                                                                                                                                       |
+-----------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```

ICMP を許可するルールを作成する。

```sh
openstack security group rule create --protocol icmp lb-mgmt-sec-grp
```

```
+-------------------------+--------------------------------------+
| Field                   | Value                                |
+-------------------------+--------------------------------------+
| created_at              | 2024-04-18T11:08:07Z                 |
| description             |                                      |
| direction               | ingress                              |
| ether_type              | IPv4                                 |
| id                      | 925f6405-f269-41dd-b0db-85651eee62ba |
| name                    | None                                 |
| port_range_max          | None                                 |
| port_range_min          | None                                 |
| project_id              | 39dd96b7af3a420096b892fabd45900a     |
| protocol                | icmp                                 |
| remote_address_group_id | None                                 |
| remote_group_id         | None                                 |
| remote_ip_prefix        | 0.0.0.0/0                            |
| revision_number         | 0                                    |
| security_group_id       | 560176ef-81fa-4deb-8699-75aa8304d47e |
| tags                    | []                                   |
| tenant_id               | 39dd96b7af3a420096b892fabd45900a     |
| updated_at              | 2024-04-18T11:08:07Z                 |
+-------------------------+--------------------------------------+
```

SSH を許可するルールを作成する。

```sh
openstack security group rule create --protocol tcp --dst-port 22 lb-mgmt-sec-grp
```

```
+-------------------------+--------------------------------------+
| Field                   | Value                                |
+-------------------------+--------------------------------------+
| created_at              | 2024-04-18T11:08:39Z                 |
| description             |                                      |
| direction               | ingress                              |
| ether_type              | IPv4                                 |
| id                      | 2cf46f72-57ee-4094-945d-73e265437f5c |
| name                    | None                                 |
| port_range_max          | 22                                   |
| port_range_min          | 22                                   |
| project_id              | 39dd96b7af3a420096b892fabd45900a     |
| protocol                | tcp                                  |
| remote_address_group_id | None                                 |
| remote_group_id         | None                                 |
| remote_ip_prefix        | 0.0.0.0/0                            |
| revision_number         | 0                                    |
| security_group_id       | 560176ef-81fa-4deb-8699-75aa8304d47e |
| tags                    | []                                   |
| tenant_id               | 39dd96b7af3a420096b892fabd45900a     |
| updated_at              | 2024-04-18T11:08:39Z                 |
+-------------------------+--------------------------------------+
```

Load Balancing Service を許可するルールを作成する。

```sh
openstack security group rule create --protocol tcp --dst-port 9443 lb-mgmt-sec-grp
```

```
+-------------------------+--------------------------------------+
| Field                   | Value                                |
+-------------------------+--------------------------------------+
| created_at              | 2024-04-18T11:09:28Z                 |
| description             |                                      |
| direction               | ingress                              |
| ether_type              | IPv4                                 |
| id                      | b367ffc9-b959-4e83-a6b8-3702bf6c2c33 |
| name                    | None                                 |
| port_range_max          | 9443                                 |
| port_range_min          | 9443                                 |
| project_id              | 39dd96b7af3a420096b892fabd45900a     |
| protocol                | tcp                                  |
| remote_address_group_id | None                                 |
| remote_group_id         | None                                 |
| remote_ip_prefix        | 0.0.0.0/0                            |
| revision_number         | 0                                    |
| security_group_id       | 560176ef-81fa-4deb-8699-75aa8304d47e |
| tags                    | []                                   |
| tenant_id               | 39dd96b7af3a420096b892fabd45900a     |
| updated_at              | 2024-04-18T11:09:28Z                 |
+-------------------------+--------------------------------------+
```

## 管理用 flat ネットワークの作成

amphora を制御するためのネットワークを作成する。

```sh
openstack network create \
    --share \
    --external \
    --provider-physical-network mgmt \
    --provider-network-type flat \
    mgmt
```

```
+---------------------------+--------------------------------------+
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| admin_state_up            | UP                                   |
| availability_zone_hints   |                                      |
| availability_zones        |                                      |
| created_at                | 2024-04-20T04:00:29Z                 |
| description               |                                      |
| dns_domain                | None                                 |
| id                        | 2549b734-562a-483c-92bf-f15621dcd30e |
| ipv4_address_scope        | None                                 |
| ipv6_address_scope        | None                                 |
| is_default                | False                                |
| is_vlan_transparent       | None                                 |
| mtu                       | 1500                                 |
| name                      | mgmt                                 |
| port_security_enabled     | True                                 |
| project_id                | 1e3ac7ae10e24515a0956beaa1d8073c     |
| provider:network_type     | flat                                 |
| provider:physical_network | mgmt                                 |
| provider:segmentation_id  | None                                 |
| qos_policy_id             | None                                 |
| revision_number           | 1                                    |
| router:external           | External                             |
| segments                  | None                                 |
| shared                    | True                                 |
| status                    | ACTIVE                               |
| subnets                   |                                      |
| tags                      |                                      |
| updated_at                | 2024-04-20T04:00:29Z                 |
+---------------------------+--------------------------------------+
```

サブネットを作成する。

```sh
openstack subnet create \
    --network mgmt \
    --allocation-pool start=10.0.0.200,end=10.0.0.219 \
    --gateway 10.0.0.1 \
    --dns-nameserver 10.0.0.1 \
    --subnet-range 10.0.0.0/24 \
    mgmt
```

```
+----------------------+--------------------------------------+
| Field                | Value                                |
+----------------------+--------------------------------------+
| allocation_pools     | 10.0.0.200-10.0.0.219                |
| cidr                 | 10.0.0.0/24                          |
| created_at           | 2024-04-20T04:03:45Z                 |
| description          |                                      |
| dns_nameservers      | 10.0.0.1                             |
| dns_publish_fixed_ip | None                                 |
| enable_dhcp          | True                                 |
| gateway_ip           | 10.0.0.1                             |
| host_routes          |                                      |
| id                   | 8997aff8-3491-43e6-bd2c-36f2152cee12 |
| ip_version           | 4                                    |
| ipv6_address_mode    | None                                 |
| ipv6_ra_mode         | None                                 |
| name                 | mgmt                                 |
| network_id           | 2549b734-562a-483c-92bf-f15621dcd30e |
| project_id           | 1e3ac7ae10e24515a0956beaa1d8073c     |
| revision_number      | 0                                    |
| segment_id           | None                                 |
| service_types        |                                      |
| subnetpool_id        | None                                 |
| tags                 |                                      |
| updated_at           | 2024-04-20T04:03:45Z                 |
+----------------------+--------------------------------------+
```

## インストール

octavia をインストールする。

```sh
dnf install -y \
    openstack-octavia-api \
    openstack-octavia-health-manager \
    openstack-octavia-housekeeping \
    openstack-octavia-worker \
    python3-octaviaclient
```

## 証明書の作成

octavia と amphora で使用する証明書を作成する。

```sh
cd octavia/bin/
source create_dual_intermediate_CA.sh
```

```
################# Verifying the Octavia files ###########################
etc/octavia/certs/client.cert-and-key.pem: OK
etc/octavia/certs/server_ca.cert.pem: OK
!!!!!!!!!!!!!!!Do not use this script for deployments!!!!!!!!!!!!!
Please use the Octavia Certificate Configuration guide:
https://docs.openstack.org/octavia/latest/admin/guides/certificates.html
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
```

証明書を配置する。

```sh
mkdir -p /etc/octavia/certs/private
chmod 755 /etc/octavia -R
cp -p etc/octavia/certs/server_ca.cert.pem /etc/octavia/certs
cp -p etc/octavia/certs/server_ca-chain.cert.pem /etc/octavia/certs
cp -p etc/octavia/certs/server_ca.key.pem /etc/octavia/certs/private
cp -p etc/octavia/certs/client_ca.cert.pem /etc/octavia/certs
cp -p etc/octavia/certs/client.cert-and-key.pem /etc/octavia/certs/private
chown -R octavia /etc/octavia/certs
```

## DHCP の設定

DHCP の設定ファイルを作成する。

```sh
mkdir -p 755 /etc/dhcp/octavia
cp octavia/etc/dhcp/dhclient.conf /etc/dhcp/octavia
```

## octavia の設定

設定ファイルを更新する。

`amp_boot_network_list` は Controller Node から amphora の TCP/9443 に接続できる必要があり、
amphora から Controller Node の UDP/5555 に接続できる必要がある。

```sh
sed \
    -e '/^\[DEFAULT]/,/^\[/ {
      /^transport_url =/d
      /^# transport_url =/atransport_url = rabbit://openstack:3c17215fe69ba1dad320@controller:5672/
    }' \
    -e '/^\[database]/,/^\[/ {
      /^connection =/d
      /^# connection = mysql+pymysql:\/\/$/aconnection = mysql+pymysql://octavia:37cf176d919e31c90e43@controller/octavia
    }' \
    -e '/^\[keystone_authtoken]/,/^\[/ {
      /^www_authenticate_uri =/d
      /^# www_authenticate_uri =/awww_authenticate_uri = http://controller:5000
      /^auth_url =/d
      /^# auth_url =/aauth_url = http://controller:5000
      /^project_domain_name =/d
      /^# project_domain_name =/aproject_domain_name = Default
      /^project_name =/d
      /^# project_name =/aproject_name = service
      /^user_domain_name =/d
      /^# user_domain_name =/auser_domain_name = Default
      /^username =/d
      /^# username =/ausername = octavia
      /^password =/d
      /^# password =/apassword = 1672b62e6cd2c1682425
      /^memcached_servers =/d
      /^\[keystone_authtoken]/amemcached_servers = controller:11211
      /^auth_type =/d
      /^\[keystone_authtoken]/aauth_type = password
    }' \
    -e '/^\[service_auth]/,/^\[/ {
      /^auth_url =/d
      /^# auth_url =/aauth_url = http://controller:5000
      /^memcached_servers =/d
      /^# memcached_servers =/amemcached_servers = controller:11211
      /^auth_type =/d
      /^# auth_type =/aauth_type = password
      /^project_domain_name =/d
      /^# project_domain_name =/aproject_domain_name = Default
      /^project_name =/d
      /^# project_name =/aproject_name = service
      /^user_domain_name =/d
      /^# user_domain_name =/auser_domain_name = Default
      /^username =/d
      /^# username =/ausername = octavia
      /^password =/d
      /^# password =/apassword = 1672b62e6cd2c1682425
    }' \
    -e '/^\[api_settings]/,/^\[/ {
      /^bind_host =/d
      /^# bind_host =/abind_host = 0.0.0.0
      /^auth_strategy =/d
      /^# auth_strategy =/aauth_strategy = keystone
      /^api_base_uri =/d
      /^# api_base_uri =/aapi_base_uri = http://controller:9876
    }' \
    -e '/^\[health_manager]/,/^\[/ {
      /^bind_ip =/d
      /^# bind_ip =/abind_ip = 0.0.0.0
      /^controller_ip_port_list =/d
      /^# controller_ip_port_list =$/acontroller_ip_port_list = 10.0.0.11:5555
    }' \
    -e '/^\[oslo_messaging]/,/^\[/ {
      /^topic =/d
      /^# topic =/atopic = octavia_prov
    }' \
    -e '/^\[certificates]/,/^\[/ {
      /^server_certs_key_passphrase =/d
      /^# server_certs_key_passphrase =/aserver_certs_key_passphrase = insecure-key-do-not-use-this-key
      /^ca_private_key_passphrase =/d
      /^# ca_private_key_passphrase =/aca_private_key_passphrase = not-secure-passphrase
      /^ca_private_key =/d
      /^# ca_private_key =/aca_private_key = /etc/octavia/certs/private/server_ca.key.pem
      /^ca_certificate =/d
      /^# ca_certificate =/aca_certificate = /etc/octavia/certs/server_ca.cert.pem
    }' \
    -e '/^\[haproxy_amphora]/,/^\[/ {
      /^bind_host =/d
      /^# bind_host =/abind_host = 0.0.0.0
      /^server_ca =/d
      /^# server_ca =/aserver_ca = /etc/octavia/certs/server_ca-chain.cert.pem
      /^client_cert =/d
      /^# client_cert =/aclient_cert = /etc/octavia/certs/private/client.cert-and-key.pem
    }' \
    -e '/^\[controller_worker]/,/^\[/ {
      /^amp_image_owner_id =/d
      /^# amp_image_owner_id =/aamp_image_owner_id = 39dd96b7af3a420096b892fabd45900a
      /^amp_image_tag =/d
      /^# amp_image_tag =/aamp_image_tag = amphora
      /^amp_ssh_key_name =/d
      /^# amp_ssh_key_name =/aamp_ssh_key_name = mykey
      /^amp_boot_network_list =/d
      /^# amp_boot_network_list =/aamp_boot_network_list = 2549b734-562a-483c-92bf-f15621dcd30e
      /^amp_flavor_id =/d
      /^# amp_flavor_id =/aamp_flavor_id = 200
      /^amp_secgroup_list =/d
      /^# amp_secgroup_list =/aamp_secgroup_list = 560176ef-81fa-4deb-8699-75aa8304d47e
      /^client_ca =/d
      /^# client_ca =/aclient_ca = /etc/octavia/certs/client_ca.cert.pem
    }' \
    -i /etc/octavia/octavia.conf
```

## データベースの構築

データベースを構築する。

```sh
octavia-db-manage --config-file /etc/octavia/octavia.conf upgrade head
```

## 起動

マシン起動時の自動起動設定とサービスを起動する。

```sh
systemctl enable --now octavia-api
systemctl enable --now octavia-health-manager
systemctl enable --now octavia-housekeeping
systemctl enable --now octavia-worker
```
