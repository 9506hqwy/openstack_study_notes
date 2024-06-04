# ルータ (Open Virtual Network)

## 前提条件

* [](../network/ovn_flat) を作成していること。
* [](../network/ovn_geneve) を作成していること。

## ルータの作成

```{tip}
myuser で実行
```

ルータを作成する。

```sh
openstack router create router
```

```
+---------------------------+--------------------------------------+
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| admin_state_up            | UP                                   |
| availability_zone_hints   |                                      |
| availability_zones        |                                      |
| created_at                | 2024-05-30T11:34:44Z                 |
| description               |                                      |
| enable_default_route_bfd  | False                                |
| enable_default_route_ecmp | False                                |
| enable_ndp_proxy          | None                                 |
| external_gateway_info     | null                                 |
| external_gateways         | []                                   |
| flavor_id                 | None                                 |
| ha                        | True                                 |
| id                        | 32ee6d25-076d-4dd3-9bed-87afb2cbf222 |
| name                      | router                               |
| project_id                | bccf406c045d401b91ba5c7552a124ae     |
| revision_number           | 1                                    |
| routes                    |                                      |
| status                    | ACTIVE                               |
| tags                      |                                      |
| tenant_id                 | bccf406c045d401b91ba5c7552a124ae     |
| updated_at                | 2024-05-30T11:34:44Z                 |
+---------------------------+--------------------------------------+
```

## サブネットに接続

ルータ router をサブネット selfservice に接続する。

```sh
openstack router add subnet router selfservice
```

## ゲートウェイの設定

ルータ router のゲートウェイを設定する。

```sh
openstack router set router --external-gateway provider
```

## ポートの確認

ルータ router に作成されたポートを確認する。

```sh
openstack port list --router router
```

```
+--------------------------------------+------+-------------------+--------------------------------------------------------------------------------+--------+
| ID                                   | Name | MAC Address       | Fixed IP Addresses                                                             | Status |
+--------------------------------------+------+-------------------+--------------------------------------------------------------------------------+--------+
| 3c5abf4a-5efc-4b6a-8d49-98d3f2cac0dd |      | fa:16:3e:15:d6:26 | ip_address='172.16.0.168', subnet_id='c2616a6d-bb5d-48b6-b80f-284954a3b7b5'    | DOWN   |
| 82afe6ae-c8f6-4b01-8833-f011bcf0dda9 |      | fa:16:3e:35:8f:09 | ip_address='192.168.101.254', subnet_id='27a80619-0344-4b90-9df3-3f8f4b10ec76' | ACTIVE |
+--------------------------------------+------+-------------------+--------------------------------------------------------------------------------+--------+
```

## 環境の確認


### Controller Node

#### ネットワーク名前空間

ネットワーク名前空間は作成されない。

#### Northband データベース

論理スイッチの作成を確認する。

```sh
ovn-nbctl list Logical_Switch
```

```
_uuid               : 8f3ec596-41e0-43f2-a4b7-7befbab52e40
acls                : []
copp                : []
dns_records         : []
external_ids        : {"neutron:availability_zone_hints"="", "neutron:mtu"="1500", "neutron:network_name"=provider, "neutron:revision_number"="2"}
forwarding_groups   : []
load_balancer       : []
load_balancer_group : []
name                : neutron-ec6ba68e-47d4-4ece-a233-718a902e68e6
other_config        : {fdb_age_threshold="0", mcast_flood_unregistered="false", mcast_snoop="false", vlan-passthru="false"}
ports               : [0eadca71-d5c6-4ff5-84f6-8e79548312e9, 19af7d65-fabb-4966-b9eb-9b4c33c5f4c4, 769582de-9c0f-406b-9dbe-70d5f5204934, f4e9a1f0-e912-4158-9030-be3c8a238795]
qos_rules           : []

_uuid               : 4618aa87-cb97-4480-b7d8-9e296ed6ead4
acls                : []
copp                : []
dns_records         : []
external_ids        : {"neutron:availability_zone_hints"="", "neutron:mtu"="1442", "neutron:network_name"=selfservice, "neutron:revision_number"="2"}
forwarding_groups   : []
load_balancer       : []
load_balancer_group : []
name                : neutron-f5e24ee8-2bcb-43a4-91d5-0abad0c58197
other_config        : {mcast_flood_unregistered="false", mcast_snoop="false", vlan-passthru="false"}
ports               : [5afb3337-b52f-4c14-9000-fc6c275b821b, 856bb6d7-842d-43bc-a22f-cf9b44818e10, 90ead9c3-36a3-442b-bda6-35d228354e15, bee7f632-da16-4b76-91b0-09b8379bc492]
qos_rules           : []
```

ポートを確認する。

```sh
ovn-nbctl list Logical_Switch_Port
```

```
_uuid               : 19af7d65-fabb-4966-b9eb-9b4c33c5f4c4
addresses           : [router]
dhcpv4_options      : []
dhcpv6_options      : []
dynamic_addresses   : []
enabled             : true
external_ids        : {"neutron:cidrs"="172.16.0.168/24", "neutron:device_id"="32ee6d25-076d-4dd3-9bed-87afb2cbf222", "neutron:device_owner"="network:router_gateway", "neutron:mtu"="", "neutron:network_name"=neutron-ec6ba68e-47d4-4ece-a233-718a902e68e6, "neutron:port_capabilities"="", "neutron:port_name"="", "neutron:project_id"="", "neutron:revision_number"="1", "neutron:security_group_ids"="", "neutron:subnet_pool_addr_scope4"="", "neutron:subnet_pool_addr_scope6"="", "neutron:vnic_type"=normal}
ha_chassis_group    : []
mirror_rules        : []
name                : "3c5abf4a-5efc-4b6a-8d49-98d3f2cac0dd"
options             : {exclude-lb-vips-from-garp="true", nat-addresses=router, router-port=lrp-3c5abf4a-5efc-4b6a-8d49-98d3f2cac0dd}
parent_name         : []
port_security       : []
tag                 : []
tag_request         : []
type                : router
up                  : true

_uuid               : 5afb3337-b52f-4c14-9000-fc6c275b821b
addresses           : [router]
dhcpv4_options      : []
dhcpv6_options      : []
dynamic_addresses   : []
enabled             : true
external_ids        : {"neutron:cidrs"="192.168.101.254/24", "neutron:device_id"="32ee6d25-076d-4dd3-9bed-87afb2cbf222", "neutron:device_owner"="network:router_interface", "neutron:mtu"="", "neutron:network_name"=neutron-f5e24ee8-2bcb-43a4-91d5-0abad0c58197, "neutron:port_capabilities"="", "neutron:port_name"="", "neutron:project_id"=bccf406c045d401b91ba5c7552a124ae, "neutron:revision_number"="3", "neutron:security_group_ids"="", "neutron:subnet_pool_addr_scope4"="", "neutron:subnet_pool_addr_scope6"="", "neutron:vnic_type"=normal}
ha_chassis_group    : []
mirror_rules        : []
name                : "82afe6ae-c8f6-4b01-8833-f011bcf0dda9"
options             : {router-port=lrp-82afe6ae-c8f6-4b01-8833-f011bcf0dda9}
parent_name         : []
port_security       : []
tag                 : []
tag_request         : []
type                : router
up                  : true
```

論理ルータを確認する。

```sh
ovn-nbctl list Logical_Router
```

```
_uuid               : 108cc266-6c52-4adc-817b-1ead115d7996
copp                : []
enabled             : true
external_ids        : {"neutron:availability_zone_hints"="", "neutron:revision_number"="4", "neutron:router_name"=router}
load_balancer       : []
load_balancer_group : []
name                : neutron-32ee6d25-076d-4dd3-9bed-87afb2cbf222
nat                 : [f708ac0c-5ebf-45c3-be48-56fa503d064c]
options             : {always_learn_from_arp_request="false", dynamic_neigh_routers="true", mac_binding_age_threshold="0"}
policies            : []
ports               : [427c434e-8c9a-41ee-ae3f-8f10f49ec48f, e22c8140-651f-4075-bbbb-fc2a19a0c6d8]
static_routes       : [df7ec0c8-0343-4c84-a1a2-9d79bfce3a66]
```

NAT を確認する。

```sh
ovn-nbctl list nat f708ac0c-5ebf-45c3-be48-56fa503d064c
```

```
_uuid               : f708ac0c-5ebf-45c3-be48-56fa503d064c
allowed_ext_ips     : []
exempted_ext_ips    : []
external_ids        : {}
external_ip         : "172.16.0.168"
external_mac        : []
external_port_range : ""
gateway_port        : []
logical_ip          : "192.168.101.0/24"
logical_port        : []
options             : {}
type                : snat
```

静的ルートを確認する。

```sh
ovn-nbctl list logical-router-static-route df7ec0c8-0343-4c84-a1a2-9d79bfce3a66
```

```
_uuid               : df7ec0c8-0343-4c84-a1a2-9d79bfce3a66
bfd                 : []
external_ids        : {"neutron:is_ext_gw"="true", "neutron:subnet_id"="c2616a6d-bb5d-48b6-b80f-284954a3b7b5"}
ip_prefix           : "0.0.0.0/0"
nexthop             : "172.16.0.254"
options             : {}
output_port         : []
policy              : []
route_table         : ""
```

論理ルータのポートを確認する。

```sh
ovn-nbctl list Logical_Router_Port
```

```
_uuid               : e22c8140-651f-4075-bbbb-fc2a19a0c6d8
enabled             : []
external_ids        : {"neutron:is_ext_gw"=False, "neutron:network_name"=neutron-f5e24ee8-2bcb-43a4-91d5-0abad0c58197, "neutron:revision_number"="3", "neutron:router_name"="32ee6d25-076d-4dd3-9bed-87afb2cbf222", "neutron:subnet_ids"="27a80619-0344-4b90-9df3-3f8f4b10ec76"}
gateway_chassis     : []
ha_chassis_group    : []
ipv6_prefix         : []
ipv6_ra_configs     : {}
mac                 : "fa:16:3e:35:8f:09"
name                : lrp-82afe6ae-c8f6-4b01-8833-f011bcf0dda9
networks            : ["192.168.101.254/24"]
options             : {}
peer                : []
status              : {}

_uuid               : 427c434e-8c9a-41ee-ae3f-8f10f49ec48f
enabled             : []
external_ids        : {"neutron:is_ext_gw"=True, "neutron:network_name"=neutron-ec6ba68e-47d4-4ece-a233-718a902e68e6, "neutron:revision_number"="1", "neutron:router_name"="32ee6d25-076d-4dd3-9bed-87afb2cbf222", "neutron:subnet_ids"="c2616a6d-bb5d-48b6-b80f-284954a3b7b5"}
gateway_chassis     : []
ha_chassis_group    : []
ipv6_prefix         : []
ipv6_ra_configs     : {}
mac                 : "fa:16:3e:15:d6:26"
name                : lrp-3c5abf4a-5efc-4b6a-8d49-98d3f2cac0dd
networks            : ["172.16.0.168/24"]
options             : {}
peer                : []
status              : {}
```

#### Southbound データベース

フローを確認する。

```sh
ovn-sbctl lflow-list
```

```
Datapath: "neutron-32ee6d25-076d-4dd3-9bed-87afb2cbf222" aka "router" (9e73dd0c-1d10-42d9-8264-d268e1c794e2)  Pipeline: ingress
  table=0 (lr_in_admission    ), priority=110  , match=(((ip4 && icmp4.type == 3 && icmp4.code == 4) || (ip6 && icmp6.type == 2 && icmp6.code == 0)) && flags.tunnel_rx == 1), action=(drop;)
  table=0 (lr_in_admission    ), priority=100  , match=(vlan.present || eth.src[40]), action=(drop;)
  table=0 (lr_in_admission    ), priority=50   , match=(eth.dst == fa:16:3e:15:d6:26 && inport == "lrp-3c5abf4a-5efc-4b6a-8d49-98d3f2cac0dd"), action=(xreg0[0..47] = fa:16:3e:15:d6:26; next;)
  table=0 (lr_in_admission    ), priority=50   , match=(eth.dst == fa:16:3e:35:8f:09 && inport == "lrp-82afe6ae-c8f6-4b01-8833-f011bcf0dda9"), action=(xreg0[0..47] = fa:16:3e:35:8f:09; next;)
  table=0 (lr_in_admission    ), priority=50   , match=(eth.mcast && inport == "lrp-3c5abf4a-5efc-4b6a-8d49-98d3f2cac0dd"), action=(xreg0[0..47] = fa:16:3e:15:d6:26; next;)
  table=0 (lr_in_admission    ), priority=50   , match=(eth.mcast && inport == "lrp-82afe6ae-c8f6-4b01-8833-f011bcf0dda9"), action=(xreg0[0..47] = fa:16:3e:35:8f:09; next;)
  table=0 (lr_in_admission    ), priority=0    , match=(1), action=(drop;)
  table=1 (lr_in_lookup_neighbor), priority=110  , match=(inport == "lrp-3c5abf4a-5efc-4b6a-8d49-98d3f2cac0dd" && arp.spa == 172.16.0.0/24 && arp.tpa == 172.16.0.168 && arp.op == 1), action=(reg9[2] = lookup_arp(inport, arp.spa, arp.sha); reg9[3] = 1; next;)
  table=1 (lr_in_lookup_neighbor), priority=110  , match=(inport == "lrp-82afe6ae-c8f6-4b01-8833-f011bcf0dda9" && arp.spa == 192.168.101.0/24 && arp.tpa == 192.168.101.254 && arp.op == 1), action=(reg9[2] = lookup_arp(inport, arp.spa, arp.sha); reg9[3] = 1; next;)
  table=1 (lr_in_lookup_neighbor), priority=110  , match=(nd_na && ip6.src == fe80::/10 && ip6.dst == ff00::/8), action=(reg9[2] = lookup_nd(inport, nd.target, nd.tll); reg9[3] = lookup_nd_ip(inport, nd.target); next;)
  table=1 (lr_in_lookup_neighbor), priority=100  , match=(arp.op == 2), action=(reg9[2] = lookup_arp(inport, arp.spa, arp.sha); reg9[3] = 1; next;)
  table=1 (lr_in_lookup_neighbor), priority=100  , match=(inport == "lrp-3c5abf4a-5efc-4b6a-8d49-98d3f2cac0dd" && arp.spa == 172.16.0.0/24 && arp.op == 1), action=(reg9[2] = lookup_arp(inport, arp.spa, arp.sha); reg9[3] = lookup_arp_ip(inport, arp.spa); next;)
  table=1 (lr_in_lookup_neighbor), priority=100  , match=(inport == "lrp-82afe6ae-c8f6-4b01-8833-f011bcf0dda9" && arp.spa == 192.168.101.0/24 && arp.op == 1), action=(reg9[2] = lookup_arp(inport, arp.spa, arp.sha); reg9[3] = lookup_arp_ip(inport, arp.spa); next;)
  table=1 (lr_in_lookup_neighbor), priority=100  , match=(nd_na), action=(reg9[2] = lookup_nd(inport, nd.target, nd.tll); reg9[3] = 1; next;)
  table=1 (lr_in_lookup_neighbor), priority=100  , match=(nd_ns), action=(reg9[2] = lookup_nd(inport, ip6.src, nd.sll); reg9[3] = lookup_nd_ip(inport, ip6.src); next;)
  table=1 (lr_in_lookup_neighbor), priority=0    , match=(1), action=(reg9[2] = 1; next;)
  table=2 (lr_in_learn_neighbor), priority=100  , match=(reg9[2] == 1 || reg9[3] == 0), action=(mac_cache_use; next;)
  table=2 (lr_in_learn_neighbor), priority=95   , match=(nd_na && nd.tll == 0), action=(put_nd(inport, nd.target, eth.src); next;)
  table=2 (lr_in_learn_neighbor), priority=95   , match=(nd_ns && (ip6.src == 0 || nd.sll == 0)), action=(next;)
  table=2 (lr_in_learn_neighbor), priority=90   , match=(arp), action=(put_arp(inport, arp.spa, arp.sha); next;)
  table=2 (lr_in_learn_neighbor), priority=90   , match=(nd_na), action=(put_nd(inport, nd.target, nd.tll); next;)
  table=2 (lr_in_learn_neighbor), priority=90   , match=(nd_ns), action=(put_nd(inport, ip6.src, nd.sll); next;)
  table=2 (lr_in_learn_neighbor), priority=0    , match=(1), action=(drop;)
  table=3 (lr_in_ip_input     ), priority=100  , match=(ip4.src == {172.16.0.168, 172.16.0.255} && reg9[0] == 0), action=(drop;)
  table=3 (lr_in_ip_input     ), priority=100  , match=(ip4.src == {192.168.101.254, 192.168.101.255} && reg9[0] == 0), action=(drop;)
  table=3 (lr_in_ip_input     ), priority=100  , match=(ip4.src_mcast ||ip4.src == 255.255.255.255 || ip4.src == 127.0.0.0/8 || ip4.dst == 127.0.0.0/8 || ip4.src == 0.0.0.0/8 || ip4.dst == 0.0.0.0/8), action=(drop;)
  table=3 (lr_in_ip_input     ), priority=100  , match=(ip6.dst == fe80::f816:3eff:fe15:d626 && udp.src == 547 && udp.dst == 546), action=(reg0 = 0; handle_dhcpv6_reply;)
  table=3 (lr_in_ip_input     ), priority=100  , match=(ip6.dst == fe80::f816:3eff:fe35:8f09 && udp.src == 547 && udp.dst == 546), action=(reg0 = 0; handle_dhcpv6_reply;)
  table=3 (lr_in_ip_input     ), priority=90   , match=(arp.op == 1 && arp.tpa == 172.16.0.168), action=(eth.dst = eth.src; eth.src = xreg0[0..47]; arp.op = 2; /* ARP reply */ arp.tha = arp.sha; arp.sha = xreg0[0..47]; arp.tpa <-> arp.spa; outport = inport; flags.loopback = 1; output;)
  table=3 (lr_in_ip_input     ), priority=90   , match=(inport == "lrp-3c5abf4a-5efc-4b6a-8d49-98d3f2cac0dd" && arp.op == 1 && arp.tpa == 172.16.0.168 && arp.spa == 172.16.0.0/24), action=(eth.dst = eth.src; eth.src = xreg0[0..47]; arp.op = 2; /* ARP reply */ arp.tha = arp.sha; arp.sha = xreg0[0..47]; arp.tpa <-> arp.spa; outport = inport; flags.loopback = 1; output;)
  table=3 (lr_in_ip_input     ), priority=90   , match=(inport == "lrp-3c5abf4a-5efc-4b6a-8d49-98d3f2cac0dd" && ip6.dst == {fe80::f816:3eff:fe15:d626, ff02::1:ff15:d626} && nd_ns && nd.target == fe80::f816:3eff:fe15:d626), action=(nd_na_router { eth.src = xreg0[0..47]; ip6.src = nd.target; nd.tll = xreg0[0..47]; outport = inport; flags.loopback = 1; output; };)
  table=3 (lr_in_ip_input     ), priority=90   , match=(inport == "lrp-82afe6ae-c8f6-4b01-8833-f011bcf0dda9" && arp.op == 1 && arp.tpa == 192.168.101.254 && arp.spa == 192.168.101.0/24), action=(eth.dst = eth.src; eth.src = xreg0[0..47]; arp.op = 2; /* ARP reply */ arp.tha = arp.sha; arp.sha = xreg0[0..47]; arp.tpa <-> arp.spa; outport = inport; flags.loopback = 1; output;)
  table=3 (lr_in_ip_input     ), priority=90   , match=(inport == "lrp-82afe6ae-c8f6-4b01-8833-f011bcf0dda9" && ip6.dst == {fe80::f816:3eff:fe35:8f09, ff02::1:ff35:8f09} && nd_ns && nd.target == fe80::f816:3eff:fe35:8f09), action=(nd_na_router { eth.src = xreg0[0..47]; ip6.src = nd.target; nd.tll = xreg0[0..47]; outport = inport; flags.loopback = 1; output; };)
  table=3 (lr_in_ip_input     ), priority=90   , match=(ip4.dst == 172.16.0.168 && icmp4.type == 8 && icmp4.code == 0), action=(ip4.dst <-> ip4.src; ip.ttl = 255; icmp4.type = 0; flags.loopback = 1; next; )
  table=3 (lr_in_ip_input     ), priority=90   , match=(ip4.dst == 192.168.101.254 && icmp4.type == 8 && icmp4.code == 0), action=(ip4.dst <-> ip4.src; ip.ttl = 255; icmp4.type = 0; flags.loopback = 1; next; )
  table=3 (lr_in_ip_input     ), priority=90   , match=(ip6.dst == fe80::f816:3eff:fe15:d626 && icmp6.type == 128 && icmp6.code == 0), action=(ip6.dst <-> ip6.src; ip.ttl = 255; icmp6.type = 129; flags.loopback = 1; next; )
  table=3 (lr_in_ip_input     ), priority=90   , match=(ip6.dst == fe80::f816:3eff:fe35:8f09 && icmp6.type == 128 && icmp6.code == 0), action=(ip6.dst <-> ip6.src; ip.ttl = 255; icmp6.type = 129; flags.loopback = 1; next; )
  table=3 (lr_in_ip_input     ), priority=85   , match=(arp || nd), action=(drop;)
  table=3 (lr_in_ip_input     ), priority=84   , match=(nd_rs || nd_ra), action=(next;)
  table=3 (lr_in_ip_input     ), priority=83   , match=(ip6.mcast_rsvd), action=(drop;)
  table=3 (lr_in_ip_input     ), priority=82   , match=(ip4.mcast || ip6.mcast), action=(drop;)
  table=3 (lr_in_ip_input     ), priority=80   , match=(ip4 && ip4.dst == 172.16.0.168 && !ip.later_frag && sctp), action=(sctp_abort {eth.dst <-> eth.src; ip4.dst <-> ip4.src; next; };)
  table=3 (lr_in_ip_input     ), priority=80   , match=(ip4 && ip4.dst == 172.16.0.168 && !ip.later_frag && tcp), action=(tcp_reset {eth.dst <-> eth.src; ip4.dst <-> ip4.src; next; };)
  table=3 (lr_in_ip_input     ), priority=80   , match=(ip4 && ip4.dst == 172.16.0.168 && !ip.later_frag && udp), action=(icmp4 {eth.dst <-> eth.src; ip4.dst <-> ip4.src; ip.ttl = 255; icmp4.type = 3; icmp4.code = 3; next; };)
  table=3 (lr_in_ip_input     ), priority=80   , match=(ip4 && ip4.dst == 192.168.101.254 && !ip.later_frag && sctp), action=(sctp_abort {eth.dst <-> eth.src; ip4.dst <-> ip4.src; next; };)
  table=3 (lr_in_ip_input     ), priority=80   , match=(ip4 && ip4.dst == 192.168.101.254 && !ip.later_frag && tcp), action=(tcp_reset {eth.dst <-> eth.src; ip4.dst <-> ip4.src; next; };)
  table=3 (lr_in_ip_input     ), priority=80   , match=(ip4 && ip4.dst == 192.168.101.254 && !ip.later_frag && udp), action=(icmp4 {eth.dst <-> eth.src; ip4.dst <-> ip4.src; ip.ttl = 255; icmp4.type = 3; icmp4.code = 3; next; };)
  table=3 (lr_in_ip_input     ), priority=80   , match=(ip6 && ip6.dst == fe80::f816:3eff:fe15:d626 && !ip.later_frag && sctp), action=(sctp_abort {eth.dst <-> eth.src; ip6.dst <-> ip6.src; next; };)
  table=3 (lr_in_ip_input     ), priority=80   , match=(ip6 && ip6.dst == fe80::f816:3eff:fe15:d626 && !ip.later_frag && tcp), action=(tcp_reset {eth.dst <-> eth.src; ip6.dst <-> ip6.src; next; };)
  table=3 (lr_in_ip_input     ), priority=80   , match=(ip6 && ip6.dst == fe80::f816:3eff:fe15:d626 && !ip.later_frag && udp), action=(icmp6 {eth.dst <-> eth.src; ip6.dst <-> ip6.src; ip.ttl = 255; icmp6.type = 1; icmp6.code = 4; next; };)
  table=3 (lr_in_ip_input     ), priority=80   , match=(ip6 && ip6.dst == fe80::f816:3eff:fe35:8f09 && !ip.later_frag && sctp), action=(sctp_abort {eth.dst <-> eth.src; ip6.dst <-> ip6.src; next; };)
  table=3 (lr_in_ip_input     ), priority=80   , match=(ip6 && ip6.dst == fe80::f816:3eff:fe35:8f09 && !ip.later_frag && tcp), action=(tcp_reset {eth.dst <-> eth.src; ip6.dst <-> ip6.src; next; };)
  table=3 (lr_in_ip_input     ), priority=80   , match=(ip6 && ip6.dst == fe80::f816:3eff:fe35:8f09 && !ip.later_frag && udp), action=(icmp6 {eth.dst <-> eth.src; ip6.dst <-> ip6.src; ip.ttl = 255; icmp6.type = 1; icmp6.code = 4; next; };)
  table=3 (lr_in_ip_input     ), priority=70   , match=(ip4 && ip4.dst == 172.16.0.168 && !ip.later_frag), action=(icmp4 {eth.dst <-> eth.src; ip4.dst <-> ip4.src; ip.ttl = 255; icmp4.type = 3; icmp4.code = 2; next; };)
  table=3 (lr_in_ip_input     ), priority=70   , match=(ip4 && ip4.dst == 192.168.101.254 && !ip.later_frag), action=(icmp4 {eth.dst <-> eth.src; ip4.dst <-> ip4.src; ip.ttl = 255; icmp4.type = 3; icmp4.code = 2; next; };)
  table=3 (lr_in_ip_input     ), priority=70   , match=(ip6 && ip6.dst == fe80::f816:3eff:fe15:d626 && !ip.later_frag), action=(icmp6 {eth.dst <-> eth.src; ip6.dst <-> ip6.src; ip.ttl = 255; icmp6.type = 1; icmp6.code = 3; next; };)
  table=3 (lr_in_ip_input     ), priority=70   , match=(ip6 && ip6.dst == fe80::f816:3eff:fe35:8f09 && !ip.later_frag), action=(icmp6 {eth.dst <-> eth.src; ip6.dst <-> ip6.src; ip.ttl = 255; icmp6.type = 1; icmp6.code = 3; next; };)
  table=3 (lr_in_ip_input     ), priority=60   , match=(ip4.dst == {192.168.101.254}), action=(drop;)
  table=3 (lr_in_ip_input     ), priority=60   , match=(ip6.dst == {fe80::f816:3eff:fe15:d626}), action=(drop;)
  table=3 (lr_in_ip_input     ), priority=60   , match=(ip6.dst == {fe80::f816:3eff:fe35:8f09}), action=(drop;)
  table=3 (lr_in_ip_input     ), priority=50   , match=(eth.bcast), action=(drop;)
  table=3 (lr_in_ip_input     ), priority=32   , match=(ip.ttl == {0, 1} && !ip.later_frag && (ip4.mcast || ip6.mcast)), action=(drop;)
  table=3 (lr_in_ip_input     ), priority=31   , match=(inport == "lrp-3c5abf4a-5efc-4b6a-8d49-98d3f2cac0dd" && ip4 && ip.ttl == {0, 1} && !ip.later_frag), action=(icmp4 {eth.dst <-> eth.src; icmp4.type = 11; /* Time exceeded */ icmp4.code = 0; /* TTL exceeded in transit */ ip4.dst = ip4.src; ip4.src = 172.16.0.168 ; ip.ttl = 254; outport = "lrp-3c5abf4a-5efc-4b6a-8d49-98d3f2cac0dd"; flags.loopback = 1; output; };)
  table=3 (lr_in_ip_input     ), priority=31   , match=(inport == "lrp-82afe6ae-c8f6-4b01-8833-f011bcf0dda9" && ip4 && ip.ttl == {0, 1} && !ip.later_frag), action=(icmp4 {eth.dst <-> eth.src; icmp4.type = 11; /* Time exceeded */ icmp4.code = 0; /* TTL exceeded in transit */ ip4.dst = ip4.src; ip4.src = 192.168.101.254 ; ip.ttl = 254; outport = "lrp-82afe6ae-c8f6-4b01-8833-f011bcf0dda9"; flags.loopback = 1; output; };)
  table=3 (lr_in_ip_input     ), priority=30   , match=(ip.ttl == {0, 1}), action=(drop;)
  table=3 (lr_in_ip_input     ), priority=0    , match=(1), action=(next;)
  table=4 (lr_in_unsnat       ), priority=0    , match=(1), action=(next;)
  table=5 (lr_in_defrag       ), priority=0    , match=(1), action=(next;)
  table=6 (lr_in_lb_aff_check ), priority=0    , match=(1), action=(next;)
  table=7 (lr_in_dnat         ), priority=0    , match=(1), action=(next;)
  table=8 (lr_in_lb_aff_learn ), priority=0    , match=(1), action=(next;)
  table=9 (lr_in_ecmp_stateful), priority=0    , match=(1), action=(next;)
  table=10(lr_in_nd_ra_options), priority=0    , match=(1), action=(next;)
  table=11(lr_in_nd_ra_response), priority=0    , match=(1), action=(next;)
  table=12(lr_in_ip_routing_pre), priority=0    , match=(1), action=(reg7 = 0; next;)
  table=13(lr_in_ip_routing   ), priority=10550, match=(nd_rs || nd_ra), action=(drop;)
  table=13(lr_in_ip_routing   ), priority=194  , match=(inport == "lrp-3c5abf4a-5efc-4b6a-8d49-98d3f2cac0dd" && ip6.dst == fe80::/64), action=(ip.ttl--; reg8[0..15] = 0; xxreg0 = ip6.dst; xxreg1 = fe80::f816:3eff:fe15:d626; eth.src = fa:16:3e:15:d6:26; outport = "lrp-3c5abf4a-5efc-4b6a-8d49-98d3f2cac0dd"; flags.loopback = 1; next;)
  table=13(lr_in_ip_routing   ), priority=194  , match=(inport == "lrp-82afe6ae-c8f6-4b01-8833-f011bcf0dda9" && ip6.dst == fe80::/64), action=(ip.ttl--; reg8[0..15] = 0; xxreg0 = ip6.dst; xxreg1 = fe80::f816:3eff:fe35:8f09; eth.src = fa:16:3e:35:8f:09; outport = "lrp-82afe6ae-c8f6-4b01-8833-f011bcf0dda9"; flags.loopback = 1; next;)
  table=13(lr_in_ip_routing   ), priority=74   , match=(ip4.dst == 172.16.0.0/24), action=(ip.ttl--; reg8[0..15] = 0; reg0 = ip4.dst; reg1 = 172.16.0.168; eth.src = fa:16:3e:15:d6:26; outport = "lrp-3c5abf4a-5efc-4b6a-8d49-98d3f2cac0dd"; flags.loopback = 1; next;)
  table=13(lr_in_ip_routing   ), priority=74   , match=(ip4.dst == 192.168.101.0/24), action=(ip.ttl--; reg8[0..15] = 0; reg0 = ip4.dst; reg1 = 192.168.101.254; eth.src = fa:16:3e:35:8f:09; outport = "lrp-82afe6ae-c8f6-4b01-8833-f011bcf0dda9"; flags.loopback = 1; next;)
  table=13(lr_in_ip_routing   ), priority=1    , match=(reg7 == 0 && ip4.dst == 0.0.0.0/0), action=(ip.ttl--; reg8[0..15] = 0; reg0 = 172.16.0.254; reg1 = 172.16.0.168; eth.src = fa:16:3e:15:d6:26; outport = "lrp-3c5abf4a-5efc-4b6a-8d49-98d3f2cac0dd"; flags.loopback = 1; next;)
  table=13(lr_in_ip_routing   ), priority=0    , match=(1), action=(drop;)
  table=14(lr_in_ip_routing_ecmp), priority=150  , match=(reg8[0..15] == 0), action=(next;)
  table=14(lr_in_ip_routing_ecmp), priority=0    , match=(1), action=(drop;)
  table=15(lr_in_policy       ), priority=0    , match=(1), action=(reg8[0..15] = 0; next;)
  table=16(lr_in_policy_ecmp  ), priority=150  , match=(reg8[0..15] == 0), action=(next;)
  table=16(lr_in_policy_ecmp  ), priority=0    , match=(1), action=(drop;)
  table=17(lr_in_arp_resolve  ), priority=500  , match=(ip4.mcast || ip6.mcast), action=(next;)
  table=17(lr_in_arp_resolve  ), priority=100  , match=(outport == "lrp-3c5abf4a-5efc-4b6a-8d49-98d3f2cac0dd" && reg0 == 172.16.0.100), action=(eth.dst = fa:16:3e:ee:57:9a; next;)
  table=17(lr_in_arp_resolve  ), priority=100  , match=(outport == "lrp-3c5abf4a-5efc-4b6a-8d49-98d3f2cac0dd" && reg0 == 172.16.0.146), action=(eth.dst = fa:16:3e:2a:eb:48; next;)
  table=17(lr_in_arp_resolve  ), priority=100  , match=(outport == "lrp-82afe6ae-c8f6-4b01-8833-f011bcf0dda9" && reg0 == 192.168.101.1), action=(eth.dst = fa:16:3e:71:9e:ad; next;)
  table=17(lr_in_arp_resolve  ), priority=100  , match=(outport == "lrp-82afe6ae-c8f6-4b01-8833-f011bcf0dda9" && reg0 == 192.168.101.162), action=(eth.dst = fa:16:3e:bf:bb:c5; next;)
  table=17(lr_in_arp_resolve  ), priority=100  , match=(outport == "lrp-82afe6ae-c8f6-4b01-8833-f011bcf0dda9" && reg0 == 192.168.101.25), action=(eth.dst = fa:16:3e:63:d4:da; next;)
  table=17(lr_in_arp_resolve  ), priority=2    , match=(ip4.dst == {172.16.0.168}), action=(drop;)
  table=17(lr_in_arp_resolve  ), priority=1    , match=(ip4), action=(get_arp(outport, reg0); next;)
  table=17(lr_in_arp_resolve  ), priority=1    , match=(ip6), action=(get_nd(outport, xxreg0); next;)
  table=17(lr_in_arp_resolve  ), priority=0    , match=(1), action=(drop;)
  table=18(lr_in_chk_pkt_len  ), priority=0    , match=(1), action=(next;)
  table=19(lr_in_larger_pkts  ), priority=0    , match=(1), action=(next;)
  table=20(lr_in_gw_redirect  ), priority=0    , match=(1), action=(next;)
  table=21(lr_in_arp_request  ), priority=100  , match=(eth.dst == 00:00:00:00:00:00 && ip4), action=(arp { eth.dst = ff:ff:ff:ff:ff:ff; arp.spa = reg1; arp.tpa = reg0; arp.op = 1; output; };)
  table=21(lr_in_arp_request  ), priority=100  , match=(eth.dst == 00:00:00:00:00:00 && ip6), action=(nd_ns { nd.target = xxreg0; output; };)
  table=21(lr_in_arp_request  ), priority=0    , match=(1), action=(output;)
Datapath: "neutron-32ee6d25-076d-4dd3-9bed-87afb2cbf222" aka "router" (9e73dd0c-1d10-42d9-8264-d268e1c794e2)  Pipeline: egress
  table=0 (lr_out_chk_dnat_local), priority=0    , match=(1), action=(reg9[4] = 0; next;)
  table=1 (lr_out_undnat      ), priority=0    , match=(1), action=(next;)
  table=2 (lr_out_post_undnat ), priority=0    , match=(1), action=(next;)
  table=3 (lr_out_snat        ), priority=120  , match=(nd_ns), action=(next;)
  table=3 (lr_out_snat        ), priority=0    , match=(1), action=(next;)
  table=4 (lr_out_post_snat   ), priority=0    , match=(1), action=(next;)
  table=5 (lr_out_egr_loop    ), priority=0    , match=(1), action=(next;)
  table=6 (lr_out_delivery    ), priority=100  , match=(outport == "lrp-3c5abf4a-5efc-4b6a-8d49-98d3f2cac0dd"), action=(output;)
  table=6 (lr_out_delivery    ), priority=100  , match=(outport == "lrp-82afe6ae-c8f6-4b01-8833-f011bcf0dda9"), action=(output;)
  table=6 (lr_out_delivery    ), priority=0    , match=(1), action=(drop;)
Datapath: "neutron-ec6ba68e-47d4-4ece-a233-718a902e68e6" aka "provider" (fa81dc7d-3704-41ab-9e5b-266d1fddcfba)  Pipeline: ingress
  table=0 (ls_in_check_port_sec), priority=120  , match=(((ip4 && icmp4.type == 3 && icmp4.code == 4) || (ip6 && icmp6.type == 2 && icmp6.code == 0)) && eth.dst == fa:16:3e:15:d6:26 && flags.tunnel_rx == 1), action=(outport <-> inport; next(pipeline=ingress,table=27);)
  table=0 (ls_in_check_port_sec), priority=110  , match=(((ip4 && icmp4.type == 3 && icmp4.code == 4) || (ip6 && icmp6.type == 2 && icmp6.code == 0)) && flags.tunnel_rx == 1), action=(drop;)
  table=0 (ls_in_check_port_sec), priority=100  , match=(eth.src[40]), action=(drop;)
  table=0 (ls_in_check_port_sec), priority=100  , match=(vlan.present), action=(drop;)
  table=0 (ls_in_check_port_sec), priority=50   , match=(1), action=(reg0[15] = check_in_port_sec(); next;)
  table=1 (ls_in_apply_port_sec), priority=50   , match=(reg0[15] == 1), action=(drop;)
  table=1 (ls_in_apply_port_sec), priority=0    , match=(1), action=(next;)
  table=2 (ls_in_lookup_fdb   ), priority=0    , match=(1), action=(next;)
  table=3 (ls_in_put_fdb      ), priority=0    , match=(1), action=(next;)
  table=4 (ls_in_pre_acl      ), priority=110  , match=(eth.dst == $svc_monitor_mac), action=(next;)
  table=4 (ls_in_pre_acl      ), priority=110  , match=(eth.mcast), action=(next;)
  table=4 (ls_in_pre_acl      ), priority=110  , match=(ip && inport == "3c5abf4a-5efc-4b6a-8d49-98d3f2cac0dd"), action=(next;)
  table=4 (ls_in_pre_acl      ), priority=110  , match=(ip && inport == "provnet-698fbbf6-da6f-4d59-8259-60d87eb5e697"), action=(next;)
  table=4 (ls_in_pre_acl      ), priority=110  , match=(nd || nd_rs || nd_ra || mldv1 || mldv2 || (udp && udp.src == 546 && udp.dst == 547)), action=(next;)
  table=4 (ls_in_pre_acl      ), priority=100  , match=(ip), action=(reg0[0] = 1; next;)
  table=4 (ls_in_pre_acl      ), priority=0    , match=(1), action=(next;)
  table=5 (ls_in_pre_lb       ), priority=110  , match=(eth.dst == $svc_monitor_mac), action=(next;)
  table=5 (ls_in_pre_lb       ), priority=110  , match=(eth.mcast), action=(next;)
  table=5 (ls_in_pre_lb       ), priority=110  , match=(ip && inport == "3c5abf4a-5efc-4b6a-8d49-98d3f2cac0dd"), action=(next;)
  table=5 (ls_in_pre_lb       ), priority=110  , match=(ip && inport == "provnet-698fbbf6-da6f-4d59-8259-60d87eb5e697"), action=(next;)
  table=5 (ls_in_pre_lb       ), priority=110  , match=(nd || nd_rs || nd_ra || mldv1 || mldv2), action=(next;)
  table=5 (ls_in_pre_lb       ), priority=110  , match=(reg0[16] == 1), action=(next;)
  table=5 (ls_in_pre_lb       ), priority=0    , match=(1), action=(next;)
  table=6 (ls_in_pre_stateful ), priority=110  , match=(reg0[2] == 1), action=(ct_lb_mark;)
  table=6 (ls_in_pre_stateful ), priority=100  , match=(reg0[0] == 1), action=(ct_next;)
  table=6 (ls_in_pre_stateful ), priority=0    , match=(1), action=(next;)
  table=7 (ls_in_acl_hint     ), priority=7    , match=(ct.new && !ct.est), action=(reg0[7] = 1; reg0[9] = 1; next;)
  table=7 (ls_in_acl_hint     ), priority=6    , match=(!ct.new && ct.est && !ct.rpl && ct_mark.blocked == 1), action=(reg0[7] = 1; reg0[9] = 1; next;)
  table=7 (ls_in_acl_hint     ), priority=5    , match=(!ct.trk), action=(reg0[8] = 1; reg0[9] = 1; next;)
  table=7 (ls_in_acl_hint     ), priority=4    , match=(!ct.new && ct.est && !ct.rpl && ct_mark.blocked == 0), action=(reg0[8] = 1; reg0[10] = 1; next;)
  table=7 (ls_in_acl_hint     ), priority=3    , match=(!ct.est), action=(reg0[9] = 1; next;)
  table=7 (ls_in_acl_hint     ), priority=2    , match=(ct.est && ct_mark.blocked == 1), action=(reg0[9] = 1; next;)
  table=7 (ls_in_acl_hint     ), priority=1    , match=(ct.est && ct_mark.blocked == 0), action=(reg0[10] = 1; next;)
  table=7 (ls_in_acl_hint     ), priority=0    , match=(1), action=(next;)
  table=8 (ls_in_acl_eval     ), priority=65532, match=(!ct.est && ct.rel && !ct.new && !ct.inv && ct_mark.blocked == 0), action=(reg0[17] = 1; reg8[16] = 1; ct_commit_nat;)
  table=8 (ls_in_acl_eval     ), priority=65532, match=(ct.est && !ct.rel && !ct.new && !ct.inv && ct.rpl && ct_mark.blocked == 0), action=(reg0[9] = 0; reg0[10] = 0; reg0[17] = 1; reg8[16] = 1; next;)
  table=8 (ls_in_acl_eval     ), priority=65532, match=(ct.inv || (ct.est && ct.rpl && ct_mark.blocked == 1)), action=(reg8[17] = 1; next;)
  table=8 (ls_in_acl_eval     ), priority=65532, match=(nd || nd_ra || nd_rs || mldv1 || mldv2), action=(reg8[16] = 1; next;)
  table=8 (ls_in_acl_eval     ), priority=34000, match=(eth.dst == $svc_monitor_mac), action=(reg8[16] = 1; next;)
  table=8 (ls_in_acl_eval     ), priority=2002 , match=(reg0[7] == 1 && (inport == @pg_158f2c45_1393_46e4_8093_6e82e4d876c9 && ip4)), action=(reg8[16] = 1; reg0[1] = 1; next;)
  table=8 (ls_in_acl_eval     ), priority=2002 , match=(reg0[7] == 1 && (inport == @pg_158f2c45_1393_46e4_8093_6e82e4d876c9 && ip6)), action=(reg8[16] = 1; reg0[1] = 1; next;)
  table=8 (ls_in_acl_eval     ), priority=2002 , match=(reg0[8] == 1 && (inport == @pg_158f2c45_1393_46e4_8093_6e82e4d876c9 && ip4)), action=(reg8[16] = 1; next;)
  table=8 (ls_in_acl_eval     ), priority=2002 , match=(reg0[8] == 1 && (inport == @pg_158f2c45_1393_46e4_8093_6e82e4d876c9 && ip6)), action=(reg8[16] = 1; next;)
  table=8 (ls_in_acl_eval     ), priority=2001 , match=(reg0[10] == 1 && (inport == @neutron_pg_drop && ip)), action=(reg8[17] = 1; ct_commit { ct_mark.blocked = 1; }; next;)
  table=8 (ls_in_acl_eval     ), priority=2001 , match=(reg0[9] == 1 && (inport == @neutron_pg_drop && ip)), action=(reg8[17] = 1; next;)
  table=8 (ls_in_acl_eval     ), priority=1    , match=(ip && !ct.est), action=(reg0[1] = 1; next;)
  table=8 (ls_in_acl_eval     ), priority=1    , match=(ip && ct.est && ct_mark.blocked == 1), action=(reg0[1] = 1; reg8[16] = 1; next;)
  table=8 (ls_in_acl_eval     ), priority=0    , match=(1), action=(next;)
  table=9 (ls_in_acl_action   ), priority=1000 , match=(reg8[16] == 1), action=(reg8[16] = 0; reg8[17] = 0; reg8[18] = 0; next;)
  table=9 (ls_in_acl_action   ), priority=1000 , match=(reg8[17] == 1), action=(reg8[16] = 0; reg8[17] = 0; reg8[18] = 0; /* drop */)
  table=9 (ls_in_acl_action   ), priority=1000 , match=(reg8[18] == 1), action=(reg8[16] = 0; reg8[17] = 0; reg8[18] = 0; reg0 = 0; reject { /* eth.dst <-> eth.src; ip.dst <-> ip.src; is implicit. */ outport <-> inport; next(pipeline=egress,table=6); };)
  table=9 (ls_in_acl_action   ), priority=0    , match=(1), action=(reg8[16] = 0; reg8[17] = 0; reg8[18] = 0; next;)
  table=10(ls_in_qos_mark     ), priority=0    , match=(1), action=(next;)
  table=11(ls_in_qos_meter    ), priority=0    , match=(1), action=(next;)
  table=12(ls_in_lb_aff_check ), priority=0    , match=(1), action=(next;)
  table=13(ls_in_lb           ), priority=0    , match=(1), action=(next;)
  table=14(ls_in_lb_aff_learn ), priority=0    , match=(1), action=(next;)
  table=15(ls_in_pre_hairpin  ), priority=0    , match=(1), action=(next;)
  table=16(ls_in_nat_hairpin  ), priority=0    , match=(1), action=(next;)
  table=17(ls_in_hairpin      ), priority=0    , match=(1), action=(next;)
  table=18(ls_in_acl_after_lb_eval), priority=65532, match=(nd || nd_ra || nd_rs || mldv1 || mldv2), action=(reg8[16] = 1; next;)
  table=18(ls_in_acl_after_lb_eval), priority=65532, match=(reg0[17] == 1), action=(reg8[16] = 1; next;)
  table=18(ls_in_acl_after_lb_eval), priority=0    , match=(1), action=(next;)
  table=19(ls_in_acl_after_lb_action), priority=1000 , match=(reg8[16] == 1), action=(reg8[16] = 0; reg8[17] = 0; reg8[18] = 0; next;)
  table=19(ls_in_acl_after_lb_action), priority=1000 , match=(reg8[17] == 1), action=(reg8[16] = 0; reg8[17] = 0; reg8[18] = 0; /* drop */)
  table=19(ls_in_acl_after_lb_action), priority=1000 , match=(reg8[18] == 1), action=(reg8[16] = 0; reg8[17] = 0; reg8[18] = 0; reg0 = 0; reject { /* eth.dst <-> eth.src; ip.dst <-> ip.src; is implicit. */ outport <-> inport; next(pipeline=egress,table=6); };)
  table=19(ls_in_acl_after_lb_action), priority=0    , match=(1), action=(reg8[16] = 0; reg8[17] = 0; reg8[18] = 0; next;)
  table=20(ls_in_stateful     ), priority=100  , match=(reg0[1] == 1 && reg0[13] == 0), action=(ct_commit { ct_mark.blocked = 0; }; next;)
  table=20(ls_in_stateful     ), priority=100  , match=(reg0[1] == 1 && reg0[13] == 1), action=(ct_commit { ct_mark.blocked = 0; ct_label.label = reg3; }; next;)
  table=20(ls_in_stateful     ), priority=0    , match=(1), action=(next;)
  table=21(ls_in_arp_rsp      ), priority=100  , match=(arp.tpa == 172.16.0.100 && arp.op == 1 && inport == "9241e97d-8312-471c-9765-48f657442262"), action=(next;)
  table=21(ls_in_arp_rsp      ), priority=100  , match=(arp.tpa == 172.16.0.168 && arp.op == 1 && inport == "3c5abf4a-5efc-4b6a-8d49-98d3f2cac0dd"), action=(next;)
  table=21(ls_in_arp_rsp      ), priority=100  , match=(inport == "provnet-698fbbf6-da6f-4d59-8259-60d87eb5e697"), action=(next;)
  table=21(ls_in_arp_rsp      ), priority=100  , match=(nd_ns && ip6.dst == {fe80::f816:3eff:fe15:d626, ff02::1:ff15:d626} && nd.target == fe80::f816:3eff:fe15:d626 && inport == "3c5abf4a-5efc-4b6a-8d49-98d3f2cac0dd"), action=(next;)
  table=21(ls_in_arp_rsp      ), priority=50   , match=(arp.tpa == 172.16.0.100 && arp.op == 1), action=(eth.dst = eth.src; eth.src = fa:16:3e:ee:57:9a; arp.op = 2; /* ARP reply */ arp.tha = arp.sha; arp.sha = fa:16:3e:ee:57:9a; arp.tpa = arp.spa; arp.spa = 172.16.0.100; outport = inport; flags.loopback = 1; output;)
  table=21(ls_in_arp_rsp      ), priority=50   , match=(arp.tpa == 172.16.0.168 && arp.op == 1), action=(eth.dst = eth.src; eth.src = fa:16:3e:15:d6:26; arp.op = 2; /* ARP reply */ arp.tha = arp.sha; arp.sha = fa:16:3e:15:d6:26; arp.tpa = arp.spa; arp.spa = 172.16.0.168; outport = inport; flags.loopback = 1; output;)
  table=21(ls_in_arp_rsp      ), priority=50   , match=(nd_ns && ip6.dst == {fe80::f816:3eff:fe15:d626, ff02::1:ff15:d626} && nd.target == fe80::f816:3eff:fe15:d626), action=(nd_na_router { eth.src = fa:16:3e:15:d6:26; ip6.src = fe80::f816:3eff:fe15:d626; nd.target = fe80::f816:3eff:fe15:d626; nd.tll = fa:16:3e:15:d6:26; outport = inport; flags.loopback = 1; output; };)
  table=21(ls_in_arp_rsp      ), priority=0    , match=(1), action=(next;)
  table=22(ls_in_dhcp_options ), priority=100  , match=(inport == "7b43909d-8ae8-4609-9872-811667056013" && eth.src == fa:16:3e:2a:eb:48 && (ip4.src == {172.16.0.146, 0.0.0.0} && ip4.dst == {172.16.0.254, 255.255.255.255}) && udp.src == 68 && udp.dst == 67), action=(reg0[3] = put_dhcp_opts(offerip = 172.16.0.146, classless_static_route = {169.254.169.254/32,172.16.0.100, 0.0.0.0/0,172.16.0.254}, dns_server = {10.0.0.254}, lease_time = 43200, mtu = 1500, netmask = 255.255.255.0, router = 172.16.0.254, server_id = 172.16.0.254); next;)
  table=22(ls_in_dhcp_options ), priority=0    , match=(1), action=(next;)
  table=23(ls_in_dhcp_response), priority=100  , match=(inport == "7b43909d-8ae8-4609-9872-811667056013" && eth.src == fa:16:3e:2a:eb:48 && ip4 && udp.src == 68 && udp.dst == 67 && reg0[3]), action=(eth.dst = eth.src; eth.src = fa:16:3e:ca:64:02; ip4.src = 172.16.0.254; udp.src = 67; udp.dst = 68; outport = inport; flags.loopback = 1; output;)
  table=23(ls_in_dhcp_response), priority=0    , match=(1), action=(next;)
  table=24(ls_in_dns_lookup   ), priority=0    , match=(1), action=(next;)
  table=25(ls_in_dns_response ), priority=0    , match=(1), action=(next;)
  table=26(ls_in_external_port), priority=0    , match=(1), action=(next;)
  table=27(ls_in_l2_lkup      ), priority=110  , match=(eth.dst == $svc_monitor_mac && (tcp || icmp || icmp6)), action=(handle_svc_check(inport);)
  table=27(ls_in_l2_lkup      ), priority=80   , match=(flags[1] == 0 && arp.op == 1 && arp.tpa == 172.16.0.168), action=(clone {outport = "3c5abf4a-5efc-4b6a-8d49-98d3f2cac0dd"; output; }; outport = "_MC_flood_l2"; output;)
  table=27(ls_in_l2_lkup      ), priority=80   , match=(flags[1] == 0 && nd_ns && nd.target == fe80::f816:3eff:fe15:d626), action=(clone {outport = "3c5abf4a-5efc-4b6a-8d49-98d3f2cac0dd"; output; }; outport = "_MC_flood_l2"; output;)
  table=27(ls_in_l2_lkup      ), priority=75   , match=(eth.src == {fa:16:3e:15:d6:26} && (arp.op == 1 || rarp.op == 3 || nd_ns)), action=(outport = "_MC_flood_l2"; output;)
  table=27(ls_in_l2_lkup      ), priority=70   , match=(eth.mcast), action=(outport = "_MC_flood"; output;)
  table=27(ls_in_l2_lkup      ), priority=50   , match=(eth.dst == fa:16:3e:15:d6:26), action=(outport = "3c5abf4a-5efc-4b6a-8d49-98d3f2cac0dd"; output;)
  table=27(ls_in_l2_lkup      ), priority=50   , match=(eth.dst == fa:16:3e:2a:eb:48), action=(outport = "7b43909d-8ae8-4609-9872-811667056013"; output;)
  table=27(ls_in_l2_lkup      ), priority=50   , match=(eth.dst == fa:16:3e:ee:57:9a), action=(outport = "9241e97d-8312-471c-9765-48f657442262"; output;)
  table=27(ls_in_l2_lkup      ), priority=0    , match=(1), action=(outport = get_fdb(eth.dst); next;)
  table=28(ls_in_l2_unknown   ), priority=50   , match=(outport == "none"), action=(outport = "_MC_unknown"; output;)
  table=28(ls_in_l2_unknown   ), priority=0    , match=(1), action=(output;)
Datapath: "neutron-ec6ba68e-47d4-4ece-a233-718a902e68e6" aka "provider" (fa81dc7d-3704-41ab-9e5b-266d1fddcfba)  Pipeline: egress
  table=0 (ls_out_pre_acl     ), priority=110  , match=(eth.mcast), action=(next;)
  table=0 (ls_out_pre_acl     ), priority=110  , match=(eth.src == $svc_monitor_mac), action=(next;)
  table=0 (ls_out_pre_acl     ), priority=110  , match=(ip && outport == "3c5abf4a-5efc-4b6a-8d49-98d3f2cac0dd"), action=(next;)
  table=0 (ls_out_pre_acl     ), priority=110  , match=(ip && outport == "provnet-698fbbf6-da6f-4d59-8259-60d87eb5e697"), action=(next;)
  table=0 (ls_out_pre_acl     ), priority=110  , match=(nd || nd_rs || nd_ra || mldv1 || mldv2 || (udp && udp.src == 546 && udp.dst == 547)), action=(next;)
  table=0 (ls_out_pre_acl     ), priority=100  , match=(ip), action=(reg0[0] = 1; next;)
  table=0 (ls_out_pre_acl     ), priority=0    , match=(1), action=(next;)
  table=1 (ls_out_pre_lb      ), priority=110  , match=(eth.mcast), action=(next;)
  table=1 (ls_out_pre_lb      ), priority=110  , match=(eth.src == $svc_monitor_mac), action=(next;)
  table=1 (ls_out_pre_lb      ), priority=110  , match=(ip && outport == "3c5abf4a-5efc-4b6a-8d49-98d3f2cac0dd"), action=(next;)
  table=1 (ls_out_pre_lb      ), priority=110  , match=(ip && outport == "provnet-698fbbf6-da6f-4d59-8259-60d87eb5e697"), action=(next;)
  table=1 (ls_out_pre_lb      ), priority=110  , match=(nd || nd_rs || nd_ra || mldv1 || mldv2), action=(next;)
  table=1 (ls_out_pre_lb      ), priority=110  , match=(reg0[16] == 1), action=(next;)
  table=1 (ls_out_pre_lb      ), priority=0    , match=(1), action=(next;)
  table=2 (ls_out_pre_stateful), priority=110  , match=(reg0[2] == 1), action=(ct_lb_mark;)
  table=2 (ls_out_pre_stateful), priority=100  , match=(reg0[0] == 1), action=(ct_next;)
  table=2 (ls_out_pre_stateful), priority=0    , match=(1), action=(next;)
  table=3 (ls_out_acl_hint    ), priority=7    , match=(ct.new && !ct.est), action=(reg0[7] = 1; reg0[9] = 1; next;)
  table=3 (ls_out_acl_hint    ), priority=6    , match=(!ct.new && ct.est && !ct.rpl && ct_mark.blocked == 1), action=(reg0[7] = 1; reg0[9] = 1; next;)
  table=3 (ls_out_acl_hint    ), priority=5    , match=(!ct.trk), action=(reg0[8] = 1; reg0[9] = 1; next;)
  table=3 (ls_out_acl_hint    ), priority=4    , match=(!ct.new && ct.est && !ct.rpl && ct_mark.blocked == 0), action=(reg0[8] = 1; reg0[10] = 1; next;)
  table=3 (ls_out_acl_hint    ), priority=3    , match=(!ct.est), action=(reg0[9] = 1; next;)
  table=3 (ls_out_acl_hint    ), priority=2    , match=(ct.est && ct_mark.blocked == 1), action=(reg0[9] = 1; next;)
  table=3 (ls_out_acl_hint    ), priority=1    , match=(ct.est && ct_mark.blocked == 0), action=(reg0[10] = 1; next;)
  table=3 (ls_out_acl_hint    ), priority=0    , match=(1), action=(next;)
  table=4 (ls_out_acl_eval    ), priority=65532, match=(!ct.est && ct.rel && !ct.new && !ct.inv && ct_mark.blocked == 0), action=(reg8[16] = 1; ct_commit_nat;)
  table=4 (ls_out_acl_eval    ), priority=65532, match=(ct.est && !ct.rel && !ct.new && !ct.inv && ct.rpl && ct_mark.blocked == 0), action=(reg8[16] = 1; next;)
  table=4 (ls_out_acl_eval    ), priority=65532, match=(ct.inv || (ct.est && ct.rpl && ct_mark.blocked == 1)), action=(reg8[17] = 1; next;)
  table=4 (ls_out_acl_eval    ), priority=65532, match=(nd || nd_ra || nd_rs || mldv1 || mldv2), action=(reg8[16] = 1; next;)
  table=4 (ls_out_acl_eval    ), priority=34000, match=(eth.src == $svc_monitor_mac), action=(reg8[16] = 1; next;)
  table=4 (ls_out_acl_eval    ), priority=34000, match=(outport == "7b43909d-8ae8-4609-9872-811667056013" && eth.src == fa:16:3e:ca:64:02 && ip4.src == 172.16.0.254 && udp && udp.src == 67 && udp.dst == 68), action=(reg8[16] = 1; next;)
  table=4 (ls_out_acl_eval    ), priority=2002 , match=(reg0[7] == 1 && (outport == @pg_158f2c45_1393_46e4_8093_6e82e4d876c9 && ip4 && ip4.src == 0.0.0.0/0 && icmp4)), action=(reg8[16] = 1; reg0[1] = 1; next;)
  table=4 (ls_out_acl_eval    ), priority=2002 , match=(reg0[7] == 1 && (outport == @pg_158f2c45_1393_46e4_8093_6e82e4d876c9 && ip4 && ip4.src == 0.0.0.0/0 && tcp && tcp.dst == 22)), action=(reg8[16] = 1; reg0[1] = 1; next;)
  table=4 (ls_out_acl_eval    ), priority=2002 , match=(reg0[8] == 1 && (outport == @pg_158f2c45_1393_46e4_8093_6e82e4d876c9 && ip4 && ip4.src == 0.0.0.0/0 && icmp4)), action=(reg8[16] = 1; next;)
  table=4 (ls_out_acl_eval    ), priority=2002 , match=(reg0[8] == 1 && (outport == @pg_158f2c45_1393_46e4_8093_6e82e4d876c9 && ip4 && ip4.src == 0.0.0.0/0 && tcp && tcp.dst == 22)), action=(reg8[16] = 1; next;)
  table=4 (ls_out_acl_eval    ), priority=2001 , match=(reg0[10] == 1 && (outport == @neutron_pg_drop && ip)), action=(reg8[17] = 1; ct_commit { ct_mark.blocked = 1; }; next;)
  table=4 (ls_out_acl_eval    ), priority=2001 , match=(reg0[9] == 1 && (outport == @neutron_pg_drop && ip)), action=(reg8[17] = 1; next;)
  table=4 (ls_out_acl_eval    ), priority=1    , match=(ip && !ct.est), action=(reg0[1] = 1; next;)
  table=4 (ls_out_acl_eval    ), priority=1    , match=(ip && ct.est && ct_mark.blocked == 1), action=(reg0[1] = 1; reg8[16] = 1; next;)
  table=4 (ls_out_acl_eval    ), priority=0    , match=(1), action=(next;)
  table=5 (ls_out_acl_action  ), priority=1000 , match=(reg8[16] == 1), action=(reg8[16] = 0; reg8[17] = 0; reg8[18] = 0; next;)
  table=5 (ls_out_acl_action  ), priority=1000 , match=(reg8[17] == 1), action=(reg8[16] = 0; reg8[17] = 0; reg8[18] = 0; /* drop */)
  table=5 (ls_out_acl_action  ), priority=1000 , match=(reg8[18] == 1), action=(reg8[16] = 0; reg8[17] = 0; reg8[18] = 0; reg0 = 0; reject { /* eth.dst <-> eth.src; ip.dst <-> ip.src; is implicit. */ outport <-> inport; next(pipeline=ingress,table=27); };)
  table=5 (ls_out_acl_action  ), priority=0    , match=(1), action=(reg8[16] = 0; reg8[17] = 0; reg8[18] = 0; next;)
  table=6 (ls_out_qos_mark    ), priority=0    , match=(1), action=(next;)
  table=7 (ls_out_qos_meter   ), priority=0    , match=(1), action=(next;)
  table=8 (ls_out_stateful    ), priority=100  , match=(reg0[1] == 1 && reg0[13] == 0), action=(ct_commit { ct_mark.blocked = 0; }; next;)
  table=8 (ls_out_stateful    ), priority=100  , match=(reg0[1] == 1 && reg0[13] == 1), action=(ct_commit { ct_mark.blocked = 0; ct_label.label = reg3; }; next;)
  table=8 (ls_out_stateful    ), priority=0    , match=(1), action=(next;)
  table=9 (ls_out_check_port_sec), priority=100  , match=(eth.mcast), action=(reg0[15] = 0; next;)
  table=9 (ls_out_check_port_sec), priority=0    , match=(1), action=(reg0[15] = check_out_port_sec(); next;)
  table=10(ls_out_apply_port_sec), priority=50   , match=(reg0[15] == 1), action=(drop;)
  table=10(ls_out_apply_port_sec), priority=0    , match=(1), action=(output;)
Datapath: "neutron-f5e24ee8-2bcb-43a4-91d5-0abad0c58197" aka "selfservice" (a6c6f842-4aea-48ea-b485-1a363553a876)  Pipeline: ingress
  table=0 (ls_in_check_port_sec), priority=120  , match=(((ip4 && icmp4.type == 3 && icmp4.code == 4) || (ip6 && icmp6.type == 2 && icmp6.code == 0)) && eth.dst == fa:16:3e:35:8f:09 && flags.tunnel_rx == 1), action=(outport <-> inport; next(pipeline=ingress,table=27);)
  table=0 (ls_in_check_port_sec), priority=110  , match=(((ip4 && icmp4.type == 3 && icmp4.code == 4) || (ip6 && icmp6.type == 2 && icmp6.code == 0)) && flags.tunnel_rx == 1), action=(drop;)
  table=0 (ls_in_check_port_sec), priority=100  , match=(eth.src[40]), action=(drop;)
  table=0 (ls_in_check_port_sec), priority=100  , match=(vlan.present), action=(drop;)
  table=0 (ls_in_check_port_sec), priority=50   , match=(1), action=(reg0[15] = check_in_port_sec(); next;)
  table=1 (ls_in_apply_port_sec), priority=50   , match=(reg0[15] == 1), action=(drop;)
  table=1 (ls_in_apply_port_sec), priority=0    , match=(1), action=(next;)
  table=2 (ls_in_lookup_fdb   ), priority=0    , match=(1), action=(next;)
  table=3 (ls_in_put_fdb      ), priority=0    , match=(1), action=(next;)
  table=4 (ls_in_pre_acl      ), priority=110  , match=(eth.dst == $svc_monitor_mac), action=(next;)
  table=4 (ls_in_pre_acl      ), priority=110  , match=(eth.mcast), action=(next;)
  table=4 (ls_in_pre_acl      ), priority=110  , match=(ip && inport == "82afe6ae-c8f6-4b01-8833-f011bcf0dda9"), action=(next;)
  table=4 (ls_in_pre_acl      ), priority=110  , match=(nd || nd_rs || nd_ra || mldv1 || mldv2 || (udp && udp.src == 546 && udp.dst == 547)), action=(next;)
  table=4 (ls_in_pre_acl      ), priority=100  , match=(ip), action=(reg0[0] = 1; next;)
  table=4 (ls_in_pre_acl      ), priority=0    , match=(1), action=(next;)
  table=5 (ls_in_pre_lb       ), priority=110  , match=(eth.dst == $svc_monitor_mac), action=(next;)
  table=5 (ls_in_pre_lb       ), priority=110  , match=(eth.mcast), action=(next;)
  table=5 (ls_in_pre_lb       ), priority=110  , match=(ip && inport == "82afe6ae-c8f6-4b01-8833-f011bcf0dda9"), action=(next;)
  table=5 (ls_in_pre_lb       ), priority=110  , match=(nd || nd_rs || nd_ra || mldv1 || mldv2), action=(next;)
  table=5 (ls_in_pre_lb       ), priority=110  , match=(reg0[16] == 1), action=(next;)
  table=5 (ls_in_pre_lb       ), priority=0    , match=(1), action=(next;)
  table=6 (ls_in_pre_stateful ), priority=110  , match=(reg0[2] == 1), action=(ct_lb_mark;)
  table=6 (ls_in_pre_stateful ), priority=100  , match=(reg0[0] == 1), action=(ct_next;)
  table=6 (ls_in_pre_stateful ), priority=0    , match=(1), action=(next;)
  table=7 (ls_in_acl_hint     ), priority=7    , match=(ct.new && !ct.est), action=(reg0[7] = 1; reg0[9] = 1; next;)
  table=7 (ls_in_acl_hint     ), priority=6    , match=(!ct.new && ct.est && !ct.rpl && ct_mark.blocked == 1), action=(reg0[7] = 1; reg0[9] = 1; next;)
  table=7 (ls_in_acl_hint     ), priority=5    , match=(!ct.trk), action=(reg0[8] = 1; reg0[9] = 1; next;)
  table=7 (ls_in_acl_hint     ), priority=4    , match=(!ct.new && ct.est && !ct.rpl && ct_mark.blocked == 0), action=(reg0[8] = 1; reg0[10] = 1; next;)
  table=7 (ls_in_acl_hint     ), priority=3    , match=(!ct.est), action=(reg0[9] = 1; next;)
  table=7 (ls_in_acl_hint     ), priority=2    , match=(ct.est && ct_mark.blocked == 1), action=(reg0[9] = 1; next;)
  table=7 (ls_in_acl_hint     ), priority=1    , match=(ct.est && ct_mark.blocked == 0), action=(reg0[10] = 1; next;)
  table=7 (ls_in_acl_hint     ), priority=0    , match=(1), action=(next;)
  table=8 (ls_in_acl_eval     ), priority=65532, match=(!ct.est && ct.rel && !ct.new && !ct.inv && ct_mark.blocked == 0), action=(reg0[17] = 1; reg8[16] = 1; ct_commit_nat;)
  table=8 (ls_in_acl_eval     ), priority=65532, match=(ct.est && !ct.rel && !ct.new && !ct.inv && ct.rpl && ct_mark.blocked == 0), action=(reg0[9] = 0; reg0[10] = 0; reg0[17] = 1; reg8[16] = 1; next;)
  table=8 (ls_in_acl_eval     ), priority=65532, match=(ct.inv || (ct.est && ct.rpl && ct_mark.blocked == 1)), action=(reg8[17] = 1; next;)
  table=8 (ls_in_acl_eval     ), priority=65532, match=(nd || nd_ra || nd_rs || mldv1 || mldv2), action=(reg8[16] = 1; next;)
  table=8 (ls_in_acl_eval     ), priority=34000, match=(eth.dst == $svc_monitor_mac), action=(reg8[16] = 1; next;)
  table=8 (ls_in_acl_eval     ), priority=2002 , match=(reg0[7] == 1 && (inport == @pg_158f2c45_1393_46e4_8093_6e82e4d876c9 && ip4)), action=(reg8[16] = 1; reg0[1] = 1; next;)
  table=8 (ls_in_acl_eval     ), priority=2002 , match=(reg0[7] == 1 && (inport == @pg_158f2c45_1393_46e4_8093_6e82e4d876c9 && ip6)), action=(reg8[16] = 1; reg0[1] = 1; next;)
  table=8 (ls_in_acl_eval     ), priority=2002 , match=(reg0[8] == 1 && (inport == @pg_158f2c45_1393_46e4_8093_6e82e4d876c9 && ip4)), action=(reg8[16] = 1; next;)
  table=8 (ls_in_acl_eval     ), priority=2002 , match=(reg0[8] == 1 && (inport == @pg_158f2c45_1393_46e4_8093_6e82e4d876c9 && ip6)), action=(reg8[16] = 1; next;)
  table=8 (ls_in_acl_eval     ), priority=2001 , match=(reg0[10] == 1 && (inport == @neutron_pg_drop && ip)), action=(reg8[17] = 1; ct_commit { ct_mark.blocked = 1; }; next;)
  table=8 (ls_in_acl_eval     ), priority=2001 , match=(reg0[9] == 1 && (inport == @neutron_pg_drop && ip)), action=(reg8[17] = 1; next;)
  table=8 (ls_in_acl_eval     ), priority=1    , match=(ip && !ct.est), action=(reg0[1] = 1; next;)
  table=8 (ls_in_acl_eval     ), priority=1    , match=(ip && ct.est && ct_mark.blocked == 1), action=(reg0[1] = 1; reg8[16] = 1; next;)
  table=8 (ls_in_acl_eval     ), priority=0    , match=(1), action=(next;)
  table=9 (ls_in_acl_action   ), priority=1000 , match=(reg8[16] == 1), action=(reg8[16] = 0; reg8[17] = 0; reg8[18] = 0; next;)
  table=9 (ls_in_acl_action   ), priority=1000 , match=(reg8[17] == 1), action=(reg8[16] = 0; reg8[17] = 0; reg8[18] = 0; /* drop */)
  table=9 (ls_in_acl_action   ), priority=1000 , match=(reg8[18] == 1), action=(reg8[16] = 0; reg8[17] = 0; reg8[18] = 0; reg0 = 0; reject { /* eth.dst <-> eth.src; ip.dst <-> ip.src; is implicit. */ outport <-> inport; next(pipeline=egress,table=6); };)
  table=9 (ls_in_acl_action   ), priority=0    , match=(1), action=(reg8[16] = 0; reg8[17] = 0; reg8[18] = 0; next;)
  table=10(ls_in_qos_mark     ), priority=0    , match=(1), action=(next;)
  table=11(ls_in_qos_meter    ), priority=0    , match=(1), action=(next;)
  table=12(ls_in_lb_aff_check ), priority=0    , match=(1), action=(next;)
  table=13(ls_in_lb           ), priority=0    , match=(1), action=(next;)
  table=14(ls_in_lb_aff_learn ), priority=0    , match=(1), action=(next;)
  table=15(ls_in_pre_hairpin  ), priority=0    , match=(1), action=(next;)
  table=16(ls_in_nat_hairpin  ), priority=0    , match=(1), action=(next;)
  table=17(ls_in_hairpin      ), priority=0    , match=(1), action=(next;)
  table=18(ls_in_acl_after_lb_eval), priority=65532, match=(nd || nd_ra || nd_rs || mldv1 || mldv2), action=(reg8[16] = 1; next;)
  table=18(ls_in_acl_after_lb_eval), priority=65532, match=(reg0[17] == 1), action=(reg8[16] = 1; next;)
  table=18(ls_in_acl_after_lb_eval), priority=0    , match=(1), action=(next;)
  table=19(ls_in_acl_after_lb_action), priority=1000 , match=(reg8[16] == 1), action=(reg8[16] = 0; reg8[17] = 0; reg8[18] = 0; next;)
  table=19(ls_in_acl_after_lb_action), priority=1000 , match=(reg8[17] == 1), action=(reg8[16] = 0; reg8[17] = 0; reg8[18] = 0; /* drop */)
  table=19(ls_in_acl_after_lb_action), priority=1000 , match=(reg8[18] == 1), action=(reg8[16] = 0; reg8[17] = 0; reg8[18] = 0; reg0 = 0; reject { /* eth.dst <-> eth.src; ip.dst <-> ip.src; is implicit. */ outport <-> inport; next(pipeline=egress,table=6); };)
  table=19(ls_in_acl_after_lb_action), priority=0    , match=(1), action=(reg8[16] = 0; reg8[17] = 0; reg8[18] = 0; next;)
  table=20(ls_in_stateful     ), priority=100  , match=(reg0[1] == 1 && reg0[13] == 0), action=(ct_commit { ct_mark.blocked = 0; }; next;)
  table=20(ls_in_stateful     ), priority=100  , match=(reg0[1] == 1 && reg0[13] == 1), action=(ct_commit { ct_mark.blocked = 0; ct_label.label = reg3; }; next;)
  table=20(ls_in_stateful     ), priority=0    , match=(1), action=(next;)
  table=21(ls_in_arp_rsp      ), priority=100  , match=(arp.tpa == 192.168.101.1 && arp.op == 1 && inport == "47a2be8a-c336-42e0-a8a6-044ff5df6bf2"), action=(next;)
  table=21(ls_in_arp_rsp      ), priority=100  , match=(arp.tpa == 192.168.101.254 && arp.op == 1 && inport == "82afe6ae-c8f6-4b01-8833-f011bcf0dda9"), action=(next;)
  table=21(ls_in_arp_rsp      ), priority=100  , match=(nd_ns && ip6.dst == {fe80::f816:3eff:fe35:8f09, ff02::1:ff35:8f09} && nd.target == fe80::f816:3eff:fe35:8f09 && inport == "82afe6ae-c8f6-4b01-8833-f011bcf0dda9"), action=(next;)
  table=21(ls_in_arp_rsp      ), priority=50   , match=(arp.tpa == 192.168.101.1 && arp.op == 1), action=(eth.dst = eth.src; eth.src = fa:16:3e:71:9e:ad; arp.op = 2; /* ARP reply */ arp.tha = arp.sha; arp.sha = fa:16:3e:71:9e:ad; arp.tpa = arp.spa; arp.spa = 192.168.101.1; outport = inport; flags.loopback = 1; output;)
  table=21(ls_in_arp_rsp      ), priority=50   , match=(arp.tpa == 192.168.101.254 && arp.op == 1), action=(eth.dst = eth.src; eth.src = fa:16:3e:35:8f:09; arp.op = 2; /* ARP reply */ arp.tha = arp.sha; arp.sha = fa:16:3e:35:8f:09; arp.tpa = arp.spa; arp.spa = 192.168.101.254; outport = inport; flags.loopback = 1; output;)
  table=21(ls_in_arp_rsp      ), priority=50   , match=(nd_ns && ip6.dst == {fe80::f816:3eff:fe35:8f09, ff02::1:ff35:8f09} && nd.target == fe80::f816:3eff:fe35:8f09), action=(nd_na_router { eth.src = fa:16:3e:35:8f:09; ip6.src = fe80::f816:3eff:fe35:8f09; nd.target = fe80::f816:3eff:fe35:8f09; nd.tll = fa:16:3e:35:8f:09; outport = inport; flags.loopback = 1; output; };)
  table=21(ls_in_arp_rsp      ), priority=0    , match=(1), action=(next;)
  table=22(ls_in_dhcp_options ), priority=100  , match=(inport == "31d88d1e-0ebe-42aa-ae42-35c28c1abe33" && eth.src == fa:16:3e:63:d4:da && (ip4.src == {192.168.101.25, 0.0.0.0} && ip4.dst == {192.168.101.254, 255.255.255.255}) && udp.src == 68 && udp.dst == 67), action=(reg0[3] = put_dhcp_opts(offerip = 192.168.101.25, classless_static_route = {169.254.169.254/32,192.168.101.1, 0.0.0.0/0,192.168.101.254}, dns_server = {10.0.0.254}, lease_time = 43200, mtu = 1442, netmask = 255.255.255.0, router = 192.168.101.254, server_id = 192.168.101.254); next;)
  table=22(ls_in_dhcp_options ), priority=100  , match=(inport == "aec5a6ee-a55c-4626-ad4a-5e4715291954" && eth.src == fa:16:3e:bf:bb:c5 && (ip4.src == {192.168.101.162, 0.0.0.0} && ip4.dst == {192.168.101.254, 255.255.255.255}) && udp.src == 68 && udp.dst == 67), action=(reg0[3] = put_dhcp_opts(offerip = 192.168.101.162, classless_static_route = {169.254.169.254/32,192.168.101.1, 0.0.0.0/0,192.168.101.254}, dns_server = {10.0.0.254}, lease_time = 43200, mtu = 1442, netmask = 255.255.255.0, router = 192.168.101.254, server_id = 192.168.101.254); next;)
  table=22(ls_in_dhcp_options ), priority=0    , match=(1), action=(next;)
  table=23(ls_in_dhcp_response), priority=100  , match=(inport == "31d88d1e-0ebe-42aa-ae42-35c28c1abe33" && eth.src == fa:16:3e:63:d4:da && ip4 && udp.src == 68 && udp.dst == 67 && reg0[3]), action=(eth.dst = eth.src; eth.src = fa:16:3e:73:d1:b0; ip4.src = 192.168.101.254; udp.src = 67; udp.dst = 68; outport = inport; flags.loopback = 1; output;)
  table=23(ls_in_dhcp_response), priority=100  , match=(inport == "aec5a6ee-a55c-4626-ad4a-5e4715291954" && eth.src == fa:16:3e:bf:bb:c5 && ip4 && udp.src == 68 && udp.dst == 67 && reg0[3]), action=(eth.dst = eth.src; eth.src = fa:16:3e:73:d1:b0; ip4.src = 192.168.101.254; udp.src = 67; udp.dst = 68; outport = inport; flags.loopback = 1; output;)
  table=23(ls_in_dhcp_response), priority=0    , match=(1), action=(next;)
  table=24(ls_in_dns_lookup   ), priority=0    , match=(1), action=(next;)
  table=25(ls_in_dns_response ), priority=0    , match=(1), action=(next;)
  table=26(ls_in_external_port), priority=0    , match=(1), action=(next;)
  table=27(ls_in_l2_lkup      ), priority=110  , match=(eth.dst == $svc_monitor_mac && (tcp || icmp || icmp6)), action=(handle_svc_check(inport);)
  table=27(ls_in_l2_lkup      ), priority=80   , match=(flags[1] == 0 && arp.op == 1 && arp.tpa == 192.168.101.254), action=(clone {outport = "82afe6ae-c8f6-4b01-8833-f011bcf0dda9"; output; }; outport = "_MC_flood_l2"; output;)
  table=27(ls_in_l2_lkup      ), priority=80   , match=(flags[1] == 0 && nd_ns && nd.target == fe80::f816:3eff:fe35:8f09), action=(clone {outport = "82afe6ae-c8f6-4b01-8833-f011bcf0dda9"; output; }; outport = "_MC_flood_l2"; output;)
  table=27(ls_in_l2_lkup      ), priority=75   , match=(eth.src == {fa:16:3e:35:8f:09} && (arp.op == 1 || rarp.op == 3 || nd_ns)), action=(outport = "_MC_flood_l2"; output;)
  table=27(ls_in_l2_lkup      ), priority=70   , match=(eth.mcast), action=(outport = "_MC_flood"; output;)
  table=27(ls_in_l2_lkup      ), priority=50   , match=(eth.dst == fa:16:3e:35:8f:09), action=(outport = "82afe6ae-c8f6-4b01-8833-f011bcf0dda9"; output;)
  table=27(ls_in_l2_lkup      ), priority=50   , match=(eth.dst == fa:16:3e:63:d4:da), action=(outport = "31d88d1e-0ebe-42aa-ae42-35c28c1abe33"; output;)
  table=27(ls_in_l2_lkup      ), priority=50   , match=(eth.dst == fa:16:3e:71:9e:ad), action=(outport = "47a2be8a-c336-42e0-a8a6-044ff5df6bf2"; output;)
  table=27(ls_in_l2_lkup      ), priority=50   , match=(eth.dst == fa:16:3e:bf:bb:c5), action=(outport = "aec5a6ee-a55c-4626-ad4a-5e4715291954"; output;)
  table=27(ls_in_l2_lkup      ), priority=0    , match=(1), action=(outport = get_fdb(eth.dst); next;)
  table=28(ls_in_l2_unknown   ), priority=50   , match=(outport == "none"), action=(drop;)
  table=28(ls_in_l2_unknown   ), priority=0    , match=(1), action=(output;)
Datapath: "neutron-f5e24ee8-2bcb-43a4-91d5-0abad0c58197" aka "selfservice" (a6c6f842-4aea-48ea-b485-1a363553a876)  Pipeline: egress
  table=0 (ls_out_pre_acl     ), priority=110  , match=(eth.mcast), action=(next;)
  table=0 (ls_out_pre_acl     ), priority=110  , match=(eth.src == $svc_monitor_mac), action=(next;)
  table=0 (ls_out_pre_acl     ), priority=110  , match=(ip && outport == "82afe6ae-c8f6-4b01-8833-f011bcf0dda9"), action=(next;)
  table=0 (ls_out_pre_acl     ), priority=110  , match=(nd || nd_rs || nd_ra || mldv1 || mldv2 || (udp && udp.src == 546 && udp.dst == 547)), action=(next;)
  table=0 (ls_out_pre_acl     ), priority=100  , match=(ip), action=(reg0[0] = 1; next;)
  table=0 (ls_out_pre_acl     ), priority=0    , match=(1), action=(next;)
  table=1 (ls_out_pre_lb      ), priority=110  , match=(eth.mcast), action=(next;)
  table=1 (ls_out_pre_lb      ), priority=110  , match=(eth.src == $svc_monitor_mac), action=(next;)
  table=1 (ls_out_pre_lb      ), priority=110  , match=(ip && outport == "82afe6ae-c8f6-4b01-8833-f011bcf0dda9"), action=(next;)
  table=1 (ls_out_pre_lb      ), priority=110  , match=(nd || nd_rs || nd_ra || mldv1 || mldv2), action=(next;)
  table=1 (ls_out_pre_lb      ), priority=110  , match=(reg0[16] == 1), action=(next;)
  table=1 (ls_out_pre_lb      ), priority=0    , match=(1), action=(next;)
  table=2 (ls_out_pre_stateful), priority=110  , match=(reg0[2] == 1), action=(ct_lb_mark;)
  table=2 (ls_out_pre_stateful), priority=100  , match=(reg0[0] == 1), action=(ct_next;)
  table=2 (ls_out_pre_stateful), priority=0    , match=(1), action=(next;)
  table=3 (ls_out_acl_hint    ), priority=7    , match=(ct.new && !ct.est), action=(reg0[7] = 1; reg0[9] = 1; next;)
  table=3 (ls_out_acl_hint    ), priority=6    , match=(!ct.new && ct.est && !ct.rpl && ct_mark.blocked == 1), action=(reg0[7] = 1; reg0[9] = 1; next;)
  table=3 (ls_out_acl_hint    ), priority=5    , match=(!ct.trk), action=(reg0[8] = 1; reg0[9] = 1; next;)
  table=3 (ls_out_acl_hint    ), priority=4    , match=(!ct.new && ct.est && !ct.rpl && ct_mark.blocked == 0), action=(reg0[8] = 1; reg0[10] = 1; next;)
  table=3 (ls_out_acl_hint    ), priority=3    , match=(!ct.est), action=(reg0[9] = 1; next;)
  table=3 (ls_out_acl_hint    ), priority=2    , match=(ct.est && ct_mark.blocked == 1), action=(reg0[9] = 1; next;)
  table=3 (ls_out_acl_hint    ), priority=1    , match=(ct.est && ct_mark.blocked == 0), action=(reg0[10] = 1; next;)
  table=3 (ls_out_acl_hint    ), priority=0    , match=(1), action=(next;)
  table=4 (ls_out_acl_eval    ), priority=65532, match=(!ct.est && ct.rel && !ct.new && !ct.inv && ct_mark.blocked == 0), action=(reg8[16] = 1; ct_commit_nat;)
  table=4 (ls_out_acl_eval    ), priority=65532, match=(ct.est && !ct.rel && !ct.new && !ct.inv && ct.rpl && ct_mark.blocked == 0), action=(reg8[16] = 1; next;)
  table=4 (ls_out_acl_eval    ), priority=65532, match=(ct.inv || (ct.est && ct.rpl && ct_mark.blocked == 1)), action=(reg8[17] = 1; next;)
  table=4 (ls_out_acl_eval    ), priority=65532, match=(nd || nd_ra || nd_rs || mldv1 || mldv2), action=(reg8[16] = 1; next;)
  table=4 (ls_out_acl_eval    ), priority=34000, match=(eth.src == $svc_monitor_mac), action=(reg8[16] = 1; next;)
  table=4 (ls_out_acl_eval    ), priority=34000, match=(outport == "31d88d1e-0ebe-42aa-ae42-35c28c1abe33" && eth.src == fa:16:3e:73:d1:b0 && ip4.src == 192.168.101.254 && udp && udp.src == 67 && udp.dst == 68), action=(reg8[16] = 1; next;)
  table=4 (ls_out_acl_eval    ), priority=34000, match=(outport == "aec5a6ee-a55c-4626-ad4a-5e4715291954" && eth.src == fa:16:3e:73:d1:b0 && ip4.src == 192.168.101.254 && udp && udp.src == 67 && udp.dst == 68), action=(reg8[16] = 1; next;)
  table=4 (ls_out_acl_eval    ), priority=2002 , match=(reg0[7] == 1 && (outport == @pg_158f2c45_1393_46e4_8093_6e82e4d876c9 && ip4 && ip4.src == 0.0.0.0/0 && icmp4)), action=(reg8[16] = 1; reg0[1] = 1; next;)
  table=4 (ls_out_acl_eval    ), priority=2002 , match=(reg0[7] == 1 && (outport == @pg_158f2c45_1393_46e4_8093_6e82e4d876c9 && ip4 && ip4.src == 0.0.0.0/0 && tcp && tcp.dst == 22)), action=(reg8[16] = 1; reg0[1] = 1; next;)
  table=4 (ls_out_acl_eval    ), priority=2002 , match=(reg0[8] == 1 && (outport == @pg_158f2c45_1393_46e4_8093_6e82e4d876c9 && ip4 && ip4.src == 0.0.0.0/0 && icmp4)), action=(reg8[16] = 1; next;)
  table=4 (ls_out_acl_eval    ), priority=2002 , match=(reg0[8] == 1 && (outport == @pg_158f2c45_1393_46e4_8093_6e82e4d876c9 && ip4 && ip4.src == 0.0.0.0/0 && tcp && tcp.dst == 22)), action=(reg8[16] = 1; next;)
  table=4 (ls_out_acl_eval    ), priority=2001 , match=(reg0[10] == 1 && (outport == @neutron_pg_drop && ip)), action=(reg8[17] = 1; ct_commit { ct_mark.blocked = 1; }; next;)
  table=4 (ls_out_acl_eval    ), priority=2001 , match=(reg0[9] == 1 && (outport == @neutron_pg_drop && ip)), action=(reg8[17] = 1; next;)
  table=4 (ls_out_acl_eval    ), priority=1    , match=(ip && !ct.est), action=(reg0[1] = 1; next;)
  table=4 (ls_out_acl_eval    ), priority=1    , match=(ip && ct.est && ct_mark.blocked == 1), action=(reg0[1] = 1; reg8[16] = 1; next;)
  table=4 (ls_out_acl_eval    ), priority=0    , match=(1), action=(next;)
  table=5 (ls_out_acl_action  ), priority=1000 , match=(reg8[16] == 1), action=(reg8[16] = 0; reg8[17] = 0; reg8[18] = 0; next;)
  table=5 (ls_out_acl_action  ), priority=1000 , match=(reg8[17] == 1), action=(reg8[16] = 0; reg8[17] = 0; reg8[18] = 0; /* drop */)
  table=5 (ls_out_acl_action  ), priority=1000 , match=(reg8[18] == 1), action=(reg8[16] = 0; reg8[17] = 0; reg8[18] = 0; reg0 = 0; reject { /* eth.dst <-> eth.src; ip.dst <-> ip.src; is implicit. */ outport <-> inport; next(pipeline=ingress,table=27); };)
  table=5 (ls_out_acl_action  ), priority=0    , match=(1), action=(reg8[16] = 0; reg8[17] = 0; reg8[18] = 0; next;)
  table=6 (ls_out_qos_mark    ), priority=0    , match=(1), action=(next;)
  table=7 (ls_out_qos_meter   ), priority=0    , match=(1), action=(next;)
  table=8 (ls_out_stateful    ), priority=100  , match=(reg0[1] == 1 && reg0[13] == 0), action=(ct_commit { ct_mark.blocked = 0; }; next;)
  table=8 (ls_out_stateful    ), priority=100  , match=(reg0[1] == 1 && reg0[13] == 1), action=(ct_commit { ct_mark.blocked = 0; ct_label.label = reg3; }; next;)
  table=8 (ls_out_stateful    ), priority=0    , match=(1), action=(next;)
  table=9 (ls_out_check_port_sec), priority=100  , match=(eth.mcast), action=(reg0[15] = 0; next;)
  table=9 (ls_out_check_port_sec), priority=0    , match=(1), action=(reg0[15] = check_out_port_sec(); next;)
  table=10(ls_out_apply_port_sec), priority=50   , match=(reg0[15] == 1), action=(drop;)
  table=10(ls_out_apply_port_sec), priority=0    , match=(1), action=(output;)
```

マルチキャストグループを確認する。

```sh
ovn-sbctl list Multicast_Group
```

```
_uuid               : 148f8e38-662b-4759-8e9b-94a1179d5771
datapath            : fa81dc7d-3704-41ab-9e5b-266d1fddcfba
name                : _MC_flood_l2
ports               : [0ef38ad1-f38a-49ac-9a51-130566377fb5, 3c1e627e-3849-4a58-9d23-8db1a7288f6f, 856a3a05-69a1-4a7a-93a3-2d150b5cf40d]
tunnel_key          : 32772

_uuid               : 4d7eedea-f419-477c-824c-a9395a9add1d
datapath            : fa81dc7d-3704-41ab-9e5b-266d1fddcfba
name                : _MC_unknown
ports               : [3c1e627e-3849-4a58-9d23-8db1a7288f6f]
tunnel_key          : 32769

_uuid               : 7265e3e4-760f-460e-9a1a-78e54377f18d
datapath            : a6c6f842-4aea-48ea-b485-1a363553a876
name                : _MC_flood_l2
ports               : [9b5b3836-fae0-4746-9cbf-948916b44321, c64e5e70-4d92-4afc-858d-a02068a2adff, c697c99c-22ab-4acf-969b-a0692f0fb25e]
tunnel_key          : 32772

_uuid               : 97bbc64c-fe4a-41d9-95ab-f259f8724a55
datapath            : fa81dc7d-3704-41ab-9e5b-266d1fddcfba
name                : _MC_mrouter_flood
ports               : [3c1e627e-3849-4a58-9d23-8db1a7288f6f]
tunnel_key          : 32770

_uuid               : 3418c045-c8c3-4639-a709-2e1eeb597490
datapath            : a6c6f842-4aea-48ea-b485-1a363553a876
name                : _MC_flood
ports               : [9b5b3836-fae0-4746-9cbf-948916b44321, a481ae91-e2ad-4b8b-b0bf-5a066558baf0, c64e5e70-4d92-4afc-858d-a02068a2adff, c697c99c-22ab-4acf-969b-a0692f0fb25e]
tunnel_key          : 32768

_uuid               : 8712f73f-31b1-408d-a0f7-8a5f535b98ff
datapath            : fa81dc7d-3704-41ab-9e5b-266d1fddcfba
name                : _MC_flood
ports               : [0ef38ad1-f38a-49ac-9a51-130566377fb5, 3c1e627e-3849-4a58-9d23-8db1a7288f6f, 856a3a05-69a1-4a7a-93a3-2d150b5cf40d, a3a7b338-0656-4d89-bc9f-6c85de210eaf]
tunnel_key          : 32768
```

データパスを確認する。

```sh
ovn-sbctl list Datapath_Binding
```

```
_uuid               : 9e73dd0c-1d10-42d9-8264-d268e1c794e2
external_ids        : {always_learn_from_arp_request="false", logical-router="108cc266-6c52-4adc-817b-1ead115d7996", name=neutron-32ee6d25-076d-4dd3-9bed-87afb2cbf222, name2=router}
load_balancers      : []
tunnel_key          : 4

_uuid               : fa81dc7d-3704-41ab-9e5b-266d1fddcfba
external_ids        : {logical-switch="8f3ec596-41e0-43f2-a4b7-7befbab52e40", name=neutron-ec6ba68e-47d4-4ece-a233-718a902e68e6, name2=provider}
load_balancers      : []
tunnel_key          : 1

_uuid               : a6c6f842-4aea-48ea-b485-1a363553a876
external_ids        : {logical-switch="4618aa87-cb97-4480-b7d8-9e296ed6ead4", name=neutron-f5e24ee8-2bcb-43a4-91d5-0abad0c58197, name2=selfservice}
load_balancers      : []
tunnel_key          : 3
```

ポートを確認する。

```sh
ovn-sbctl list Port_Binding
```

```
_uuid               : a3a7b338-0656-4d89-bc9f-6c85de210eaf
additional_chassis  : []
additional_encap    : []
chassis             : []
datapath            : fa81dc7d-3704-41ab-9e5b-266d1fddcfba
encap               : []
external_ids        : {"neutron:cidrs"="172.16.0.168/24", "neutron:device_id"="32ee6d25-076d-4dd3-9bed-87afb2cbf222", "neutron:device_owner"="network:router_gateway", "neutron:mtu"="", "neutron:network_name"=neutron-ec6ba68e-47d4-4ece-a233-718a902e68e6, "neutron:port_capabilities"="", "neutron:port_name"="", "neutron:project_id"="", "neutron:revision_number"="1", "neutron:security_group_ids"="", "neutron:subnet_pool_addr_scope4"="", "neutron:subnet_pool_addr_scope6"="", "neutron:vnic_type"=normal}
gateway_chassis     : []
ha_chassis_group    : []
logical_port        : "3c5abf4a-5efc-4b6a-8d49-98d3f2cac0dd"
mac                 : [router]
mirror_rules        : []
nat_addresses       : []
options             : {peer=lrp-3c5abf4a-5efc-4b6a-8d49-98d3f2cac0dd}
parent_port         : []
port_security       : []
requested_additional_chassis: []
requested_chassis   : []
tag                 : []
tunnel_key          : 4
type                : patch
up                  : false
virtual_parent      : []

_uuid               : a481ae91-e2ad-4b8b-b0bf-5a066558baf0
additional_chassis  : []
additional_encap    : []
chassis             : []
datapath            : a6c6f842-4aea-48ea-b485-1a363553a876
encap               : []
external_ids        : {"neutron:cidrs"="192.168.101.254/24", "neutron:device_id"="32ee6d25-076d-4dd3-9bed-87afb2cbf222", "neutron:device_owner"="network:router_interface", "neutron:mtu"="", "neutron:network_name"=neutron-f5e24ee8-2bcb-43a4-91d5-0abad0c58197, "neutron:port_capabilities"="", "neutron:port_name"="", "neutron:project_id"=bccf406c045d401b91ba5c7552a124ae, "neutron:revision_number"="3", "neutron:security_group_ids"="", "neutron:subnet_pool_addr_scope4"="", "neutron:subnet_pool_addr_scope6"="", "neutron:vnic_type"=normal}
gateway_chassis     : []
ha_chassis_group    : []
logical_port        : "82afe6ae-c8f6-4b01-8833-f011bcf0dda9"
mac                 : [router]
mirror_rules        : []
nat_addresses       : []
options             : {peer=lrp-82afe6ae-c8f6-4b01-8833-f011bcf0dda9}
parent_port         : []
port_security       : []
requested_additional_chassis: []
requested_chassis   : []
tag                 : []
tunnel_key          : 5
type                : patch
up                  : false
virtual_parent      : []
```

#### Open vSwitch

Open vSwitch にブリッジは作成されない。

```sh
ovs-vsctl show
```

```
f6d63fd3-3bf4-45eb-afc6-603984ff8667
    ovs_version: "3.3.1"
```

### Compute Node

#### ネットワーク名前空間

ネットワーク名前空間は作成されない。

#### Open vSwitch

ブリッジの構成を確認する。

```sh
ovs-vsctl show
```

```
210309a0-2cc9-4a7f-96af-960f78c1c32d
    Bridge br-int
        fail_mode: secure
        datapath_type: system
        Port br-int
            Interface br-int
                type: internal
        Port tapf5e24ee8-20
            Interface tapf5e24ee8-20
        Port tap31d88d1e-0e
            Interface tap31d88d1e-0e
        Port patch-br-int-to-provnet-698fbbf6-da6f-4d59-8259-60d87eb5e697
            Interface patch-br-int-to-provnet-698fbbf6-da6f-4d59-8259-60d87eb5e697
                type: patch
                options: {peer=patch-provnet-698fbbf6-da6f-4d59-8259-60d87eb5e697-to-br-int}
        Port ovn-3bb30c-0
            Interface ovn-3bb30c-0
                type: geneve
                options: {csum="true", key=flow, remote_ip="172.16.0.32"}
    Bridge br-mgmt
        Port eth3
            Interface eth3
                type: system
    Bridge br-provider
        Port patch-provnet-698fbbf6-da6f-4d59-8259-60d87eb5e697-to-br-int
            Interface patch-provnet-698fbbf6-da6f-4d59-8259-60d87eb5e697-to-br-int
                type: patch
                options: {peer=patch-br-int-to-provnet-698fbbf6-da6f-4d59-8259-60d87eb5e697}
        Port eth2
            Interface eth2
                type: system
    ovs_version: "3.3.1"
```

データパスを確認する。

```sh
ovs-dpctl show
```

```
system@ovs-system:
  lookups: hit:910 missed:309 lost:0
  flows: 4
  masks: hit:2075 total:4 hit/pkt:1.70
  cache: hit:594 hit-rate:48.73%
  caches:
    masks-cache: size:256
  port 0: ovs-system (internal)
  port 1: eth2
  port 2: eth3
  port 3: br-int (internal)
  port 4: genev_sys_6081 (geneve: packet_type=ptap)
  port 5: tap31d88d1e-0e
  port 6: tapf5e24ee8-20
```

ブリッジ br-int のフローのエントリを確認する。

```sh
ovs-ofctl dump-flows br-int
```

```
 cookie=0x1b707d7, duration=297.879s, table=0, n_packets=0, n_bytes=0, priority=180,conj_id=100,in_port="patch-br-int-to",vlan_tci=0x0000/0x1000 actions=load:0x8->NXM_NX_REG13[0..15],load:0x6->NXM_NX_REG11[],load:0x7->NXM_NX_REG12[],load:0x1->OXM_OF_METADATA[],load:0x1->NXM_NX_REG14[],load:0->NXM_NX_REG13[16..31],mod_dl_src:fa:16:3e:15:d6:26,resubmit(,8)
 cookie=0x1b707d7, duration=297.879s, table=0, n_packets=0, n_bytes=0, priority=180,vlan_tci=0x0000/0x1000 actions=conjunction(100,2/2)
 cookie=0x0, duration=2225.463s, table=0, n_packets=0, n_bytes=0, priority=120,icmp6,in_port="ovn-3bb30c-0",icmp_type=2,icmp_code=0 actions=move:NXM_NX_TUN_ID[0..23]->OXM_OF_METADATA[0..23],move:NXM_NX_TUN_METADATA0[16..30]->NXM_NX_REG14[0..14],move:NXM_NX_TUN_METADATA0[0..15]->NXM_NX_REG15[0..15],resubmit(,40),load:0x1->NXM_NX_REG10[16],resubmit(,8)
 cookie=0x0, duration=2225.463s, table=0, n_packets=0, n_bytes=0, priority=120,icmp,in_port="ovn-3bb30c-0",icmp_type=3,icmp_code=4 actions=move:NXM_NX_TUN_ID[0..23]->OXM_OF_METADATA[0..23],move:NXM_NX_TUN_METADATA0[16..30]->NXM_NX_REG14[0..14],move:NXM_NX_TUN_METADATA0[0..15]->NXM_NX_REG15[0..15],resubmit(,40),load:0x1->NXM_NX_REG10[16],resubmit(,8)
 cookie=0x0, duration=2225.463s, table=0, n_packets=0, n_bytes=0, priority=100,in_port="ovn-3bb30c-0" actions=move:NXM_NX_TUN_ID[0..23]->OXM_OF_METADATA[0..23],move:NXM_NX_TUN_METADATA0[16..30]->NXM_NX_REG14[0..14],move:NXM_NX_TUN_METADATA0[0..15]->NXM_NX_REG15[0..15],resubmit(,40)
 cookie=0xc697c99c, duration=297.909s, table=0, n_packets=131, n_bytes=11562, priority=100,in_port="tap31d88d1e-0e" actions=load:0x2->NXM_NX_REG13[0..15],load:0x5->NXM_NX_REG11[],load:0x4->NXM_NX_REG12[],load:0x3->OXM_OF_METADATA[],load:0x2->NXM_NX_REG14[],load:0->NXM_NX_REG13[16..31],resubmit(,8)
 cookie=0x9b5b3836, duration=290.821s, table=0, n_packets=62, n_bytes=7091, priority=100,in_port="tapf5e24ee8-20" actions=load:0x9->NXM_NX_REG13[0..15],load:0x5->NXM_NX_REG11[],load:0x4->NXM_NX_REG12[],load:0x3->OXM_OF_METADATA[],load:0x1->NXM_NX_REG14[],load:0->NXM_NX_REG13[16..31],load:0x1->NXM_NX_REG10[10],resubmit(,8)
 cookie=0x3c1e627e, duration=297.879s, table=0, n_packets=0, n_bytes=0, priority=100,in_port="patch-br-int-to",dl_vlan=0 actions=strip_vlan,load:0x8->NXM_NX_REG13[0..15],load:0x6->NXM_NX_REG11[],load:0x7->NXM_NX_REG12[],load:0x1->OXM_OF_METADATA[],load:0x1->NXM_NX_REG14[],load:0->NXM_NX_REG13[16..31],resubmit(,8)
 cookie=0x3c1e627e, duration=297.879s, table=0, n_packets=28, n_bytes=5032, priority=100,in_port="patch-br-int-to",vlan_tci=0x0000/0x1000 actions=load:0x8->NXM_NX_REG13[0..15],load:0x6->NXM_NX_REG11[],load:0x7->NXM_NX_REG12[],load:0x1->OXM_OF_METADATA[],load:0x1->NXM_NX_REG14[],load:0->NXM_NX_REG13[16..31],resubmit(,8)
 cookie=0x0, duration=2294s, table=0, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0xe5be180, duration=297.901s, table=8, n_packets=0, n_bytes=0, priority=120,icmp,reg10=0x10000/0x10000,metadata=0x1,dl_dst=fa:16:3e:15:d6:26,icmp_type=3,icmp_code=4 actions=push:NXM_NX_REG14[],push:NXM_NX_REG15[],pop:NXM_NX_REG14[],pop:NXM_NX_REG15[],resubmit(,35)
 cookie=0xbbf002be, duration=297.884s, table=8, n_packets=0, n_bytes=0, priority=120,icmp,reg10=0x10000/0x10000,metadata=0x3,dl_dst=fa:16:3e:35:8f:09,icmp_type=3,icmp_code=4 actions=push:NXM_NX_REG14[],push:NXM_NX_REG15[],pop:NXM_NX_REG14[],pop:NXM_NX_REG15[],resubmit(,35)
 cookie=0xe5be180, duration=297.884s, table=8, n_packets=0, n_bytes=0, priority=120,icmp6,reg10=0x10000/0x10000,metadata=0x1,dl_dst=fa:16:3e:15:d6:26,icmp_type=2,icmp_code=0 actions=push:NXM_NX_REG14[],push:NXM_NX_REG15[],pop:NXM_NX_REG14[],pop:NXM_NX_REG15[],resubmit(,35)
 cookie=0xbbf002be, duration=297.883s, table=8, n_packets=0, n_bytes=0, priority=120,icmp6,reg10=0x10000/0x10000,metadata=0x3,dl_dst=fa:16:3e:35:8f:09,icmp_type=2,icmp_code=0 actions=push:NXM_NX_REG14[],push:NXM_NX_REG15[],pop:NXM_NX_REG14[],pop:NXM_NX_REG15[],resubmit(,35)
 cookie=0xe79cb8cc, duration=297.967s, table=8, n_packets=0, n_bytes=0, priority=110,icmp6,reg10=0x10000/0x10000,metadata=0x1,icmp_type=2,icmp_code=0 actions=drop
 cookie=0xe79cb8cc, duration=297.966s, table=8, n_packets=0, n_bytes=0, priority=110,icmp,reg10=0x10000/0x10000,metadata=0x1,icmp_type=3,icmp_code=4 actions=drop
 cookie=0xe79cb8cc, duration=297.909s, table=8, n_packets=0, n_bytes=0, priority=110,icmp6,reg10=0x10000/0x10000,metadata=0x3,icmp_type=2,icmp_code=0 actions=drop
 cookie=0xe79cb8cc, duration=297.909s, table=8, n_packets=0, n_bytes=0, priority=110,icmp,reg10=0x10000/0x10000,metadata=0x3,icmp_type=3,icmp_code=4 actions=drop
 cookie=0xe9a37b6c, duration=297.884s, table=8, n_packets=0, n_bytes=0, priority=110,icmp,reg10=0x10000/0x10000,metadata=0x4,icmp_type=3,icmp_code=4 actions=drop
 cookie=0xe9a37b6c, duration=297.883s, table=8, n_packets=0, n_bytes=0, priority=110,icmp6,reg10=0x10000/0x10000,metadata=0x4,icmp_type=2,icmp_code=0 actions=drop
 cookie=0x6fe066f3, duration=298.004s, table=8, n_packets=0, n_bytes=0, priority=100,metadata=0x1,vlan_tci=0x1000/0x1000 actions=drop
 cookie=0x6fe066f3, duration=297.914s, table=8, n_packets=0, n_bytes=0, priority=100,metadata=0x3,vlan_tci=0x1000/0x1000 actions=drop
 cookie=0xe7b57a37, duration=297.883s, table=8, n_packets=0, n_bytes=0, priority=100,metadata=0x4,vlan_tci=0x1000/0x1000 actions=drop
 cookie=0xc6a42719, duration=297.967s, table=8, n_packets=0, n_bytes=0, priority=100,metadata=0x1,dl_src=01:00:00:00:00:00/01:00:00:00:00:00 actions=drop
 cookie=0xc6a42719, duration=297.911s, table=8, n_packets=0, n_bytes=0, priority=100,metadata=0x3,dl_src=01:00:00:00:00:00/01:00:00:00:00:00 actions=drop
 cookie=0xe7b57a37, duration=297.884s, table=8, n_packets=0, n_bytes=0, priority=100,metadata=0x4,dl_src=01:00:00:00:00:00/01:00:00:00:00:00 actions=drop
 cookie=0x9b517dee, duration=298.004s, table=8, n_packets=42, n_bytes=6292, priority=50,metadata=0x1 actions=load:0->NXM_NX_REG10[12],resubmit(,73),move:NXM_NX_REG10[12]->NXM_NX_XXREG0[111],resubmit(,9)
 cookie=0x9b517dee, duration=297.911s, table=8, n_packets=196, n_bytes=18947, priority=50,metadata=0x3 actions=load:0->NXM_NX_REG10[12],resubmit(,73),move:NXM_NX_REG10[12]->NXM_NX_XXREG0[111],resubmit(,9)
 cookie=0x5de99443, duration=297.883s, table=8, n_packets=0, n_bytes=0, priority=0,metadata=0x4 actions=drop
 cookie=0x6de3094f, duration=297.884s, table=8, n_packets=2, n_bytes=84, priority=50,reg14=0x2,metadata=0x4,dl_dst=fa:16:3e:15:d6:26 actions=load:0xfa163e15d626->NXM_NX_XXREG0[64..111],resubmit(,9)
 cookie=0x751f1d2c, duration=297.883s, table=8, n_packets=15, n_bytes=1470, priority=50,reg14=0x1,metadata=0x4,dl_dst=fa:16:3e:35:8f:09 actions=load:0xfa163e358f09->NXM_NX_XXREG0[64..111],resubmit(,9)
 cookie=0x19e2d6dc, duration=297.884s, table=8, n_packets=2, n_bytes=84, priority=50,reg14=0x1,metadata=0x4,dl_dst=01:00:00:00:00:00/01:00:00:00:00:00 actions=load:0xfa163e358f09->NXM_NX_XXREG0[64..111],resubmit(,9)
 cookie=0x764e8bdb, duration=297.882s, table=8, n_packets=26, n_bytes=4948, priority=50,reg14=0x2,metadata=0x4,dl_dst=01:00:00:00:00:00/01:00:00:00:00:00 actions=load:0xfa163e15d626->NXM_NX_XXREG0[64..111],resubmit(,9)
 cookie=0xa39dcfd9, duration=297.884s, table=9, n_packets=0, n_bytes=0, priority=110,icmp6,metadata=0x4,ipv6_src=fe80::/10,ipv6_dst=ff00::/8,nw_ttl=255,icmp_type=136,icmp_code=0 actions=push:NXM_NX_XXREG0[],push:NXM_OF_ETH_SRC[],push:NXM_NX_ND_TLL[],push:NXM_NX_ND_TARGET[],pop:NXM_NX_XXREG0[],pop:NXM_OF_ETH_SRC[],load:0->NXM_NX_REG10[6],resubmit(,67),move:NXM_NX_REG10[6]->OXM_OF_PKT_REG4[2],pop:NXM_OF_ETH_SRC[],pop:NXM_NX_XXREG0[],push:NXM_NX_REG15[],push:NXM_NX_XXREG0[],push:NXM_NX_ND_TARGET[],push:NXM_NX_REG14[],pop:NXM_NX_REG15[],pop:NXM_NX_XXREG0[],push:NXM_OF_ETH_DST[],load:0->NXM_NX_REG10[6],resubmit(,66),move:NXM_NX_REG10[6]->OXM_OF_PKT_REG4[3],pop:NXM_OF_ETH_DST[],pop:NXM_NX_XXREG0[],pop:NXM_NX_REG15[],resubmit(,10)
 cookie=0x8f1f39da, duration=297.884s, table=9, n_packets=0, n_bytes=0, priority=110,arp,reg14=0x1,metadata=0x4,arp_spa=192.168.101.0/24,arp_tpa=192.168.101.254,arp_op=1 actions=push:NXM_NX_REG0[],push:NXM_OF_ETH_SRC[],push:NXM_NX_ARP_SHA[],push:NXM_OF_ARP_SPA[],pop:NXM_NX_REG0[],pop:NXM_OF_ETH_SRC[],load:0->NXM_NX_REG10[6],resubmit(,67),move:NXM_NX_REG10[6]->OXM_OF_PKT_REG4[2],pop:NXM_OF_ETH_SRC[],pop:NXM_NX_REG0[],load:0x1->OXM_OF_PKT_REG4[3],resubmit(,10)
 cookie=0xe4154e89, duration=297.884s, table=9, n_packets=0, n_bytes=0, priority=110,arp,reg14=0x2,metadata=0x4,arp_spa=172.16.0.0/24,arp_tpa=172.16.0.168,arp_op=1 actions=push:NXM_NX_REG0[],push:NXM_OF_ETH_SRC[],push:NXM_NX_ARP_SHA[],push:NXM_OF_ARP_SPA[],pop:NXM_NX_REG0[],pop:NXM_OF_ETH_SRC[],load:0->NXM_NX_REG10[6],resubmit(,67),move:NXM_NX_REG10[6]->OXM_OF_PKT_REG4[2],pop:NXM_OF_ETH_SRC[],pop:NXM_NX_REG0[],load:0x1->OXM_OF_PKT_REG4[3],resubmit(,10)
 cookie=0xa75ca2df, duration=297.884s, table=9, n_packets=0, n_bytes=0, priority=100,icmp6,metadata=0x4,nw_ttl=255,icmp_type=136,icmp_code=0 actions=push:NXM_NX_XXREG0[],push:NXM_OF_ETH_SRC[],push:NXM_NX_ND_TLL[],push:NXM_NX_ND_TARGET[],pop:NXM_NX_XXREG0[],pop:NXM_OF_ETH_SRC[],load:0->NXM_NX_REG10[6],resubmit(,67),move:NXM_NX_REG10[6]->OXM_OF_PKT_REG4[2],pop:NXM_OF_ETH_SRC[],pop:NXM_NX_XXREG0[],load:0x1->OXM_OF_PKT_REG4[3],resubmit(,10)
 cookie=0x3bfbcc41, duration=297.883s, table=9, n_packets=1, n_bytes=86, priority=100,icmp6,metadata=0x4,nw_ttl=255,icmp_type=135,icmp_code=0 actions=push:NXM_NX_XXREG0[],push:NXM_OF_ETH_SRC[],push:NXM_NX_ND_SLL[],push:NXM_NX_IPV6_SRC[],pop:NXM_NX_XXREG0[],pop:NXM_OF_ETH_SRC[],load:0->NXM_NX_REG10[6],resubmit(,67),move:NXM_NX_REG10[6]->OXM_OF_PKT_REG4[2],pop:NXM_OF_ETH_SRC[],pop:NXM_NX_XXREG0[],push:NXM_NX_REG15[],push:NXM_NX_XXREG0[],push:NXM_NX_IPV6_SRC[],push:NXM_NX_REG14[],pop:NXM_NX_REG15[],pop:NXM_NX_XXREG0[],push:NXM_OF_ETH_DST[],load:0->NXM_NX_REG10[6],resubmit(,66),move:NXM_NX_REG10[6]->OXM_OF_PKT_REG4[3],pop:NXM_OF_ETH_DST[],pop:NXM_NX_XXREG0[],pop:NXM_NX_REG15[],resubmit(,10)
 cookie=0xb518107, duration=297.884s, table=9, n_packets=2, n_bytes=84, priority=100,arp,reg14=0x1,metadata=0x4,arp_spa=192.168.101.0/24,arp_op=1 actions=push:NXM_NX_REG0[],push:NXM_OF_ETH_SRC[],push:NXM_NX_ARP_SHA[],push:NXM_OF_ARP_SPA[],pop:NXM_NX_REG0[],pop:NXM_OF_ETH_SRC[],load:0->NXM_NX_REG10[6],resubmit(,67),move:NXM_NX_REG10[6]->OXM_OF_PKT_REG4[2],pop:NXM_OF_ETH_SRC[],pop:NXM_NX_REG0[],push:NXM_NX_REG15[],push:NXM_NX_REG0[],push:NXM_OF_ARP_SPA[],push:NXM_NX_REG14[],pop:NXM_NX_REG15[],pop:NXM_NX_REG0[],push:NXM_OF_ETH_DST[],load:0->NXM_NX_REG10[6],resubmit(,66),move:NXM_NX_REG10[6]->OXM_OF_PKT_REG4[3],pop:NXM_OF_ETH_DST[],pop:NXM_NX_REG0[],pop:NXM_NX_REG15[],resubmit(,10)
 cookie=0x5b715e1f, duration=297.883s, table=9, n_packets=0, n_bytes=0, priority=100,arp,reg14=0x2,metadata=0x4,arp_spa=172.16.0.0/24,arp_op=1 actions=push:NXM_NX_REG0[],push:NXM_OF_ETH_SRC[],push:NXM_NX_ARP_SHA[],push:NXM_OF_ARP_SPA[],pop:NXM_NX_REG0[],pop:NXM_OF_ETH_SRC[],load:0->NXM_NX_REG10[6],resubmit(,67),move:NXM_NX_REG10[6]->OXM_OF_PKT_REG4[2],pop:NXM_OF_ETH_SRC[],pop:NXM_NX_REG0[],push:NXM_NX_REG15[],push:NXM_NX_REG0[],push:NXM_OF_ARP_SPA[],push:NXM_NX_REG14[],pop:NXM_NX_REG15[],pop:NXM_NX_REG0[],push:NXM_OF_ETH_DST[],load:0->NXM_NX_REG10[6],resubmit(,66),move:NXM_NX_REG10[6]->OXM_OF_PKT_REG4[3],pop:NXM_OF_ETH_DST[],pop:NXM_NX_REG0[],pop:NXM_NX_REG15[],resubmit(,10)
 cookie=0x4cabb6d7, duration=297.883s, table=9, n_packets=2, n_bytes=84, priority=100,arp,metadata=0x4,arp_op=2 actions=push:NXM_NX_REG0[],push:NXM_OF_ETH_SRC[],push:NXM_NX_ARP_SHA[],push:NXM_OF_ARP_SPA[],pop:NXM_NX_REG0[],pop:NXM_OF_ETH_SRC[],load:0->NXM_NX_REG10[6],resubmit(,67),move:NXM_NX_REG10[6]->OXM_OF_PKT_REG4[2],pop:NXM_OF_ETH_SRC[],pop:NXM_NX_REG0[],load:0x1->OXM_OF_PKT_REG4[3],resubmit(,10)
 cookie=0xd524109e, duration=297.967s, table=9, n_packets=0, n_bytes=0, priority=50,reg0=0x8000/0x8000,metadata=0x1 actions=drop
 cookie=0xd524109e, duration=297.909s, table=9, n_packets=12, n_bytes=852, priority=50,reg0=0x8000/0x8000,metadata=0x3 actions=drop
 cookie=0x50d7abc2, duration=298.005s, table=9, n_packets=42, n_bytes=6292, priority=0,metadata=0x1 actions=resubmit(,10)
 cookie=0x50d7abc2, duration=297.914s, table=9, n_packets=184, n_bytes=18095, priority=0,metadata=0x3 actions=resubmit(,10)
 cookie=0x76e2cfe5, duration=297.884s, table=9, n_packets=40, n_bytes=6332, priority=0,metadata=0x4 actions=load:0x1->OXM_OF_PKT_REG4[2],resubmit(,10)
 cookie=0xdd819e5a, duration=297.882s, table=10, n_packets=43, n_bytes=6502, priority=100,reg9=0/0x8,metadata=0x4 actions=resubmit(,79),resubmit(,11)
 cookie=0xdd819e5a, duration=297.882s, table=10, n_packets=0, n_bytes=0, priority=100,reg9=0x4/0x4,metadata=0x4 actions=resubmit(,79),resubmit(,11)
 cookie=0xb5a810cb, duration=297.883s, table=10, n_packets=0, n_bytes=0, priority=95,icmp6,metadata=0x4,ipv6_src=::,nw_ttl=255,icmp_type=135,icmp_code=0 actions=resubmit(,11)
 cookie=0x8432270, duration=297.883s, table=10, n_packets=0, n_bytes=0, priority=95,icmp6,metadata=0x4,nw_ttl=255,icmp_type=136,icmp_code=0,nd_tll=00:00:00:00:00:00 actions=push:NXM_NX_XXREG0[],push:NXM_NX_ND_TARGET[],pop:NXM_NX_XXREG0[],controller(userdata=00.00.00.04.00.00.00.00),pop:NXM_NX_XXREG0[],resubmit(,11)
 cookie=0xb5a810cb, duration=297.883s, table=10, n_packets=0, n_bytes=0, priority=95,icmp6,metadata=0x4,nw_ttl=255,icmp_type=135,icmp_code=0,nd_sll=00:00:00:00:00:00 actions=resubmit(,11)
 cookie=0x2bf787fd, duration=297.884s, table=10, n_packets=0, n_bytes=0, priority=90,icmp6,metadata=0x4,nw_ttl=255,icmp_type=136,icmp_code=0 actions=push:NXM_NX_XXREG0[],push:NXM_OF_ETH_SRC[],push:NXM_NX_ND_TLL[],push:NXM_NX_ND_TARGET[],pop:NXM_NX_XXREG0[],pop:NXM_OF_ETH_SRC[],controller(userdata=00.00.00.04.00.00.00.00),pop:NXM_OF_ETH_SRC[],pop:NXM_NX_XXREG0[],resubmit(,11)
 cookie=0xaee3f3e, duration=297.882s, table=10, n_packets=0, n_bytes=0, priority=90,icmp6,metadata=0x4,nw_ttl=255,icmp_type=135,icmp_code=0 actions=push:NXM_NX_XXREG0[],push:NXM_OF_ETH_SRC[],push:NXM_NX_ND_SLL[],push:NXM_NX_IPV6_SRC[],pop:NXM_NX_XXREG0[],pop:NXM_OF_ETH_SRC[],controller(userdata=00.00.00.04.00.00.00.00),pop:NXM_OF_ETH_SRC[],pop:NXM_NX_XXREG0[],resubmit(,11)
 cookie=0x83c33055, duration=297.883s, table=10, n_packets=2, n_bytes=84, priority=90,arp,metadata=0x4 actions=push:NXM_NX_REG0[],push:NXM_OF_ETH_SRC[],push:NXM_NX_ARP_SHA[],push:NXM_OF_ARP_SPA[],pop:NXM_NX_REG0[],pop:NXM_OF_ETH_SRC[],controller(userdata=00.00.00.01.00.00.00.00),pop:NXM_OF_ETH_SRC[],pop:NXM_NX_REG0[],resubmit(,11)
 cookie=0x94d4f04, duration=298.021s, table=10, n_packets=42, n_bytes=6292, priority=0,metadata=0x1 actions=resubmit(,11)
 cookie=0x94d4f04, duration=297.934s, table=10, n_packets=184, n_bytes=18095, priority=0,metadata=0x3 actions=resubmit(,11)
 cookie=0xf35ad4a3, duration=297.884s, table=10, n_packets=0, n_bytes=0, priority=0,metadata=0x4 actions=drop
 cookie=0xaa733c67, duration=297.902s, table=11, n_packets=0, n_bytes=0, priority=90,icmp6,metadata=0x4,ipv6_dst=fe80::f816:3eff:fe15:d626,icmp_type=128,icmp_code=0 actions=push:NXM_NX_IPV6_SRC[],push:NXM_NX_IPV6_DST[],pop:NXM_NX_IPV6_SRC[],pop:NXM_NX_IPV6_DST[],load:0xff->NXM_NX_IP_TTL[],load:0x81->NXM_NX_ICMPV6_TYPE[],load:0x1->NXM_NX_REG10[0],resubmit(,12)
 cookie=0x290d3b3d, duration=297.884s, table=11, n_packets=0, n_bytes=0, priority=100,udp6,metadata=0x4,ipv6_dst=fe80::f816:3eff:fe15:d626,tp_src=547,tp_dst=546 actions=load:0->NXM_NX_XXREG0[96..127],controller(userdata=00.00.00.13.00.00.00.00)
 cookie=0x2dcc780b, duration=297.884s, table=11, n_packets=0, n_bytes=0, priority=90,icmp6,metadata=0x4,ipv6_dst=fe80::f816:3eff:fe35:8f09,icmp_type=128,icmp_code=0 actions=push:NXM_NX_IPV6_SRC[],push:NXM_NX_IPV6_DST[],pop:NXM_NX_IPV6_SRC[],pop:NXM_NX_IPV6_DST[],load:0xff->NXM_NX_IP_TTL[],load:0x81->NXM_NX_ICMPV6_TYPE[],load:0x1->NXM_NX_REG10[0],resubmit(,12)
 cookie=0x25a5aa4d, duration=297.884s, table=11, n_packets=0, n_bytes=0, priority=100,udp6,metadata=0x4,ipv6_dst=fe80::f816:3eff:fe35:8f09,tp_src=547,tp_dst=546 actions=load:0->NXM_NX_XXREG0[96..127],controller(userdata=00.00.00.13.00.00.00.00)
 cookie=0x25ea5baa, duration=297.885s, table=11, n_packets=0, n_bytes=0, priority=100,ip,metadata=0x4,nw_src=224.0.0.0/4 actions=drop
 cookie=0x25ea5baa, duration=297.885s, table=11, n_packets=0, n_bytes=0, priority=100,ip,metadata=0x4,nw_src=255.255.255.255 actions=drop
 cookie=0x25ea5baa, duration=297.885s, table=11, n_packets=0, n_bytes=0, priority=100,ip,metadata=0x4,nw_dst=0.0.0.0/8 actions=drop
 cookie=0x25ea5baa, duration=297.883s, table=11, n_packets=0, n_bytes=0, priority=100,ip,metadata=0x4,nw_dst=127.0.0.0/8 actions=drop
 cookie=0x24124c59, duration=297.885s, table=11, n_packets=0, n_bytes=0, priority=100,ip,reg9=0/0x1,metadata=0x4,nw_src=172.16.0.168 actions=drop
 cookie=0x7a62cfee, duration=297.884s, table=11, n_packets=0, n_bytes=0, priority=100,ip,reg9=0/0x1,metadata=0x4,nw_src=192.168.101.254 actions=drop
 cookie=0x24124c59, duration=297.884s, table=11, n_packets=0, n_bytes=0, priority=100,ip,reg9=0/0x1,metadata=0x4,nw_src=172.16.0.255 actions=drop
 cookie=0x7a62cfee, duration=297.883s, table=11, n_packets=0, n_bytes=0, priority=100,ip,reg9=0/0x1,metadata=0x4,nw_src=192.168.101.255 actions=drop
 cookie=0x25ea5baa, duration=297.884s, table=11, n_packets=5, n_bytes=1650, priority=100,ip,metadata=0x4,nw_src=0.0.0.0/8 actions=drop
 cookie=0x25ea5baa, duration=297.883s, table=11, n_packets=0, n_bytes=0, priority=100,ip,metadata=0x4,nw_src=127.0.0.0/8 actions=drop
 cookie=0x41612e99, duration=297.885s, table=11, n_packets=0, n_bytes=0, priority=90,icmp6,reg14=0x2,metadata=0x4,ipv6_dst=fe80::f816:3eff:fe15:d626,nw_ttl=255,icmp_type=135,icmp_code=0,nd_target=fe80::f816:3eff:fe15:d626 actions=controller(userdata=00.00.00.0c.00.00.00.00.00.1c.00.18.00.30.00.40.00.00.00.00.00.01.de.10.80.00.08.06.00.00.00.00.00.1c.00.18.00.80.00.00.00.00.00.00.80.00.3e.10.80.00.34.10.00.00.00.00.00.1c.00.18.00.30.00.40.00.00.00.00.00.01.de.10.80.00.42.06.00.00.00.00.00.1c.00.18.00.20.00.00.00.00.00.00.00.01.1c.04.00.01.1e.04.00.00.00.00.00.19.00.10.00.01.15.08.00.00.00.01.00.00.00.01.ff.ff.00.10.00.00.23.20.00.0e.ff.f8.25.00.00.00)
 cookie=0x81e9690a, duration=297.884s, table=11, n_packets=0, n_bytes=0, priority=90,icmp6,reg14=0x1,metadata=0x4,ipv6_dst=ff02::1:ff35:8f09,nw_ttl=255,icmp_type=135,icmp_code=0,nd_target=fe80::f816:3eff:fe35:8f09 actions=controller(userdata=00.00.00.0c.00.00.00.00.00.1c.00.18.00.30.00.40.00.00.00.00.00.01.de.10.80.00.08.06.00.00.00.00.00.1c.00.18.00.80.00.00.00.00.00.00.80.00.3e.10.80.00.34.10.00.00.00.00.00.1c.00.18.00.30.00.40.00.00.00.00.00.01.de.10.80.00.42.06.00.00.00.00.00.1c.00.18.00.20.00.00.00.00.00.00.00.01.1c.04.00.01.1e.04.00.00.00.00.00.19.00.10.00.01.15.08.00.00.00.01.00.00.00.01.ff.ff.00.10.00.00.23.20.00.0e.ff.f8.25.00.00.00)
 cookie=0x81e9690a, duration=297.884s, table=11, n_packets=0, n_bytes=0, priority=90,icmp6,reg14=0x1,metadata=0x4,ipv6_dst=fe80::f816:3eff:fe35:8f09,nw_ttl=255,icmp_type=135,icmp_code=0,nd_target=fe80::f816:3eff:fe35:8f09 actions=controller(userdata=00.00.00.0c.00.00.00.00.00.1c.00.18.00.30.00.40.00.00.00.00.00.01.de.10.80.00.08.06.00.00.00.00.00.1c.00.18.00.80.00.00.00.00.00.00.80.00.3e.10.80.00.34.10.00.00.00.00.00.1c.00.18.00.30.00.40.00.00.00.00.00.01.de.10.80.00.42.06.00.00.00.00.00.1c.00.18.00.20.00.00.00.00.00.00.00.01.1c.04.00.01.1e.04.00.00.00.00.00.19.00.10.00.01.15.08.00.00.00.01.00.00.00.01.ff.ff.00.10.00.00.23.20.00.0e.ff.f8.25.00.00.00)
 cookie=0x41612e99, duration=297.883s, table=11, n_packets=0, n_bytes=0, priority=90,icmp6,reg14=0x2,metadata=0x4,ipv6_dst=ff02::1:ff15:d626,nw_ttl=255,icmp_type=135,icmp_code=0,nd_target=fe80::f816:3eff:fe15:d626 actions=controller(userdata=00.00.00.0c.00.00.00.00.00.1c.00.18.00.30.00.40.00.00.00.00.00.01.de.10.80.00.08.06.00.00.00.00.00.1c.00.18.00.80.00.00.00.00.00.00.80.00.3e.10.80.00.34.10.00.00.00.00.00.1c.00.18.00.30.00.40.00.00.00.00.00.01.de.10.80.00.42.06.00.00.00.00.00.1c.00.18.00.20.00.00.00.00.00.00.00.01.1c.04.00.01.1e.04.00.00.00.00.00.19.00.10.00.01.15.08.00.00.00.01.00.00.00.01.ff.ff.00.10.00.00.23.20.00.0e.ff.f8.25.00.00.00)
 cookie=0xc6100ae0, duration=297.885s, table=11, n_packets=3, n_bytes=294, priority=90,icmp,metadata=0x4,nw_dst=192.168.101.254,icmp_type=8,icmp_code=0 actions=push:NXM_OF_IP_SRC[],push:NXM_OF_IP_DST[],pop:NXM_OF_IP_SRC[],pop:NXM_OF_IP_DST[],load:0xff->NXM_NX_IP_TTL[],load:0->NXM_OF_ICMP_TYPE[],load:0x1->NXM_NX_REG10[0],resubmit(,12)
 cookie=0x95c4827f, duration=297.885s, table=11, n_packets=0, n_bytes=0, priority=90,icmp,metadata=0x4,nw_dst=172.16.0.168,icmp_type=8,icmp_code=0 actions=push:NXM_OF_IP_SRC[],push:NXM_OF_IP_DST[],pop:NXM_OF_IP_SRC[],pop:NXM_OF_IP_DST[],load:0xff->NXM_NX_IP_TTL[],load:0->NXM_OF_ICMP_TYPE[],load:0x1->NXM_NX_REG10[0],resubmit(,12)
 cookie=0x7c39240e, duration=297.885s, table=11, n_packets=0, n_bytes=0, priority=90,arp,metadata=0x4,arp_tpa=172.16.0.168,arp_op=1 actions=move:NXM_OF_ETH_SRC[]->NXM_OF_ETH_DST[],move:NXM_NX_XXREG0[64..111]->NXM_OF_ETH_SRC[],load:0x2->NXM_OF_ARP_OP[],move:NXM_NX_ARP_SHA[]->NXM_NX_ARP_THA[],move:NXM_NX_XXREG0[64..111]->NXM_NX_ARP_SHA[],push:NXM_OF_ARP_SPA[],push:NXM_OF_ARP_TPA[],pop:NXM_OF_ARP_SPA[],pop:NXM_OF_ARP_TPA[],move:NXM_NX_REG14[]->NXM_NX_REG15[],load:0x1->NXM_NX_REG10[0],resubmit(,37)
 cookie=0xd244aae2, duration=297.884s, table=11, n_packets=0, n_bytes=0, priority=90,arp,reg14=0x1,metadata=0x4,arp_spa=192.168.101.0/24,arp_tpa=192.168.101.254,arp_op=1 actions=move:NXM_OF_ETH_SRC[]->NXM_OF_ETH_DST[],move:NXM_NX_XXREG0[64..111]->NXM_OF_ETH_SRC[],load:0x2->NXM_OF_ARP_OP[],move:NXM_NX_ARP_SHA[]->NXM_NX_ARP_THA[],move:NXM_NX_XXREG0[64..111]->NXM_NX_ARP_SHA[],push:NXM_OF_ARP_SPA[],push:NXM_OF_ARP_TPA[],pop:NXM_OF_ARP_SPA[],pop:NXM_OF_ARP_TPA[],move:NXM_NX_REG14[]->NXM_NX_REG15[],load:0x1->NXM_NX_REG10[0],resubmit(,37)
 cookie=0x41d652f0, duration=297.884s, table=11, n_packets=0, n_bytes=0, priority=90,arp,reg14=0x2,metadata=0x4,arp_spa=172.16.0.0/24,arp_tpa=172.16.0.168,arp_op=1 actions=move:NXM_OF_ETH_SRC[]->NXM_OF_ETH_DST[],move:NXM_NX_XXREG0[64..111]->NXM_OF_ETH_SRC[],load:0x2->NXM_OF_ARP_OP[],move:NXM_NX_ARP_SHA[]->NXM_NX_ARP_THA[],move:NXM_NX_XXREG0[64..111]->NXM_NX_ARP_SHA[],push:NXM_OF_ARP_SPA[],push:NXM_OF_ARP_TPA[],pop:NXM_OF_ARP_SPA[],pop:NXM_OF_ARP_TPA[],move:NXM_NX_REG14[]->NXM_NX_REG15[],load:0x1->NXM_NX_REG10[0],resubmit(,37)
 cookie=0x775c7e37, duration=297.885s, table=11, n_packets=4, n_bytes=168, priority=85,arp,metadata=0x4 actions=drop
 cookie=0x775c7e37, duration=297.885s, table=11, n_packets=0, n_bytes=0, priority=85,icmp6,metadata=0x4,nw_ttl=255,icmp_type=136,icmp_code=0 actions=drop
 cookie=0x226fe0c3, duration=297.885s, table=11, n_packets=4, n_bytes=248, priority=84,icmp6,metadata=0x4,nw_ttl=255,icmp_type=133,icmp_code=0 actions=resubmit(,12)
 cookie=0x775c7e37, duration=297.885s, table=11, n_packets=1, n_bytes=86, priority=85,icmp6,metadata=0x4,nw_ttl=255,icmp_type=135,icmp_code=0 actions=drop
 cookie=0x226fe0c3, duration=297.884s, table=11, n_packets=0, n_bytes=0, priority=84,icmp6,metadata=0x4,nw_ttl=255,icmp_type=134,icmp_code=0 actions=resubmit(,12)
 cookie=0xf35cd412, duration=297.884s, table=11, n_packets=0, n_bytes=0, priority=83,ipv6,metadata=0x4,ipv6_dst=ff00::/fff0:ffff:ffff:ffff:ffff:ffff:ffff:ffff actions=drop
 cookie=0xb871c55e, duration=297.885s, table=11, n_packets=4, n_bytes=360, priority=82,ipv6,metadata=0x4,dl_dst=33:33:00:00:00:00/ff:ff:00:00:00:00,ipv6_dst=ff00::/8 actions=drop
 cookie=0xb871c55e, duration=297.885s, table=11, n_packets=12, n_bytes=2604, priority=82,ip,metadata=0x4,dl_dst=01:00:00:00:00:00/01:00:00:00:00:00,nw_dst=224.0.0.0/4 actions=drop
 cookie=0x12869f19, duration=297.885s, table=11, n_packets=0, n_bytes=0, priority=80,udp,metadata=0x4,nw_dst=172.16.0.168,nw_frag=not_later actions=controller(userdata=00.00.00.0a.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.04.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.02.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.04.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.02.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.0e.04.00.20.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.10.04.00.20.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.0e.04.00.20.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.10.04.00.20.00.00.00.00.00.00.00.19.00.10.00.01.3a.01.ff.00.00.00.00.00.00.00.00.19.00.10.80.00.26.01.03.00.00.00.00.00.00.00.00.19.00.10.80.00.28.01.03.00.00.00.00.00.00.00.ff.ff.00.10.00.00.23.20.00.0e.ff.f8.0c.00.00.00)
 cookie=0x1042020, duration=297.885s, table=11, n_packets=0, n_bytes=0, priority=80,sctp,metadata=0x4,nw_dst=172.16.0.168,nw_frag=not_later actions=controller(userdata=00.00.00.18.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.04.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.02.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.04.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.02.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.0e.04.00.20.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.10.04.00.20.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.0e.04.00.20.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.10.04.00.20.00.00.00.00.00.00.ff.ff.00.10.00.00.23.20.00.0e.ff.f8.0c.00.00.00)
 cookie=0xbd7d1eda, duration=297.885s, table=11, n_packets=0, n_bytes=0, priority=80,sctp,metadata=0x4,nw_dst=192.168.101.254,nw_frag=not_later actions=controller(userdata=00.00.00.18.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.04.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.02.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.04.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.02.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.0e.04.00.20.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.10.04.00.20.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.0e.04.00.20.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.10.04.00.20.00.00.00.00.00.00.ff.ff.00.10.00.00.23.20.00.0e.ff.f8.0c.00.00.00)
 cookie=0x4a76dd09, duration=297.885s, table=11, n_packets=0, n_bytes=0, priority=80,udp,metadata=0x4,nw_dst=192.168.101.254,nw_frag=not_later actions=controller(userdata=00.00.00.0a.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.04.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.02.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.04.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.02.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.0e.04.00.20.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.10.04.00.20.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.0e.04.00.20.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.10.04.00.20.00.00.00.00.00.00.00.19.00.10.00.01.3a.01.ff.00.00.00.00.00.00.00.00.19.00.10.80.00.26.01.03.00.00.00.00.00.00.00.00.19.00.10.80.00.28.01.03.00.00.00.00.00.00.00.ff.ff.00.10.00.00.23.20.00.0e.ff.f8.0c.00.00.00)
 cookie=0xd603ef0b, duration=297.884s, table=11, n_packets=0, n_bytes=0, priority=80,tcp,metadata=0x4,nw_dst=192.168.101.254,nw_frag=not_later actions=controller(userdata=00.00.00.0b.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.04.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.02.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.04.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.02.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.0e.04.00.20.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.10.04.00.20.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.0e.04.00.20.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.10.04.00.20.00.00.00.00.00.00.ff.ff.00.10.00.00.23.20.00.0e.ff.f8.0c.00.00.00)
 cookie=0xdd8eb07e, duration=297.883s, table=11, n_packets=0, n_bytes=0, priority=80,tcp,metadata=0x4,nw_dst=172.16.0.168,nw_frag=not_later actions=controller(userdata=00.00.00.0b.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.04.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.02.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.04.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.02.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.0e.04.00.20.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.10.04.00.20.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.0e.04.00.20.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.10.04.00.20.00.00.00.00.00.00.ff.ff.00.10.00.00.23.20.00.0e.ff.f8.0c.00.00.00)
 cookie=0xda7a9229, duration=297.885s, table=11, n_packets=0, n_bytes=0, priority=80,sctp6,metadata=0x4,ipv6_dst=fe80::f816:3eff:fe35:8f09,nw_frag=not_later actions=controller(userdata=00.00.00.18.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.04.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.02.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.04.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.02.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.01.26.10.00.80.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.01.28.10.00.80.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.01.26.10.00.80.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.01.28.10.00.80.00.00.00.00.00.00.ff.ff.00.10.00.00.23.20.00.0e.ff.f8.0c.00.00.00)
 cookie=0x8f4d1eab, duration=297.885s, table=11, n_packets=0, n_bytes=0, priority=80,tcp6,metadata=0x4,ipv6_dst=fe80::f816:3eff:fe35:8f09,nw_frag=not_later actions=controller(userdata=00.00.00.0b.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.04.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.02.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.04.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.02.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.01.26.10.00.80.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.01.28.10.00.80.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.01.26.10.00.80.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.01.28.10.00.80.00.00.00.00.00.00.ff.ff.00.10.00.00.23.20.00.0e.ff.f8.0c.00.00.00)
 cookie=0x6568d241, duration=297.884s, table=11, n_packets=0, n_bytes=0, priority=80,udp6,metadata=0x4,ipv6_dst=fe80::f816:3eff:fe15:d626,nw_frag=not_later actions=controller(userdata=00.00.00.0a.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.04.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.02.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.04.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.02.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.01.26.10.00.80.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.01.28.10.00.80.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.01.26.10.00.80.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.01.28.10.00.80.00.00.00.00.00.00.00.19.00.10.00.01.3a.01.ff.00.00.00.00.00.00.00.00.19.00.10.80.00.3a.01.01.00.00.00.00.00.00.00.00.19.00.10.80.00.3c.01.04.00.00.00.00.00.00.00.ff.ff.00.10.00.00.23.20.00.0e.ff.f8.0c.00.00.00)
 cookie=0x468e29e6, duration=297.884s, table=11, n_packets=0, n_bytes=0, priority=80,tcp6,metadata=0x4,ipv6_dst=fe80::f816:3eff:fe15:d626,nw_frag=not_later actions=controller(userdata=00.00.00.0b.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.04.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.02.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.04.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.02.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.01.26.10.00.80.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.01.28.10.00.80.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.01.26.10.00.80.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.01.28.10.00.80.00.00.00.00.00.00.ff.ff.00.10.00.00.23.20.00.0e.ff.f8.0c.00.00.00)
 cookie=0x6d159a82, duration=297.884s, table=11, n_packets=0, n_bytes=0, priority=80,sctp6,metadata=0x4,ipv6_dst=fe80::f816:3eff:fe15:d626,nw_frag=not_later actions=controller(userdata=00.00.00.18.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.04.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.02.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.04.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.02.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.01.26.10.00.80.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.01.28.10.00.80.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.01.26.10.00.80.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.01.28.10.00.80.00.00.00.00.00.00.ff.ff.00.10.00.00.23.20.00.0e.ff.f8.0c.00.00.00)
 cookie=0x2925aac8, duration=297.883s, table=11, n_packets=0, n_bytes=0, priority=80,udp6,metadata=0x4,ipv6_dst=fe80::f816:3eff:fe35:8f09,nw_frag=not_later actions=controller(userdata=00.00.00.0a.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.04.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.02.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.04.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.02.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.01.26.10.00.80.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.01.28.10.00.80.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.01.26.10.00.80.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.01.28.10.00.80.00.00.00.00.00.00.00.19.00.10.00.01.3a.01.ff.00.00.00.00.00.00.00.00.19.00.10.80.00.3a.01.01.00.00.00.00.00.00.00.00.19.00.10.80.00.3c.01.04.00.00.00.00.00.00.00.ff.ff.00.10.00.00.23.20.00.0e.ff.f8.0c.00.00.00)
 cookie=0xef1096e1, duration=297.885s, table=11, n_packets=0, n_bytes=0, priority=70,ipv6,metadata=0x4,ipv6_dst=fe80::f816:3eff:fe35:8f09,nw_frag=not_later actions=controller(userdata=00.00.00.0a.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.04.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.02.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.04.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.02.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.01.26.10.00.80.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.01.28.10.00.80.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.01.26.10.00.80.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.01.28.10.00.80.00.00.00.00.00.00.00.19.00.10.00.01.3a.01.ff.00.00.00.00.00.00.00.00.19.00.10.80.00.3a.01.01.00.00.00.00.00.00.00.00.19.00.10.80.00.3c.01.03.00.00.00.00.00.00.00.ff.ff.00.10.00.00.23.20.00.0e.ff.f8.0c.00.00.00)
 cookie=0xe321f3b7, duration=297.884s, table=11, n_packets=0, n_bytes=0, priority=70,ipv6,metadata=0x4,ipv6_dst=fe80::f816:3eff:fe15:d626,nw_frag=not_later actions=controller(userdata=00.00.00.0a.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.04.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.02.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.04.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.02.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.01.26.10.00.80.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.01.28.10.00.80.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.01.26.10.00.80.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.01.28.10.00.80.00.00.00.00.00.00.00.19.00.10.00.01.3a.01.ff.00.00.00.00.00.00.00.00.19.00.10.80.00.3a.01.01.00.00.00.00.00.00.00.00.19.00.10.80.00.3c.01.03.00.00.00.00.00.00.00.ff.ff.00.10.00.00.23.20.00.0e.ff.f8.0c.00.00.00)
 cookie=0x2d4f9b99, duration=297.885s, table=11, n_packets=0, n_bytes=0, priority=70,ip,metadata=0x4,nw_dst=172.16.0.168,nw_frag=not_later actions=controller(userdata=00.00.00.0a.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.04.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.02.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.04.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.02.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.0e.04.00.20.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.10.04.00.20.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.0e.04.00.20.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.10.04.00.20.00.00.00.00.00.00.00.19.00.10.00.01.3a.01.ff.00.00.00.00.00.00.00.00.19.00.10.80.00.26.01.03.00.00.00.00.00.00.00.00.19.00.10.80.00.28.01.02.00.00.00.00.00.00.00.ff.ff.00.10.00.00.23.20.00.0e.ff.f8.0c.00.00.00)
 cookie=0x82cb20c4, duration=297.884s, table=11, n_packets=0, n_bytes=0, priority=70,ip,metadata=0x4,nw_dst=192.168.101.254,nw_frag=not_later actions=controller(userdata=00.00.00.0a.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.04.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.02.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.04.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.02.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.0e.04.00.20.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.10.04.00.20.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.0e.04.00.20.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.10.04.00.20.00.00.00.00.00.00.00.19.00.10.00.01.3a.01.ff.00.00.00.00.00.00.00.00.19.00.10.80.00.26.01.03.00.00.00.00.00.00.00.00.19.00.10.80.00.28.01.02.00.00.00.00.00.00.00.ff.ff.00.10.00.00.23.20.00.0e.ff.f8.0c.00.00.00)
 cookie=0x5d3b2d23, duration=297.885s, table=11, n_packets=0, n_bytes=0, priority=60,ipv6,metadata=0x4,ipv6_dst=fe80::f816:3eff:fe15:d626 actions=drop
 cookie=0x50f06cf7, duration=297.885s, table=11, n_packets=0, n_bytes=0, priority=60,ipv6,metadata=0x4,ipv6_dst=fe80::f816:3eff:fe35:8f09 actions=drop
 cookie=0xbef403a9, duration=297.884s, table=11, n_packets=0, n_bytes=0, priority=60,ip,metadata=0x4,nw_dst=192.168.101.254 actions=drop
 cookie=0x2e92e948, duration=297.884s, table=11, n_packets=0, n_bytes=0, priority=50,metadata=0x4,dl_dst=ff:ff:ff:ff:ff:ff actions=drop
 cookie=0xd8513780, duration=297.885s, table=11, n_packets=0, n_bytes=0, priority=32,ipv6,metadata=0x4,dl_dst=33:33:00:00:00:00/ff:ff:00:00:00:00,ipv6_dst=ff00::/8,nw_ttl=0,nw_frag=not_later actions=drop
 cookie=0xd8513780, duration=297.883s, table=11, n_packets=0, n_bytes=0, priority=32,ipv6,metadata=0x4,dl_dst=33:33:00:00:00:00/ff:ff:00:00:00:00,ipv6_dst=ff00::/8,nw_ttl=1,nw_frag=not_later actions=drop
 cookie=0xd8513780, duration=297.885s, table=11, n_packets=0, n_bytes=0, priority=32,ip,metadata=0x4,dl_dst=01:00:00:00:00:00/01:00:00:00:00:00,nw_dst=224.0.0.0/4,nw_ttl=1,nw_frag=not_later actions=drop
 cookie=0xd8513780, duration=297.884s, table=11, n_packets=0, n_bytes=0, priority=32,ip,metadata=0x4,dl_dst=01:00:00:00:00:00/01:00:00:00:00:00,nw_dst=224.0.0.0/4,nw_ttl=0,nw_frag=not_later actions=drop
 cookie=0xfd1c35f8, duration=297.885s, table=11, n_packets=0, n_bytes=0, priority=31,ip,reg14=0x1,metadata=0x4,nw_ttl=1,nw_frag=not_later actions=controller(userdata=00.00.00.0a.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.04.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.02.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.04.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.02.06.00.30.00.00.00.00.00.00.00.19.00.10.80.00.26.01.0b.00.00.00.00.00.00.00.00.19.00.10.80.00.28.01.00.00.00.00.00.00.00.00.00.1c.00.18.00.20.00.00.00.00.00.00.80.00.16.04.80.00.18.04.00.00.00.00.00.19.00.10.80.00.16.04.c0.a8.65.fe.00.00.00.00.00.19.00.10.00.01.3a.01.fe.00.00.00.00.00.00.00.00.19.00.10.00.01.1e.04.00.00.00.01.00.00.00.00.00.19.00.10.00.01.15.08.00.00.00.01.00.00.00.01.ff.ff.00.10.00.00.23.20.00.0e.ff.f8.25.00.00.00)
 cookie=0x1e2c528e, duration=297.884s, table=11, n_packets=0, n_bytes=0, priority=31,ip,reg14=0x2,metadata=0x4,nw_ttl=1,nw_frag=not_later actions=controller(userdata=00.00.00.0a.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.04.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.02.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.04.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.02.06.00.30.00.00.00.00.00.00.00.19.00.10.80.00.26.01.0b.00.00.00.00.00.00.00.00.19.00.10.80.00.28.01.00.00.00.00.00.00.00.00.00.1c.00.18.00.20.00.00.00.00.00.00.80.00.16.04.80.00.18.04.00.00.00.00.00.19.00.10.80.00.16.04.ac.10.00.a8.00.00.00.00.00.19.00.10.00.01.3a.01.fe.00.00.00.00.00.00.00.00.19.00.10.00.01.1e.04.00.00.00.02.00.00.00.00.00.19.00.10.00.01.15.08.00.00.00.01.00.00.00.01.ff.ff.00.10.00.00.23.20.00.0e.ff.f8.25.00.00.00)
 cookie=0x1e2c528e, duration=297.884s, table=11, n_packets=0, n_bytes=0, priority=31,ip,reg14=0x2,metadata=0x4,nw_ttl=0,nw_frag=not_later actions=controller(userdata=00.00.00.0a.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.04.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.02.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.04.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.02.06.00.30.00.00.00.00.00.00.00.19.00.10.80.00.26.01.0b.00.00.00.00.00.00.00.00.19.00.10.80.00.28.01.00.00.00.00.00.00.00.00.00.1c.00.18.00.20.00.00.00.00.00.00.80.00.16.04.80.00.18.04.00.00.00.00.00.19.00.10.80.00.16.04.ac.10.00.a8.00.00.00.00.00.19.00.10.00.01.3a.01.fe.00.00.00.00.00.00.00.00.19.00.10.00.01.1e.04.00.00.00.02.00.00.00.00.00.19.00.10.00.01.15.08.00.00.00.01.00.00.00.01.ff.ff.00.10.00.00.23.20.00.0e.ff.f8.25.00.00.00)
 cookie=0xfd1c35f8, duration=297.884s, table=11, n_packets=0, n_bytes=0, priority=31,ip,reg14=0x1,metadata=0x4,nw_ttl=0,nw_frag=not_later actions=controller(userdata=00.00.00.0a.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.04.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.00.02.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.04.06.00.30.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.00.02.06.00.30.00.00.00.00.00.00.00.19.00.10.80.00.26.01.0b.00.00.00.00.00.00.00.00.19.00.10.80.00.28.01.00.00.00.00.00.00.00.00.00.1c.00.18.00.20.00.00.00.00.00.00.80.00.16.04.80.00.18.04.00.00.00.00.00.19.00.10.80.00.16.04.c0.a8.65.fe.00.00.00.00.00.19.00.10.00.01.3a.01.fe.00.00.00.00.00.00.00.00.19.00.10.00.01.1e.04.00.00.00.01.00.00.00.00.00.19.00.10.00.01.15.08.00.00.00.01.00.00.00.01.ff.ff.00.10.00.00.23.20.00.0e.ff.f8.25.00.00.00)
 cookie=0xfed9207b, duration=297.885s, table=11, n_packets=0, n_bytes=0, priority=30,ip,metadata=0x4,nw_ttl=1 actions=drop
 cookie=0xfed9207b, duration=297.885s, table=11, n_packets=0, n_bytes=0, priority=30,ipv6,metadata=0x4,nw_ttl=1 actions=drop
 cookie=0xfed9207b, duration=297.884s, table=11, n_packets=0, n_bytes=0, priority=30,ip,metadata=0x4,nw_ttl=0 actions=drop
 cookie=0xfed9207b, duration=297.884s, table=11, n_packets=0, n_bytes=0, priority=30,ipv6,metadata=0x4,nw_ttl=0 actions=drop
 cookie=0x99a645, duration=298.021s, table=11, n_packets=42, n_bytes=6292, priority=0,metadata=0x1 actions=resubmit(,12)
 cookie=0x99a645, duration=297.967s, table=11, n_packets=184, n_bytes=18095, priority=0,metadata=0x3 actions=resubmit(,12)
 cookie=0xb7a7f01b, duration=297.885s, table=11, n_packets=12, n_bytes=1176, priority=0,metadata=0x4 actions=resubmit(,12)
 cookie=0x1c40e6e, duration=298.021s, table=12, n_packets=0, n_bytes=0, priority=110,metadata=0x1,dl_dst=b6:9c:90:73:34:da actions=resubmit(,13)
 cookie=0x1c40e6e, duration=297.934s, table=12, n_packets=0, n_bytes=0, priority=110,metadata=0x3,dl_dst=b6:9c:90:73:34:da actions=resubmit(,13)
 cookie=0x4ed1c2cd, duration=298.006s, table=12, n_packets=1, n_bytes=86, priority=110,icmp6,metadata=0x1,nw_ttl=255,icmp_type=135,icmp_code=0 actions=resubmit(,13)
 cookie=0x4ed1c2cd, duration=298.006s, table=12, n_packets=0, n_bytes=0, priority=110,icmp6,metadata=0x1,nw_ttl=255,icmp_type=136,icmp_code=0 actions=resubmit(,13)
 cookie=0x4ed1c2cd, duration=298.006s, table=12, n_packets=0, n_bytes=0, priority=110,icmp6,metadata=0x1,nw_ttl=255,icmp_type=134,icmp_code=0 actions=resubmit(,13)
 cookie=0x4ed1c2cd, duration=298.006s, table=12, n_packets=4, n_bytes=248, priority=110,icmp6,metadata=0x1,nw_ttl=255,icmp_type=133,icmp_code=0 actions=resubmit(,13)
 cookie=0x4ed1c2cd, duration=297.915s, table=12, n_packets=2, n_bytes=172, priority=110,icmp6,metadata=0x3,nw_ttl=255,icmp_type=135,icmp_code=0 actions=resubmit(,13)
 cookie=0x4ed1c2cd, duration=297.915s, table=12, n_packets=0, n_bytes=0, priority=110,icmp6,metadata=0x3,nw_ttl=255,icmp_type=136,icmp_code=0 actions=resubmit(,13)
 cookie=0x4ed1c2cd, duration=297.915s, table=12, n_packets=0, n_bytes=0, priority=110,icmp6,metadata=0x3,nw_ttl=255,icmp_type=134,icmp_code=0 actions=resubmit(,13)
 cookie=0x4ed1c2cd, duration=297.915s, table=12, n_packets=7, n_bytes=490, priority=110,icmp6,metadata=0x3,nw_ttl=255,icmp_type=133,icmp_code=0 actions=resubmit(,13)
 cookie=0x4ed1c2cd, duration=298.006s, table=12, n_packets=0, n_bytes=0, priority=110,icmp6,metadata=0x1,ipv6_src=fe80::/10,icmp_type=130 actions=resubmit(,13)
 cookie=0x4ed1c2cd, duration=298.006s, table=12, n_packets=0, n_bytes=0, priority=110,icmp6,metadata=0x1,ipv6_src=fe80::/10,icmp_type=131 actions=resubmit(,13)
 cookie=0x4ed1c2cd, duration=298.006s, table=12, n_packets=0, n_bytes=0, priority=110,icmp6,metadata=0x1,ipv6_src=fe80::/10,icmp_type=132 actions=resubmit(,13)
 cookie=0x4ed1c2cd, duration=297.915s, table=12, n_packets=0, n_bytes=0, priority=110,icmp6,metadata=0x3,ipv6_src=fe80::/10,icmp_type=130 actions=resubmit(,13)
 cookie=0x4ed1c2cd, duration=297.915s, table=12, n_packets=0, n_bytes=0, priority=110,icmp6,metadata=0x3,ipv6_src=fe80::/10,icmp_type=131 actions=resubmit(,13)
 cookie=0x4ed1c2cd, duration=297.915s, table=12, n_packets=0, n_bytes=0, priority=110,icmp6,metadata=0x3,ipv6_src=fe80::/10,icmp_type=132 actions=resubmit(,13)
 cookie=0x4ed1c2cd, duration=298.006s, table=12, n_packets=0, n_bytes=0, priority=110,udp6,metadata=0x1,tp_src=546,tp_dst=547 actions=resubmit(,13)
 cookie=0x4ed1c2cd, duration=298.006s, table=12, n_packets=0, n_bytes=0, priority=110,udp,metadata=0x1,tp_src=546,tp_dst=547 actions=resubmit(,13)
 cookie=0x4ed1c2cd, duration=297.915s, table=12, n_packets=0, n_bytes=0, priority=110,udp6,metadata=0x3,tp_src=546,tp_dst=547 actions=resubmit(,13)
 cookie=0x4ed1c2cd, duration=297.915s, table=12, n_packets=0, n_bytes=0, priority=110,udp,metadata=0x3,tp_src=546,tp_dst=547 actions=resubmit(,13)
 cookie=0x4ed1c2cd, duration=298.006s, table=12, n_packets=4, n_bytes=360, priority=110,icmp6,metadata=0x1,ipv6_dst=ff02::16,icmp_type=143 actions=resubmit(,13)
 cookie=0x4ed1c2cd, duration=297.915s, table=12, n_packets=2, n_bytes=220, priority=110,icmp6,metadata=0x3,ipv6_dst=ff02::16,icmp_type=143 actions=resubmit(,13)
 cookie=0x8a859fb5, duration=298.005s, table=12, n_packets=19, n_bytes=4338, priority=110,metadata=0x1,dl_dst=01:00:00:00:00:00/01:00:00:00:00:00 actions=resubmit(,13)
 cookie=0x8a859fb5, duration=297.912s, table=12, n_packets=7, n_bytes=894, priority=110,metadata=0x3,dl_dst=01:00:00:00:00:00/01:00:00:00:00:00 actions=resubmit(,13)
 cookie=0xc9e0f69c, duration=297.885s, table=12, n_packets=3, n_bytes=294, priority=110,ip,reg14=0x5,metadata=0x3 actions=resubmit(,13)
 cookie=0xbfd6acdb, duration=297.885s, table=12, n_packets=0, n_bytes=0, priority=110,ip,reg14=0x1,metadata=0x1 actions=resubmit(,13)
 cookie=0xbfd6acdb, duration=297.885s, table=12, n_packets=0, n_bytes=0, priority=110,ipv6,reg14=0x1,metadata=0x1 actions=resubmit(,13)
 cookie=0x5c05e464, duration=297.885s, table=12, n_packets=12, n_bytes=1176, priority=110,ip,reg14=0x4,metadata=0x1 actions=resubmit(,13)
 cookie=0x5c05e464, duration=297.885s, table=12, n_packets=0, n_bytes=0, priority=110,ipv6,reg14=0x4,metadata=0x1 actions=resubmit(,13)
 cookie=0xc9e0f69c, duration=297.884s, table=12, n_packets=0, n_bytes=0, priority=110,ipv6,reg14=0x5,metadata=0x3 actions=resubmit(,13)
 cookie=0xd09ee163, duration=297.968s, table=12, n_packets=0, n_bytes=0, priority=100,ipv6,metadata=0x1 actions=load:0x1->NXM_NX_XXREG0[96],resubmit(,13)
 cookie=0xd09ee163, duration=297.968s, table=12, n_packets=0, n_bytes=0, priority=100,ip,metadata=0x1 actions=load:0x1->NXM_NX_XXREG0[96],resubmit(,13)
 cookie=0xd09ee163, duration=297.910s, table=12, n_packets=0, n_bytes=0, priority=100,ipv6,metadata=0x3 actions=load:0x1->NXM_NX_XXREG0[96],resubmit(,13)
 cookie=0xd09ee163, duration=297.910s, table=12, n_packets=161, n_bytes=15941, priority=100,ip,metadata=0x3 actions=load:0x1->NXM_NX_XXREG0[96],resubmit(,13)
 cookie=0xa8889c2a, duration=298.005s, table=12, n_packets=2, n_bytes=84, priority=0,metadata=0x1 actions=resubmit(,13)
 cookie=0xa8889c2a, duration=297.912s, table=12, n_packets=2, n_bytes=84, priority=0,metadata=0x3 actions=resubmit(,13)
 cookie=0x5db6329d, duration=297.885s, table=12, n_packets=19, n_bytes=1718, priority=0,metadata=0x4 actions=resubmit(,13)
 cookie=0x18ef1d8, duration=298.021s, table=13, n_packets=1, n_bytes=86, priority=110,icmp6,metadata=0x1,nw_ttl=255,icmp_type=135,icmp_code=0 actions=resubmit(,14)
 cookie=0x18ef1d8, duration=298.021s, table=13, n_packets=0, n_bytes=0, priority=110,icmp6,metadata=0x1,nw_ttl=255,icmp_type=136,icmp_code=0 actions=resubmit(,14)
 cookie=0x18ef1d8, duration=298.021s, table=13, n_packets=0, n_bytes=0, priority=110,icmp6,metadata=0x1,nw_ttl=255,icmp_type=134,icmp_code=0 actions=resubmit(,14)
 cookie=0x18ef1d8, duration=298.021s, table=13, n_packets=4, n_bytes=248, priority=110,icmp6,metadata=0x1,nw_ttl=255,icmp_type=133,icmp_code=0 actions=resubmit(,14)
 cookie=0x18ef1d8, duration=297.967s, table=13, n_packets=2, n_bytes=172, priority=110,icmp6,metadata=0x3,nw_ttl=255,icmp_type=135,icmp_code=0 actions=resubmit(,14)
 cookie=0x18ef1d8, duration=297.934s, table=13, n_packets=0, n_bytes=0, priority=110,icmp6,metadata=0x3,nw_ttl=255,icmp_type=136,icmp_code=0 actions=resubmit(,14)
 cookie=0x18ef1d8, duration=297.934s, table=13, n_packets=0, n_bytes=0, priority=110,icmp6,metadata=0x3,nw_ttl=255,icmp_type=134,icmp_code=0 actions=resubmit(,14)
 cookie=0x18ef1d8, duration=297.934s, table=13, n_packets=7, n_bytes=490, priority=110,icmp6,metadata=0x3,nw_ttl=255,icmp_type=133,icmp_code=0 actions=resubmit(,14)
 cookie=0x18ef1d8, duration=298.021s, table=13, n_packets=0, n_bytes=0, priority=110,icmp6,metadata=0x1,ipv6_src=fe80::/10,icmp_type=130 actions=resubmit(,14)
 cookie=0x18ef1d8, duration=298.021s, table=13, n_packets=0, n_bytes=0, priority=110,icmp6,metadata=0x1,ipv6_src=fe80::/10,icmp_type=131 actions=resubmit(,14)
 cookie=0x18ef1d8, duration=298.021s, table=13, n_packets=0, n_bytes=0, priority=110,icmp6,metadata=0x1,ipv6_src=fe80::/10,icmp_type=132 actions=resubmit(,14)
 cookie=0x18ef1d8, duration=297.967s, table=13, n_packets=0, n_bytes=0, priority=110,icmp6,metadata=0x3,ipv6_src=fe80::/10,icmp_type=130 actions=resubmit(,14)
 cookie=0x18ef1d8, duration=297.967s, table=13, n_packets=0, n_bytes=0, priority=110,icmp6,metadata=0x3,ipv6_src=fe80::/10,icmp_type=131 actions=resubmit(,14)
 cookie=0x18ef1d8, duration=297.934s, table=13, n_packets=0, n_bytes=0, priority=110,icmp6,metadata=0x3,ipv6_src=fe80::/10,icmp_type=132 actions=resubmit(,14)
 cookie=0x18ef1d8, duration=298.021s, table=13, n_packets=4, n_bytes=360, priority=110,icmp6,metadata=0x1,ipv6_dst=ff02::16,icmp_type=143 actions=resubmit(,14)
 cookie=0x18ef1d8, duration=297.934s, table=13, n_packets=2, n_bytes=220, priority=110,icmp6,metadata=0x3,ipv6_dst=ff02::16,icmp_type=143 actions=resubmit(,14)
 cookie=0x13514752, duration=298.021s, table=13, n_packets=0, n_bytes=0, priority=110,metadata=0x1,dl_dst=b6:9c:90:73:34:da actions=resubmit(,14)
 cookie=0x13514752, duration=297.934s, table=13, n_packets=0, n_bytes=0, priority=110,metadata=0x3,dl_dst=b6:9c:90:73:34:da actions=resubmit(,14)
 cookie=0x3dcac7c2, duration=298.006s, table=13, n_packets=19, n_bytes=4338, priority=110,metadata=0x1,dl_dst=01:00:00:00:00:00/01:00:00:00:00:00 actions=resubmit(,14)
 cookie=0x3dcac7c2, duration=297.934s, table=13, n_packets=7, n_bytes=894, priority=110,metadata=0x3,dl_dst=01:00:00:00:00:00/01:00:00:00:00:00 actions=resubmit(,14)
 cookie=0xe6437fe2, duration=297.968s, table=13, n_packets=0, n_bytes=0, priority=110,reg0=0x10000/0x10000,metadata=0x1 actions=resubmit(,14)
 cookie=0xe6437fe2, duration=297.910s, table=13, n_packets=0, n_bytes=0, priority=110,reg0=0x10000/0x10000,metadata=0x3 actions=resubmit(,14)
 cookie=0x4d6644b3, duration=297.902s, table=13, n_packets=12, n_bytes=1176, priority=110,ip,reg14=0x4,metadata=0x1 actions=resubmit(,14)
 cookie=0xf92a04b1, duration=297.902s, table=13, n_packets=0, n_bytes=0, priority=110,ip,reg14=0x1,metadata=0x1 actions=resubmit(,14)
 cookie=0xf92a04b1, duration=297.885s, table=13, n_packets=0, n_bytes=0, priority=110,ipv6,reg14=0x1,metadata=0x1 actions=resubmit(,14)
 cookie=0x4d6644b3, duration=297.885s, table=13, n_packets=0, n_bytes=0, priority=110,ipv6,reg14=0x4,metadata=0x1 actions=resubmit(,14)
 cookie=0x51fcfe4d, duration=297.884s, table=13, n_packets=0, n_bytes=0, priority=110,ipv6,reg14=0x5,metadata=0x3 actions=resubmit(,14)
 cookie=0x51fcfe4d, duration=297.884s, table=13, n_packets=3, n_bytes=294, priority=110,ip,reg14=0x5,metadata=0x3 actions=resubmit(,14)
 cookie=0x7fcfc69d, duration=298.005s, table=13, n_packets=2, n_bytes=84, priority=0,metadata=0x1 actions=resubmit(,14)
 cookie=0x7fcfc69d, duration=297.915s, table=13, n_packets=163, n_bytes=16025, priority=0,metadata=0x3 actions=resubmit(,14)
 cookie=0x66a92f9a, duration=297.884s, table=13, n_packets=19, n_bytes=1718, priority=0,metadata=0x4 actions=resubmit(,14)
 cookie=0xb6a4cceb, duration=298.005s, table=14, n_packets=0, n_bytes=0, priority=110,ipv6,reg0=0x4/0x4,metadata=0x1 actions=ct(table=15,zone=NXM_NX_REG13[0..15],nat)
 cookie=0xb6a4cceb, duration=298.005s, table=14, n_packets=0, n_bytes=0, priority=110,ip,reg0=0x4/0x4,metadata=0x1 actions=ct(table=15,zone=NXM_NX_REG13[0..15],nat)
 cookie=0xb6a4cceb, duration=297.912s, table=14, n_packets=0, n_bytes=0, priority=110,ipv6,reg0=0x4/0x4,metadata=0x3 actions=ct(table=15,zone=NXM_NX_REG13[0..15],nat)
 cookie=0xb6a4cceb, duration=297.912s, table=14, n_packets=0, n_bytes=0, priority=110,ip,reg0=0x4/0x4,metadata=0x3 actions=ct(table=15,zone=NXM_NX_REG13[0..15],nat)
 cookie=0xdd7d3fe8, duration=297.968s, table=14, n_packets=0, n_bytes=0, priority=100,ipv6,reg0=0x1/0x1,metadata=0x1 actions=ct(table=15,zone=NXM_NX_REG13[0..15])
 cookie=0xdd7d3fe8, duration=297.968s, table=14, n_packets=0, n_bytes=0, priority=100,ip,reg0=0x1/0x1,metadata=0x1 actions=ct(table=15,zone=NXM_NX_REG13[0..15])
 cookie=0xdd7d3fe8, duration=297.910s, table=14, n_packets=0, n_bytes=0, priority=100,ipv6,reg0=0x1/0x1,metadata=0x3 actions=ct(table=15,zone=NXM_NX_REG13[0..15])
 cookie=0xdd7d3fe8, duration=297.910s, table=14, n_packets=161, n_bytes=15941, priority=100,ip,reg0=0x1/0x1,metadata=0x3 actions=ct(table=15,zone=NXM_NX_REG13[0..15])
 cookie=0x63cc265d, duration=298.006s, table=14, n_packets=42, n_bytes=6292, priority=0,metadata=0x1 actions=resubmit(,15)
 cookie=0x63cc265d, duration=297.915s, table=14, n_packets=23, n_bytes=2154, priority=0,metadata=0x3 actions=resubmit(,15)
 cookie=0xcc5eecf, duration=297.884s, table=14, n_packets=19, n_bytes=1718, priority=0,metadata=0x4 actions=resubmit(,15)
 cookie=0xa610f851, duration=298.005s, table=15, n_packets=0, n_bytes=0, priority=7,ct_state=+new-est+trk,metadata=0x1 actions=load:0x1->NXM_NX_XXREG0[103],load:0x1->NXM_NX_XXREG0[105],resubmit(,16)
 cookie=0xa610f851, duration=297.912s, table=15, n_packets=31, n_bytes=2654, priority=7,ct_state=+new-est+trk,metadata=0x3 actions=load:0x1->NXM_NX_XXREG0[103],load:0x1->NXM_NX_XXREG0[105],resubmit(,16)
 cookie=0x5418d02b, duration=298.006s, table=15, n_packets=0, n_bytes=0, priority=6,ct_state=-new+est-rpl+trk,ct_mark=0x1/0x1,metadata=0x1 actions=load:0x1->NXM_NX_XXREG0[103],load:0x1->NXM_NX_XXREG0[105],resubmit(,16)
 cookie=0x58a7f520, duration=298.006s, table=15, n_packets=0, n_bytes=0, priority=4,ct_state=-new+est-rpl+trk,ct_mark=0/0x1,metadata=0x1 actions=load:0x1->NXM_NX_XXREG0[104],load:0x1->NXM_NX_XXREG0[106],resubmit(,16)
 cookie=0x5418d02b, duration=297.915s, table=15, n_packets=0, n_bytes=0, priority=6,ct_state=-new+est-rpl+trk,ct_mark=0x1/0x1,metadata=0x3 actions=load:0x1->NXM_NX_XXREG0[103],load:0x1->NXM_NX_XXREG0[105],resubmit(,16)
 cookie=0x58a7f520, duration=297.915s, table=15, n_packets=80, n_bytes=7120, priority=4,ct_state=-new+est-rpl+trk,ct_mark=0/0x1,metadata=0x3 actions=load:0x1->NXM_NX_XXREG0[104],load:0x1->NXM_NX_XXREG0[106],resubmit(,16)
 cookie=0xad4144c7, duration=298.005s, table=15, n_packets=42, n_bytes=6292, priority=5,ct_state=-trk,metadata=0x1 actions=load:0x1->NXM_NX_XXREG0[104],load:0x1->NXM_NX_XXREG0[105],resubmit(,16)
 cookie=0xad4144c7, duration=297.912s, table=15, n_packets=23, n_bytes=2154, priority=5,ct_state=-trk,metadata=0x3 actions=load:0x1->NXM_NX_XXREG0[104],load:0x1->NXM_NX_XXREG0[105],resubmit(,16)
 cookie=0xc9f7c6e8, duration=297.968s, table=15, n_packets=0, n_bytes=0, priority=3,ct_state=-est+trk,metadata=0x1 actions=load:0x1->NXM_NX_XXREG0[105],resubmit(,16)
 cookie=0xc9f7c6e8, duration=297.912s, table=15, n_packets=0, n_bytes=0, priority=3,ct_state=-est+trk,metadata=0x3 actions=load:0x1->NXM_NX_XXREG0[105],resubmit(,16)
 cookie=0xb65e896, duration=298.021s, table=15, n_packets=0, n_bytes=0, priority=1,ct_state=+est+trk,ct_mark=0/0x1,metadata=0x1 actions=load:0x1->NXM_NX_XXREG0[106],resubmit(,16)
 cookie=0x83467ed0, duration=298.005s, table=15, n_packets=0, n_bytes=0, priority=2,ct_state=+est+trk,ct_mark=0x1/0x1,metadata=0x1 actions=load:0x1->NXM_NX_XXREG0[105],resubmit(,16)
 cookie=0xb65e896, duration=297.934s, table=15, n_packets=50, n_bytes=6167, priority=1,ct_state=+est+trk,ct_mark=0/0x1,metadata=0x3 actions=load:0x1->NXM_NX_XXREG0[106],resubmit(,16)
 cookie=0x83467ed0, duration=297.915s, table=15, n_packets=0, n_bytes=0, priority=2,ct_state=+est+trk,ct_mark=0x1/0x1,metadata=0x3 actions=load:0x1->NXM_NX_XXREG0[105],resubmit(,16)
 cookie=0x8753dab9, duration=298.005s, table=15, n_packets=0, n_bytes=0, priority=0,metadata=0x1 actions=resubmit(,16)
 cookie=0x8753dab9, duration=297.915s, table=15, n_packets=0, n_bytes=0, priority=0,metadata=0x3 actions=resubmit(,16)
 cookie=0x8462b77b, duration=297.884s, table=15, n_packets=19, n_bytes=1718, priority=0,metadata=0x4 actions=resubmit(,16)
 cookie=0x699c65a3, duration=298.006s, table=16, n_packets=0, n_bytes=0, priority=65532,ct_state=+inv+trk,metadata=0x1 actions=load:0x1->OXM_OF_PKT_REG4[49],resubmit(,17)
 cookie=0x699c65a3, duration=297.915s, table=16, n_packets=0, n_bytes=0, priority=65532,ct_state=+inv+trk,metadata=0x3 actions=load:0x1->OXM_OF_PKT_REG4[49],resubmit(,17)
 cookie=0x699c65a3, duration=298.006s, table=16, n_packets=0, n_bytes=0, priority=65532,ct_state=+est+rpl+trk,ct_mark=0x1/0x1,metadata=0x1 actions=load:0x1->OXM_OF_PKT_REG4[49],resubmit(,17)
 cookie=0x699c65a3, duration=297.915s, table=16, n_packets=0, n_bytes=0, priority=65532,ct_state=+est+rpl+trk,ct_mark=0x1/0x1,metadata=0x3 actions=load:0x1->OXM_OF_PKT_REG4[49],resubmit(,17)
 cookie=0x9057aa86, duration=298.005s, table=16, n_packets=0, n_bytes=0, priority=65532,ct_state=-new+est-rel+rpl-inv+trk,ct_mark=0/0x1,metadata=0x1 actions=load:0->NXM_NX_XXREG0[105],load:0->NXM_NX_XXREG0[106],load:0x1->NXM_NX_XXREG0[113],load:0x1->OXM_OF_PKT_REG4[48],resubmit(,17)
 cookie=0x9057aa86, duration=297.912s, table=16, n_packets=50, n_bytes=6167, priority=65532,ct_state=-new+est-rel+rpl-inv+trk,ct_mark=0/0x1,metadata=0x3 actions=load:0->NXM_NX_XXREG0[105],load:0->NXM_NX_XXREG0[106],load:0x1->NXM_NX_XXREG0[113],load:0x1->OXM_OF_PKT_REG4[48],resubmit(,17)
 cookie=0xaca2a410, duration=298.005s, table=16, n_packets=1, n_bytes=86, priority=65532,icmp6,metadata=0x1,nw_ttl=255,icmp_type=135,icmp_code=0 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,17)
 cookie=0xaca2a410, duration=298.005s, table=16, n_packets=0, n_bytes=0, priority=65532,icmp6,metadata=0x1,nw_ttl=255,icmp_type=136,icmp_code=0 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,17)
 cookie=0xaca2a410, duration=298.005s, table=16, n_packets=0, n_bytes=0, priority=65532,icmp6,metadata=0x1,nw_ttl=255,icmp_type=134,icmp_code=0 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,17)
 cookie=0xaca2a410, duration=298.005s, table=16, n_packets=4, n_bytes=248, priority=65532,icmp6,metadata=0x1,nw_ttl=255,icmp_type=133,icmp_code=0 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,17)
 cookie=0xaca2a410, duration=297.912s, table=16, n_packets=2, n_bytes=172, priority=65532,icmp6,metadata=0x3,nw_ttl=255,icmp_type=135,icmp_code=0 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,17)
 cookie=0xaca2a410, duration=297.912s, table=16, n_packets=0, n_bytes=0, priority=65532,icmp6,metadata=0x3,nw_ttl=255,icmp_type=136,icmp_code=0 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,17)
 cookie=0xaca2a410, duration=297.912s, table=16, n_packets=0, n_bytes=0, priority=65532,icmp6,metadata=0x3,nw_ttl=255,icmp_type=134,icmp_code=0 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,17)
 cookie=0xaca2a410, duration=297.912s, table=16, n_packets=7, n_bytes=490, priority=65532,icmp6,metadata=0x3,nw_ttl=255,icmp_type=133,icmp_code=0 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,17)
 cookie=0xaca2a410, duration=298.005s, table=16, n_packets=0, n_bytes=0, priority=65532,icmp6,metadata=0x1,ipv6_src=fe80::/10,icmp_type=130 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,17)
 cookie=0xaca2a410, duration=298.005s, table=16, n_packets=0, n_bytes=0, priority=65532,icmp6,metadata=0x1,ipv6_src=fe80::/10,icmp_type=131 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,17)
 cookie=0xaca2a410, duration=298.005s, table=16, n_packets=0, n_bytes=0, priority=65532,icmp6,metadata=0x1,ipv6_src=fe80::/10,icmp_type=132 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,17)
 cookie=0xaca2a410, duration=297.912s, table=16, n_packets=0, n_bytes=0, priority=65532,icmp6,metadata=0x3,ipv6_src=fe80::/10,icmp_type=130 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,17)
 cookie=0xaca2a410, duration=297.912s, table=16, n_packets=0, n_bytes=0, priority=65532,icmp6,metadata=0x3,ipv6_src=fe80::/10,icmp_type=131 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,17)
 cookie=0xaca2a410, duration=297.912s, table=16, n_packets=0, n_bytes=0, priority=65532,icmp6,metadata=0x3,ipv6_src=fe80::/10,icmp_type=132 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,17)
 cookie=0xaca2a410, duration=298.005s, table=16, n_packets=4, n_bytes=360, priority=65532,icmp6,metadata=0x1,ipv6_dst=ff02::16,icmp_type=143 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,17)
 cookie=0xaca2a410, duration=297.912s, table=16, n_packets=2, n_bytes=220, priority=65532,icmp6,metadata=0x3,ipv6_dst=ff02::16,icmp_type=143 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,17)
 cookie=0xb7784997, duration=298.005s, table=16, n_packets=0, n_bytes=0, priority=65532,ct_state=-new-est+rel-inv+trk,ct_mark=0/0x1,ipv6,metadata=0x1 actions=load:0x1->NXM_NX_XXREG0[113],load:0x1->OXM_OF_PKT_REG4[48],ct(commit,table=17,zone=NXM_NX_REG13[0..15],nat)
 cookie=0xb7784997, duration=298.005s, table=16, n_packets=0, n_bytes=0, priority=65532,ct_state=-new-est+rel-inv+trk,ct_mark=0/0x1,ip,metadata=0x1 actions=load:0x1->NXM_NX_XXREG0[113],load:0x1->OXM_OF_PKT_REG4[48],ct(commit,table=17,zone=NXM_NX_REG13[0..15],nat)
 cookie=0xb7784997, duration=297.912s, table=16, n_packets=0, n_bytes=0, priority=65532,ct_state=-new-est+rel-inv+trk,ct_mark=0/0x1,ipv6,metadata=0x3 actions=load:0x1->NXM_NX_XXREG0[113],load:0x1->OXM_OF_PKT_REG4[48],ct(commit,table=17,zone=NXM_NX_REG13[0..15],nat)
 cookie=0xb7784997, duration=297.912s, table=16, n_packets=0, n_bytes=0, priority=65532,ct_state=-new-est+rel-inv+trk,ct_mark=0/0x1,ip,metadata=0x3 actions=load:0x1->NXM_NX_XXREG0[113],load:0x1->OXM_OF_PKT_REG4[48],ct(commit,table=17,zone=NXM_NX_REG13[0..15],nat)
 cookie=0x27bede58, duration=298.021s, table=16, n_packets=0, n_bytes=0, priority=34000,metadata=0x1,dl_dst=b6:9c:90:73:34:da actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,17)
 cookie=0x27bede58, duration=297.934s, table=16, n_packets=0, n_bytes=0, priority=34000,metadata=0x3,dl_dst=b6:9c:90:73:34:da actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,17)
 cookie=0xd64d1438, duration=297.910s, table=16, n_packets=82, n_bytes=7804, priority=2002,ip,reg0=0x100/0x100,reg14=0x2,metadata=0x3 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,17)
 cookie=0xa8785c27, duration=297.910s, table=16, n_packets=0, n_bytes=0, priority=2002,ipv6,reg0=0x100/0x100,reg14=0x2,metadata=0x3 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,17)
 cookie=0x97c7289d, duration=297.910s, table=16, n_packets=31, n_bytes=2654, priority=2002,ip,reg0=0x80/0x80,reg14=0x2,metadata=0x3 actions=load:0x1->OXM_OF_PKT_REG4[48],load:0x1->NXM_NX_XXREG0[97],resubmit(,17)
 cookie=0xeae4871e, duration=297.910s, table=16, n_packets=0, n_bytes=0, priority=2002,ipv6,reg0=0x80/0x80,reg14=0x2,metadata=0x3 actions=load:0x1->OXM_OF_PKT_REG4[48],load:0x1->NXM_NX_XXREG0[97],resubmit(,17)
 cookie=0xf2ba8941, duration=297.910s, table=16, n_packets=0, n_bytes=0, priority=2001,ipv6,reg0=0x200/0x200,reg14=0x2,metadata=0x3 actions=load:0x1->OXM_OF_PKT_REG4[49],resubmit(,17)
 cookie=0xf2ba8941, duration=297.910s, table=16, n_packets=0, n_bytes=0, priority=2001,ip,reg0=0x200/0x200,reg14=0x2,metadata=0x3 actions=load:0x1->OXM_OF_PKT_REG4[49],resubmit(,17)
 cookie=0xff1901d5, duration=297.910s, table=16, n_packets=0, n_bytes=0, priority=2001,ip,reg0=0x400/0x400,reg14=0x2,metadata=0x3 actions=load:0x1->OXM_OF_PKT_REG4[49],resubmit(,17)
 cookie=0xff1901d5, duration=297.910s, table=16, n_packets=0, n_bytes=0, priority=2001,ipv6,reg0=0x400/0x400,reg14=0x2,metadata=0x3 actions=load:0x1->OXM_OF_PKT_REG4[49],resubmit(,17)
 cookie=0x88f83281, duration=298.005s, table=16, n_packets=0, n_bytes=0, priority=1,ct_state=+est+trk,ct_mark=0x1/0x1,ip,metadata=0x1 actions=load:0x1->NXM_NX_XXREG0[97],load:0x1->OXM_OF_PKT_REG4[48],resubmit(,17)
 cookie=0x88f83281, duration=298.005s, table=16, n_packets=0, n_bytes=0, priority=1,ct_state=+est+trk,ct_mark=0x1/0x1,ipv6,metadata=0x1 actions=load:0x1->NXM_NX_XXREG0[97],load:0x1->OXM_OF_PKT_REG4[48],resubmit(,17)
 cookie=0x88f83281, duration=297.912s, table=16, n_packets=0, n_bytes=0, priority=1,ct_state=+est+trk,ct_mark=0x1/0x1,ip,metadata=0x3 actions=load:0x1->NXM_NX_XXREG0[97],load:0x1->OXM_OF_PKT_REG4[48],resubmit(,17)
 cookie=0x88f83281, duration=297.912s, table=16, n_packets=0, n_bytes=0, priority=1,ct_state=+est+trk,ct_mark=0x1/0x1,ipv6,metadata=0x3 actions=load:0x1->NXM_NX_XXREG0[97],load:0x1->OXM_OF_PKT_REG4[48],resubmit(,17)
 cookie=0xe179bee6, duration=297.968s, table=16, n_packets=0, n_bytes=0, priority=1,ct_state=-est+trk,ip,metadata=0x1 actions=load:0x1->NXM_NX_XXREG0[97],resubmit(,17)
 cookie=0xe179bee6, duration=297.968s, table=16, n_packets=0, n_bytes=0, priority=1,ct_state=-est+trk,ipv6,metadata=0x1 actions=load:0x1->NXM_NX_XXREG0[97],resubmit(,17)
 cookie=0xe179bee6, duration=297.910s, table=16, n_packets=0, n_bytes=0, priority=1,ct_state=-est+trk,ip,metadata=0x3 actions=load:0x1->NXM_NX_XXREG0[97],resubmit(,17)
 cookie=0xe179bee6, duration=297.910s, table=16, n_packets=0, n_bytes=0, priority=1,ct_state=-est+trk,ipv6,metadata=0x3 actions=load:0x1->NXM_NX_XXREG0[97],resubmit(,17)
 cookie=0xc0887f73, duration=297.968s, table=16, n_packets=33, n_bytes=5598, priority=0,metadata=0x1 actions=resubmit(,17)
 cookie=0xc0887f73, duration=297.912s, table=16, n_packets=10, n_bytes=588, priority=0,metadata=0x3 actions=resubmit(,17)
 cookie=0xf1546b68, duration=297.884s, table=16, n_packets=19, n_bytes=1718, priority=0,metadata=0x4 actions=resubmit(,17)
 cookie=0x54304388, duration=298.006s, table=17, n_packets=0, n_bytes=0, priority=1000,reg8=0x20000/0x20000,metadata=0x1 actions=load:0->OXM_OF_PKT_REG4[48],load:0->OXM_OF_PKT_REG4[49],load:0->OXM_OF_PKT_REG4[50]
 cookie=0x54304388, duration=297.915s, table=17, n_packets=0, n_bytes=0, priority=1000,reg8=0x20000/0x20000,metadata=0x3 actions=load:0->OXM_OF_PKT_REG4[48],load:0->OXM_OF_PKT_REG4[49],load:0->OXM_OF_PKT_REG4[50]
 cookie=0xb23c013f, duration=298.005s, table=17, n_packets=9, n_bytes=694, priority=1000,reg8=0x10000/0x10000,metadata=0x1 actions=load:0->OXM_OF_PKT_REG4[48],load:0->OXM_OF_PKT_REG4[49],load:0->OXM_OF_PKT_REG4[50],resubmit(,18)
 cookie=0xb23c013f, duration=297.912s, table=17, n_packets=174, n_bytes=17507, priority=1000,reg8=0x10000/0x10000,metadata=0x3 actions=load:0->OXM_OF_PKT_REG4[48],load:0->OXM_OF_PKT_REG4[49],load:0->OXM_OF_PKT_REG4[50],resubmit(,18)
 cookie=0xfe496347, duration=297.967s, table=17, n_packets=0, n_bytes=0, priority=1000,reg8=0x40000/0x40000,metadata=0x1 actions=load:0->OXM_OF_PKT_REG4[48],load:0->OXM_OF_PKT_REG4[49],load:0->OXM_OF_PKT_REG4[50],load:0->NXM_NX_XXREG0[96..127],controller(userdata=00.00.00.16.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.01.1c.04.00.20.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.01.1e.04.00.20.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.01.1c.04.00.20.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.01.1e.04.00.20.00.00.00.00.00.00.ff.ff.00.10.00.00.23.20.00.0e.ff.f8.30.00.00.00)
 cookie=0xfe496347, duration=297.910s, table=17, n_packets=0, n_bytes=0, priority=1000,reg8=0x40000/0x40000,metadata=0x3 actions=load:0->OXM_OF_PKT_REG4[48],load:0->OXM_OF_PKT_REG4[49],load:0->OXM_OF_PKT_REG4[50],load:0->NXM_NX_XXREG0[96..127],controller(userdata=00.00.00.16.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.01.1c.04.00.20.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.01.1e.04.00.20.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.01.1c.04.00.20.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.01.1e.04.00.20.00.00.00.00.00.00.ff.ff.00.10.00.00.23.20.00.0e.ff.f8.30.00.00.00)
 cookie=0x9980aba1, duration=298.005s, table=17, n_packets=33, n_bytes=5598, priority=0,metadata=0x1 actions=load:0->OXM_OF_PKT_REG4[48],load:0->OXM_OF_PKT_REG4[49],load:0->OXM_OF_PKT_REG4[50],resubmit(,18)
 cookie=0x9980aba1, duration=297.912s, table=17, n_packets=10, n_bytes=588, priority=0,metadata=0x3 actions=load:0->OXM_OF_PKT_REG4[48],load:0->OXM_OF_PKT_REG4[49],load:0->OXM_OF_PKT_REG4[50],resubmit(,18)
 cookie=0xa100323d, duration=297.885s, table=17, n_packets=19, n_bytes=1718, priority=0,metadata=0x4 actions=resubmit(,18)
 cookie=0x51dd0f56, duration=298.006s, table=18, n_packets=42, n_bytes=6292, priority=0,metadata=0x1 actions=resubmit(,19)
 cookie=0x51dd0f56, duration=297.915s, table=18, n_packets=184, n_bytes=18095, priority=0,metadata=0x3 actions=resubmit(,19)
 cookie=0x42a97ef0, duration=297.884s, table=18, n_packets=19, n_bytes=1718, priority=0,metadata=0x4 actions=resubmit(,19)
 cookie=0xefdd4d3, duration=298.021s, table=19, n_packets=42, n_bytes=6292, priority=0,metadata=0x1 actions=resubmit(,20)
 cookie=0xefdd4d3, duration=297.934s, table=19, n_packets=184, n_bytes=18095, priority=0,metadata=0x3 actions=resubmit(,20)
 cookie=0x5a5f0429, duration=297.884s, table=19, n_packets=19, n_bytes=1718, priority=0,metadata=0x4 actions=resubmit(,20)
 cookie=0xfbd881a, duration=298.021s, table=20, n_packets=42, n_bytes=6292, priority=0,metadata=0x1 actions=resubmit(,21)
 cookie=0xfbd881a, duration=297.934s, table=20, n_packets=184, n_bytes=18095, priority=0,metadata=0x3 actions=resubmit(,21)
 cookie=0x3bb03059, duration=297.883s, table=20, n_packets=19, n_bytes=1718, priority=0,metadata=0x4 actions=load:0->NXM_NX_XXREG1[0..31],resubmit(,21)
 cookie=0xade6940b, duration=297.884s, table=21, n_packets=4, n_bytes=248, priority=10550,icmp6,metadata=0x4,nw_ttl=255,icmp_type=133,icmp_code=0 actions=drop
 cookie=0xade6940b, duration=297.883s, table=21, n_packets=0, n_bytes=0, priority=10550,icmp6,metadata=0x4,nw_ttl=255,icmp_type=134,icmp_code=0 actions=drop
 cookie=0x24aa8e52, duration=297.885s, table=21, n_packets=0, n_bytes=0, priority=194,ipv6,reg14=0x1,metadata=0x4,ipv6_dst=fe80::/64 actions=dec_ttl(),load:0->OXM_OF_PKT_REG4[32..47],move:NXM_NX_IPV6_DST[]->NXM_NX_XXREG0[],load:0xf8163efffe358f09->NXM_NX_XXREG1[0..63],load:0xfe80000000000000->NXM_NX_XXREG1[64..127],mod_dl_src:fa:16:3e:35:8f:09,load:0x1->NXM_NX_REG15[],load:0x1->NXM_NX_REG10[0],resubmit(,22)
 cookie=0x64588ded, duration=297.884s, table=21, n_packets=0, n_bytes=0, priority=194,ipv6,reg14=0x2,metadata=0x4,ipv6_dst=fe80::/64 actions=dec_ttl(),load:0->OXM_OF_PKT_REG4[32..47],move:NXM_NX_IPV6_DST[]->NXM_NX_XXREG0[],load:0xf8163efffe15d626->NXM_NX_XXREG1[0..63],load:0xfe80000000000000->NXM_NX_XXREG1[64..127],mod_dl_src:fa:16:3e:15:d6:26,load:0x2->NXM_NX_REG15[],load:0x1->NXM_NX_REG10[0],resubmit(,22)
 cookie=0xacfac2f6, duration=297.885s, table=21, n_packets=4, n_bytes=392, priority=74,ip,metadata=0x4,nw_dst=172.16.0.0/24 actions=dec_ttl(),load:0->OXM_OF_PKT_REG4[32..47],move:NXM_OF_IP_DST[]->NXM_NX_XXREG0[96..127],load:0xac1000a8->NXM_NX_XXREG0[64..95],mod_dl_src:fa:16:3e:15:d6:26,load:0x2->NXM_NX_REG15[],load:0x1->NXM_NX_REG10[0],resubmit(,22)
 cookie=0x90658c0f, duration=297.884s, table=21, n_packets=3, n_bytes=294, priority=74,ip,metadata=0x4,nw_dst=192.168.101.0/24 actions=dec_ttl(),load:0->OXM_OF_PKT_REG4[32..47],move:NXM_OF_IP_DST[]->NXM_NX_XXREG0[96..127],load:0xc0a865fe->NXM_NX_XXREG0[64..95],mod_dl_src:fa:16:3e:35:8f:09,load:0x1->NXM_NX_REG15[],load:0x1->NXM_NX_REG10[0],resubmit(,22)
 cookie=0x1580b0ad, duration=297.885s, table=21, n_packets=8, n_bytes=784, priority=1,ip,reg7=0,metadata=0x4 actions=dec_ttl(),load:0->OXM_OF_PKT_REG4[32..47],load:0xac1000fe->NXM_NX_XXREG0[96..127],load:0xac1000a8->NXM_NX_XXREG0[64..95],mod_dl_src:fa:16:3e:15:d6:26,load:0x2->NXM_NX_REG15[],load:0x1->NXM_NX_REG10[0],resubmit(,22)
 cookie=0xca40fba0, duration=297.968s, table=21, n_packets=42, n_bytes=6292, priority=0,metadata=0x1 actions=resubmit(,22)
 cookie=0xca40fba0, duration=297.912s, table=21, n_packets=184, n_bytes=18095, priority=0,metadata=0x3 actions=resubmit(,22)
 cookie=0x3afb33fe, duration=297.883s, table=21, n_packets=0, n_bytes=0, priority=0,metadata=0x4 actions=drop
 cookie=0xe691cabc, duration=297.884s, table=22, n_packets=15, n_bytes=1470, priority=150,reg8=0/0xffff,metadata=0x4 actions=resubmit(,23)
 cookie=0x4ceeb63d, duration=298.006s, table=22, n_packets=42, n_bytes=6292, priority=0,metadata=0x1 actions=resubmit(,23)
 cookie=0x4ceeb63d, duration=297.934s, table=22, n_packets=184, n_bytes=18095, priority=0,metadata=0x3 actions=resubmit(,23)
 cookie=0xeca5ce6f, duration=297.902s, table=22, n_packets=0, n_bytes=0, priority=0,metadata=0x4 actions=drop
 cookie=0xc554645a, duration=297.968s, table=23, n_packets=42, n_bytes=6292, priority=0,metadata=0x1 actions=resubmit(,24)
 cookie=0xc554645a, duration=297.912s, table=23, n_packets=184, n_bytes=18095, priority=0,metadata=0x3 actions=resubmit(,24)
 cookie=0x96e025c5, duration=297.885s, table=23, n_packets=15, n_bytes=1470, priority=0,metadata=0x4 actions=load:0->OXM_OF_PKT_REG4[32..47],resubmit(,24)
 cookie=0x9a01b0a0, duration=297.884s, table=24, n_packets=15, n_bytes=1470, priority=150,reg8=0/0xffff,metadata=0x4 actions=resubmit(,25)
 cookie=0x729fa573, duration=298.005s, table=24, n_packets=42, n_bytes=6292, priority=0,metadata=0x1 actions=resubmit(,25)
 cookie=0x729fa573, duration=297.915s, table=24, n_packets=184, n_bytes=18095, priority=0,metadata=0x3 actions=resubmit(,25)
 cookie=0xd7217bb2, duration=297.883s, table=24, n_packets=0, n_bytes=0, priority=0,metadata=0x4 actions=drop
 cookie=0x7a0f66c0, duration=297.885s, table=25, n_packets=0, n_bytes=0, priority=500,ipv6,metadata=0x4,dl_dst=33:33:00:00:00:00/ff:ff:00:00:00:00,ipv6_dst=ff00::/8 actions=resubmit(,26)
 cookie=0x7a0f66c0, duration=297.884s, table=25, n_packets=0, n_bytes=0, priority=500,ip,metadata=0x4,dl_dst=01:00:00:00:00:00/01:00:00:00:00:00,nw_dst=224.0.0.0/4 actions=resubmit(,26)
 cookie=0xdb62a891, duration=297.885s, table=25, n_packets=0, n_bytes=0, priority=100,reg0=0xc0a86501,reg15=0x1,metadata=0x4 actions=mod_dl_dst:fa:16:3e:71:9e:ad,resubmit(,26)
 cookie=0x519257e8, duration=297.885s, table=25, n_packets=0, n_bytes=0, priority=100,reg0=0xac100064,reg15=0x2,metadata=0x4 actions=mod_dl_dst:fa:16:3e:ee:57:9a,resubmit(,26)
 cookie=0xd4b3f60e, duration=297.884s, table=25, n_packets=0, n_bytes=0, priority=100,reg0=0xc0a865a2,reg15=0x1,metadata=0x4 actions=mod_dl_dst:fa:16:3e:bf:bb:c5,resubmit(,26)
 cookie=0x5ee7a228, duration=297.884s, table=25, n_packets=0, n_bytes=0, priority=100,reg0=0xac100092,reg15=0x2,metadata=0x4 actions=mod_dl_dst:fa:16:3e:2a:eb:48,resubmit(,26)
 cookie=0xf405d387, duration=297.884s, table=25, n_packets=3, n_bytes=294, priority=100,reg0=0xc0a86519,reg15=0x1,metadata=0x4 actions=mod_dl_dst:fa:16:3e:63:d4:da,resubmit(,26)
 cookie=0x705e606e, duration=297.885s, table=25, n_packets=0, n_bytes=0, priority=2,ip,metadata=0x4,nw_dst=172.16.0.168 actions=drop
 cookie=0xb57bbc9, duration=297.885s, table=25, n_packets=0, n_bytes=0, priority=1,ipv6,metadata=0x4 actions=mod_dl_dst:00:00:00:00:00:00,resubmit(,66),resubmit(,26)
 cookie=0xbc32a929, duration=297.884s, table=25, n_packets=12, n_bytes=1176, priority=1,ip,metadata=0x4 actions=push:NXM_NX_REG0[],push:NXM_NX_XXREG0[96..127],pop:NXM_NX_REG0[],mod_dl_dst:00:00:00:00:00:00,resubmit(,66),pop:NXM_NX_REG0[],resubmit(,26)
 cookie=0xfeeff10a, duration=297.967s, table=25, n_packets=42, n_bytes=6292, priority=0,metadata=0x1 actions=resubmit(,26)
 cookie=0xfeeff10a, duration=297.910s, table=25, n_packets=184, n_bytes=18095, priority=0,metadata=0x3 actions=resubmit(,26)
 cookie=0x7657d2c6, duration=297.883s, table=25, n_packets=0, n_bytes=0, priority=0,metadata=0x4 actions=drop
 cookie=0x4f3941f3, duration=298.006s, table=26, n_packets=1, n_bytes=86, priority=65532,icmp6,metadata=0x1,nw_ttl=255,icmp_type=135,icmp_code=0 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,27)
 cookie=0x4f3941f3, duration=298.006s, table=26, n_packets=0, n_bytes=0, priority=65532,icmp6,metadata=0x1,nw_ttl=255,icmp_type=136,icmp_code=0 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,27)
 cookie=0x4f3941f3, duration=298.006s, table=26, n_packets=0, n_bytes=0, priority=65532,icmp6,metadata=0x1,nw_ttl=255,icmp_type=134,icmp_code=0 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,27)
 cookie=0x4f3941f3, duration=298.006s, table=26, n_packets=4, n_bytes=248, priority=65532,icmp6,metadata=0x1,nw_ttl=255,icmp_type=133,icmp_code=0 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,27)
 cookie=0x4f3941f3, duration=297.915s, table=26, n_packets=2, n_bytes=172, priority=65532,icmp6,metadata=0x3,nw_ttl=255,icmp_type=135,icmp_code=0 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,27)
 cookie=0x4f3941f3, duration=297.915s, table=26, n_packets=0, n_bytes=0, priority=65532,icmp6,metadata=0x3,nw_ttl=255,icmp_type=136,icmp_code=0 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,27)
 cookie=0x4f3941f3, duration=297.915s, table=26, n_packets=0, n_bytes=0, priority=65532,icmp6,metadata=0x3,nw_ttl=255,icmp_type=134,icmp_code=0 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,27)
 cookie=0x4f3941f3, duration=297.915s, table=26, n_packets=7, n_bytes=490, priority=65532,icmp6,metadata=0x3,nw_ttl=255,icmp_type=133,icmp_code=0 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,27)
 cookie=0x4f3941f3, duration=298.006s, table=26, n_packets=0, n_bytes=0, priority=65532,icmp6,metadata=0x1,ipv6_src=fe80::/10,icmp_type=130 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,27)
 cookie=0x4f3941f3, duration=298.006s, table=26, n_packets=0, n_bytes=0, priority=65532,icmp6,metadata=0x1,ipv6_src=fe80::/10,icmp_type=131 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,27)
 cookie=0x4f3941f3, duration=298.006s, table=26, n_packets=0, n_bytes=0, priority=65532,icmp6,metadata=0x1,ipv6_src=fe80::/10,icmp_type=132 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,27)
 cookie=0x4f3941f3, duration=297.915s, table=26, n_packets=0, n_bytes=0, priority=65532,icmp6,metadata=0x3,ipv6_src=fe80::/10,icmp_type=130 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,27)
 cookie=0x4f3941f3, duration=297.915s, table=26, n_packets=0, n_bytes=0, priority=65532,icmp6,metadata=0x3,ipv6_src=fe80::/10,icmp_type=131 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,27)
 cookie=0x4f3941f3, duration=297.915s, table=26, n_packets=0, n_bytes=0, priority=65532,icmp6,metadata=0x3,ipv6_src=fe80::/10,icmp_type=132 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,27)
 cookie=0x4f3941f3, duration=298.006s, table=26, n_packets=4, n_bytes=360, priority=65532,icmp6,metadata=0x1,ipv6_dst=ff02::16,icmp_type=143 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,27)
 cookie=0x4f3941f3, duration=297.915s, table=26, n_packets=2, n_bytes=220, priority=65532,icmp6,metadata=0x3,ipv6_dst=ff02::16,icmp_type=143 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,27)
 cookie=0x88f43f04, duration=298.005s, table=26, n_packets=0, n_bytes=0, priority=65532,reg0=0x20000/0x20000,metadata=0x1 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,27)
 cookie=0x88f43f04, duration=297.912s, table=26, n_packets=50, n_bytes=6167, priority=65532,reg0=0x20000/0x20000,metadata=0x3 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,27)
 cookie=0x2287e941, duration=298.021s, table=26, n_packets=33, n_bytes=5598, priority=0,metadata=0x1 actions=resubmit(,27)
 cookie=0x2287e941, duration=297.934s, table=26, n_packets=123, n_bytes=11046, priority=0,metadata=0x3 actions=resubmit(,27)
 cookie=0x21035fca, duration=297.885s, table=26, n_packets=15, n_bytes=1470, priority=0,metadata=0x4 actions=resubmit(,27)
 cookie=0x154425f, duration=298.021s, table=27, n_packets=0, n_bytes=0, priority=1000,reg8=0x40000/0x40000,metadata=0x1 actions=load:0->OXM_OF_PKT_REG4[48],load:0->OXM_OF_PKT_REG4[49],load:0->OXM_OF_PKT_REG4[50],load:0->NXM_NX_XXREG0[96..127],controller(userdata=00.00.00.16.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.01.1c.04.00.20.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.01.1e.04.00.20.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.01.1c.04.00.20.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.01.1e.04.00.20.00.00.00.00.00.00.ff.ff.00.10.00.00.23.20.00.0e.ff.f8.30.00.00.00)
 cookie=0x154425f, duration=297.967s, table=27, n_packets=0, n_bytes=0, priority=1000,reg8=0x40000/0x40000,metadata=0x3 actions=load:0->OXM_OF_PKT_REG4[48],load:0->OXM_OF_PKT_REG4[49],load:0->OXM_OF_PKT_REG4[50],load:0->NXM_NX_XXREG0[96..127],controller(userdata=00.00.00.16.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.01.1c.04.00.20.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.01.1e.04.00.20.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.01.1c.04.00.20.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.01.1e.04.00.20.00.00.00.00.00.00.ff.ff.00.10.00.00.23.20.00.0e.ff.f8.30.00.00.00)
 cookie=0x8eb10ae1, duration=298.005s, table=27, n_packets=9, n_bytes=694, priority=1000,reg8=0x10000/0x10000,metadata=0x1 actions=load:0->OXM_OF_PKT_REG4[48],load:0->OXM_OF_PKT_REG4[49],load:0->OXM_OF_PKT_REG4[50],resubmit(,28)
 cookie=0x8eb10ae1, duration=297.912s, table=27, n_packets=61, n_bytes=7049, priority=1000,reg8=0x10000/0x10000,metadata=0x3 actions=load:0->OXM_OF_PKT_REG4[48],load:0->OXM_OF_PKT_REG4[49],load:0->OXM_OF_PKT_REG4[50],resubmit(,28)
 cookie=0xc04ee3cc, duration=297.968s, table=27, n_packets=0, n_bytes=0, priority=1000,reg8=0x20000/0x20000,metadata=0x1 actions=load:0->OXM_OF_PKT_REG4[48],load:0->OXM_OF_PKT_REG4[49],load:0->OXM_OF_PKT_REG4[50]
 cookie=0xc04ee3cc, duration=297.912s, table=27, n_packets=0, n_bytes=0, priority=1000,reg8=0x20000/0x20000,metadata=0x3 actions=load:0->OXM_OF_PKT_REG4[48],load:0->OXM_OF_PKT_REG4[49],load:0->OXM_OF_PKT_REG4[50]
 cookie=0x758e98d1, duration=298.005s, table=27, n_packets=33, n_bytes=5598, priority=0,metadata=0x1 actions=load:0->OXM_OF_PKT_REG4[48],load:0->OXM_OF_PKT_REG4[49],load:0->OXM_OF_PKT_REG4[50],resubmit(,28)
 cookie=0x758e98d1, duration=297.915s, table=27, n_packets=123, n_bytes=11046, priority=0,metadata=0x3 actions=load:0->OXM_OF_PKT_REG4[48],load:0->OXM_OF_PKT_REG4[49],load:0->OXM_OF_PKT_REG4[50],resubmit(,28)
 cookie=0x43312337, duration=297.883s, table=27, n_packets=15, n_bytes=1470, priority=0,metadata=0x4 actions=resubmit(,28)
 cookie=0x6d95a6d5, duration=298.006s, table=28, n_packets=0, n_bytes=0, priority=100,ipv6,reg0=0x2002/0x2002,metadata=0x1 actions=ct(commit,zone=NXM_NX_REG13[0..15],nat(src),exec(load:0->NXM_NX_CT_MARK[0],move:NXM_NX_XXREG0[0..31]->NXM_NX_CT_LABEL[96..127])),resubmit(,29)
 cookie=0x6d95a6d5, duration=298.005s, table=28, n_packets=0, n_bytes=0, priority=100,ip,reg0=0x2002/0x2002,metadata=0x1 actions=ct(commit,zone=NXM_NX_REG13[0..15],nat(src),exec(load:0->NXM_NX_CT_MARK[0],move:NXM_NX_XXREG0[0..31]->NXM_NX_CT_LABEL[96..127])),resubmit(,29)
 cookie=0x73c54fdb, duration=298.005s, table=28, n_packets=0, n_bytes=0, priority=100,ip,reg0=0x2/0x2002,metadata=0x1 actions=ct(commit,zone=NXM_NX_REG13[0..15],nat(src),exec(load:0->NXM_NX_CT_MARK[0])),resubmit(,29)
 cookie=0x73c54fdb, duration=298.005s, table=28, n_packets=0, n_bytes=0, priority=100,ipv6,reg0=0x2/0x2002,metadata=0x1 actions=ct(commit,zone=NXM_NX_REG13[0..15],nat(src),exec(load:0->NXM_NX_CT_MARK[0])),resubmit(,29)
 cookie=0x6d95a6d5, duration=297.915s, table=28, n_packets=0, n_bytes=0, priority=100,ipv6,reg0=0x2002/0x2002,metadata=0x3 actions=ct(commit,zone=NXM_NX_REG13[0..15],nat(src),exec(load:0->NXM_NX_CT_MARK[0],move:NXM_NX_XXREG0[0..31]->NXM_NX_CT_LABEL[96..127])),resubmit(,29)
 cookie=0x6d95a6d5, duration=297.915s, table=28, n_packets=0, n_bytes=0, priority=100,ip,reg0=0x2002/0x2002,metadata=0x3 actions=ct(commit,zone=NXM_NX_REG13[0..15],nat(src),exec(load:0->NXM_NX_CT_MARK[0],move:NXM_NX_XXREG0[0..31]->NXM_NX_CT_LABEL[96..127])),resubmit(,29)
 cookie=0x73c54fdb, duration=297.915s, table=28, n_packets=31, n_bytes=2654, priority=100,ip,reg0=0x2/0x2002,metadata=0x3 actions=ct(commit,zone=NXM_NX_REG13[0..15],nat(src),exec(load:0->NXM_NX_CT_MARK[0])),resubmit(,29)
 cookie=0x73c54fdb, duration=297.915s, table=28, n_packets=0, n_bytes=0, priority=100,ipv6,reg0=0x2/0x2002,metadata=0x3 actions=ct(commit,zone=NXM_NX_REG13[0..15],nat(src),exec(load:0->NXM_NX_CT_MARK[0])),resubmit(,29)
 cookie=0x76504d1, duration=298.021s, table=28, n_packets=42, n_bytes=6292, priority=0,metadata=0x1 actions=resubmit(,29)
 cookie=0x76504d1, duration=297.934s, table=28, n_packets=153, n_bytes=15441, priority=0,metadata=0x3 actions=resubmit(,29)
 cookie=0x259e1b5c, duration=297.883s, table=28, n_packets=15, n_bytes=1470, priority=0,metadata=0x4 actions=resubmit(,29)
 cookie=0xcdac59db, duration=297.885s, table=29, n_packets=0, n_bytes=0, priority=100,ipv6,metadata=0x4,dl_dst=00:00:00:00:00:00 actions=controller(userdata=00.00.00.09.00.00.00.00.00.1c.00.18.00.80.00.00.00.00.00.00.00.01.de.10.80.00.3e.10.00.00.00.00.ff.ff.00.10.00.00.23.20.00.0e.ff.f8.25.00.00.00)
 cookie=0x9856503d, duration=297.884s, table=29, n_packets=2, n_bytes=196, priority=100,ip,metadata=0x4,dl_dst=00:00:00:00:00:00 actions=controller(userdata=00.00.00.00.00.00.00.00.00.19.00.10.80.00.06.06.ff.ff.ff.ff.ff.ff.00.00.00.1c.00.18.00.20.00.40.00.00.00.00.00.01.de.10.80.00.2c.04.00.00.00.00.00.1c.00.18.00.20.00.60.00.00.00.00.00.01.de.10.80.00.2e.04.00.00.00.00.00.19.00.10.80.00.2a.02.00.01.00.00.00.00.00.00.ff.ff.00.10.00.00.23.20.00.0e.ff.f8.25.00.00.00)
 cookie=0x4b4f6a96, duration=297.884s, table=29, n_packets=0, n_bytes=0, priority=100,arp,reg14=0x5,metadata=0x3,arp_tpa=192.168.101.254,arp_op=1 actions=resubmit(,30)
 cookie=0x9788c626, duration=297.884s, table=29, n_packets=0, n_bytes=0, priority=100,arp,reg14=0x4,metadata=0x1,arp_tpa=172.16.0.168,arp_op=1 actions=resubmit(,30)
 cookie=0x978b178b, duration=297.859s, table=29, n_packets=2, n_bytes=84, priority=100,arp,reg14=0x2,metadata=0x3,arp_tpa=192.168.101.25,arp_op=1 actions=resubmit(,30)
 cookie=0xd758b859, duration=290.822s, table=29, n_packets=0, n_bytes=0, priority=100,arp,reg14=0x1,metadata=0x3,arp_tpa=192.168.101.1,arp_op=1 actions=resubmit(,30)
 cookie=0xd0f6471e, duration=297.884s, table=29, n_packets=0, n_bytes=0, priority=100,icmp6,reg14=0x5,metadata=0x3,ipv6_dst=fe80::f816:3eff:fe35:8f09,nw_ttl=255,icmp_type=135,icmp_code=0,nd_target=fe80::f816:3eff:fe35:8f09 actions=resubmit(,30)
 cookie=0xd0f6471e, duration=297.884s, table=29, n_packets=0, n_bytes=0, priority=100,icmp6,reg14=0x5,metadata=0x3,ipv6_dst=ff02::1:ff35:8f09,nw_ttl=255,icmp_type=135,icmp_code=0,nd_target=fe80::f816:3eff:fe35:8f09 actions=resubmit(,30)
 cookie=0xbe2dad7c, duration=297.884s, table=29, n_packets=0, n_bytes=0, priority=100,icmp6,reg14=0x4,metadata=0x1,ipv6_dst=ff02::1:ff15:d626,nw_ttl=255,icmp_type=135,icmp_code=0,nd_target=fe80::f816:3eff:fe15:d626 actions=resubmit(,30)
 cookie=0xbe2dad7c, duration=297.883s, table=29, n_packets=0, n_bytes=0, priority=100,icmp6,reg14=0x4,metadata=0x1,ipv6_dst=fe80::f816:3eff:fe15:d626,nw_ttl=255,icmp_type=135,icmp_code=0,nd_target=fe80::f816:3eff:fe15:d626 actions=resubmit(,30)
 cookie=0x66e3367a, duration=297.884s, table=29, n_packets=28, n_bytes=5032, priority=100,reg14=0x1,metadata=0x1 actions=resubmit(,30)
 cookie=0xc5faa0cc, duration=297.885s, table=29, n_packets=0, n_bytes=0, priority=50,arp,metadata=0x1,arp_tpa=172.16.0.100,arp_op=1 actions=move:NXM_OF_ETH_SRC[]->NXM_OF_ETH_DST[],mod_dl_src:fa:16:3e:ee:57:9a,load:0x2->NXM_OF_ARP_OP[],move:NXM_NX_ARP_SHA[]->NXM_NX_ARP_THA[],load:0xfa163eee579a->NXM_NX_ARP_SHA[],move:NXM_OF_ARP_SPA[]->NXM_OF_ARP_TPA[],load:0xac100064->NXM_OF_ARP_SPA[],move:NXM_NX_REG14[]->NXM_NX_REG15[],load:0x1->NXM_NX_REG10[0],resubmit(,37)
 cookie=0xff945f30, duration=297.885s, table=29, n_packets=3, n_bytes=126, priority=50,arp,metadata=0x3,arp_tpa=192.168.101.254,arp_op=1 actions=move:NXM_OF_ETH_SRC[]->NXM_OF_ETH_DST[],mod_dl_src:fa:16:3e:35:8f:09,load:0x2->NXM_OF_ARP_OP[],move:NXM_NX_ARP_SHA[]->NXM_NX_ARP_THA[],load:0xfa163e358f09->NXM_NX_ARP_SHA[],move:NXM_OF_ARP_SPA[]->NXM_OF_ARP_TPA[],load:0xc0a865fe->NXM_OF_ARP_SPA[],move:NXM_NX_REG14[]->NXM_NX_REG15[],load:0x1->NXM_NX_REG10[0],resubmit(,37)
 cookie=0xeca1669, duration=297.885s, table=29, n_packets=1, n_bytes=42, priority=50,arp,metadata=0x3,arp_tpa=192.168.101.1,arp_op=1 actions=move:NXM_OF_ETH_SRC[]->NXM_OF_ETH_DST[],mod_dl_src:fa:16:3e:71:9e:ad,load:0x2->NXM_OF_ARP_OP[],move:NXM_NX_ARP_SHA[]->NXM_NX_ARP_THA[],load:0xfa163e719ead->NXM_NX_ARP_SHA[],move:NXM_OF_ARP_SPA[]->NXM_OF_ARP_TPA[],load:0xc0a86501->NXM_OF_ARP_SPA[],move:NXM_NX_REG14[]->NXM_NX_REG15[],load:0x1->NXM_NX_REG10[0],resubmit(,37)
 cookie=0x87fcda8, duration=297.884s, table=29, n_packets=0, n_bytes=0, priority=50,arp,metadata=0x1,arp_tpa=172.16.0.168,arp_op=1 actions=move:NXM_OF_ETH_SRC[]->NXM_OF_ETH_DST[],mod_dl_src:fa:16:3e:15:d6:26,load:0x2->NXM_OF_ARP_OP[],move:NXM_NX_ARP_SHA[]->NXM_NX_ARP_THA[],load:0xfa163e15d626->NXM_NX_ARP_SHA[],move:NXM_OF_ARP_SPA[]->NXM_OF_ARP_TPA[],load:0xac1000a8->NXM_OF_ARP_SPA[],move:NXM_NX_REG14[]->NXM_NX_REG15[],load:0x1->NXM_NX_REG10[0],resubmit(,37)
 cookie=0x96c3bafb, duration=297.859s, table=29, n_packets=1, n_bytes=42, priority=50,arp,metadata=0x3,arp_tpa=192.168.101.25,arp_op=1 actions=move:NXM_OF_ETH_SRC[]->NXM_OF_ETH_DST[],mod_dl_src:fa:16:3e:63:d4:da,load:0x2->NXM_OF_ARP_OP[],move:NXM_NX_ARP_SHA[]->NXM_NX_ARP_THA[],load:0xfa163e63d4da->NXM_NX_ARP_SHA[],move:NXM_OF_ARP_SPA[]->NXM_OF_ARP_TPA[],load:0xc0a86519->NXM_OF_ARP_SPA[],move:NXM_NX_REG14[]->NXM_NX_REG15[],load:0x1->NXM_NX_REG10[0],resubmit(,37)
 cookie=0x7e9711b7, duration=297.885s, table=29, n_packets=0, n_bytes=0, priority=50,icmp6,metadata=0x3,ipv6_dst=ff02::1:ff35:8f09,nw_ttl=255,icmp_type=135,icmp_code=0,nd_target=fe80::f816:3eff:fe35:8f09 actions=controller(userdata=00.00.00.0c.00.00.00.00.00.19.00.10.80.00.08.06.fa.16.3e.35.8f.09.00.00.00.19.00.18.80.00.34.10.fe.80.00.00.00.00.00.00.f8.16.3e.ff.fe.35.8f.09.00.19.00.18.80.00.3e.10.fe.80.00.00.00.00.00.00.f8.16.3e.ff.fe.35.8f.09.00.19.00.10.80.00.42.06.fa.16.3e.35.8f.09.00.00.00.1c.00.18.00.20.00.00.00.00.00.00.00.01.1c.04.00.01.1e.04.00.00.00.00.00.19.00.10.00.01.15.08.00.00.00.01.00.00.00.01.ff.ff.00.10.00.00.23.20.00.0e.ff.f8.25.00.00.00)
 cookie=0x191b77b6, duration=297.884s, table=29, n_packets=0, n_bytes=0, priority=50,icmp6,metadata=0x1,ipv6_dst=fe80::f816:3eff:fe15:d626,nw_ttl=255,icmp_type=135,icmp_code=0,nd_target=fe80::f816:3eff:fe15:d626 actions=controller(userdata=00.00.00.0c.00.00.00.00.00.19.00.10.80.00.08.06.fa.16.3e.15.d6.26.00.00.00.19.00.18.80.00.34.10.fe.80.00.00.00.00.00.00.f8.16.3e.ff.fe.15.d6.26.00.19.00.18.80.00.3e.10.fe.80.00.00.00.00.00.00.f8.16.3e.ff.fe.15.d6.26.00.19.00.10.80.00.42.06.fa.16.3e.15.d6.26.00.00.00.1c.00.18.00.20.00.00.00.00.00.00.00.01.1c.04.00.01.1e.04.00.00.00.00.00.19.00.10.00.01.15.08.00.00.00.01.00.00.00.01.ff.ff.00.10.00.00.23.20.00.0e.ff.f8.25.00.00.00)
 cookie=0x7e9711b7, duration=297.884s, table=29, n_packets=0, n_bytes=0, priority=50,icmp6,metadata=0x3,ipv6_dst=fe80::f816:3eff:fe35:8f09,nw_ttl=255,icmp_type=135,icmp_code=0,nd_target=fe80::f816:3eff:fe35:8f09 actions=controller(userdata=00.00.00.0c.00.00.00.00.00.19.00.10.80.00.08.06.fa.16.3e.35.8f.09.00.00.00.19.00.18.80.00.34.10.fe.80.00.00.00.00.00.00.f8.16.3e.ff.fe.35.8f.09.00.19.00.18.80.00.3e.10.fe.80.00.00.00.00.00.00.f8.16.3e.ff.fe.35.8f.09.00.19.00.10.80.00.42.06.fa.16.3e.35.8f.09.00.00.00.1c.00.18.00.20.00.00.00.00.00.00.00.01.1c.04.00.01.1e.04.00.00.00.00.00.19.00.10.00.01.15.08.00.00.00.01.00.00.00.01.ff.ff.00.10.00.00.23.20.00.0e.ff.f8.25.00.00.00)
 cookie=0x191b77b6, duration=297.883s, table=29, n_packets=0, n_bytes=0, priority=50,icmp6,metadata=0x1,ipv6_dst=ff02::1:ff15:d626,nw_ttl=255,icmp_type=135,icmp_code=0,nd_target=fe80::f816:3eff:fe15:d626 actions=controller(userdata=00.00.00.0c.00.00.00.00.00.19.00.10.80.00.08.06.fa.16.3e.15.d6.26.00.00.00.19.00.18.80.00.34.10.fe.80.00.00.00.00.00.00.f8.16.3e.ff.fe.15.d6.26.00.19.00.18.80.00.3e.10.fe.80.00.00.00.00.00.00.f8.16.3e.ff.fe.15.d6.26.00.19.00.10.80.00.42.06.fa.16.3e.15.d6.26.00.00.00.1c.00.18.00.20.00.00.00.00.00.00.00.01.1c.04.00.01.1e.04.00.00.00.00.00.19.00.10.00.01.15.08.00.00.00.01.00.00.00.01.ff.ff.00.10.00.00.23.20.00.0e.ff.f8.25.00.00.00)
 cookie=0x97308c22, duration=298.005s, table=29, n_packets=14, n_bytes=1260, priority=0,metadata=0x1 actions=resubmit(,30)
 cookie=0x97308c22, duration=297.912s, table=29, n_packets=177, n_bytes=17801, priority=0,metadata=0x3 actions=resubmit(,30)
 cookie=0x4538dba0, duration=297.884s, table=29, n_packets=13, n_bytes=1274, priority=0,metadata=0x4 actions=resubmit(,37)
 cookie=0x7d1e4119, duration=297.885s, table=30, n_packets=2, n_bytes=684, priority=100,conj_id=3117231089,udp,reg14=0x2,metadata=0x3,dl_src=fa:16:3e:63:d4:da,tp_src=68,tp_dst=67 actions=controller(userdata=00.00.00.02.00.00.00.00.00.01.de.10.00.00.00.63.c0.a8.65.19.79.0e.20.a9.fe.a9.fe.c0.a8.65.01.00.c0.a8.65.fe.06.04.0a.00.00.fe.33.04.00.00.a8.c0.1a.02.05.a2.01.04.ff.ff.ff.00.03.04.c0.a8.65.fe.36.04.c0.a8.65.fe,pause),resubmit(,31)
 cookie=0x0, duration=297.885s, table=30, n_packets=0, n_bytes=0, priority=100,udp,reg14=0x2,metadata=0x3,dl_src=fa:16:3e:63:d4:da,nw_src=192.168.101.25,tp_src=68,tp_dst=67 actions=conjunction(3117231089,2/2)
 cookie=0x0, duration=297.884s, table=30, n_packets=0, n_bytes=0, priority=100,udp,reg14=0x2,metadata=0x3,dl_src=fa:16:3e:63:d4:da,nw_src=0.0.0.0,tp_src=68,tp_dst=67 actions=conjunction(3117231089,2/2)
 cookie=0x0, duration=297.884s, table=30, n_packets=0, n_bytes=0, priority=100,udp,reg14=0x2,metadata=0x3,dl_src=fa:16:3e:63:d4:da,nw_dst=192.168.101.254,tp_src=68,tp_dst=67 actions=conjunction(3117231089,1/2)
 cookie=0x0, duration=297.883s, table=30, n_packets=0, n_bytes=0, priority=100,udp,reg14=0x2,metadata=0x3,dl_src=fa:16:3e:63:d4:da,nw_dst=255.255.255.255,tp_src=68,tp_dst=67 actions=conjunction(3117231089,1/2)
 cookie=0xb85f6e91, duration=297.968s, table=30, n_packets=42, n_bytes=6292, priority=0,metadata=0x1 actions=resubmit(,31)
 cookie=0xb85f6e91, duration=297.912s, table=30, n_packets=177, n_bytes=17201, priority=0,metadata=0x3 actions=resubmit(,31)
 cookie=0x719d4847, duration=297.883s, table=31, n_packets=2, n_bytes=688, priority=100,udp,reg0=0x8/0x8,reg14=0x2,metadata=0x3,dl_src=fa:16:3e:63:d4:da,tp_src=68,tp_dst=67 actions=move:NXM_OF_ETH_SRC[]->NXM_OF_ETH_DST[],mod_dl_src:fa:16:3e:73:d1:b0,mod_nw_src:192.168.101.254,mod_tp_src:67,mod_tp_dst:68,move:NXM_NX_REG14[]->NXM_NX_REG15[],load:0x1->NXM_NX_REG10[0],resubmit(,37)
 cookie=0xe0ec5c52, duration=297.968s, table=31, n_packets=42, n_bytes=6292, priority=0,metadata=0x1 actions=resubmit(,32)
 cookie=0xe0ec5c52, duration=297.910s, table=31, n_packets=177, n_bytes=17201, priority=0,metadata=0x3 actions=resubmit(,32)
 cookie=0x7db3f230, duration=298.005s, table=32, n_packets=42, n_bytes=6292, priority=0,metadata=0x1 actions=resubmit(,33)
 cookie=0x7db3f230, duration=297.915s, table=32, n_packets=177, n_bytes=17201, priority=0,metadata=0x3 actions=resubmit(,33)
 cookie=0xe941f155, duration=297.967s, table=33, n_packets=42, n_bytes=6292, priority=0,metadata=0x1 actions=resubmit(,34)
 cookie=0xe941f155, duration=297.910s, table=33, n_packets=177, n_bytes=17201, priority=0,metadata=0x3 actions=resubmit(,34)
 cookie=0x2ada6e51, duration=298.021s, table=34, n_packets=42, n_bytes=6292, priority=0,metadata=0x1 actions=resubmit(,35)
 cookie=0x2ada6e51, duration=297.934s, table=34, n_packets=177, n_bytes=17201, priority=0,metadata=0x3 actions=resubmit(,35)
 cookie=0x4de78d87, duration=298.006s, table=35, n_packets=0, n_bytes=0, priority=110,tcp,metadata=0x1,dl_dst=b6:9c:90:73:34:da actions=controller(userdata=00.00.00.12.00.00.00.00)
 cookie=0x4de78d87, duration=298.006s, table=35, n_packets=0, n_bytes=0, priority=110,icmp6,metadata=0x1,dl_dst=b6:9c:90:73:34:da actions=controller(userdata=00.00.00.12.00.00.00.00)
 cookie=0x4de78d87, duration=298.006s, table=35, n_packets=0, n_bytes=0, priority=110,icmp,metadata=0x1,dl_dst=b6:9c:90:73:34:da actions=controller(userdata=00.00.00.12.00.00.00.00)
 cookie=0x4de78d87, duration=298.006s, table=35, n_packets=0, n_bytes=0, priority=110,tcp6,metadata=0x1,dl_dst=b6:9c:90:73:34:da actions=controller(userdata=00.00.00.12.00.00.00.00)
 cookie=0x4de78d87, duration=297.934s, table=35, n_packets=0, n_bytes=0, priority=110,tcp,metadata=0x3,dl_dst=b6:9c:90:73:34:da actions=controller(userdata=00.00.00.12.00.00.00.00)
 cookie=0x4de78d87, duration=297.934s, table=35, n_packets=0, n_bytes=0, priority=110,icmp6,metadata=0x3,dl_dst=b6:9c:90:73:34:da actions=controller(userdata=00.00.00.12.00.00.00.00)
 cookie=0x4de78d87, duration=297.915s, table=35, n_packets=0, n_bytes=0, priority=110,icmp,metadata=0x3,dl_dst=b6:9c:90:73:34:da actions=controller(userdata=00.00.00.12.00.00.00.00)
 cookie=0x4de78d87, duration=297.915s, table=35, n_packets=0, n_bytes=0, priority=110,tcp6,metadata=0x3,dl_dst=b6:9c:90:73:34:da actions=controller(userdata=00.00.00.12.00.00.00.00)
 cookie=0xf1d1d355, duration=297.885s, table=35, n_packets=0, n_bytes=0, priority=80,arp,reg10=0/0x2,metadata=0x1,arp_tpa=172.16.0.168,arp_op=1 actions=clone(load:0x4->NXM_NX_REG15[],resubmit(,37)),load:0x8004->NXM_NX_REG15[],resubmit(,37)
 cookie=0xf863795e, duration=297.885s, table=35, n_packets=0, n_bytes=0, priority=80,arp,reg10=0/0x2,metadata=0x3,arp_tpa=192.168.101.254,arp_op=1 actions=clone(load:0x5->NXM_NX_REG15[],resubmit(,37)),load:0x8004->NXM_NX_REG15[],resubmit(,37)
 cookie=0xe27e2b66, duration=297.885s, table=35, n_packets=0, n_bytes=0, priority=80,icmp6,reg10=0/0x2,metadata=0x3,nw_ttl=255,icmp_type=135,icmp_code=0,nd_target=fe80::f816:3eff:fe35:8f09 actions=clone(load:0x5->NXM_NX_REG15[],resubmit(,37)),load:0x8004->NXM_NX_REG15[],resubmit(,37)
 cookie=0xe8894875, duration=297.883s, table=35, n_packets=0, n_bytes=0, priority=80,icmp6,reg10=0/0x2,metadata=0x1,nw_ttl=255,icmp_type=135,icmp_code=0,nd_target=fe80::f816:3eff:fe15:d626 actions=clone(load:0x4->NXM_NX_REG15[],resubmit(,37)),load:0x8004->NXM_NX_REG15[],resubmit(,37)
 cookie=0xce433baf, duration=297.885s, table=35, n_packets=0, n_bytes=0, priority=75,arp,metadata=0x3,dl_src=fa:16:3e:35:8f:09,arp_op=1 actions=load:0x8004->NXM_NX_REG15[],resubmit(,37)
 cookie=0x32ef7337, duration=297.885s, table=35, n_packets=0, n_bytes=0, priority=75,rarp,metadata=0x1,dl_src=fa:16:3e:15:d6:26,arp_op=3 actions=load:0x8004->NXM_NX_REG15[],resubmit(,37)
 cookie=0xce433baf, duration=297.885s, table=35, n_packets=0, n_bytes=0, priority=75,rarp,metadata=0x3,dl_src=fa:16:3e:35:8f:09,arp_op=3 actions=load:0x8004->NXM_NX_REG15[],resubmit(,37)
 cookie=0x32ef7337, duration=297.884s, table=35, n_packets=2, n_bytes=84, priority=75,arp,metadata=0x1,dl_src=fa:16:3e:15:d6:26,arp_op=1 actions=load:0x8004->NXM_NX_REG15[],resubmit(,37)
 cookie=0xce433baf, duration=297.885s, table=35, n_packets=0, n_bytes=0, priority=75,icmp6,metadata=0x3,dl_src=fa:16:3e:35:8f:09,nw_ttl=255,icmp_type=135,icmp_code=0 actions=load:0x8004->NXM_NX_REG15[],resubmit(,37)
 cookie=0x32ef7337, duration=297.885s, table=35, n_packets=0, n_bytes=0, priority=75,icmp6,metadata=0x1,dl_src=fa:16:3e:15:d6:26,nw_ttl=255,icmp_type=135,icmp_code=0 actions=load:0x8004->NXM_NX_REG15[],resubmit(,37)
 cookie=0x2dface73, duration=297.934s, table=35, n_packets=13, n_bytes=966, priority=70,metadata=0x3,dl_dst=01:00:00:00:00:00/01:00:00:00:00:00 actions=load:0x8000->NXM_NX_REG15[],resubmit(,37)
 cookie=0x2dface73, duration=298.006s, table=35, n_packets=26, n_bytes=4948, priority=70,metadata=0x1,dl_dst=01:00:00:00:00:00/01:00:00:00:00:00 actions=load:0x8000->NXM_NX_REG15[],resubmit(,37)
 cookie=0x2d7b143c, duration=297.885s, table=35, n_packets=96, n_bytes=8304, priority=50,metadata=0x3,dl_dst=fa:16:3e:71:9e:ad actions=load:0x1->NXM_NX_REG15[],resubmit(,37)
 cookie=0x73e3b16a, duration=297.884s, table=35, n_packets=15, n_bytes=1470, priority=50,metadata=0x3,dl_dst=fa:16:3e:35:8f:09 actions=load:0x5->NXM_NX_REG15[],resubmit(,37)
 cookie=0x7b8da6, duration=297.884s, table=35, n_packets=0, n_bytes=0, priority=50,metadata=0x1,dl_dst=fa:16:3e:2a:eb:48 actions=load:0x3->NXM_NX_REG15[],resubmit(,37)
 cookie=0x17e71698, duration=297.884s, table=35, n_packets=0, n_bytes=0, priority=50,metadata=0x1,dl_dst=fa:16:3e:ee:57:9a actions=load:0x2->NXM_NX_REG15[],resubmit(,37)
 cookie=0x7d236373, duration=297.884s, table=35, n_packets=0, n_bytes=0, priority=50,metadata=0x3,dl_dst=fa:16:3e:bf:bb:c5 actions=load:0x4->NXM_NX_REG15[],resubmit(,37)
 cookie=0x7990b163, duration=297.883s, table=35, n_packets=53, n_bytes=6461, priority=50,metadata=0x3,dl_dst=fa:16:3e:63:d4:da actions=load:0x2->NXM_NX_REG15[],resubmit(,37)
 cookie=0xfeebbe95, duration=297.883s, table=35, n_packets=2, n_bytes=84, priority=50,metadata=0x1,dl_dst=fa:16:3e:15:d6:26 actions=load:0x4->NXM_NX_REG15[],resubmit(,37)
 cookie=0xa8092c9c, duration=298.005s, table=35, n_packets=12, n_bytes=1176, priority=0,metadata=0x1 actions=load:0->NXM_NX_REG15[],resubmit(,71),resubmit(,36)
 cookie=0xa8092c9c, duration=297.912s, table=35, n_packets=0, n_bytes=0, priority=0,metadata=0x3 actions=load:0->NXM_NX_REG15[],resubmit(,71),resubmit(,36)
 cookie=0x61ea2c49, duration=297.967s, table=36, n_packets=12, n_bytes=1176, priority=50,reg15=0,metadata=0x1 actions=load:0x8001->NXM_NX_REG15[],resubmit(,37)
 cookie=0x754a7ef0, duration=297.885s, table=36, n_packets=0, n_bytes=0, priority=50,reg15=0,metadata=0x3 actions=drop
 cookie=0x98c8931d, duration=298.005s, table=36, n_packets=0, n_bytes=0, priority=0,metadata=0x1 actions=resubmit(,37)
 cookie=0x98c8931d, duration=297.912s, table=36, n_packets=0, n_bytes=0, priority=0,metadata=0x3 actions=resubmit(,37)
 cookie=0x0, duration=2294.001s, table=37, n_packets=243, n_bytes=25945, priority=0 actions=resubmit(,39)
 cookie=0x0, duration=2294.001s, table=38, n_packets=0, n_bytes=0, priority=10,reg10=0x1/0x1 actions=resubmit(,40)
 cookie=0x0, duration=2294.001s, table=38, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,39)
 cookie=0x0, duration=2294.001s, table=39, n_packets=0, n_bytes=0, priority=150,reg10=0x2/0x3 actions=resubmit(,40)
 cookie=0x0, duration=2294.001s, table=39, n_packets=0, n_bytes=0, priority=150,reg10=0x10/0x10 actions=resubmit(,40)
 cookie=0x9b5b3836, duration=290.822s, table=39, n_packets=62, n_bytes=7091, priority=150,reg14=0x1,metadata=0x3 actions=resubmit(,40)
 cookie=0x7265e3e4, duration=297.883s, table=39, n_packets=0, n_bytes=0, priority=100,reg15=0x8004,metadata=0x3 actions=load:0->NXM_NX_REG6[],load:0x1->NXM_NX_REG15[],resubmit(,41),load:0x8004->NXM_NX_REG15[],resubmit(,40)
 cookie=0x148f8e38, duration=297.883s, table=39, n_packets=2, n_bytes=84, priority=100,reg15=0x8004,metadata=0x1 actions=load:0->NXM_NX_REG6[],load:0x2->NXM_NX_REG15[],resubmit(,41),load:0x8004->NXM_NX_REG15[],resubmit(,40)
 cookie=0x3418c045, duration=297.883s, table=39, n_packets=2, n_bytes=84, priority=100,reg15=0x8000,metadata=0x3 actions=load:0->NXM_NX_REG6[],load:0x1->NXM_NX_REG15[],resubmit(,41),load:0x5->NXM_NX_REG15[],resubmit(,41),load:0x8000->NXM_NX_REG15[],resubmit(,40)
 cookie=0x8712f73f, duration=297.883s, table=39, n_packets=26, n_bytes=4948, priority=100,reg15=0x8000,metadata=0x1 actions=load:0->NXM_NX_REG6[],load:0x2->NXM_NX_REG15[],resubmit(,41),load:0x4->NXM_NX_REG15[],resubmit(,41),load:0x8000->NXM_NX_REG15[],resubmit(,40)
 cookie=0x0, duration=2294.001s, table=39, n_packets=151, n_bytes=13738, priority=0 actions=resubmit(,40)
 cookie=0xa3a7b338, duration=297.910s, table=40, n_packets=2, n_bytes=84, priority=100,reg15=0x4,metadata=0x1 actions=load:0x6->NXM_NX_REG11[],load:0x7->NXM_NX_REG12[],resubmit(,41)
 cookie=0xa481ae91, duration=297.910s, table=40, n_packets=15, n_bytes=1470, priority=100,reg15=0x5,metadata=0x3 actions=load:0x5->NXM_NX_REG11[],load:0x4->NXM_NX_REG12[],resubmit(,41)
 cookie=0x839bff69, duration=297.910s, table=40, n_packets=3, n_bytes=294, priority=100,reg15=0x1,metadata=0x4 actions=load:0x3->NXM_NX_REG11[],load:0x1->NXM_NX_REG12[],resubmit(,41)
 cookie=0xc697c99c, duration=297.910s, table=40, n_packets=59, n_bytes=7317, priority=100,reg15=0x2,metadata=0x3 actions=load:0x2->NXM_NX_REG13[0..15],load:0x5->NXM_NX_REG11[],load:0x4->NXM_NX_REG12[],resubmit(,41)
 cookie=0x1b707d7, duration=297.910s, table=40, n_packets=14, n_bytes=1260, priority=100,reg15=0x2,metadata=0x4 actions=load:0x3->NXM_NX_REG11[],load:0x1->NXM_NX_REG12[],resubmit(,41)
 cookie=0x7265e3e4, duration=297.883s, table=40, n_packets=0, n_bytes=0, priority=100,reg15=0x8004,metadata=0x3 actions=load:0->NXM_NX_REG6[],load:0x2->NXM_NX_REG13[0..15],load:0x2->NXM_NX_REG15[],resubmit(,41),load:0x8004->NXM_NX_REG15[]
 cookie=0x3418c045, duration=297.883s, table=40, n_packets=13, n_bytes=966, priority=100,reg15=0x8000,metadata=0x3 actions=load:0->NXM_NX_REG6[],load:0x2->NXM_NX_REG13[0..15],load:0x2->NXM_NX_REG15[],resubmit(,41),load:0x8000->NXM_NX_REG15[]
 cookie=0x4d7eedea, duration=297.883s, table=40, n_packets=12, n_bytes=1176, priority=100,reg15=0x8001,metadata=0x1 actions=load:0->NXM_NX_REG6[],load:0x8->NXM_NX_REG13[0..15],load:0x1->NXM_NX_REG15[],resubmit(,41),load:0x8001->NXM_NX_REG15[]
 cookie=0x97bbc64c, duration=297.883s, table=40, n_packets=0, n_bytes=0, priority=100,reg15=0x8002,metadata=0x1 actions=load:0->NXM_NX_REG6[],load:0x8->NXM_NX_REG13[0..15],load:0x1->NXM_NX_REG15[],resubmit(,41),load:0x8002->NXM_NX_REG15[]
 cookie=0x148f8e38, duration=297.883s, table=40, n_packets=2, n_bytes=84, priority=100,reg15=0x8004,metadata=0x1 actions=load:0->NXM_NX_REG6[],load:0x8->NXM_NX_REG13[0..15],load:0x1->NXM_NX_REG15[],resubmit(,41),load:0x8004->NXM_NX_REG15[]
 cookie=0x8712f73f, duration=297.883s, table=40, n_packets=26, n_bytes=4948, priority=100,reg15=0x8000,metadata=0x1 actions=load:0->NXM_NX_REG6[],load:0x8->NXM_NX_REG13[0..15],load:0x1->NXM_NX_REG15[],resubmit(,41),load:0x8000->NXM_NX_REG15[]
 cookie=0xef38ad1, duration=297.880s, table=40, n_packets=0, n_bytes=0, priority=100,reg15=0x3,metadata=0x1 actions=load:0x1->NXM_NX_REG15[],resubmit(,40)
 cookie=0x3c1e627e, duration=297.880s, table=40, n_packets=0, n_bytes=0, priority=100,reg15=0x1,metadata=0x1 actions=load:0x8->NXM_NX_REG13[0..15],load:0x6->NXM_NX_REG11[],load:0x7->NXM_NX_REG12[],resubmit(,41)
 cookie=0x856a3a05, duration=297.880s, table=40, n_packets=0, n_bytes=0, priority=100,reg15=0x2,metadata=0x1 actions=load:0x1->NXM_NX_REG15[],resubmit(,40)
 cookie=0x9b5b3836, duration=290.822s, table=40, n_packets=97, n_bytes=8346, priority=100,reg15=0x1,metadata=0x3 actions=load:0x9->NXM_NX_REG13[0..15],load:0x5->NXM_NX_REG11[],load:0x4->NXM_NX_REG12[],resubmit(,41)
 cookie=0x0, duration=2294.001s, table=40, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x3c1e627e, duration=297.880s, table=41, n_packets=0, n_bytes=0, priority=160,reg10=0x400/0x400,reg15=0x1,metadata=0x1 actions=drop
 cookie=0x3c1e627e, duration=297.880s, table=41, n_packets=0, n_bytes=0, priority=160,reg10=0x10/0x10,reg15=0x1,metadata=0x1 actions=drop
 cookie=0x1b707d7, duration=297.910s, table=41, n_packets=0, n_bytes=0, priority=100,reg10=0/0x1,reg14=0x2,reg15=0x2,metadata=0x4 actions=drop
 cookie=0xa481ae91, duration=297.910s, table=41, n_packets=0, n_bytes=0, priority=100,reg10=0/0x1,reg14=0x5,reg15=0x5,metadata=0x3 actions=drop
 cookie=0x839bff69, duration=297.910s, table=41, n_packets=0, n_bytes=0, priority=100,reg10=0/0x1,reg14=0x1,reg15=0x1,metadata=0x4 actions=drop
 cookie=0xa3a7b338, duration=297.910s, table=41, n_packets=0, n_bytes=0, priority=100,reg10=0/0x1,reg14=0x4,reg15=0x4,metadata=0x1 actions=drop
 cookie=0xc697c99c, duration=297.910s, table=41, n_packets=2, n_bytes=84, priority=100,reg10=0/0x1,reg14=0x2,reg15=0x2,metadata=0x3 actions=drop
 cookie=0x3c1e627e, duration=297.880s, table=41, n_packets=26, n_bytes=4948, priority=100,reg10=0/0x1,reg14=0x1,reg15=0x1,metadata=0x1 actions=drop
 cookie=0x9b5b3836, duration=290.822s, table=41, n_packets=0, n_bytes=0, priority=100,reg10=0/0x1,reg14=0x1,reg15=0x1,metadata=0x3 actions=drop
 cookie=0x0, duration=2294.001s, table=41, n_packets=273, n_bytes=31061, priority=0 actions=load:0->NXM_NX_REG0[],load:0->NXM_NX_REG1[],load:0->NXM_NX_REG2[],load:0->NXM_NX_REG3[],load:0->NXM_NX_REG4[],load:0->NXM_NX_REG5[],load:0->NXM_NX_REG6[],load:0->NXM_NX_REG7[],load:0->NXM_NX_REG8[],load:0->NXM_NX_REG9[],resubmit(,42)
 cookie=0x2c598801, duration=298.021s, table=42, n_packets=2, n_bytes=172, priority=110,icmp6,metadata=0x1,nw_ttl=255,icmp_type=135,icmp_code=0 actions=resubmit(,43)
 cookie=0x2c598801, duration=298.021s, table=42, n_packets=0, n_bytes=0, priority=110,icmp6,metadata=0x1,nw_ttl=255,icmp_type=136,icmp_code=0 actions=resubmit(,43)
 cookie=0x2c598801, duration=298.021s, table=42, n_packets=0, n_bytes=0, priority=110,icmp6,metadata=0x1,nw_ttl=255,icmp_type=134,icmp_code=0 actions=resubmit(,43)
 cookie=0x2c598801, duration=298.006s, table=42, n_packets=8, n_bytes=496, priority=110,icmp6,metadata=0x1,nw_ttl=255,icmp_type=133,icmp_code=0 actions=resubmit(,43)
 cookie=0x2c598801, duration=297.934s, table=42, n_packets=2, n_bytes=172, priority=110,icmp6,metadata=0x3,nw_ttl=255,icmp_type=135,icmp_code=0 actions=resubmit(,43)
 cookie=0x2c598801, duration=297.934s, table=42, n_packets=0, n_bytes=0, priority=110,icmp6,metadata=0x3,nw_ttl=255,icmp_type=136,icmp_code=0 actions=resubmit(,43)
 cookie=0x2c598801, duration=297.934s, table=42, n_packets=0, n_bytes=0, priority=110,icmp6,metadata=0x3,nw_ttl=255,icmp_type=134,icmp_code=0 actions=resubmit(,43)
 cookie=0x2c598801, duration=297.934s, table=42, n_packets=7, n_bytes=490, priority=110,icmp6,metadata=0x3,nw_ttl=255,icmp_type=133,icmp_code=0 actions=resubmit(,43)
 cookie=0x2c598801, duration=298.021s, table=42, n_packets=0, n_bytes=0, priority=110,icmp6,metadata=0x1,ipv6_src=fe80::/10,icmp_type=130 actions=resubmit(,43)
 cookie=0x2c598801, duration=298.021s, table=42, n_packets=0, n_bytes=0, priority=110,icmp6,metadata=0x1,ipv6_src=fe80::/10,icmp_type=131 actions=resubmit(,43)
 cookie=0x2c598801, duration=298.021s, table=42, n_packets=0, n_bytes=0, priority=110,icmp6,metadata=0x1,ipv6_src=fe80::/10,icmp_type=132 actions=resubmit(,43)
 cookie=0x2c598801, duration=297.934s, table=42, n_packets=0, n_bytes=0, priority=110,icmp6,metadata=0x3,ipv6_src=fe80::/10,icmp_type=130 actions=resubmit(,43)
 cookie=0x2c598801, duration=297.934s, table=42, n_packets=0, n_bytes=0, priority=110,icmp6,metadata=0x3,ipv6_src=fe80::/10,icmp_type=131 actions=resubmit(,43)
 cookie=0x2c598801, duration=297.934s, table=42, n_packets=0, n_bytes=0, priority=110,icmp6,metadata=0x3,ipv6_src=fe80::/10,icmp_type=132 actions=resubmit(,43)
 cookie=0x2c598801, duration=298.021s, table=42, n_packets=0, n_bytes=0, priority=110,udp6,metadata=0x1,tp_src=546,tp_dst=547 actions=resubmit(,43)
 cookie=0x2c598801, duration=298.021s, table=42, n_packets=0, n_bytes=0, priority=110,udp,metadata=0x1,tp_src=546,tp_dst=547 actions=resubmit(,43)
 cookie=0x2c598801, duration=297.934s, table=42, n_packets=0, n_bytes=0, priority=110,udp6,metadata=0x3,tp_src=546,tp_dst=547 actions=resubmit(,43)
 cookie=0x2c598801, duration=297.934s, table=42, n_packets=0, n_bytes=0, priority=110,udp,metadata=0x3,tp_src=546,tp_dst=547 actions=resubmit(,43)
 cookie=0x2c598801, duration=298.021s, table=42, n_packets=8, n_bytes=720, priority=110,icmp6,metadata=0x1,ipv6_dst=ff02::16,icmp_type=143 actions=resubmit(,43)
 cookie=0x2c598801, duration=297.934s, table=42, n_packets=2, n_bytes=220, priority=110,icmp6,metadata=0x3,ipv6_dst=ff02::16,icmp_type=143 actions=resubmit(,43)
 cookie=0x340f8bcf, duration=298.006s, table=42, n_packets=0, n_bytes=0, priority=110,metadata=0x1,dl_src=b6:9c:90:73:34:da actions=resubmit(,43)
 cookie=0x340f8bcf, duration=297.934s, table=42, n_packets=0, n_bytes=0, priority=110,metadata=0x3,dl_src=b6:9c:90:73:34:da actions=resubmit(,43)
 cookie=0x9946f595, duration=298.005s, table=42, n_packets=38, n_bytes=8676, priority=110,metadata=0x1,dl_dst=01:00:00:00:00:00/01:00:00:00:00:00 actions=resubmit(,43)
 cookie=0x9946f595, duration=297.912s, table=42, n_packets=4, n_bytes=168, priority=110,metadata=0x3,dl_dst=01:00:00:00:00:00/01:00:00:00:00:00 actions=resubmit(,43)
 cookie=0x8d9212ac, duration=297.884s, table=42, n_packets=0, n_bytes=0, priority=110,ip,reg15=0x4,metadata=0x1 actions=resubmit(,43)
 cookie=0x8d9212ac, duration=297.884s, table=42, n_packets=0, n_bytes=0, priority=110,ipv6,reg15=0x4,metadata=0x1 actions=resubmit(,43)
 cookie=0x8265c27a, duration=297.884s, table=42, n_packets=0, n_bytes=0, priority=110,ipv6,reg15=0x1,metadata=0x1 actions=resubmit(,43)
 cookie=0x8265c27a, duration=297.884s, table=42, n_packets=12, n_bytes=1176, priority=110,ip,reg15=0x1,metadata=0x1 actions=resubmit(,43)
 cookie=0x520b5e2, duration=297.884s, table=42, n_packets=0, n_bytes=0, priority=110,ipv6,reg15=0x5,metadata=0x3 actions=resubmit(,43)
 cookie=0x520b5e2, duration=297.883s, table=42, n_packets=15, n_bytes=1470, priority=110,ip,reg15=0x5,metadata=0x3 actions=resubmit(,43)
 cookie=0x2201098a, duration=298.021s, table=42, n_packets=0, n_bytes=0, priority=100,ipv6,metadata=0x1 actions=load:0x1->NXM_NX_XXREG0[96],resubmit(,43)
 cookie=0x2201098a, duration=298.021s, table=42, n_packets=0, n_bytes=0, priority=100,ip,metadata=0x1 actions=load:0x1->NXM_NX_XXREG0[96],resubmit(,43)
 cookie=0x2201098a, duration=297.934s, table=42, n_packets=0, n_bytes=0, priority=100,ipv6,metadata=0x3 actions=load:0x1->NXM_NX_XXREG0[96],resubmit(,43)
 cookie=0x2201098a, duration=297.934s, table=42, n_packets=151, n_bytes=15453, priority=100,ip,metadata=0x3 actions=load:0x1->NXM_NX_XXREG0[96],resubmit(,43)
 cookie=0x4d6ec236, duration=298.006s, table=42, n_packets=2, n_bytes=84, priority=0,metadata=0x1 actions=resubmit(,43)
 cookie=0x4d6ec236, duration=297.934s, table=42, n_packets=5, n_bytes=210, priority=0,metadata=0x3 actions=resubmit(,43)
 cookie=0x649c5cac, duration=297.884s, table=42, n_packets=17, n_bytes=1554, priority=0,metadata=0x4 actions=load:0->OXM_OF_PKT_REG4[4],resubmit(,43)
 cookie=0x22e7aaf5, duration=298.021s, table=43, n_packets=2, n_bytes=172, priority=110,icmp6,metadata=0x1,nw_ttl=255,icmp_type=135,icmp_code=0 actions=resubmit(,44)
 cookie=0x22e7aaf5, duration=298.021s, table=43, n_packets=0, n_bytes=0, priority=110,icmp6,metadata=0x1,nw_ttl=255,icmp_type=136,icmp_code=0 actions=resubmit(,44)
 cookie=0x22e7aaf5, duration=298.021s, table=43, n_packets=0, n_bytes=0, priority=110,icmp6,metadata=0x1,nw_ttl=255,icmp_type=134,icmp_code=0 actions=resubmit(,44)
 cookie=0x22e7aaf5, duration=298.021s, table=43, n_packets=8, n_bytes=496, priority=110,icmp6,metadata=0x1,nw_ttl=255,icmp_type=133,icmp_code=0 actions=resubmit(,44)
 cookie=0x22e7aaf5, duration=297.934s, table=43, n_packets=2, n_bytes=172, priority=110,icmp6,metadata=0x3,nw_ttl=255,icmp_type=135,icmp_code=0 actions=resubmit(,44)
 cookie=0x22e7aaf5, duration=297.934s, table=43, n_packets=0, n_bytes=0, priority=110,icmp6,metadata=0x3,nw_ttl=255,icmp_type=136,icmp_code=0 actions=resubmit(,44)
 cookie=0x22e7aaf5, duration=297.934s, table=43, n_packets=0, n_bytes=0, priority=110,icmp6,metadata=0x3,nw_ttl=255,icmp_type=134,icmp_code=0 actions=resubmit(,44)
 cookie=0x22e7aaf5, duration=297.934s, table=43, n_packets=7, n_bytes=490, priority=110,icmp6,metadata=0x3,nw_ttl=255,icmp_type=133,icmp_code=0 actions=resubmit(,44)
 cookie=0x22e7aaf5, duration=298.021s, table=43, n_packets=0, n_bytes=0, priority=110,icmp6,metadata=0x1,ipv6_src=fe80::/10,icmp_type=130 actions=resubmit(,44)
 cookie=0x22e7aaf5, duration=298.021s, table=43, n_packets=0, n_bytes=0, priority=110,icmp6,metadata=0x1,ipv6_src=fe80::/10,icmp_type=131 actions=resubmit(,44)
 cookie=0x22e7aaf5, duration=298.021s, table=43, n_packets=0, n_bytes=0, priority=110,icmp6,metadata=0x1,ipv6_src=fe80::/10,icmp_type=132 actions=resubmit(,44)
 cookie=0x22e7aaf5, duration=297.934s, table=43, n_packets=0, n_bytes=0, priority=110,icmp6,metadata=0x3,ipv6_src=fe80::/10,icmp_type=130 actions=resubmit(,44)
 cookie=0x22e7aaf5, duration=297.934s, table=43, n_packets=0, n_bytes=0, priority=110,icmp6,metadata=0x3,ipv6_src=fe80::/10,icmp_type=131 actions=resubmit(,44)
 cookie=0x22e7aaf5, duration=297.934s, table=43, n_packets=0, n_bytes=0, priority=110,icmp6,metadata=0x3,ipv6_src=fe80::/10,icmp_type=132 actions=resubmit(,44)
 cookie=0x22e7aaf5, duration=298.021s, table=43, n_packets=8, n_bytes=720, priority=110,icmp6,metadata=0x1,ipv6_dst=ff02::16,icmp_type=143 actions=resubmit(,44)
 cookie=0x22e7aaf5, duration=297.934s, table=43, n_packets=2, n_bytes=220, priority=110,icmp6,metadata=0x3,ipv6_dst=ff02::16,icmp_type=143 actions=resubmit(,44)
 cookie=0x69538864, duration=298.006s, table=43, n_packets=38, n_bytes=8676, priority=110,metadata=0x1,dl_dst=01:00:00:00:00:00/01:00:00:00:00:00 actions=resubmit(,44)
 cookie=0x69538864, duration=297.915s, table=43, n_packets=4, n_bytes=168, priority=110,metadata=0x3,dl_dst=01:00:00:00:00:00/01:00:00:00:00:00 actions=resubmit(,44)
 cookie=0x6a27c333, duration=298.006s, table=43, n_packets=0, n_bytes=0, priority=110,reg0=0x10000/0x10000,metadata=0x1 actions=resubmit(,44)
 cookie=0x6a27c333, duration=297.915s, table=43, n_packets=0, n_bytes=0, priority=110,reg0=0x10000/0x10000,metadata=0x3 actions=resubmit(,44)
 cookie=0xc1c30dd5, duration=297.968s, table=43, n_packets=0, n_bytes=0, priority=110,metadata=0x1,dl_src=b6:9c:90:73:34:da actions=resubmit(,44)
 cookie=0xc1c30dd5, duration=297.912s, table=43, n_packets=0, n_bytes=0, priority=110,metadata=0x3,dl_src=b6:9c:90:73:34:da actions=resubmit(,44)
 cookie=0xbc58637b, duration=297.885s, table=43, n_packets=12, n_bytes=1176, priority=110,ip,reg15=0x1,metadata=0x1 actions=resubmit(,44)
 cookie=0x8af3d1b5, duration=297.885s, table=43, n_packets=0, n_bytes=0, priority=110,ipv6,reg15=0x5,metadata=0x3 actions=resubmit(,44)
 cookie=0xe7231d7a, duration=297.885s, table=43, n_packets=0, n_bytes=0, priority=110,ipv6,reg15=0x4,metadata=0x1 actions=resubmit(,44)
 cookie=0x8af3d1b5, duration=297.884s, table=43, n_packets=15, n_bytes=1470, priority=110,ip,reg15=0x5,metadata=0x3 actions=resubmit(,44)
 cookie=0xe7231d7a, duration=297.884s, table=43, n_packets=0, n_bytes=0, priority=110,ip,reg15=0x4,metadata=0x1 actions=resubmit(,44)
 cookie=0xbc58637b, duration=297.884s, table=43, n_packets=0, n_bytes=0, priority=110,ipv6,reg15=0x1,metadata=0x1 actions=resubmit(,44)
 cookie=0xca2e2bcc, duration=297.968s, table=43, n_packets=2, n_bytes=84, priority=0,metadata=0x1 actions=resubmit(,44)
 cookie=0xca2e2bcc, duration=297.912s, table=43, n_packets=156, n_bytes=15663, priority=0,metadata=0x3 actions=resubmit(,44)
 cookie=0x2fd6ed21, duration=297.885s, table=43, n_packets=17, n_bytes=1554, priority=0,metadata=0x4 actions=resubmit(,44)
 cookie=0x401b44bd, duration=298.006s, table=44, n_packets=0, n_bytes=0, priority=110,ipv6,reg0=0x4/0x4,metadata=0x1 actions=ct(table=45,zone=NXM_NX_REG13[0..15],nat)
 cookie=0x401b44bd, duration=298.006s, table=44, n_packets=0, n_bytes=0, priority=110,ip,reg0=0x4/0x4,metadata=0x1 actions=ct(table=45,zone=NXM_NX_REG13[0..15],nat)
 cookie=0x401b44bd, duration=297.934s, table=44, n_packets=0, n_bytes=0, priority=110,ipv6,reg0=0x4/0x4,metadata=0x3 actions=ct(table=45,zone=NXM_NX_REG13[0..15],nat)
 cookie=0x401b44bd, duration=297.934s, table=44, n_packets=0, n_bytes=0, priority=110,ip,reg0=0x4/0x4,metadata=0x3 actions=ct(table=45,zone=NXM_NX_REG13[0..15],nat)
 cookie=0xdb8c395c, duration=297.968s, table=44, n_packets=0, n_bytes=0, priority=100,ipv6,reg0=0x1/0x1,metadata=0x1 actions=ct(table=45,zone=NXM_NX_REG13[0..15])
 cookie=0xdb8c395c, duration=297.968s, table=44, n_packets=0, n_bytes=0, priority=100,ip,reg0=0x1/0x1,metadata=0x1 actions=ct(table=45,zone=NXM_NX_REG13[0..15])
 cookie=0xdb8c395c, duration=297.910s, table=44, n_packets=0, n_bytes=0, priority=100,ipv6,reg0=0x1/0x1,metadata=0x3 actions=ct(table=45,zone=NXM_NX_REG13[0..15])
 cookie=0xdb8c395c, duration=297.910s, table=44, n_packets=151, n_bytes=15453, priority=100,ip,reg0=0x1/0x1,metadata=0x3 actions=ct(table=45,zone=NXM_NX_REG13[0..15])
 cookie=0x5ae2385c, duration=298.006s, table=44, n_packets=70, n_bytes=11324, priority=0,metadata=0x1 actions=resubmit(,45)
 cookie=0x5ae2385c, duration=297.915s, table=44, n_packets=35, n_bytes=2730, priority=0,metadata=0x3 actions=resubmit(,45)
 cookie=0x66e04f91, duration=297.883s, table=44, n_packets=17, n_bytes=1554, priority=0,metadata=0x4 actions=resubmit(,45)
 cookie=0x6f93ac11, duration=297.884s, table=45, n_packets=0, n_bytes=0, priority=120,icmp6,metadata=0x4,nw_ttl=255,icmp_type=135,icmp_code=0 actions=resubmit(,46)
 cookie=0x5d97bb3c, duration=298.006s, table=45, n_packets=0, n_bytes=0, priority=7,ct_state=+new-est+trk,metadata=0x1 actions=load:0x1->NXM_NX_XXREG0[103],load:0x1->NXM_NX_XXREG0[105],resubmit(,46)
 cookie=0x5d97bb3c, duration=297.915s, table=45, n_packets=21, n_bytes=2166, priority=7,ct_state=+new-est+trk,metadata=0x3 actions=load:0x1->NXM_NX_XXREG0[103],load:0x1->NXM_NX_XXREG0[105],resubmit(,46)
 cookie=0x77b42c79, duration=298.005s, table=45, n_packets=0, n_bytes=0, priority=4,ct_state=-new+est-rpl+trk,ct_mark=0/0x1,metadata=0x1 actions=load:0x1->NXM_NX_XXREG0[104],load:0x1->NXM_NX_XXREG0[106],resubmit(,46)
 cookie=0xb06d0ee4, duration=298.005s, table=45, n_packets=0, n_bytes=0, priority=6,ct_state=-new+est-rpl+trk,ct_mark=0x1/0x1,metadata=0x1 actions=load:0x1->NXM_NX_XXREG0[103],load:0x1->NXM_NX_XXREG0[105],resubmit(,46)
 cookie=0x77b42c79, duration=297.915s, table=45, n_packets=80, n_bytes=7120, priority=4,ct_state=-new+est-rpl+trk,ct_mark=0/0x1,metadata=0x3 actions=load:0x1->NXM_NX_XXREG0[104],load:0x1->NXM_NX_XXREG0[106],resubmit(,46)
 cookie=0xb06d0ee4, duration=297.912s, table=45, n_packets=0, n_bytes=0, priority=6,ct_state=-new+est-rpl+trk,ct_mark=0x1/0x1,metadata=0x3 actions=load:0x1->NXM_NX_XXREG0[103],load:0x1->NXM_NX_XXREG0[105],resubmit(,46)
 cookie=0x4275ecd8, duration=298.006s, table=45, n_packets=70, n_bytes=11324, priority=5,ct_state=-trk,metadata=0x1 actions=load:0x1->NXM_NX_XXREG0[104],load:0x1->NXM_NX_XXREG0[105],resubmit(,46)
 cookie=0x4275ecd8, duration=297.934s, table=45, n_packets=35, n_bytes=2730, priority=5,ct_state=-trk,metadata=0x3 actions=load:0x1->NXM_NX_XXREG0[104],load:0x1->NXM_NX_XXREG0[105],resubmit(,46)
 cookie=0x2b46bae1, duration=298.021s, table=45, n_packets=0, n_bytes=0, priority=3,ct_state=-est+trk,metadata=0x1 actions=load:0x1->NXM_NX_XXREG0[105],resubmit(,46)
 cookie=0x2b46bae1, duration=297.934s, table=45, n_packets=0, n_bytes=0, priority=3,ct_state=-est+trk,metadata=0x3 actions=load:0x1->NXM_NX_XXREG0[105],resubmit(,46)
 cookie=0xbbf6f3fd, duration=297.968s, table=45, n_packets=0, n_bytes=0, priority=2,ct_state=+est+trk,ct_mark=0x1/0x1,metadata=0x1 actions=load:0x1->NXM_NX_XXREG0[105],resubmit(,46)
 cookie=0xf6de0452, duration=297.967s, table=45, n_packets=0, n_bytes=0, priority=1,ct_state=+est+trk,ct_mark=0/0x1,metadata=0x1 actions=load:0x1->NXM_NX_XXREG0[106],resubmit(,46)
 cookie=0xbbf6f3fd, duration=297.912s, table=45, n_packets=0, n_bytes=0, priority=2,ct_state=+est+trk,ct_mark=0x1/0x1,metadata=0x3 actions=load:0x1->NXM_NX_XXREG0[105],resubmit(,46)
 cookie=0xf6de0452, duration=297.910s, table=45, n_packets=50, n_bytes=6167, priority=1,ct_state=+est+trk,ct_mark=0/0x1,metadata=0x3 actions=load:0x1->NXM_NX_XXREG0[106],resubmit(,46)
 cookie=0xe629d07, duration=298.021s, table=45, n_packets=0, n_bytes=0, priority=0,metadata=0x1 actions=resubmit(,46)
 cookie=0xe629d07, duration=297.934s, table=45, n_packets=0, n_bytes=0, priority=0,metadata=0x3 actions=resubmit(,46)
 cookie=0x7d77e883, duration=297.884s, table=45, n_packets=17, n_bytes=1554, priority=0,metadata=0x4 actions=resubmit(,46)
 cookie=0x793e8d13, duration=298.005s, table=46, n_packets=0, n_bytes=0, priority=65532,ct_state=-new+est-rel+rpl-inv+trk,ct_mark=0/0x1,metadata=0x1 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,47)
 cookie=0x793e8d13, duration=297.915s, table=46, n_packets=50, n_bytes=6167, priority=65532,ct_state=-new+est-rel+rpl-inv+trk,ct_mark=0/0x1,metadata=0x3 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,47)
 cookie=0x9b3cd3bb, duration=298.005s, table=46, n_packets=0, n_bytes=0, priority=65532,ct_state=-new-est+rel-inv+trk,ct_mark=0/0x1,ipv6,metadata=0x1 actions=load:0x1->OXM_OF_PKT_REG4[48],ct(commit,table=47,zone=NXM_NX_REG13[0..15],nat)
 cookie=0x9b3cd3bb, duration=298.005s, table=46, n_packets=0, n_bytes=0, priority=65532,ct_state=-new-est+rel-inv+trk,ct_mark=0/0x1,ip,metadata=0x1 actions=load:0x1->OXM_OF_PKT_REG4[48],ct(commit,table=47,zone=NXM_NX_REG13[0..15],nat)
 cookie=0x9b3cd3bb, duration=297.912s, table=46, n_packets=0, n_bytes=0, priority=65532,ct_state=-new-est+rel-inv+trk,ct_mark=0/0x1,ipv6,metadata=0x3 actions=load:0x1->OXM_OF_PKT_REG4[48],ct(commit,table=47,zone=NXM_NX_REG13[0..15],nat)
 cookie=0x9b3cd3bb, duration=297.912s, table=46, n_packets=0, n_bytes=0, priority=65532,ct_state=-new-est+rel-inv+trk,ct_mark=0/0x1,ip,metadata=0x3 actions=load:0x1->OXM_OF_PKT_REG4[48],ct(commit,table=47,zone=NXM_NX_REG13[0..15],nat)
 cookie=0xcd683e89, duration=297.968s, table=46, n_packets=0, n_bytes=0, priority=65532,ct_state=+inv+trk,metadata=0x1 actions=load:0x1->OXM_OF_PKT_REG4[49],resubmit(,47)
 cookie=0xcd683e89, duration=297.910s, table=46, n_packets=0, n_bytes=0, priority=65532,ct_state=+inv+trk,metadata=0x3 actions=load:0x1->OXM_OF_PKT_REG4[49],resubmit(,47)
 cookie=0xcd683e89, duration=297.968s, table=46, n_packets=0, n_bytes=0, priority=65532,ct_state=+est+rpl+trk,ct_mark=0x1/0x1,metadata=0x1 actions=load:0x1->OXM_OF_PKT_REG4[49],resubmit(,47)
 cookie=0xcd683e89, duration=297.910s, table=46, n_packets=0, n_bytes=0, priority=65532,ct_state=+est+rpl+trk,ct_mark=0x1/0x1,metadata=0x3 actions=load:0x1->OXM_OF_PKT_REG4[49],resubmit(,47)
 cookie=0xf8093db0, duration=297.967s, table=46, n_packets=2, n_bytes=172, priority=65532,icmp6,metadata=0x1,nw_ttl=255,icmp_type=135,icmp_code=0 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,47)
 cookie=0xf8093db0, duration=297.967s, table=46, n_packets=0, n_bytes=0, priority=65532,icmp6,metadata=0x1,nw_ttl=255,icmp_type=136,icmp_code=0 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,47)
 cookie=0xf8093db0, duration=297.967s, table=46, n_packets=0, n_bytes=0, priority=65532,icmp6,metadata=0x1,nw_ttl=255,icmp_type=134,icmp_code=0 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,47)
 cookie=0xf8093db0, duration=297.967s, table=46, n_packets=8, n_bytes=496, priority=65532,icmp6,metadata=0x1,nw_ttl=255,icmp_type=133,icmp_code=0 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,47)
 cookie=0xf8093db0, duration=297.910s, table=46, n_packets=2, n_bytes=172, priority=65532,icmp6,metadata=0x3,nw_ttl=255,icmp_type=135,icmp_code=0 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,47)
 cookie=0xf8093db0, duration=297.910s, table=46, n_packets=0, n_bytes=0, priority=65532,icmp6,metadata=0x3,nw_ttl=255,icmp_type=136,icmp_code=0 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,47)
 cookie=0xf8093db0, duration=297.910s, table=46, n_packets=0, n_bytes=0, priority=65532,icmp6,metadata=0x3,nw_ttl=255,icmp_type=134,icmp_code=0 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,47)
 cookie=0xf8093db0, duration=297.910s, table=46, n_packets=7, n_bytes=490, priority=65532,icmp6,metadata=0x3,nw_ttl=255,icmp_type=133,icmp_code=0 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,47)
 cookie=0xf8093db0, duration=297.967s, table=46, n_packets=0, n_bytes=0, priority=65532,icmp6,metadata=0x1,ipv6_src=fe80::/10,icmp_type=130 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,47)
 cookie=0xf8093db0, duration=297.967s, table=46, n_packets=0, n_bytes=0, priority=65532,icmp6,metadata=0x1,ipv6_src=fe80::/10,icmp_type=131 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,47)
 cookie=0xf8093db0, duration=297.967s, table=46, n_packets=0, n_bytes=0, priority=65532,icmp6,metadata=0x1,ipv6_src=fe80::/10,icmp_type=132 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,47)
 cookie=0xf8093db0, duration=297.910s, table=46, n_packets=0, n_bytes=0, priority=65532,icmp6,metadata=0x3,ipv6_src=fe80::/10,icmp_type=130 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,47)
 cookie=0xf8093db0, duration=297.910s, table=46, n_packets=0, n_bytes=0, priority=65532,icmp6,metadata=0x3,ipv6_src=fe80::/10,icmp_type=131 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,47)
 cookie=0xf8093db0, duration=297.910s, table=46, n_packets=0, n_bytes=0, priority=65532,icmp6,metadata=0x3,ipv6_src=fe80::/10,icmp_type=132 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,47)
 cookie=0xf8093db0, duration=297.967s, table=46, n_packets=8, n_bytes=720, priority=65532,icmp6,metadata=0x1,ipv6_dst=ff02::16,icmp_type=143 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,47)
 cookie=0xf8093db0, duration=297.910s, table=46, n_packets=2, n_bytes=220, priority=65532,icmp6,metadata=0x3,ipv6_dst=ff02::16,icmp_type=143 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,47)
 cookie=0xbd3202b9, duration=297.968s, table=46, n_packets=0, n_bytes=0, priority=34000,metadata=0x1,dl_src=b6:9c:90:73:34:da actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,47)
 cookie=0xbd3202b9, duration=297.912s, table=46, n_packets=0, n_bytes=0, priority=34000,metadata=0x3,dl_src=b6:9c:90:73:34:da actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,47)
 cookie=0x2e9ea961, duration=297.902s, table=46, n_packets=2, n_bytes=688, priority=34000,udp,reg15=0x2,metadata=0x3,dl_src=fa:16:3e:73:d1:b0,nw_src=192.168.101.254,tp_src=67,tp_dst=68 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,47)
 cookie=0xe20f1c10, duration=297.910s, table=46, n_packets=3, n_bytes=294, priority=2002,icmp,reg0=0x80/0x80,reg15=0x2,metadata=0x3 actions=load:0x1->OXM_OF_PKT_REG4[48],load:0x1->NXM_NX_XXREG0[97],resubmit(,47)
 cookie=0x8037f919, duration=297.910s, table=46, n_packets=0, n_bytes=0, priority=2002,tcp,reg0=0x80/0x80,reg15=0x2,metadata=0x3,tp_dst=22 actions=load:0x1->OXM_OF_PKT_REG4[48],load:0x1->NXM_NX_XXREG0[97],resubmit(,47)
 cookie=0x75db984e, duration=297.910s, table=46, n_packets=0, n_bytes=0, priority=2002,tcp,reg0=0x100/0x100,reg15=0x2,metadata=0x3,tp_dst=22 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,47)
 cookie=0xf3aa0b16, duration=297.910s, table=46, n_packets=0, n_bytes=0, priority=2002,icmp,reg0=0x100/0x100,reg15=0x2,metadata=0x3 actions=load:0x1->OXM_OF_PKT_REG4[48],resubmit(,47)
 cookie=0xa4beae28, duration=297.910s, table=46, n_packets=0, n_bytes=0, priority=2001,ip,reg0=0x200/0x200,reg15=0x2,metadata=0x3 actions=load:0x1->OXM_OF_PKT_REG4[49],resubmit(,47)
 cookie=0xa4beae28, duration=297.910s, table=46, n_packets=0, n_bytes=0, priority=2001,ipv6,reg0=0x200/0x200,reg15=0x2,metadata=0x3 actions=load:0x1->OXM_OF_PKT_REG4[49],resubmit(,47)
 cookie=0x2997c07b, duration=297.910s, table=46, n_packets=0, n_bytes=0, priority=2001,ipv6,reg0=0x400/0x400,reg15=0x2,metadata=0x3 actions=load:0x1->OXM_OF_PKT_REG4[49],resubmit(,47)
 cookie=0x2997c07b, duration=297.910s, table=46, n_packets=0, n_bytes=0, priority=2001,ip,reg0=0x400/0x400,reg15=0x2,metadata=0x3 actions=load:0x1->OXM_OF_PKT_REG4[49],resubmit(,47)
 cookie=0x5c31dec7, duration=298.006s, table=46, n_packets=0, n_bytes=0, priority=1,ct_state=+est+trk,ct_mark=0x1/0x1,ip,metadata=0x1 actions=load:0x1->NXM_NX_XXREG0[97],load:0x1->OXM_OF_PKT_REG4[48],resubmit(,47)
 cookie=0x5c31dec7, duration=298.006s, table=46, n_packets=0, n_bytes=0, priority=1,ct_state=+est+trk,ct_mark=0x1/0x1,ipv6,metadata=0x1 actions=load:0x1->NXM_NX_XXREG0[97],load:0x1->OXM_OF_PKT_REG4[48],resubmit(,47)
 cookie=0x5c31dec7, duration=297.915s, table=46, n_packets=0, n_bytes=0, priority=1,ct_state=+est+trk,ct_mark=0x1/0x1,ip,metadata=0x3 actions=load:0x1->NXM_NX_XXREG0[97],load:0x1->OXM_OF_PKT_REG4[48],resubmit(,47)
 cookie=0x5c31dec7, duration=297.915s, table=46, n_packets=0, n_bytes=0, priority=1,ct_state=+est+trk,ct_mark=0x1/0x1,ipv6,metadata=0x3 actions=load:0x1->NXM_NX_XXREG0[97],load:0x1->OXM_OF_PKT_REG4[48],resubmit(,47)
 cookie=0xf60851a2, duration=297.967s, table=46, n_packets=0, n_bytes=0, priority=1,ct_state=-est+trk,ip,metadata=0x1 actions=load:0x1->NXM_NX_XXREG0[97],resubmit(,47)
 cookie=0xf60851a2, duration=297.967s, table=46, n_packets=0, n_bytes=0, priority=1,ct_state=-est+trk,ipv6,metadata=0x1 actions=load:0x1->NXM_NX_XXREG0[97],resubmit(,47)
 cookie=0xf60851a2, duration=297.910s, table=46, n_packets=16, n_bytes=1184, priority=1,ct_state=-est+trk,ip,metadata=0x3 actions=load:0x1->NXM_NX_XXREG0[97],resubmit(,47)
 cookie=0xf60851a2, duration=297.910s, table=46, n_packets=0, n_bytes=0, priority=1,ct_state=-est+trk,ipv6,metadata=0x3 actions=load:0x1->NXM_NX_XXREG0[97],resubmit(,47)
 cookie=0x75f1a714, duration=298.005s, table=46, n_packets=52, n_bytes=9936, priority=0,metadata=0x1 actions=resubmit(,47)
 cookie=0x75f1a714, duration=297.915s, table=46, n_packets=104, n_bytes=8968, priority=0,metadata=0x3 actions=resubmit(,47)
 cookie=0x1e70d73b, duration=297.885s, table=46, n_packets=17, n_bytes=1554, priority=0,metadata=0x4 actions=resubmit(,47)
 cookie=0x93662581, duration=298.005s, table=47, n_packets=0, n_bytes=0, priority=1000,reg8=0x20000/0x20000,metadata=0x1 actions=load:0->OXM_OF_PKT_REG4[48],load:0->OXM_OF_PKT_REG4[49],load:0->OXM_OF_PKT_REG4[50]
 cookie=0x93662581, duration=297.912s, table=47, n_packets=0, n_bytes=0, priority=1000,reg8=0x20000/0x20000,metadata=0x3 actions=load:0->OXM_OF_PKT_REG4[48],load:0->OXM_OF_PKT_REG4[49],load:0->OXM_OF_PKT_REG4[50]
 cookie=0xcc2d80c0, duration=297.968s, table=47, n_packets=18, n_bytes=1388, priority=1000,reg8=0x10000/0x10000,metadata=0x1 actions=load:0->OXM_OF_PKT_REG4[48],load:0->OXM_OF_PKT_REG4[49],load:0->OXM_OF_PKT_REG4[50],resubmit(,48)
 cookie=0xcc2d80c0, duration=297.912s, table=47, n_packets=66, n_bytes=8031, priority=1000,reg8=0x10000/0x10000,metadata=0x3 actions=load:0->OXM_OF_PKT_REG4[48],load:0->OXM_OF_PKT_REG4[49],load:0->OXM_OF_PKT_REG4[50],resubmit(,48)
 cookie=0xe663e59c, duration=297.968s, table=47, n_packets=0, n_bytes=0, priority=1000,reg8=0x40000/0x40000,metadata=0x1 actions=load:0->OXM_OF_PKT_REG4[48],load:0->OXM_OF_PKT_REG4[49],load:0->OXM_OF_PKT_REG4[50],load:0->NXM_NX_XXREG0[96..127],controller(userdata=00.00.00.16.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.01.1c.04.00.20.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.01.1e.04.00.20.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.01.1c.04.00.20.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.01.1e.04.00.20.00.00.00.00.00.00.ff.ff.00.10.00.00.23.20.00.0e.ff.f8.23.00.00.00)
 cookie=0xe663e59c, duration=297.910s, table=47, n_packets=0, n_bytes=0, priority=1000,reg8=0x40000/0x40000,metadata=0x3 actions=load:0->OXM_OF_PKT_REG4[48],load:0->OXM_OF_PKT_REG4[49],load:0->OXM_OF_PKT_REG4[50],load:0->NXM_NX_XXREG0[96..127],controller(userdata=00.00.00.16.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.01.1c.04.00.20.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1b.00.00.00.01.1e.04.00.20.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.01.1c.04.00.20.00.00.00.00.00.00.ff.ff.00.18.00.00.23.20.00.1c.00.00.00.01.1e.04.00.20.00.00.00.00.00.00.ff.ff.00.10.00.00.23.20.00.0e.ff.f8.23.00.00.00)
 cookie=0xb4d7dee5, duration=298.005s, table=47, n_packets=52, n_bytes=9936, priority=0,metadata=0x1 actions=load:0->OXM_OF_PKT_REG4[48],load:0->OXM_OF_PKT_REG4[49],load:0->OXM_OF_PKT_REG4[50],resubmit(,48)
 cookie=0xb4d7dee5, duration=297.912s, table=47, n_packets=120, n_bytes=10152, priority=0,metadata=0x3 actions=load:0->OXM_OF_PKT_REG4[48],load:0->OXM_OF_PKT_REG4[49],load:0->OXM_OF_PKT_REG4[50],resubmit(,48)
 cookie=0x29eaffd, duration=297.885s, table=47, n_packets=17, n_bytes=1554, priority=0,metadata=0x4 actions=resubmit(,48)
 cookie=0x64cab41e, duration=297.885s, table=48, n_packets=14, n_bytes=1260, priority=100,reg15=0x2,metadata=0x4 actions=resubmit(,64)
 cookie=0x1f657040, duration=297.885s, table=48, n_packets=3, n_bytes=294, priority=100,reg15=0x1,metadata=0x4 actions=resubmit(,64)
 cookie=0x32c957cf, duration=298.006s, table=48, n_packets=70, n_bytes=11324, priority=0,metadata=0x1 actions=resubmit(,49)
 cookie=0x32c957cf, duration=297.934s, table=48, n_packets=186, n_bytes=18183, priority=0,metadata=0x3 actions=resubmit(,49)
 cookie=0x6ad99c7a, duration=297.884s, table=48, n_packets=0, n_bytes=0, priority=0,metadata=0x4 actions=drop
 cookie=0x6115534a, duration=298.006s, table=49, n_packets=70, n_bytes=11324, priority=0,metadata=0x1 actions=resubmit(,50)
 cookie=0x6115534a, duration=297.915s, table=49, n_packets=186, n_bytes=18183, priority=0,metadata=0x3 actions=resubmit(,50)
 cookie=0x973651a0, duration=298.005s, table=50, n_packets=0, n_bytes=0, priority=100,ipv6,reg0=0x2002/0x2002,metadata=0x1 actions=ct(commit,zone=NXM_NX_REG13[0..15],nat(src),exec(load:0->NXM_NX_CT_MARK[0],move:NXM_NX_XXREG0[0..31]->NXM_NX_CT_LABEL[96..127])),resubmit(,51)
 cookie=0x973651a0, duration=298.005s, table=50, n_packets=0, n_bytes=0, priority=100,ip,reg0=0x2002/0x2002,metadata=0x1 actions=ct(commit,zone=NXM_NX_REG13[0..15],nat(src),exec(load:0->NXM_NX_CT_MARK[0],move:NXM_NX_XXREG0[0..31]->NXM_NX_CT_LABEL[96..127])),resubmit(,51)
 cookie=0xb65a221c, duration=298.005s, table=50, n_packets=0, n_bytes=0, priority=100,ip,reg0=0x2/0x2002,metadata=0x1 actions=ct(commit,zone=NXM_NX_REG13[0..15],nat(src),exec(load:0->NXM_NX_CT_MARK[0])),resubmit(,51)
 cookie=0xb65a221c, duration=298.005s, table=50, n_packets=0, n_bytes=0, priority=100,ipv6,reg0=0x2/0x2002,metadata=0x1 actions=ct(commit,zone=NXM_NX_REG13[0..15],nat(src),exec(load:0->NXM_NX_CT_MARK[0])),resubmit(,51)
 cookie=0x973651a0, duration=297.912s, table=50, n_packets=0, n_bytes=0, priority=100,ipv6,reg0=0x2002/0x2002,metadata=0x3 actions=ct(commit,zone=NXM_NX_REG13[0..15],nat(src),exec(load:0->NXM_NX_CT_MARK[0],move:NXM_NX_XXREG0[0..31]->NXM_NX_CT_LABEL[96..127])),resubmit(,51)
 cookie=0x973651a0, duration=297.912s, table=50, n_packets=0, n_bytes=0, priority=100,ip,reg0=0x2002/0x2002,metadata=0x3 actions=ct(commit,zone=NXM_NX_REG13[0..15],nat(src),exec(load:0->NXM_NX_CT_MARK[0],move:NXM_NX_XXREG0[0..31]->NXM_NX_CT_LABEL[96..127])),resubmit(,51)
 cookie=0xb65a221c, duration=297.912s, table=50, n_packets=19, n_bytes=1478, priority=100,ip,reg0=0x2/0x2002,metadata=0x3 actions=ct(commit,zone=NXM_NX_REG13[0..15],nat(src),exec(load:0->NXM_NX_CT_MARK[0])),resubmit(,51)
 cookie=0xb65a221c, duration=297.912s, table=50, n_packets=0, n_bytes=0, priority=100,ipv6,reg0=0x2/0x2002,metadata=0x3 actions=ct(commit,zone=NXM_NX_REG13[0..15],nat(src),exec(load:0->NXM_NX_CT_MARK[0])),resubmit(,51)
 cookie=0x301b84d9, duration=298.006s, table=50, n_packets=70, n_bytes=11324, priority=0,metadata=0x1 actions=resubmit(,51)
 cookie=0x301b84d9, duration=297.934s, table=50, n_packets=167, n_bytes=16705, priority=0,metadata=0x3 actions=resubmit(,51)
 cookie=0xec2ce71d, duration=297.967s, table=51, n_packets=56, n_bytes=10064, priority=100,metadata=0x1,dl_dst=01:00:00:00:00:00/01:00:00:00:00:00 actions=load:0->NXM_NX_XXREG0[111],resubmit(,52)
 cookie=0xec2ce71d, duration=297.910s, table=51, n_packets=15, n_bytes=1050, priority=100,metadata=0x3,dl_dst=01:00:00:00:00:00/01:00:00:00:00:00 actions=load:0->NXM_NX_XXREG0[111],resubmit(,52)
 cookie=0x5415bc0d, duration=298.006s, table=51, n_packets=14, n_bytes=1260, priority=0,metadata=0x1 actions=load:0->NXM_NX_REG10[12],resubmit(,75),move:NXM_NX_REG10[12]->NXM_NX_XXREG0[111],resubmit(,52)
 cookie=0x5415bc0d, duration=297.915s, table=51, n_packets=171, n_bytes=17133, priority=0,metadata=0x3 actions=load:0->NXM_NX_REG10[12],resubmit(,75),move:NXM_NX_REG10[12]->NXM_NX_XXREG0[111],resubmit(,52)
 cookie=0xbdf1791d, duration=297.968s, table=52, n_packets=0, n_bytes=0, priority=50,reg0=0x8000/0x8000,metadata=0x1 actions=drop
 cookie=0xbdf1791d, duration=297.912s, table=52, n_packets=0, n_bytes=0, priority=50,reg0=0x8000/0x8000,metadata=0x3 actions=drop
 cookie=0x265bdfed, duration=298.021s, table=52, n_packets=70, n_bytes=11324, priority=0,metadata=0x1 actions=resubmit(,64)
 cookie=0x265bdfed, duration=297.934s, table=52, n_packets=186, n_bytes=18183, priority=0,metadata=0x3 actions=resubmit(,64)
 cookie=0xa481ae91, duration=297.910s, table=64, n_packets=0, n_bytes=0, priority=100,reg10=0x1/0x1,reg15=0x5,metadata=0x3 actions=push:NXM_OF_IN_PORT[],load:0xffff->NXM_OF_IN_PORT[],resubmit(,65),pop:NXM_OF_IN_PORT[]
 cookie=0xc697c99c, duration=297.910s, table=64, n_packets=6, n_bytes=856, priority=100,reg10=0x1/0x1,reg15=0x2,metadata=0x3 actions=push:NXM_OF_IN_PORT[],load:0xffff->NXM_OF_IN_PORT[],resubmit(,65),pop:NXM_OF_IN_PORT[]
 cookie=0x1b707d7, duration=297.910s, table=64, n_packets=14, n_bytes=1260, priority=100,reg10=0x1/0x1,reg15=0x2,metadata=0x4 actions=push:NXM_OF_IN_PORT[],load:0xffff->NXM_OF_IN_PORT[],resubmit(,65),pop:NXM_OF_IN_PORT[]
 cookie=0x839bff69, duration=297.910s, table=64, n_packets=3, n_bytes=294, priority=100,reg10=0x1/0x1,reg15=0x1,metadata=0x4 actions=push:NXM_OF_IN_PORT[],load:0xffff->NXM_OF_IN_PORT[],resubmit(,65),pop:NXM_OF_IN_PORT[]
 cookie=0xa3a7b338, duration=297.910s, table=64, n_packets=0, n_bytes=0, priority=100,reg10=0x1/0x1,reg15=0x4,metadata=0x1 actions=push:NXM_OF_IN_PORT[],load:0xffff->NXM_OF_IN_PORT[],resubmit(,65),pop:NXM_OF_IN_PORT[]
 cookie=0x3c1e627e, duration=297.880s, table=64, n_packets=0, n_bytes=0, priority=100,reg10=0x1/0x1,reg15=0x1,metadata=0x1 actions=push:NXM_OF_IN_PORT[],load:0xffff->NXM_OF_IN_PORT[],resubmit(,65),pop:NXM_OF_IN_PORT[]
 cookie=0x9b5b3836, duration=290.822s, table=64, n_packets=1, n_bytes=42, priority=100,reg10=0x1/0x1,reg15=0x1,metadata=0x3 actions=push:NXM_OF_IN_PORT[],load:0xffff->NXM_OF_IN_PORT[],resubmit(,65),pop:NXM_OF_IN_PORT[]
 cookie=0x0, duration=2294.001s, table=64, n_packets=249, n_bytes=28609, priority=0 actions=resubmit(,65)
 cookie=0xa3a7b338, duration=297.910s, table=65, n_packets=28, n_bytes=5032, priority=100,reg15=0x4,metadata=0x1 actions=clone(ct_clear,load:0->NXM_NX_REG11[],load:0->NXM_NX_REG12[],load:0->NXM_NX_REG13[0..15],load:0x3->NXM_NX_REG11[],load:0x1->NXM_NX_REG12[],load:0x4->OXM_OF_METADATA[],load:0x2->NXM_NX_REG14[],load:0->NXM_NX_REG10[],load:0->NXM_NX_REG15[],load:0->NXM_NX_REG0[],load:0->NXM_NX_REG1[],load:0->NXM_NX_REG2[],load:0->NXM_NX_REG3[],load:0->NXM_NX_REG4[],load:0->NXM_NX_REG5[],load:0->NXM_NX_REG6[],load:0->NXM_NX_REG7[],load:0->NXM_NX_REG8[],load:0->NXM_NX_REG9[],resubmit(,8))
 cookie=0x839bff69, duration=297.910s, table=65, n_packets=3, n_bytes=294, priority=100,reg15=0x1,metadata=0x4 actions=clone(ct_clear,load:0->NXM_NX_REG11[],load:0->NXM_NX_REG12[],load:0->NXM_NX_REG13[0..15],load:0x5->NXM_NX_REG11[],load:0x4->NXM_NX_REG12[],load:0x3->OXM_OF_METADATA[],load:0x5->NXM_NX_REG14[],load:0->NXM_NX_REG10[],load:0->NXM_NX_REG15[],load:0->NXM_NX_REG0[],load:0->NXM_NX_REG1[],load:0->NXM_NX_REG2[],load:0->NXM_NX_REG3[],load:0->NXM_NX_REG4[],load:0->NXM_NX_REG5[],load:0->NXM_NX_REG6[],load:0->NXM_NX_REG7[],load:0->NXM_NX_REG8[],load:0->NXM_NX_REG9[],resubmit(,8))
 cookie=0x1b707d7, duration=297.910s, table=65, n_packets=14, n_bytes=1260, priority=100,reg15=0x2,metadata=0x4 actions=clone(ct_clear,load:0->NXM_NX_REG11[],load:0->NXM_NX_REG12[],load:0->NXM_NX_REG13[0..15],load:0x6->NXM_NX_REG11[],load:0x7->NXM_NX_REG12[],load:0x1->OXM_OF_METADATA[],load:0x4->NXM_NX_REG14[],load:0->NXM_NX_REG10[],load:0->NXM_NX_REG15[],load:0->NXM_NX_REG0[],load:0->NXM_NX_REG1[],load:0->NXM_NX_REG2[],load:0->NXM_NX_REG3[],load:0->NXM_NX_REG4[],load:0->NXM_NX_REG5[],load:0->NXM_NX_REG6[],load:0->NXM_NX_REG7[],load:0->NXM_NX_REG8[],load:0->NXM_NX_REG9[],resubmit(,8))
 cookie=0xa481ae91, duration=297.910s, table=65, n_packets=17, n_bytes=1554, priority=100,reg15=0x5,metadata=0x3 actions=clone(ct_clear,load:0->NXM_NX_REG11[],load:0->NXM_NX_REG12[],load:0->NXM_NX_REG13[0..15],load:0x3->NXM_NX_REG11[],load:0x1->NXM_NX_REG12[],load:0x4->OXM_OF_METADATA[],load:0x1->NXM_NX_REG14[],load:0->NXM_NX_REG10[],load:0->NXM_NX_REG15[],load:0->NXM_NX_REG0[],load:0->NXM_NX_REG1[],load:0->NXM_NX_REG2[],load:0->NXM_NX_REG3[],load:0->NXM_NX_REG4[],load:0->NXM_NX_REG5[],load:0->NXM_NX_REG6[],load:0->NXM_NX_REG7[],load:0->NXM_NX_REG8[],load:0->NXM_NX_REG9[],resubmit(,8))
 cookie=0xc697c99c, duration=297.910s, table=65, n_packets=70, n_bytes=8199, priority=100,reg15=0x2,metadata=0x3 actions=output:"tap31d88d1e-0e"
 cookie=0x3c1e627e, duration=297.880s, table=65, n_packets=14, n_bytes=1260, priority=100,reg15=0x1,metadata=0x1 actions=output:"patch-br-int-to"
 cookie=0x9b5b3836, duration=290.822s, table=65, n_packets=99, n_bytes=8430, priority=100,reg15=0x1,metadata=0x3 actions=output:"tapf5e24ee8-20"
 cookie=0x0, duration=2294.001s, table=65, n_packets=28, n_bytes=5032, priority=0 actions=drop
 cookie=0x5e37605f, duration=161.412s, table=66, n_packets=7, n_bytes=686, priority=100,reg0=0xac1000fe,reg15=0x2,metadata=0x4 actions=mod_dl_dst:00:15:5d:bf:ba:52,load:0x1->NXM_NX_REG10[6]
 cookie=0xd176ba88, duration=125.867s, table=66, n_packets=3, n_bytes=294, priority=100,reg0=0xac10001f,reg15=0x2,metadata=0x4 actions=mod_dl_dst:00:15:5d:bf:ba:50,load:0x1->NXM_NX_REG10[6]
 cookie=0x5e37605f, duration=161.412s, table=67, n_packets=0, n_bytes=0, priority=100,arp,reg0=0xac1000fe,reg14=0x2,metadata=0x4,dl_src=00:15:5d:bf:ba:52 actions=load:0x1->NXM_NX_REG10[6]
 cookie=0xd176ba88, duration=125.867s, table=67, n_packets=0, n_bytes=0, priority=100,arp,reg0=0xac10001f,reg14=0x2,metadata=0x4,dl_src=00:15:5d:bf:ba:50 actions=load:0x1->NXM_NX_REG10[6]
 cookie=0xc697c99c, duration=297.910s, table=73, n_packets=9, n_bytes=378, priority=95,arp,reg14=0x2,metadata=0x3 actions=resubmit(,74)
 cookie=0xc697c99c, duration=297.910s, table=73, n_packets=111, n_bytes=9774, priority=90,ip,reg14=0x2,metadata=0x3,dl_src=fa:16:3e:63:d4:da,nw_src=192.168.101.25 actions=load:0->NXM_NX_REG10[12]
 cookie=0xc697c99c, duration=297.910s, table=73, n_packets=2, n_bytes=684, priority=90,udp,reg14=0x2,metadata=0x3,dl_src=fa:16:3e:63:d4:da,nw_src=0.0.0.0,nw_dst=255.255.255.255,tp_src=68,tp_dst=67 actions=load:0->NXM_NX_REG10[12]
 cookie=0xc697c99c, duration=297.910s, table=73, n_packets=9, n_bytes=726, priority=80,reg14=0x2,metadata=0x3 actions=load:0x1->NXM_NX_REG10[12]
 cookie=0xc697c99c, duration=297.910s, table=74, n_packets=6, n_bytes=252, priority=90,arp,reg14=0x2,metadata=0x3,dl_src=fa:16:3e:63:d4:da,arp_spa=192.168.101.25,arp_sha=fa:16:3e:63:d4:da actions=load:0->NXM_NX_REG10[12]
 cookie=0xc697c99c, duration=297.910s, table=74, n_packets=0, n_bytes=0, priority=90,icmp6,reg14=0x2,metadata=0x3,nw_ttl=255,icmp_type=135,icmp_code=0,nd_sll=00:00:00:00:00:00 actions=load:0->NXM_NX_REG10[12]
 cookie=0xc697c99c, duration=297.910s, table=74, n_packets=0, n_bytes=0, priority=90,icmp6,reg14=0x2,metadata=0x3,nw_ttl=255,icmp_type=135,icmp_code=0,nd_sll=fa:16:3e:63:d4:da actions=load:0->NXM_NX_REG10[12]
 cookie=0xc697c99c, duration=297.910s, table=74, n_packets=0, n_bytes=0, priority=90,icmp6,reg14=0x2,metadata=0x3,nw_ttl=255,icmp_type=136,icmp_code=0,nd_tll=00:00:00:00:00:00 actions=load:0->NXM_NX_REG10[12]
 cookie=0xc697c99c, duration=297.910s, table=74, n_packets=0, n_bytes=0, priority=90,icmp6,reg14=0x2,metadata=0x3,nw_ttl=255,icmp_type=136,icmp_code=0,nd_tll=fa:16:3e:63:d4:da actions=load:0->NXM_NX_REG10[12]
 cookie=0xc697c99c, duration=297.910s, table=74, n_packets=3, n_bytes=126, priority=80,arp,reg14=0x2,metadata=0x3 actions=load:0x1->NXM_NX_REG10[12]
 cookie=0xc697c99c, duration=297.910s, table=74, n_packets=0, n_bytes=0, priority=80,icmp6,reg14=0x2,metadata=0x3,nw_ttl=255,icmp_type=136 actions=load:0x1->NXM_NX_REG10[12]
 cookie=0xc697c99c, duration=297.910s, table=74, n_packets=0, n_bytes=0, priority=80,icmp6,reg14=0x2,metadata=0x3,nw_ttl=255,icmp_type=135 actions=load:0->NXM_NX_REG10[12]
 cookie=0xc697c99c, duration=297.910s, table=75, n_packets=55, n_bytes=7149, priority=95,ip,reg15=0x2,metadata=0x3,dl_dst=fa:16:3e:63:d4:da,nw_dst=192.168.101.25 actions=load:0->NXM_NX_REG10[12]
 cookie=0xc697c99c, duration=297.910s, table=75, n_packets=0, n_bytes=0, priority=95,ip,reg15=0x2,metadata=0x3,dl_dst=fa:16:3e:63:d4:da,nw_dst=255.255.255.255 actions=load:0->NXM_NX_REG10[12]
 cookie=0xc697c99c, duration=297.910s, table=75, n_packets=0, n_bytes=0, priority=95,ip,reg15=0x2,metadata=0x3,dl_dst=fa:16:3e:63:d4:da,nw_dst=224.0.0.0/4 actions=load:0->NXM_NX_REG10[12]
 cookie=0xc697c99c, duration=297.910s, table=75, n_packets=0, n_bytes=0, priority=90,ip,reg15=0x2,metadata=0x3,dl_dst=fa:16:3e:63:d4:da actions=load:0x1->NXM_NX_REG10[12]
 cookie=0xc697c99c, duration=297.910s, table=75, n_packets=0, n_bytes=0, priority=90,ipv6,reg15=0x2,metadata=0x3,dl_dst=fa:16:3e:63:d4:da actions=load:0x1->NXM_NX_REG10[12]
 cookie=0xc697c99c, duration=297.910s, table=75, n_packets=4, n_bytes=168, priority=85,reg15=0x2,metadata=0x3,dl_dst=fa:16:3e:63:d4:da actions=load:0->NXM_NX_REG10[12]
 cookie=0xc697c99c, duration=297.910s, table=75, n_packets=0, n_bytes=0, priority=80,reg15=0x2,metadata=0x3 actions=load:0x1->NXM_NX_REG10[12]
 cookie=0x5e37605f, duration=161.412s, table=79, n_packets=0, n_bytes=0, priority=100,ip,reg14=0x2,metadata=0x4,dl_src=00:15:5d:bf:ba:52,nw_src=172.16.0.254 actions=drop
 cookie=0xd176ba88, duration=125.867s, table=79, n_packets=0, n_bytes=0, priority=100,ip,reg14=0x2,metadata=0x4,dl_src=00:15:5d:bf:ba:50,nw_src=172.16.0.31 actions=drop
```

ブリッジ br-provider のフローのエントリを確認する。

```sh
ovs-ofctl dump-flows br-provider
```

```
 cookie=0x0, duration=2364.149s, table=0, n_packets=391, n_bytes=65262, priority=0 actions=NORMAL
```

ブリッジ br-mgmt のフローのエントリを確認する。

```sh
ovs-ofctl dump-flows br-mgmt
```

```
 cookie=0x0, duration=2374.576s, table=0, n_packets=392, n_bytes=65948, priority=0 actions=NORMAL
```
