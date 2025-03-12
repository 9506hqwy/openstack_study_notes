# flat ネットワーク (Open Virtual Network)

Open Virtual Network を利用した flat ネットワークを作成する。

## 外部ネットワークの作成

eth2 に繋がる外部ネットワークに flat ネットワークを作成する。

| オプション                  | 説明                               |
| --------------------------- | ---------------------------------- |
| --share                     | プロジェクトで共有                 |
| -external                   | OpenStack 外部のネットワーク       |
| --provider-physical-network | `ovn-bridge-mappings` に指定した値 |
| --provider-physical-network | flat                               |

```sh
openstack network create \
    --share \
    --external \
    --provider-physical-network provider \
    --provider-network-type flat \
    provider
```

```text
+---------------------------+--------------------------------------+
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| admin_state_up            | UP                                   |
| availability_zone_hints   |                                      |
| availability_zones        |                                      |
| created_at                | 2024-06-04T13:32:13Z                 |
| description               |                                      |
| dns_domain                | None                                 |
| id                        | a6370456-5bb9-4dde-8b73-3fe5ba0c414a |
| ipv4_address_scope        | None                                 |
| ipv6_address_scope        | None                                 |
| is_default                | False                                |
| is_vlan_transparent       | None                                 |
| mtu                       | 1500                                 |
| name                      | provider                             |
| port_security_enabled     | True                                 |
| project_id                | be94f4411bd74f249f5e25f642209b82     |
| provider:network_type     | flat                                 |
| provider:physical_network | provider                             |
| provider:segmentation_id  | None                                 |
| qos_policy_id             | None                                 |
| revision_number           | 1                                    |
| router:external           | External                             |
| segments                  | None                                 |
| shared                    | True                                 |
| status                    | ACTIVE                               |
| subnets                   |                                      |
| tags                      |                                      |
| tenant_id                 | be94f4411bd74f249f5e25f642209b82     |
| updated_at                | 2024-06-04T13:32:13Z                 |
+---------------------------+--------------------------------------+
```

メタデータサービスのポートを確認する。

```sh
openstack port list
```

```text
+--------------------------------------+------+-------------------+--------------------+--------+
| ID                                   | Name | MAC Address       | Fixed IP Addresses | Status |
+--------------------------------------+------+-------------------+--------------------+--------+
| 465fdd06-6fa7-4fb9-8d66-35595337ff6a |      | fa:16:3e:61:51:6f |                    | DOWN   |
+--------------------------------------+------+-------------------+--------------------+--------+
```

```sh
openstack port show 465fdd06-6fa7-4fb9-8d66-35595337ff6a
```

```text
+-------------------------+----------------------------------------------+
| Field                   | Value                                        |
+-------------------------+----------------------------------------------+
| admin_state_up          | UP                                           |
| allowed_address_pairs   |                                              |
| binding_host_id         |                                              |
| binding_profile         |                                              |
| binding_vif_details     |                                              |
| binding_vif_type        | unbound                                      |
| binding_vnic_type       | normal                                       |
| created_at              | 2024-06-04T13:32:13Z                         |
| data_plane_status       | None                                         |
| description             |                                              |
| device_id               | ovnmeta-a6370456-5bb9-4dde-8b73-3fe5ba0c414a |
| device_owner            | network:distributed                          |
| device_profile          | None                                         |
| dns_assignment          | None                                         |
| dns_domain              | None                                         |
| dns_name                | None                                         |
| extra_dhcp_opts         |                                              |
| fixed_ips               |                                              |
| hardware_offload_type   | None                                         |
| hints                   |                                              |
| id                      | 465fdd06-6fa7-4fb9-8d66-35595337ff6a         |
| ip_allocation           | None                                         |
| mac_address             | fa:16:3e:61:51:6f                            |
| name                    |                                              |
| network_id              | a6370456-5bb9-4dde-8b73-3fe5ba0c414a         |
| numa_affinity_policy    | None                                         |
| port_security_enabled   | False                                        |
| project_id              | be94f4411bd74f249f5e25f642209b82             |
| propagate_uplink_status | None                                         |
| resource_request        | None                                         |
| revision_number         | 1                                            |
| qos_network_policy_id   | None                                         |
| qos_policy_id           | None                                         |
| security_group_ids      |                                              |
| status                  | DOWN                                         |
| tags                    |                                              |
| trunk_details           | None                                         |
| updated_at              | 2024-06-04T13:32:13Z                         |
+-------------------------+----------------------------------------------+
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

```text
_uuid               : 6c84a035-ea9a-4c8a-a1e4-aba91462d704
acls                : []
copp                : []
dns_records         : []
external_ids        : {"neutron:availability_zone_hints"="", "neutron:mtu"="1500", "neutron:network_name"=provider, "neutron:revision_number"="1"}
forwarding_groups   : []
load_balancer       : []
load_balancer_group : []
name                : neutron-a6370456-5bb9-4dde-8b73-3fe5ba0c414a
other_config        : {fdb_age_threshold="0", mcast_flood_unregistered="false", mcast_snoop="false", vlan-passthru="false"}
ports               : [6679b27b-68b5-42e6-9053-2dd493b252d9, 85fb967c-89c3-49ac-970d-f1c008cfa476]
qos_rules           : []
```

ポートを確認する。

```sh
ovn-nbctl list Logical_Switch_Port
```

```text
_uuid               : 85fb967c-89c3-49ac-970d-f1c008cfa476
addresses           : ["fa:16:3e:61:51:6f"]
dhcpv4_options      : []
dhcpv6_options      : []
dynamic_addresses   : []
enabled             : true
external_ids        : {"neutron:cidrs"="", "neutron:device_id"=ovnmeta-a6370456-5bb9-4dde-8b73-3fe5ba0c414a, "neutron:device_owner"="network:distributed", "neutron:mtu"="", "neutron:network_name"=neutron-a6370456-5bb9-4dde-8b73-3fe5ba0c414a, "neutron:port_capabilities"="", "neutron:port_name"="", "neutron:project_id"=be94f4411bd74f249f5e25f642209b82, "neutron:revision_number"="1", "neutron:security_group_ids"="", "neutron:subnet_pool_addr_scope4"="", "neutron:subnet_pool_addr_scope6"="", "neutron:vnic_type"=normal}
ha_chassis_group    : []
mirror_rules        : []
name                : "465fdd06-6fa7-4fb9-8d66-35595337ff6a"
options             : {}
parent_name         : []
port_security       : []
tag                 : []
tag_request         : []
type                : localport
up                  : false

_uuid               : 6679b27b-68b5-42e6-9053-2dd493b252d9
addresses           : [unknown]
dhcpv4_options      : []
dhcpv6_options      : []
dynamic_addresses   : []
enabled             : []
external_ids        : {}
ha_chassis_group    : []
mirror_rules        : []
name                : provnet-ac334841-8877-453f-91de-38264a45b3bf
options             : {localnet_learn_fdb="false", mcast_flood="false", mcast_flood_reports="true", network_name=provider}
parent_name         : []
port_security       : []
tag                 : []
tag_request         : []
type                : localnet
up                  : false
```

セキュリティグループを確認する。

```sh
ovn-nbctl list ACL
```

```text
_uuid               : 2e9f13d4-032a-4301-a04a-5551d0e792ad
action              : allow-related
direction           : from-lport
external_ids        : {"neutron:security_group_rule_id"="ef2b2f41-234d-4693-b427-a9fcfeea849d"}
label               : 0
log                 : false
match               : "inport == @pg_10343956_af05_47e3_8a7a_8022d9a85945 && ip6"
meter               : []
name                : []
options             : {}
priority            : 1002
severity            : []
tier                : 0

_uuid               : d2413d59-4e1a-4585-b1f0-267b763f3d0b
action              : allow-related
direction           : from-lport
external_ids        : {"neutron:security_group_rule_id"="2f926dd1-f804-42dd-8660-31c201486a93"}
label               : 0
log                 : false
match               : "inport == @pg_10343956_af05_47e3_8a7a_8022d9a85945 && ip4"
meter               : []
name                : []
options             : {}
priority            : 1002
severity            : []
tier                : 0

_uuid               : 3b1768c2-3f33-4e57-a614-427811758daf
action              : drop
direction           : from-lport
external_ids        : {}
label               : 0
log                 : false
match               : "inport == @neutron_pg_drop && ip"
meter               : []
name                : []
options             : {}
priority            : 1001
severity            : []
tier                : 0

_uuid               : 64fbccaf-1326-4a91-8ac6-5b1ff4ecc4e0
action              : allow-related
direction           : to-lport
external_ids        : {"neutron:security_group_rule_id"="748ef95f-0095-4065-b97f-5d27a601ffcb"}
label               : 0
log                 : false
match               : "outport == @pg_10343956_af05_47e3_8a7a_8022d9a85945 && ip4 && ip4.src == $pg_10343956_af05_47e3_8a7a_8022d9a85945_ip4"
meter               : []
name                : []
options             : {}
priority            : 1002
severity            : []
tier                : 0

_uuid               : 6c5f59b3-6eae-4662-bcdf-3088eaf83095
action              : allow-related
direction           : to-lport
external_ids        : {"neutron:security_group_rule_id"="41a78521-61f9-4a1a-8148-7d5b7eb38219"}
label               : 0
log                 : false
match               : "outport == @pg_10343956_af05_47e3_8a7a_8022d9a85945 && ip6 && ip6.src == $pg_10343956_af05_47e3_8a7a_8022d9a85945_ip6"
meter               : []
name                : []
options             : {}
priority            : 1002
severity            : []
tier                : 0

_uuid               : 3c8b2fef-ddd1-4040-8e26-ea373222b198
action              : drop
direction           : to-lport
external_ids        : {}
label               : 0
log                 : false
match               : "outport == @neutron_pg_drop && ip"
meter               : []
name                : []
options             : {}
priority            : 1001
severity            : []
tier                : 0
```

#### Southbound データベース

アドレスセットを確認する。

```sh
ovn-sbctl list Address_Set
```

```text
_uuid               : 11f9d440-b848-436b-89a8-15b0a7474cc5
addresses           : []
name                : pg_10343956_af05_47e3_8a7a_8022d9a85945_ip6

_uuid               : d0cae968-1582-4c82-8324-88900c57b6ec
addresses           : ["2a:91:79:ab:04:2d"]
name                : svc_monitor_mac

_uuid               : 840d070d-12a7-4434-8047-95526f7b0186
addresses           : []
name                : neutron_pg_drop_ip6

_uuid               : 8c00b67a-a5af-4829-8b0f-aefcd6c5abd7
addresses           : []
name                : pg_10343956_af05_47e3_8a7a_8022d9a85945_ip4

_uuid               : a0eda553-388f-4810-b31e-63d7edf879f8
addresses           : []
name                : neutron_pg_drop_ip4
```

フローを確認する。

```sh
ovn-sbctl lflow-list
```

```text
Datapath: "neutron-a6370456-5bb9-4dde-8b73-3fe5ba0c414a" aka "provider" (a69f8cd6-1f9d-48b5-a8e3-30a5edd951fe)  Pipeline: ingress
  table=0 (ls_in_check_port_sec), priority=110  , match=(((ip4 && icmp4.type == 3 && icmp4.code == 4) || (ip6 && icmp6.type == 2 && icmp6.code == 0)) && flags.tunnel_rx == 1), action=(drop;)
  table=0 (ls_in_check_port_sec), priority=100  , match=(eth.src[40]), action=(drop;)
  table=0 (ls_in_check_port_sec), priority=100  , match=(vlan.present), action=(drop;)
  table=0 (ls_in_check_port_sec), priority=50   , match=(1), action=(reg0[15] = check_in_port_sec(); next;)
  table=1 (ls_in_apply_port_sec), priority=50   , match=(reg0[15] == 1), action=(drop;)
  table=1 (ls_in_apply_port_sec), priority=0    , match=(1), action=(next;)
  table=2 (ls_in_lookup_fdb   ), priority=0    , match=(1), action=(next;)
  table=3 (ls_in_put_fdb      ), priority=0    , match=(1), action=(next;)
  table=4 (ls_in_pre_acl      ), priority=110  , match=(eth.dst == $svc_monitor_mac), action=(next;)
  table=4 (ls_in_pre_acl      ), priority=0    , match=(1), action=(next;)
  table=5 (ls_in_pre_lb       ), priority=110  , match=(eth.dst == $svc_monitor_mac), action=(next;)
  table=5 (ls_in_pre_lb       ), priority=110  , match=(eth.mcast), action=(next;)
  table=5 (ls_in_pre_lb       ), priority=110  , match=(ip && inport == "provnet-ac334841-8877-453f-91de-38264a45b3bf"), action=(next;)
  table=5 (ls_in_pre_lb       ), priority=110  , match=(nd || nd_rs || nd_ra || mldv1 || mldv2), action=(next;)
  table=5 (ls_in_pre_lb       ), priority=110  , match=(reg0[16] == 1), action=(next;)
  table=5 (ls_in_pre_lb       ), priority=0    , match=(1), action=(next;)
  table=6 (ls_in_pre_stateful ), priority=110  , match=(reg0[2] == 1), action=(ct_lb_mark;)
  table=6 (ls_in_pre_stateful ), priority=100  , match=(reg0[0] == 1), action=(ct_next;)
  table=6 (ls_in_pre_stateful ), priority=0    , match=(1), action=(next;)
  table=7 (ls_in_acl_hint     ), priority=65535, match=(1), action=(next;)
  table=8 (ls_in_acl_eval     ), priority=65535, match=(1), action=(next;)
  table=8 (ls_in_acl_eval     ), priority=65532, match=(nd || nd_ra || nd_rs || mldv1 || mldv2), action=(reg8[16] = 1; next;)
  table=9 (ls_in_acl_action   ), priority=0    , match=(1), action=(next;)
  table=10(ls_in_qos_mark     ), priority=0    , match=(1), action=(next;)
  table=11(ls_in_qos_meter    ), priority=0    , match=(1), action=(next;)
  table=12(ls_in_lb_aff_check ), priority=0    , match=(1), action=(next;)
  table=13(ls_in_lb           ), priority=0    , match=(1), action=(next;)
  table=14(ls_in_lb_aff_learn ), priority=0    , match=(1), action=(next;)
  table=15(ls_in_pre_hairpin  ), priority=0    , match=(1), action=(next;)
  table=16(ls_in_nat_hairpin  ), priority=0    , match=(1), action=(next;)
  table=17(ls_in_hairpin      ), priority=0    , match=(1), action=(next;)
  table=18(ls_in_acl_after_lb_eval), priority=65532, match=(nd || nd_ra || nd_rs || mldv1 || mldv2), action=(reg8[16] = 1; next;)
  table=18(ls_in_acl_after_lb_eval), priority=0    , match=(1), action=(next;)
  table=19(ls_in_acl_after_lb_action), priority=0    , match=(1), action=(next;)
  table=20(ls_in_stateful     ), priority=100  , match=(reg0[1] == 1 && reg0[13] == 0), action=(ct_commit { ct_mark.blocked = 0; }; next;)
  table=20(ls_in_stateful     ), priority=100  , match=(reg0[1] == 1 && reg0[13] == 1), action=(ct_commit { ct_mark.blocked = 0; ct_label.label = reg3; }; next;)
  table=20(ls_in_stateful     ), priority=0    , match=(1), action=(next;)
  table=21(ls_in_arp_rsp      ), priority=100  , match=(inport == "provnet-ac334841-8877-453f-91de-38264a45b3bf"), action=(next;)
  table=21(ls_in_arp_rsp      ), priority=0    , match=(1), action=(next;)
  table=22(ls_in_dhcp_options ), priority=0    , match=(1), action=(next;)
  table=23(ls_in_dhcp_response), priority=0    , match=(1), action=(next;)
  table=24(ls_in_dns_lookup   ), priority=0    , match=(1), action=(next;)
  table=25(ls_in_dns_response ), priority=0    , match=(1), action=(next;)
  table=26(ls_in_external_port), priority=0    , match=(1), action=(next;)
  table=27(ls_in_l2_lkup      ), priority=110  , match=(eth.dst == $svc_monitor_mac && (tcp || icmp || icmp6)), action=(handle_svc_check(inport);)
  table=27(ls_in_l2_lkup      ), priority=70   , match=(eth.mcast), action=(outport = "_MC_flood"; output;)
  table=27(ls_in_l2_lkup      ), priority=50   , match=(eth.dst == fa:16:3e:61:51:6f), action=(outport = "465fdd06-6fa7-4fb9-8d66-35595337ff6a"; output;)
  table=27(ls_in_l2_lkup      ), priority=0    , match=(1), action=(outport = get_fdb(eth.dst); next;)
  table=28(ls_in_l2_unknown   ), priority=50   , match=(outport == "none"), action=(outport = "_MC_unknown"; output;)
  table=28(ls_in_l2_unknown   ), priority=0    , match=(1), action=(output;)
Datapath: "neutron-a6370456-5bb9-4dde-8b73-3fe5ba0c414a" aka "provider" (a69f8cd6-1f9d-48b5-a8e3-30a5edd951fe)  Pipeline: egress
  table=0 (ls_out_pre_acl     ), priority=110  , match=(eth.src == $svc_monitor_mac), action=(next;)
  table=0 (ls_out_pre_acl     ), priority=0    , match=(1), action=(next;)
  table=1 (ls_out_pre_lb      ), priority=110  , match=(eth.mcast), action=(next;)
  table=1 (ls_out_pre_lb      ), priority=110  , match=(eth.src == $svc_monitor_mac), action=(next;)
  table=1 (ls_out_pre_lb      ), priority=110  , match=(ip && outport == "provnet-ac334841-8877-453f-91de-38264a45b3bf"), action=(ct_clear; next;)
  table=1 (ls_out_pre_lb      ), priority=110  , match=(nd || nd_rs || nd_ra || mldv1 || mldv2), action=(next;)
  table=1 (ls_out_pre_lb      ), priority=110  , match=(reg0[16] == 1), action=(next;)
  table=1 (ls_out_pre_lb      ), priority=0    , match=(1), action=(next;)
  table=2 (ls_out_pre_stateful), priority=110  , match=(reg0[2] == 1), action=(ct_lb_mark;)
  table=2 (ls_out_pre_stateful), priority=100  , match=(reg0[0] == 1), action=(ct_next;)
  table=2 (ls_out_pre_stateful), priority=0    , match=(1), action=(next;)
  table=3 (ls_out_acl_hint    ), priority=65535, match=(1), action=(next;)
  table=4 (ls_out_acl_eval    ), priority=65535, match=(1), action=(next;)
  table=4 (ls_out_acl_eval    ), priority=65532, match=(nd || nd_ra || nd_rs || mldv1 || mldv2), action=(reg8[16] = 1; next;)
  table=5 (ls_out_acl_action  ), priority=0    , match=(1), action=(next;)
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

```text
_uuid               : 2677d319-557d-406b-91de-e5c9cc7f1b3b
datapath            : a69f8cd6-1f9d-48b5-a8e3-30a5edd951fe
name                : _MC_flood
ports               : [479775c9-8632-4e1c-b6c2-5562759e4fb5, 9e39cafe-0eb6-461b-92c2-e7f699e9963b]
tunnel_key          : 32768

_uuid               : 4dafb955-9f7a-4912-96b5-8a0e70339051
datapath            : a69f8cd6-1f9d-48b5-a8e3-30a5edd951fe
name                : _MC_unknown
ports               : [9e39cafe-0eb6-461b-92c2-e7f699e9963b]
tunnel_key          : 32769

_uuid               : 3f08aaea-32b3-4c0e-adca-1a9465c4131b
datapath            : a69f8cd6-1f9d-48b5-a8e3-30a5edd951fe
name                : _MC_mrouter_flood
ports               : [9e39cafe-0eb6-461b-92c2-e7f699e9963b]
tunnel_key          : 32770

_uuid               : d1184936-e20e-40b5-8e69-5e0e8af04735
datapath            : a69f8cd6-1f9d-48b5-a8e3-30a5edd951fe
name                : _MC_flood_l2
ports               : [479775c9-8632-4e1c-b6c2-5562759e4fb5, 9e39cafe-0eb6-461b-92c2-e7f699e9963b]
tunnel_key          : 32772
```

データパスを確認する。

```sh
ovn-sbctl list Datapath_Binding
```

```text
_uuid               : a69f8cd6-1f9d-48b5-a8e3-30a5edd951fe
external_ids        : {logical-switch="6c84a035-ea9a-4c8a-a1e4-aba91462d704", name=neutron-a6370456-5bb9-4dde-8b73-3fe5ba0c414a, name2=provider}
load_balancers      : []
tunnel_key          : 1
```

ポートを確認する。

```sh
ovn-sbctl list Port_Binding
```

```text
_uuid               : 479775c9-8632-4e1c-b6c2-5562759e4fb5
additional_chassis  : []
additional_encap    : []
chassis             : []
datapath            : a69f8cd6-1f9d-48b5-a8e3-30a5edd951fe
encap               : []
external_ids        : {"neutron:cidrs"="", "neutron:device_id"=ovnmeta-a6370456-5bb9-4dde-8b73-3fe5ba0c414a, "neutron:device_owner"="network:distributed", "neutron:mtu"="", "neutron:network_name"=neutron-a6370456-5bb9-4dde-8b73-3fe5ba0c414a, "neutron:port_capabilities"="", "neutron:port_name"="", "neutron:project_id"=be94f4411bd74f249f5e25f642209b82, "neutron:revision_number"="1", "neutron:security_group_ids"="", "neutron:subnet_pool_addr_scope4"="", "neutron:subnet_pool_addr_scope6"="", "neutron:vnic_type"=normal}
gateway_chassis     : []
ha_chassis_group    : []
logical_port        : "465fdd06-6fa7-4fb9-8d66-35595337ff6a"
mac                 : ["fa:16:3e:61:51:6f"]
mirror_rules        : []
nat_addresses       : []
options             : {}
parent_port         : []
port_security       : []
requested_additional_chassis: []
requested_chassis   : []
tag                 : []
tunnel_key          : 2
type                : localport
up                  : false
virtual_parent      : []

_uuid               : 9e39cafe-0eb6-461b-92c2-e7f699e9963b
additional_chassis  : []
additional_encap    : []
chassis             : []
datapath            : a69f8cd6-1f9d-48b5-a8e3-30a5edd951fe
encap               : []
external_ids        : {}
gateway_chassis     : []
ha_chassis_group    : []
logical_port        : provnet-ac334841-8877-453f-91de-38264a45b3bf
mac                 : [unknown]
mirror_rules        : []
nat_addresses       : []
options             : {localnet_learn_fdb="false", mcast_flood="false", mcast_flood_reports="true", network_name=provider}
parent_port         : []
port_security       : []
requested_additional_chassis: []
requested_chassis   : []
tag                 : []
tunnel_key          : 1
type                : localnet
up                  : false
virtual_parent      : []
```

#### Open vSwitch

ブリッジを確認する。

```sh
ovs-vsctl show
```

```text
9869c49f-a522-47dc-a744-bf85afff8b76
    Bridge br-int
        fail_mode: secure
        datapath_type: system
        Port ovn-32a2f4-0
            Interface ovn-32a2f4-0
                type: geneve
                options: {csum="true", key=flow, remote_ip="172.16.0.31"}
        Port br-int
            Interface br-int
                type: internal
    Bridge br-provider
        Port eth2
            Interface eth2
                type: system
    Bridge br-mgmt
        Port eth3
            Interface eth3
                type: system
    ovs_version: "3.3.1"
```

データパスを確認する。

```sh
ovs-dpctl show
```

```text
system@ovs-system:
  lookups: hit:627 missed:298 lost:0
  flows: 0
  masks: hit:1006 total:0 hit/pkt:1.09
  cache: hit:350 hit-rate:37.84%
  caches:
    masks-cache: size:256
  port 0: ovs-system (internal)
  port 1: br-int (internal)
  port 2: eth2
  port 3: eth3
  port 4: genev_sys_6081 (geneve: packet_type=ptap)
```

ブリッジ br-int のフローのエントリを確認する。

```sh
ovs-ofctl dump-flows br-int
```

```text
 cookie=0x0, duration=5124.343s, table=0, n_packets=0, n_bytes=0, priority=120,icmp6,in_port="ovn-32a2f4-0",icmp_type=2,icmp_code=0 actions=move:NXM_NX_TUN_ID[0..23]->OXM_OF_METADATA[0..23],move:NXM_NX_TUN_METADATA0[16..30]->NXM_NX_REG14[0..14],move:NXM_NX_TUN_METADATA0[0..15]->NXM_NX_REG15[0..15],resubmit(,40),load:0x1->NXM_NX_REG10[16],resubmit(,8)
 cookie=0x0, duration=5124.343s, table=0, n_packets=0, n_bytes=0, priority=120,icmp,in_port="ovn-32a2f4-0",icmp_type=3,icmp_code=4 actions=move:NXM_NX_TUN_ID[0..23]->OXM_OF_METADATA[0..23],move:NXM_NX_TUN_METADATA0[16..30]->NXM_NX_REG14[0..14],move:NXM_NX_TUN_METADATA0[0..15]->NXM_NX_REG15[0..15],resubmit(,40),load:0x1->NXM_NX_REG10[16],resubmit(,8)
 cookie=0x0, duration=5124.343s, table=0, n_packets=0, n_bytes=0, priority=100,in_port="ovn-32a2f4-0" actions=move:NXM_NX_TUN_ID[0..23]->OXM_OF_METADATA[0..23],move:NXM_NX_TUN_METADATA0[16..30]->NXM_NX_REG14[0..14],move:NXM_NX_TUN_METADATA0[0..15]->NXM_NX_REG15[0..15],resubmit(,40)
 cookie=0x0, duration=6605.126s, table=0, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x0, duration=6605.126s, table=37, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,39)
 cookie=0x0, duration=6605.126s, table=38, n_packets=0, n_bytes=0, priority=10,reg10=0x1/0x1 actions=resubmit(,40)
 cookie=0x0, duration=6605.126s, table=38, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,39)
 cookie=0x0, duration=6605.126s, table=39, n_packets=0, n_bytes=0, priority=150,reg10=0x10/0x10 actions=resubmit(,40)
 cookie=0x0, duration=6605.126s, table=39, n_packets=0, n_bytes=0, priority=150,reg10=0x2/0x3 actions=resubmit(,40)
 cookie=0x0, duration=6605.126s, table=39, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,40)
 cookie=0x0, duration=6605.126s, table=40, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x0, duration=6605.126s, table=41, n_packets=0, n_bytes=0, priority=0 actions=load:0->NXM_NX_REG0[],load:0->NXM_NX_REG1[],load:0->NXM_NX_REG2[],load:0->NXM_NX_REG3[],load:0->NXM_NX_REG4[],load:0->NXM_NX_REG5[],load:0->NXM_NX_REG6[],load:0->NXM_NX_REG7[],load:0->NXM_NX_REG8[],load:0->NXM_NX_REG9[],resubmit(,42)
 cookie=0x0, duration=6605.126s, table=64, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,65)
 cookie=0x0, duration=6605.126s, table=65, n_packets=0, n_bytes=0, priority=0 actions=drop
```

ブリッジ br-provider のフローのエントリを確認する。

```sh
ovs-ofctl dump-flows br-provider
```

```text
 cookie=0x0, duration=6612.411s, table=0, n_packets=449, n_bytes=80868, priority=0 actions=NORMAL
```

ブリッジ br-mgmt のフローのエントリを確認する。

```sh
ovs-ofctl dump-flows br-mgmt
```

```text
 cookie=0x0, duration=6617.362s, table=0, n_packets=476, n_bytes=83862, priority=0 actions=NORMAL
```

トンネルを確認する。

```sh
ovs-appctl ofproto/list-tunnels
```

```text
port 4: ovn-32a2f4-0 (geneve: ::->172.16.0.31, key=flow, legacy_l2, dp port=4, ttl=64, csum=true)
```

### Compute Node

#### ネットワーク名前空間

ネットワーク名前空間は作成されない。

#### Open vSwitch

ブリッジの構成を確認する。

```sh
ovs-vsctl show
```

```text
46423187-0689-4323-afe2-2ad6d689596b
    Bridge br-mgmt
        Port eth3
            Interface eth3
                type: system
    Bridge br-int
        fail_mode: secure
        datapath_type: system
        Port ovn-3732c7-0
            Interface ovn-3732c7-0
                type: geneve
                options: {csum="true", key=flow, remote_ip="172.16.0.11"}
        Port br-int
            Interface br-int
                type: internal
    Bridge br-provider
        Port eth2
            Interface eth2
                type: system
    ovs_version: "3.3.1"
```

データパスを確認する。

```sh
ovs-dpctl show
```

```text
system@ovs-system:
  lookups: hit:355 missed:120 lost:0
  flows: 0
  masks: hit:386 total:0 hit/pkt:0.81
  cache: hit:235 hit-rate:49.47%
  caches:
    masks-cache: size:256
  port 0: ovs-system (internal)
  port 1: br-int (internal)
  port 2: genev_sys_6081 (geneve: packet_type=ptap)
  port 3: eth2
  port 4: eth3
```

ブリッジ br-int のフローのエントリを確認する。

```sh
ovs-ofctl dump-flows br-int
```

```text
 cookie=0x0, duration=5256.531s, table=0, n_packets=0, n_bytes=0, priority=120,icmp6,in_port="ovn-3732c7-0",icmp_type=2,icmp_code=0 actions=move:NXM_NX_TUN_ID[0..23]->OXM_OF_METADATA[0..23],move:NXM_NX_TUN_METADATA0[16..30]->NXM_NX_REG14[0..14],move:NXM_NX_TUN_METADATA0[0..15]->NXM_NX_REG15[0..15],resubmit(,40),load:0x1->NXM_NX_REG10[16],resubmit(,8)
 cookie=0x0, duration=5256.531s, table=0, n_packets=0, n_bytes=0, priority=120,icmp,in_port="ovn-3732c7-0",icmp_type=3,icmp_code=4 actions=move:NXM_NX_TUN_ID[0..23]->OXM_OF_METADATA[0..23],move:NXM_NX_TUN_METADATA0[16..30]->NXM_NX_REG14[0..14],move:NXM_NX_TUN_METADATA0[0..15]->NXM_NX_REG15[0..15],resubmit(,40),load:0x1->NXM_NX_REG10[16],resubmit(,8)
 cookie=0x0, duration=5256.531s, table=0, n_packets=0, n_bytes=0, priority=100,in_port="ovn-3732c7-0" actions=move:NXM_NX_TUN_ID[0..23]->OXM_OF_METADATA[0..23],move:NXM_NX_TUN_METADATA0[16..30]->NXM_NX_REG14[0..14],move:NXM_NX_TUN_METADATA0[0..15]->NXM_NX_REG15[0..15],resubmit(,40)
 cookie=0x0, duration=5256.531s, table=0, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x0, duration=5256.531s, table=37, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,39)
 cookie=0x0, duration=5256.531s, table=38, n_packets=0, n_bytes=0, priority=10,reg10=0x1/0x1 actions=resubmit(,40)
 cookie=0x0, duration=5256.531s, table=38, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,39)
 cookie=0x0, duration=5256.531s, table=39, n_packets=0, n_bytes=0, priority=150,reg10=0x10/0x10 actions=resubmit(,40)
 cookie=0x0, duration=5256.531s, table=39, n_packets=0, n_bytes=0, priority=150,reg10=0x2/0x3 actions=resubmit(,40)
 cookie=0x0, duration=5256.531s, table=39, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,40)
 cookie=0x0, duration=5256.531s, table=40, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x0, duration=5256.531s, table=41, n_packets=0, n_bytes=0, priority=0 actions=load:0->NXM_NX_REG0[],load:0->NXM_NX_REG1[],load:0->NXM_NX_REG2[],load:0->NXM_NX_REG3[],load:0->NXM_NX_REG4[],load:0->NXM_NX_REG5[],load:0->NXM_NX_REG6[],load:0->NXM_NX_REG7[],load:0->NXM_NX_REG8[],load:0->NXM_NX_REG9[],resubmit(,42)
 cookie=0x0, duration=5256.531s, table=64, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,65)
 cookie=0x0, duration=5256.531s, table=65, n_packets=0, n_bytes=0, priority=0 actions=drop
```

ブリッジ br-provider のフローのエントリを確認する。

```sh
ovs-ofctl dump-flows br-provider
```

```text
 cookie=0x0, duration=5264.915s, table=0, n_packets=224, n_bytes=41858, priority=0 actions=NORMAL
```

ブリッジ br-mgmt のフローのエントリを確認する。

```sh
ovs-ofctl dump-flows br-mgmt
```

```text
 cookie=0x0, duration=5270.046s, table=0, n_packets=251, n_bytes=44900, priority=0 actions=NORMAL
```

トンネルを確認する。

```sh
ovs-appctl ofproto/list-tunnels
```

```text
port 2: ovn-3732c7-0 (geneve: ::->172.16.0.11, key=flow, legacy_l2, dp port=2, ttl=64, csum=true)
```

## サブネットの作成

サブネットを作成する。

| オプション        | 説明                     |
| ----------------- | ------------------------ |
| --network         | ネットワーク             |
| --allocation-pool | IP アドレス範囲          |
| --gateway         | ゲートウェイ IP アドレス |
| -subnet-range     | サブネットの CIDR        |

```sh
openstack subnet create \
    --network provider \
    --allocation-pool start=172.16.0.100,end=172.16.0.199 \
    --gateway 172.16.0.254 \
    --subnet-range 172.16.0.0/24 \
    provider
```

```text
+----------------------+--------------------------------------+
| Field                | Value                                |
+----------------------+--------------------------------------+
| allocation_pools     | 172.16.0.100-172.16.0.199            |
| cidr                 | 172.16.0.0/24                        |
| created_at           | 2024-06-04T13:52:45Z                 |
| description          |                                      |
| dns_nameservers      |                                      |
| dns_publish_fixed_ip | None                                 |
| enable_dhcp          | True                                 |
| gateway_ip           | 172.16.0.254                         |
| host_routes          |                                      |
| id                   | be32ef49-5ab1-4ba7-ab26-9518dc2b063f |
| ip_version           | 4                                    |
| ipv6_address_mode    | None                                 |
| ipv6_ra_mode         | None                                 |
| name                 | provider                             |
| network_id           | a6370456-5bb9-4dde-8b73-3fe5ba0c414a |
| project_id           | be94f4411bd74f249f5e25f642209b82     |
| revision_number      | 0                                    |
| segment_id           | None                                 |
| service_types        |                                      |
| subnetpool_id        | None                                 |
| tags                 |                                      |
| updated_at           | 2024-06-04T13:52:45Z                 |
+----------------------+--------------------------------------+
```

メタデータサービスのポートを確認する。

```sh
openstack port list
```

```text
+--------------------------------------+------+-------------------+-----------------------------------------------------------------------------+--------+
| ID                                   | Name | MAC Address       | Fixed IP Addresses                                                          | Status |
+--------------------------------------+------+-------------------+-----------------------------------------------------------------------------+--------+
| 465fdd06-6fa7-4fb9-8d66-35595337ff6a |      | fa:16:3e:61:51:6f | ip_address='172.16.0.100', subnet_id='be32ef49-5ab1-4ba7-ab26-9518dc2b063f' | DOWN   |
+--------------------------------------+------+-------------------+-----------------------------------------------------------------------------+--------+
```

```sh
openstack port show 465fdd06-6fa7-4fb9-8d66-35595337ff6a
```

```text
+-------------------------+-----------------------------------------------------------------------------+
| Field                   | Value                                                                       |
+-------------------------+-----------------------------------------------------------------------------+
| admin_state_up          | UP                                                                          |
| allowed_address_pairs   |                                                                             |
| binding_host_id         |                                                                             |
| binding_profile         |                                                                             |
| binding_vif_details     |                                                                             |
| binding_vif_type        | unbound                                                                     |
| binding_vnic_type       | normal                                                                      |
| created_at              | 2024-06-04T13:32:13Z                                                        |
| data_plane_status       | None                                                                        |
| description             |                                                                             |
| device_id               | ovnmeta-a6370456-5bb9-4dde-8b73-3fe5ba0c414a                                |
| device_owner            | network:distributed                                                         |
| device_profile          | None                                                                        |
| dns_assignment          | None                                                                        |
| dns_domain              | None                                                                        |
| dns_name                | None                                                                        |
| extra_dhcp_opts         |                                                                             |
| fixed_ips               | ip_address='172.16.0.100', subnet_id='be32ef49-5ab1-4ba7-ab26-9518dc2b063f' |
| hardware_offload_type   | None                                                                        |
| hints                   |                                                                             |
| id                      | 465fdd06-6fa7-4fb9-8d66-35595337ff6a                                        |
| ip_allocation           | None                                                                        |
| mac_address             | fa:16:3e:61:51:6f                                                           |
| name                    |                                                                             |
| network_id              | a6370456-5bb9-4dde-8b73-3fe5ba0c414a                                        |
| numa_affinity_policy    | None                                                                        |
| port_security_enabled   | False                                                                       |
| project_id              | be94f4411bd74f249f5e25f642209b82                                            |
| propagate_uplink_status | None                                                                        |
| resource_request        | None                                                                        |
| revision_number         | 2                                                                           |
| qos_network_policy_id   | None                                                                        |
| qos_policy_id           | None                                                                        |
| security_group_ids      |                                                                             |
| status                  | DOWN                                                                        |
| tags                    |                                                                             |
| trunk_details           | None                                                                        |
| updated_at              | 2024-06-04T13:52:45Z                                                        |
+-------------------------+-----------------------------------------------------------------------------+
```

## 環境の確認

### Controller Node

#### ネットワーク名前空間

ネットワーク名前空間は作成されない。

#### Northband データベース

ポートを確認する。

```sh
ovn-nbctl list Logical_Switch_Port
```

```text
_uuid               : 85fb967c-89c3-49ac-970d-f1c008cfa476
addresses           : ["fa:16:3e:61:51:6f 172.16.0.100"]
dhcpv4_options      : []
dhcpv6_options      : []
dynamic_addresses   : []
enabled             : true
external_ids        : {"neutron:cidrs"="172.16.0.100/24", "neutron:device_id"=ovnmeta-a6370456-5bb9-4dde-8b73-3fe5ba0c414a, "neutron:device_owner"="network:distributed", "neutron:mtu"="", "neutron:network_name"=neutron-a6370456-5bb9-4dde-8b73-3fe5ba0c414a, "neutron:port_capabilities"="", "neutron:port_name"="", "neutron:project_id"=be94f4411bd74f249f5e25f642209b82, "neutron:revision_number"="2", "neutron:security_group_ids"="", "neutron:subnet_pool_addr_scope4"="", "neutron:subnet_pool_addr_scope6"="", "neutron:vnic_type"=normal}
ha_chassis_group    : []
mirror_rules        : []
name                : "465fdd06-6fa7-4fb9-8d66-35595337ff6a"
options             : {}
parent_name         : []
port_security       : []
tag                 : []
tag_request         : []
type                : localport
up                  : false

_uuid               : 6679b27b-68b5-42e6-9053-2dd493b252d9
addresses           : [unknown]
dhcpv4_options      : []
dhcpv6_options      : []
dynamic_addresses   : []
enabled             : []
external_ids        : {}
ha_chassis_group    : []
mirror_rules        : []
name                : provnet-ac334841-8877-453f-91de-38264a45b3bf
options             : {localnet_learn_fdb="false", mcast_flood="false", mcast_flood_reports="true", network_name=provider}
parent_name         : []
port_security       : []
tag                 : []
tag_request         : []
type                : localnet
up                  : false
```

#### Southbound データベース

フローを確認する。

```sh
ovn-sbctl lflow-list
```

```text
Datapath: "neutron-a6370456-5bb9-4dde-8b73-3fe5ba0c414a" aka "provider" (a69f8cd6-1f9d-48b5-a8e3-30a5edd951fe)  Pipeline: ingress
  table=0 (ls_in_check_port_sec), priority=110  , match=(((ip4 && icmp4.type == 3 && icmp4.code == 4) || (ip6 && icmp6.type == 2 && icmp6.code == 0)) && flags.tunnel_rx == 1), action=(drop;)
  table=0 (ls_in_check_port_sec), priority=100  , match=(eth.src[40]), action=(drop;)
  table=0 (ls_in_check_port_sec), priority=100  , match=(vlan.present), action=(drop;)
  table=0 (ls_in_check_port_sec), priority=50   , match=(1), action=(reg0[15] = check_in_port_sec(); next;)
  table=1 (ls_in_apply_port_sec), priority=50   , match=(reg0[15] == 1), action=(drop;)
  table=1 (ls_in_apply_port_sec), priority=0    , match=(1), action=(next;)
  table=2 (ls_in_lookup_fdb   ), priority=0    , match=(1), action=(next;)
  table=3 (ls_in_put_fdb      ), priority=0    , match=(1), action=(next;)
  table=4 (ls_in_pre_acl      ), priority=110  , match=(eth.dst == $svc_monitor_mac), action=(next;)
  table=4 (ls_in_pre_acl      ), priority=0    , match=(1), action=(next;)
  table=5 (ls_in_pre_lb       ), priority=110  , match=(eth.dst == $svc_monitor_mac), action=(next;)
  table=5 (ls_in_pre_lb       ), priority=110  , match=(eth.mcast), action=(next;)
  table=5 (ls_in_pre_lb       ), priority=110  , match=(ip && inport == "provnet-ac334841-8877-453f-91de-38264a45b3bf"), action=(next;)
  table=5 (ls_in_pre_lb       ), priority=110  , match=(nd || nd_rs || nd_ra || mldv1 || mldv2), action=(next;)
  table=5 (ls_in_pre_lb       ), priority=110  , match=(reg0[16] == 1), action=(next;)
  table=5 (ls_in_pre_lb       ), priority=0    , match=(1), action=(next;)
  table=6 (ls_in_pre_stateful ), priority=110  , match=(reg0[2] == 1), action=(ct_lb_mark;)
  table=6 (ls_in_pre_stateful ), priority=100  , match=(reg0[0] == 1), action=(ct_next;)
  table=6 (ls_in_pre_stateful ), priority=0    , match=(1), action=(next;)
  table=7 (ls_in_acl_hint     ), priority=65535, match=(1), action=(next;)
  table=8 (ls_in_acl_eval     ), priority=65535, match=(1), action=(next;)
  table=8 (ls_in_acl_eval     ), priority=65532, match=(nd || nd_ra || nd_rs || mldv1 || mldv2), action=(reg8[16] = 1; next;)
  table=9 (ls_in_acl_action   ), priority=0    , match=(1), action=(next;)
  table=10(ls_in_qos_mark     ), priority=0    , match=(1), action=(next;)
  table=11(ls_in_qos_meter    ), priority=0    , match=(1), action=(next;)
  table=12(ls_in_lb_aff_check ), priority=0    , match=(1), action=(next;)
  table=13(ls_in_lb           ), priority=0    , match=(1), action=(next;)
  table=14(ls_in_lb_aff_learn ), priority=0    , match=(1), action=(next;)
  table=15(ls_in_pre_hairpin  ), priority=0    , match=(1), action=(next;)
  table=16(ls_in_nat_hairpin  ), priority=0    , match=(1), action=(next;)
  table=17(ls_in_hairpin      ), priority=0    , match=(1), action=(next;)
  table=18(ls_in_acl_after_lb_eval), priority=65532, match=(nd || nd_ra || nd_rs || mldv1 || mldv2), action=(reg8[16] = 1; next;)
  table=18(ls_in_acl_after_lb_eval), priority=0    , match=(1), action=(next;)
  table=19(ls_in_acl_after_lb_action), priority=0    , match=(1), action=(next;)
  table=20(ls_in_stateful     ), priority=100  , match=(reg0[1] == 1 && reg0[13] == 0), action=(ct_commit { ct_mark.blocked = 0; }; next;)
  table=20(ls_in_stateful     ), priority=100  , match=(reg0[1] == 1 && reg0[13] == 1), action=(ct_commit { ct_mark.blocked = 0; ct_label.label = reg3; }; next;)
  table=20(ls_in_stateful     ), priority=0    , match=(1), action=(next;)
  table=21(ls_in_arp_rsp      ), priority=100  , match=(arp.tpa == 172.16.0.100 && arp.op == 1 && inport == "465fdd06-6fa7-4fb9-8d66-35595337ff6a"), action=(next;)
  table=21(ls_in_arp_rsp      ), priority=100  , match=(inport == "provnet-ac334841-8877-453f-91de-38264a45b3bf"), action=(next;)
  table=21(ls_in_arp_rsp      ), priority=50   , match=(arp.tpa == 172.16.0.100 && arp.op == 1), action=(eth.dst = eth.src; eth.src = fa:16:3e:61:51:6f; arp.op = 2; /* ARP reply */ arp.tha = arp.sha; arp.sha = fa:16:3e:61:51:6f; arp.tpa = arp.spa; arp.spa = 172.16.0.100; outport = inport; flags.loopback = 1; output;)
  table=21(ls_in_arp_rsp      ), priority=0    , match=(1), action=(next;)
  table=22(ls_in_dhcp_options ), priority=0    , match=(1), action=(next;)
  table=23(ls_in_dhcp_response), priority=0    , match=(1), action=(next;)
  table=24(ls_in_dns_lookup   ), priority=0    , match=(1), action=(next;)
  table=25(ls_in_dns_response ), priority=0    , match=(1), action=(next;)
  table=26(ls_in_external_port), priority=0    , match=(1), action=(next;)
  table=27(ls_in_l2_lkup      ), priority=110  , match=(eth.dst == $svc_monitor_mac && (tcp || icmp || icmp6)), action=(handle_svc_check(inport);)
  table=27(ls_in_l2_lkup      ), priority=70   , match=(eth.mcast), action=(outport = "_MC_flood"; output;)
  table=27(ls_in_l2_lkup      ), priority=50   , match=(eth.dst == fa:16:3e:61:51:6f), action=(outport = "465fdd06-6fa7-4fb9-8d66-35595337ff6a"; output;)
  table=27(ls_in_l2_lkup      ), priority=0    , match=(1), action=(outport = get_fdb(eth.dst); next;)
  table=28(ls_in_l2_unknown   ), priority=50   , match=(outport == "none"), action=(outport = "_MC_unknown"; output;)
  table=28(ls_in_l2_unknown   ), priority=0    , match=(1), action=(output;)
Datapath: "neutron-a6370456-5bb9-4dde-8b73-3fe5ba0c414a" aka "provider" (a69f8cd6-1f9d-48b5-a8e3-30a5edd951fe)  Pipeline: egress
  table=0 (ls_out_pre_acl     ), priority=110  , match=(eth.src == $svc_monitor_mac), action=(next;)
  table=0 (ls_out_pre_acl     ), priority=0    , match=(1), action=(next;)
  table=1 (ls_out_pre_lb      ), priority=110  , match=(eth.mcast), action=(next;)
  table=1 (ls_out_pre_lb      ), priority=110  , match=(eth.src == $svc_monitor_mac), action=(next;)
  table=1 (ls_out_pre_lb      ), priority=110  , match=(ip && outport == "provnet-ac334841-8877-453f-91de-38264a45b3bf"), action=(ct_clear; next;)
  table=1 (ls_out_pre_lb      ), priority=110  , match=(nd || nd_rs || nd_ra || mldv1 || mldv2), action=(next;)
  table=1 (ls_out_pre_lb      ), priority=110  , match=(reg0[16] == 1), action=(next;)
  table=1 (ls_out_pre_lb      ), priority=0    , match=(1), action=(next;)
  table=2 (ls_out_pre_stateful), priority=110  , match=(reg0[2] == 1), action=(ct_lb_mark;)
  table=2 (ls_out_pre_stateful), priority=100  , match=(reg0[0] == 1), action=(ct_next;)
  table=2 (ls_out_pre_stateful), priority=0    , match=(1), action=(next;)
  table=3 (ls_out_acl_hint    ), priority=65535, match=(1), action=(next;)
  table=4 (ls_out_acl_eval    ), priority=65535, match=(1), action=(next;)
  table=4 (ls_out_acl_eval    ), priority=65532, match=(nd || nd_ra || nd_rs || mldv1 || mldv2), action=(reg8[16] = 1; next;)
  table=5 (ls_out_acl_action  ), priority=0    , match=(1), action=(next;)
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

ポートを確認する。

```sh
ovn-sbctl list Port_Binding
```

```text
_uuid               : 479775c9-8632-4e1c-b6c2-5562759e4fb5
additional_chassis  : []
additional_encap    : []
chassis             : []
datapath            : a69f8cd6-1f9d-48b5-a8e3-30a5edd951fe
encap               : []
external_ids        : {"neutron:cidrs"="172.16.0.100/24", "neutron:device_id"=ovnmeta-a6370456-5bb9-4dde-8b73-3fe5ba0c414a, "neutron:device_owner"="network:distributed", "neutron:mtu"="", "neutron:network_name"=neutron-a6370456-5bb9-4dde-8b73-3fe5ba0c414a, "neutron:port_capabilities"="", "neutron:port_name"="", "neutron:project_id"=be94f4411bd74f249f5e25f642209b82, "neutron:revision_number"="2", "neutron:security_group_ids"="", "neutron:subnet_pool_addr_scope4"="", "neutron:subnet_pool_addr_scope6"="", "neutron:vnic_type"=normal}
gateway_chassis     : []
ha_chassis_group    : []
logical_port        : "465fdd06-6fa7-4fb9-8d66-35595337ff6a"
mac                 : ["fa:16:3e:61:51:6f 172.16.0.100"]
mirror_rules        : []
nat_addresses       : []
options             : {}
parent_port         : []
port_security       : []
requested_additional_chassis: []
requested_chassis   : []
tag                 : []
tunnel_key          : 2
type                : localport
up                  : false
virtual_parent      : []

_uuid               : 9e39cafe-0eb6-461b-92c2-e7f699e9963b
additional_chassis  : []
additional_encap    : []
chassis             : []
datapath            : a69f8cd6-1f9d-48b5-a8e3-30a5edd951fe
encap               : []
external_ids        : {}
gateway_chassis     : []
ha_chassis_group    : []
logical_port        : provnet-ac334841-8877-453f-91de-38264a45b3bf
mac                 : [unknown]
mirror_rules        : []
nat_addresses       : []
options             : {localnet_learn_fdb="false", mcast_flood="false", mcast_flood_reports="true", network_name=provider}
parent_port         : []
port_security       : []
requested_additional_chassis: []
requested_chassis   : []
tag                 : []
tunnel_key          : 1
type                : localnet
up                  : false
virtual_parent      : []
```

#### Open vSwitch

ブリッジを確認する。

```sh
ovs-vsctl show
```

```text
9869c49f-a522-47dc-a744-bf85afff8b76
    Bridge br-int
        fail_mode: secure
        datapath_type: system
        Port ovn-32a2f4-0
            Interface ovn-32a2f4-0
                type: geneve
                options: {csum="true", key=flow, remote_ip="172.16.0.31"}
        Port br-int
            Interface br-int
                type: internal
    Bridge br-provider
        Port eth2
            Interface eth2
                type: system
    Bridge br-mgmt
        Port eth3
            Interface eth3
                type: system
    ovs_version: "3.3.1"
```

データパスを確認する。

```sh
ovs-dpctl show
```

```text
system@ovs-system:
  lookups: hit:702 missed:318 lost:0
  flows: 0
  masks: hit:1091 total:0 hit/pkt:1.07
  cache: hit:405 hit-rate:39.71%
  caches:
    masks-cache: size:256
  port 0: ovs-system (internal)
  port 1: br-int (internal)
  port 2: eth2
  port 3: eth3
  port 4: genev_sys_6081 (geneve: packet_type=ptap)
```

ブリッジ br-int のフローのエントリを確認する。

```sh
ovs-ofctl dump-flows br-int
```

```text
 cookie=0x0, duration=6026.434s, table=0, n_packets=0, n_bytes=0, priority=120,icmp6,in_port="ovn-32a2f4-0",icmp_type=2,icmp_code=0 actions=move:NXM_NX_TUN_ID[0..23]->OXM_OF_METADATA[0..23],move:NXM_NX_TUN_METADATA0[16..30]->NXM_NX_REG14[0..14],move:NXM_NX_TUN_METADATA0[0..15]->NXM_NX_REG15[0..15],resubmit(,40),load:0x1->NXM_NX_REG10[16],resubmit(,8)
 cookie=0x0, duration=6026.434s, table=0, n_packets=0, n_bytes=0, priority=120,icmp,in_port="ovn-32a2f4-0",icmp_type=3,icmp_code=4 actions=move:NXM_NX_TUN_ID[0..23]->OXM_OF_METADATA[0..23],move:NXM_NX_TUN_METADATA0[16..30]->NXM_NX_REG14[0..14],move:NXM_NX_TUN_METADATA0[0..15]->NXM_NX_REG15[0..15],resubmit(,40),load:0x1->NXM_NX_REG10[16],resubmit(,8)
 cookie=0x0, duration=6026.434s, table=0, n_packets=0, n_bytes=0, priority=100,in_port="ovn-32a2f4-0" actions=move:NXM_NX_TUN_ID[0..23]->OXM_OF_METADATA[0..23],move:NXM_NX_TUN_METADATA0[16..30]->NXM_NX_REG14[0..14],move:NXM_NX_TUN_METADATA0[0..15]->NXM_NX_REG15[0..15],resubmit(,40)
 cookie=0x0, duration=7507.217s, table=0, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x0, duration=7507.217s, table=37, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,39)
 cookie=0x0, duration=7507.217s, table=38, n_packets=0, n_bytes=0, priority=10,reg10=0x1/0x1 actions=resubmit(,40)
 cookie=0x0, duration=7507.217s, table=38, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,39)
 cookie=0x0, duration=7507.217s, table=39, n_packets=0, n_bytes=0, priority=150,reg10=0x10/0x10 actions=resubmit(,40)
 cookie=0x0, duration=7507.217s, table=39, n_packets=0, n_bytes=0, priority=150,reg10=0x2/0x3 actions=resubmit(,40)
 cookie=0x0, duration=7507.217s, table=39, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,40)
 cookie=0x0, duration=7507.217s, table=40, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x0, duration=7507.217s, table=41, n_packets=0, n_bytes=0, priority=0 actions=load:0->NXM_NX_REG0[],load:0->NXM_NX_REG1[],load:0->NXM_NX_REG2[],load:0->NXM_NX_REG3[],load:0->NXM_NX_REG4[],load:0->NXM_NX_REG5[],load:0->NXM_NX_REG6[],load:0->NXM_NX_REG7[],load:0->NXM_NX_REG8[],load:0->NXM_NX_REG9[],resubmit(,42)
 cookie=0x0, duration=7507.217s, table=64, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,65)
 cookie=0x0, duration=7507.217s, table=65, n_packets=0, n_bytes=0, priority=0 actions=drop
```

ブリッジ br-provider のフローのエントリを確認する。

```sh
ovs-ofctl dump-flows br-provider
```

```text
 cookie=0x0, duration=7516.579s, table=0, n_packets=494, n_bytes=88698, priority=0 actions=NORMAL
```

ブリッジ br-mgmt のフローのエントリを確認する。

```sh
ovs-ofctl dump-flows br-mgmt
```

```text
 cookie=0x0, duration=7519.684s, table=0, n_packets=534, n_bytes=93270, priority=0 actions=NORMAL
```

トンネルを確認する。

```sh
ovs-appctl ofproto/list-tunnels
```

```text
port 4: ovn-32a2f4-0 (geneve: ::->172.16.0.31, key=flow, legacy_l2, dp port=4, ttl=64, csum=true)
```

### Compute Node

#### ネットワーク名前空間

ネットワーク名前空間は作成されない。

#### Open vSwitch

ブリッジの構成を確認する。

```sh
ovs-vsctl show
```

```text
46423187-0689-4323-afe2-2ad6d689596b
    Bridge br-mgmt
        Port eth3
            Interface eth3
                type: system
    Bridge br-int
        fail_mode: secure
        datapath_type: system
        Port ovn-3732c7-0
            Interface ovn-3732c7-0
                type: geneve
                options: {csum="true", key=flow, remote_ip="172.16.0.11"}
        Port br-int
            Interface br-int
                type: internal
    Bridge br-provider
        Port eth2
            Interface eth2
                type: system
    ovs_version: "3.3.1"
```

データパスを確認する。

```sh
ovs-dpctl show
```

```text
system@ovs-system:
  lookups: hit:430 missed:140 lost:0
  flows: 0
  masks: hit:472 total:0 hit/pkt:0.83
  cache: hit:287 hit-rate:50.35%
  caches:
    masks-cache: size:256
  port 0: ovs-system (internal)
  port 1: br-int (internal)
  port 2: genev_sys_6081 (geneve: packet_type=ptap)
  port 3: eth2
  port 4: eth3
```

ブリッジ br-int のフローのエントリを確認する。

```sh
ovs-ofctl dump-flows br-int
```

```text
 cookie=0x0, duration=6125.341s, table=0, n_packets=0, n_bytes=0, priority=120,icmp6,in_port="ovn-3732c7-0",icmp_type=2,icmp_code=0 actions=move:NXM_NX_TUN_ID[0..23]->OXM_OF_METADATA[0..23],move:NXM_NX_TUN_METADATA0[16..30]->NXM_NX_REG14[0..14],move:NXM_NX_TUN_METADATA0[0..15]->NXM_NX_REG15[0..15],resubmit(,40),load:0x1->NXM_NX_REG10[16],resubmit(,8)
 cookie=0x0, duration=6125.341s, table=0, n_packets=0, n_bytes=0, priority=120,icmp,in_port="ovn-3732c7-0",icmp_type=3,icmp_code=4 actions=move:NXM_NX_TUN_ID[0..23]->OXM_OF_METADATA[0..23],move:NXM_NX_TUN_METADATA0[16..30]->NXM_NX_REG14[0..14],move:NXM_NX_TUN_METADATA0[0..15]->NXM_NX_REG15[0..15],resubmit(,40),load:0x1->NXM_NX_REG10[16],resubmit(,8)
 cookie=0x0, duration=6125.341s, table=0, n_packets=0, n_bytes=0, priority=100,in_port="ovn-3732c7-0" actions=move:NXM_NX_TUN_ID[0..23]->OXM_OF_METADATA[0..23],move:NXM_NX_TUN_METADATA0[16..30]->NXM_NX_REG14[0..14],move:NXM_NX_TUN_METADATA0[0..15]->NXM_NX_REG15[0..15],resubmit(,40)
 cookie=0x0, duration=6125.341s, table=0, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x0, duration=6125.341s, table=37, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,39)
 cookie=0x0, duration=6125.341s, table=38, n_packets=0, n_bytes=0, priority=10,reg10=0x1/0x1 actions=resubmit(,40)
 cookie=0x0, duration=6125.341s, table=38, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,39)
 cookie=0x0, duration=6125.341s, table=39, n_packets=0, n_bytes=0, priority=150,reg10=0x10/0x10 actions=resubmit(,40)
 cookie=0x0, duration=6125.341s, table=39, n_packets=0, n_bytes=0, priority=150,reg10=0x2/0x3 actions=resubmit(,40)
 cookie=0x0, duration=6125.341s, table=39, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,40)
 cookie=0x0, duration=6125.341s, table=40, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x0, duration=6125.341s, table=41, n_packets=0, n_bytes=0, priority=0 actions=load:0->NXM_NX_REG0[],load:0->NXM_NX_REG1[],load:0->NXM_NX_REG2[],load:0->NXM_NX_REG3[],load:0->NXM_NX_REG4[],load:0->NXM_NX_REG5[],load:0->NXM_NX_REG6[],load:0->NXM_NX_REG7[],load:0->NXM_NX_REG8[],load:0->NXM_NX_REG9[],resubmit(,42)
 cookie=0x0, duration=6125.341s, table=64, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,65)
 cookie=0x0, duration=6125.341s, table=65, n_packets=0, n_bytes=0, priority=0 actions=drop
```

ブリッジ br-provider のフローのエントリを確認する。

```sh
ovs-ofctl dump-flows br-provider
```

```text
 cookie=0x0, duration=6132.919s, table=0, n_packets=268, n_bytes=49471, priority=0 actions=NORMAL
```

ブリッジ br-mgmt のフローのエントリを確認する。

```sh
ovs-ofctl dump-flows br-mgmt
```

```text
 cookie=0x0, duration=6142.187s, table=0, n_packets=309, n_bytes=54308, priority=0 actions=NORMAL
```

トンネルを確認する。

```sh
ovs-appctl ofproto/list-tunnels
```

```text
port 2: ovn-3732c7-0 (geneve: ::->172.16.0.11, key=flow, legacy_l2, dp port=2, ttl=64, csum=true)
```
