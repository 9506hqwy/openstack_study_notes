# geneve ネットワーク (Open Virtual Network)

Open Virtual Network を利用した geneve ネットワークを作成する。

## セルフサービスネットワークの作成

```{tip}
myuser で実行
```

eth0 に繋がるセルフサービスネットワークとして geneve ネットワークを作成する。

```sh
openstack network create selfservice
```

```
+---------------------------+--------------------------------------+
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| admin_state_up            | UP                                   |
| availability_zone_hints   |                                      |
| availability_zones        |                                      |
| created_at                | 2024-05-26T15:06:24Z                 |
| description               |                                      |
| dns_domain                | None                                 |
| id                        | f5e24ee8-2bcb-43a4-91d5-0abad0c58197 |
| ipv4_address_scope        | None                                 |
| ipv6_address_scope        | None                                 |
| is_default                | False                                |
| is_vlan_transparent       | None                                 |
| mtu                       | 1442                                 |
| name                      | selfservice                          |
| port_security_enabled     | True                                 |
| project_id                | bccf406c045d401b91ba5c7552a124ae     |
| provider:network_type     | geneve                               |
| provider:physical_network | None                                 |
| provider:segmentation_id  | 348                                  |
| qos_policy_id             | None                                 |
| revision_number           | 1                                    |
| router:external           | Internal                             |
| segments                  | None                                 |
| shared                    | False                                |
| status                    | ACTIVE                               |
| subnets                   |                                      |
| tags                      |                                      |
| tenant_id                 | bccf406c045d401b91ba5c7552a124ae     |
| updated_at                | 2024-05-26T15:06:24Z                 |
+---------------------------+--------------------------------------+
```

メタデータサービスのポートを確認する。

```sh
openstack port list --network selfservice
```

```
+--------------------------------------+------+-------------------+--------------------+--------+
| ID                                   | Name | MAC Address       | Fixed IP Addresses | Status |
+--------------------------------------+------+-------------------+--------------------+--------+
| 47a2be8a-c336-42e0-a8a6-044ff5df6bf2 |      | fa:16:3e:71:9e:ad |                    | DOWN   |
+--------------------------------------+------+-------------------+--------------------+--------+
```

```sh
openstack port show 47a2be8a-c336-42e0-a8a6-044ff5df6bf2
```

```
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
| created_at              | 2024-05-26T15:06:24Z                         |
| data_plane_status       | None                                         |
| description             |                                              |
| device_id               | ovnmeta-f5e24ee8-2bcb-43a4-91d5-0abad0c58197 |
| device_owner            | network:distributed                          |
| device_profile          | None                                         |
| dns_assignment          | None                                         |
| dns_domain              | None                                         |
| dns_name                | None                                         |
| extra_dhcp_opts         |                                              |
| fixed_ips               |                                              |
| hardware_offload_type   | None                                         |
| hints                   |                                              |
| id                      | 47a2be8a-c336-42e0-a8a6-044ff5df6bf2         |
| ip_allocation           | None                                         |
| mac_address             | fa:16:3e:71:9e:ad                            |
| name                    |                                              |
| network_id              | f5e24ee8-2bcb-43a4-91d5-0abad0c58197         |
| numa_affinity_policy    | None                                         |
| port_security_enabled   | False                                        |
| project_id              | bccf406c045d401b91ba5c7552a124ae             |
| propagate_uplink_status | None                                         |
| resource_request        | None                                         |
| revision_number         | 1                                            |
| qos_network_policy_id   | None                                         |
| qos_policy_id           | None                                         |
| security_group_ids      |                                              |
| status                  | DOWN                                         |
| tags                    |                                              |
| trunk_details           | None                                         |
| updated_at              | 2024-05-26T15:06:24Z                         |
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

```
_uuid               : 4618aa87-cb97-4480-b7d8-9e296ed6ead4
acls                : []
copp                : []
dns_records         : []
external_ids        : {"neutron:availability_zone_hints"="", "neutron:mtu"="1442", "neutron:network_name"=selfservice, "neutron:revision_number"="1"}
forwarding_groups   : []
load_balancer       : []
load_balancer_group : []
name                : neutron-f5e24ee8-2bcb-43a4-91d5-0abad0c58197
other_config        : {mcast_flood_unregistered="false", mcast_snoop="false", vlan-passthru="false"}
ports               : [bee7f632-da16-4b76-91b0-09b8379bc492]
qos_rules           : []
```

ポートを確認する。

```sh
ovn-nbctl list Logical_Switch_Port
```

```
_uuid               : bee7f632-da16-4b76-91b0-09b8379bc492
addresses           : ["fa:16:3e:71:9e:ad"]
dhcpv4_options      : []
dhcpv6_options      : []
dynamic_addresses   : []
enabled             : true
external_ids        : {"neutron:cidrs"="", "neutron:device_id"=ovnmeta-f5e24ee8-2bcb-43a4-91d5-0abad0c58197, "neutron:device_owner"="network:distributed", "neutron:mtu"="", "neutron:network_name"=neutron-f5e24ee8-2bcb-43a4-91d5-0abad0c58197, "neutron:port_capabilities"="", "neutron:port_name"="", "neutron:project_id"=bccf406c045d401b91ba5c7552a124ae, "neutron:revision_number"="1", "neutron:security_group_ids"="", "neutron:subnet_pool_addr_scope4"="", "neutron:subnet_pool_addr_scope6"="", "neutron:vnic_type"=normal}
ha_chassis_group    : []
mirror_rules        : []
name                : "47a2be8a-c336-42e0-a8a6-044ff5df6bf2"
options             : {}
parent_name         : []
port_security       : []
tag                 : []
tag_request         : []
type                : localport
up                  : false
```

セキュリティグループの設定を確認する。

```sh
ovn-nbctl list ACL
```

```
_uuid               : 5c253e91-9c9d-4ce6-840b-1db3d7e29015
action              : allow-related
direction           : to-lport
external_ids        : {"neutron:security_group_rule_id"="64a457e0-829a-4118-aae3-6d48a28b377f"}
label               : 0
log                 : false
match               : "outport == @pg_a1b42dfb_939f_49e6_bcd3_493e2e3edbc0 && ip6 && ip6.src == $pg_a1b42dfb_939f_49e6_bcd3_493e2e3edbc0_ip6"
meter               : []
name                : []
options             : {}
priority            : 1002
severity            : []
tier                : 0

_uuid               : 472a9f32-7b90-4552-a846-1a7662a35dc6
action              : allow-related
direction           : from-lport
external_ids        : {"neutron:security_group_rule_id"="f52115e6-ae60-4fd9-ab69-c856641c96cc"}
label               : 0
log                 : false
match               : "inport == @pg_dcf5b841_bca6_4d31_98b5_1e41d4bf4472 && ip4"
meter               : []
name                : []
options             : {}
priority            : 1002
severity            : []
tier                : 0

_uuid               : 3588d152-70fd-4789-b22d-5c248d02a20e
action              : allow-related
direction           : from-lport
external_ids        : {"neutron:security_group_rule_id"="4ff9dca5-ccd4-4eea-be42-bcd0b7f17060"}
label               : 0
log                 : false
match               : "inport == @pg_a1b42dfb_939f_49e6_bcd3_493e2e3edbc0 && ip6"
meter               : []
name                : []
options             : {}
priority            : 1002
severity            : []
tier                : 0

_uuid               : 054add72-4335-48e3-af9d-4ec6ccf7eeaf
action              : allow-related
direction           : to-lport
external_ids        : {"neutron:security_group_rule_id"="ef15382c-9918-4218-97e2-b7b64d985641"}
label               : 0
log                 : false
match               : "outport == @pg_20a04bf0_dc15_4813_941c_3bba803605e8 && ip4 && ip4.src == $pg_20a04bf0_dc15_4813_941c_3bba803605e8_ip4"
meter               : []
name                : []
options             : {}
priority            : 1002
severity            : []
tier                : 0

_uuid               : 380e1843-4c00-4549-bc99-8cd02cc4fe47
action              : allow-related
direction           : from-lport
external_ids        : {"neutron:security_group_rule_id"="386e33c0-3d7e-49fd-a168-3c244bd2ca2d"}
label               : 0
log                 : false
match               : "inport == @pg_dcf5b841_bca6_4d31_98b5_1e41d4bf4472 && ip6"
meter               : []
name                : []
options             : {}
priority            : 1002
severity            : []
tier                : 0

_uuid               : ba161ff7-0dd4-4ec5-95bc-39bcdf5de62f
action              : allow-related
direction           : from-lport
external_ids        : {"neutron:security_group_rule_id"="8a7930bf-fa4d-4501-bb07-b24e2aab5efa"}
label               : 0
log                 : false
match               : "inport == @pg_20a04bf0_dc15_4813_941c_3bba803605e8 && ip4"
meter               : []
name                : []
options             : {}
priority            : 1002
severity            : []
tier                : 0

_uuid               : 29e28c69-b2e9-46d5-8519-be84292c255d
action              : allow-related
direction           : to-lport
external_ids        : {"neutron:security_group_rule_id"="c8082036-fb98-47a4-af34-7e88aac07f79"}
label               : 0
log                 : false
match               : "outport == @pg_a1b42dfb_939f_49e6_bcd3_493e2e3edbc0 && ip4 && ip4.src == $pg_a1b42dfb_939f_49e6_bcd3_493e2e3edbc0_ip4"
meter               : []
name                : []
options             : {}
priority            : 1002
severity            : []
tier                : 0

_uuid               : a800a64a-93c6-4608-8a48-2142357adde4
action              : allow-related
direction           : to-lport
external_ids        : {"neutron:security_group_rule_id"="2d69318e-8455-4e9a-9c80-f5d0cfad0bc0"}
label               : 0
log                 : false
match               : "outport == @pg_dcf5b841_bca6_4d31_98b5_1e41d4bf4472 && ip6 && ip6.src == $pg_dcf5b841_bca6_4d31_98b5_1e41d4bf4472_ip6"
meter               : []
name                : []
options             : {}
priority            : 1002
severity            : []
tier                : 0

_uuid               : 89909a2b-d545-4026-b270-3b2bc9a948f9
action              : allow-related
direction           : from-lport
external_ids        : {"neutron:security_group_rule_id"="a2980711-49ad-4028-8cf8-37afd65fccd8"}
label               : 0
log                 : false
match               : "inport == @pg_20a04bf0_dc15_4813_941c_3bba803605e8 && ip6"
meter               : []
name                : []
options             : {}
priority            : 1002
severity            : []
tier                : 0

_uuid               : c8261cbc-473a-44c4-8950-4325ed3fc75a
action              : allow-related
direction           : to-lport
external_ids        : {"neutron:security_group_rule_id"="7d682bde-565b-4c97-b22f-1f8bb9f8e176"}
label               : 0
log                 : false
match               : "outport == @pg_20a04bf0_dc15_4813_941c_3bba803605e8 && ip6 && ip6.src == $pg_20a04bf0_dc15_4813_941c_3bba803605e8_ip6"
meter               : []
name                : []
options             : {}
priority            : 1002
severity            : []
tier                : 0

_uuid               : f3a5a37c-ea4a-4d45-bebd-bf8e1795a502
action              : allow-related
direction           : from-lport
external_ids        : {"neutron:security_group_rule_id"="4a672c1f-fa4a-4623-95f6-7a390b2fb0b2"}
label               : 0
log                 : false
match               : "inport == @pg_a1b42dfb_939f_49e6_bcd3_493e2e3edbc0 && ip4"
meter               : []
name                : []
options             : {}
priority            : 1002
severity            : []
tier                : 0

_uuid               : 633a039c-a369-4cae-b6ce-bc96e0596136
action              : allow-related
direction           : to-lport
external_ids        : {"neutron:security_group_rule_id"="f400ed5b-d2cd-4179-bad7-b6099640aa72"}
label               : 0
log                 : false
match               : "outport == @pg_dcf5b841_bca6_4d31_98b5_1e41d4bf4472 && ip4 && ip4.src == $pg_dcf5b841_bca6_4d31_98b5_1e41d4bf4472_ip4"
meter               : []
name                : []
options             : {}
priority            : 1002
severity            : []
tier                : 0
```

#### Southbound データベース

フローを確認する。

```sh
ovn-sbctl lflow-list
```

```
Datapath: "neutron-f5e24ee8-2bcb-43a4-91d5-0abad0c58197" aka "selfservice" (a6c6f842-4aea-48ea-b485-1a363553a876)  Pipeline: ingress
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
  table=21(ls_in_arp_rsp      ), priority=0    , match=(1), action=(next;)
  table=22(ls_in_dhcp_options ), priority=0    , match=(1), action=(next;)
  table=23(ls_in_dhcp_response), priority=0    , match=(1), action=(next;)
  table=24(ls_in_dns_lookup   ), priority=0    , match=(1), action=(next;)
  table=25(ls_in_dns_response ), priority=0    , match=(1), action=(next;)
  table=26(ls_in_external_port), priority=0    , match=(1), action=(next;)
  table=27(ls_in_l2_lkup      ), priority=110  , match=(eth.dst == $svc_monitor_mac && (tcp || icmp || icmp6)), action=(handle_svc_check(inport);)
  table=27(ls_in_l2_lkup      ), priority=70   , match=(eth.mcast), action=(outport = "_MC_flood"; output;)
  table=27(ls_in_l2_lkup      ), priority=50   , match=(eth.dst == fa:16:3e:71:9e:ad), action=(outport = "47a2be8a-c336-42e0-a8a6-044ff5df6bf2"; output;)
  table=27(ls_in_l2_lkup      ), priority=0    , match=(1), action=(outport = get_fdb(eth.dst); next;)
  table=28(ls_in_l2_unknown   ), priority=50   , match=(outport == "none"), action=(drop;)
  table=28(ls_in_l2_unknown   ), priority=0    , match=(1), action=(output;)
Datapath: "neutron-f5e24ee8-2bcb-43a4-91d5-0abad0c58197" aka "selfservice" (a6c6f842-4aea-48ea-b485-1a363553a876)  Pipeline: egress
  table=0 (ls_out_pre_acl     ), priority=110  , match=(eth.src == $svc_monitor_mac), action=(next;)
  table=0 (ls_out_pre_acl     ), priority=0    , match=(1), action=(next;)
  table=1 (ls_out_pre_lb      ), priority=110  , match=(eth.mcast), action=(next;)
  table=1 (ls_out_pre_lb      ), priority=110  , match=(eth.src == $svc_monitor_mac), action=(next;)
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

```
_uuid               : 7265e3e4-760f-460e-9a1a-78e54377f18d
datapath            : a6c6f842-4aea-48ea-b485-1a363553a876
name                : _MC_flood_l2
ports               : [9b5b3836-fae0-4746-9cbf-948916b44321]
tunnel_key          : 32772

_uuid               : 3418c045-c8c3-4639-a709-2e1eeb597490
datapath            : a6c6f842-4aea-48ea-b485-1a363553a876
name                : _MC_flood
ports               : [9b5b3836-fae0-4746-9cbf-948916b44321]
tunnel_key          : 32768
```

データパスを確認する。

```sh
ovn-sbctl list Datapath_Binding
```

```
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
_uuid               : 9b5b3836-fae0-4746-9cbf-948916b44321
additional_chassis  : []
additional_encap    : []
chassis             : []
datapath            : a6c6f842-4aea-48ea-b485-1a363553a876
encap               : []
external_ids        : {"neutron:cidrs"="", "neutron:device_id"=ovnmeta-f5e24ee8-2bcb-43a4-91d5-0abad0c58197, "neutron:device_owner"="network:distributed", "neutron:mtu"="", "neutron:network_name"=neutron-f5e24ee8-2bcb-43a4-91d5-0abad0c58197, "neutron:port_capabilities"="", "neutron:port_name"="", "neutron:project_id"=bccf406c045d401b91ba5c7552a124ae, "neutron:revision_number"="1", "neutron:security_group_ids"="", "neutron:subnet_pool_addr_scope4"="", "neutron:subnet_pool_addr_scope6"="", "neutron:vnic_type"=normal}
gateway_chassis     : []
ha_chassis_group    : []
logical_port        : "47a2be8a-c336-42e0-a8a6-044ff5df6bf2"
mac                 : ["fa:16:3e:71:9e:ad"]
mirror_rules        : []
nat_addresses       : []
options             : {}
parent_port         : []
port_security       : []
requested_additional_chassis: []
requested_chassis   : []
tag                 : []
tunnel_key          : 1
type                : localport
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
        Port patch-br-int-to-provnet-ed5b0e1e-fa10-4dd3-ac28-afec8481da62
            Interface patch-br-int-to-provnet-ed5b0e1e-fa10-4dd3-ac28-afec8481da62
                type: patch
                options: {peer=patch-provnet-ed5b0e1e-fa10-4dd3-ac28-afec8481da62-to-br-int}
    Bridge br-mgmt
        Port eth3
            Interface eth3
                type: system
    Bridge br-provider
        Port eth2
            Interface eth2
                type: system
        Port patch-provnet-ed5b0e1e-fa10-4dd3-ac28-afec8481da62-to-br-int
            Interface patch-provnet-ed5b0e1e-fa10-4dd3-ac28-afec8481da62-to-br-int
                type: patch
                options: {peer=patch-br-int-to-provnet-ed5b0e1e-fa10-4dd3-ac28-afec8481da62}
    ovs_version: "3.3.1"
```

データパスを確認する。

```sh
ovs-dpctl show
```

```
system@ovs-system:
  lookups: hit:887 missed:375 lost:0
  flows: 0
  masks: hit:1583 total:0 hit/pkt:1.25
  cache: hit:522 hit-rate:41.36%
  caches:
    masks-cache: size:256
  port 0: ovs-system (internal)
  port 1: eth2
  port 2: eth3
  port 3: br-int (internal)
```

ブリッジ br-int のフローのエントリを確認する。

```sh
ovs-ofctl dump-flows br-int
```

```
 cookie=0x0, duration=3050.248s, table=0, n_packets=595, n_bytes=89906, priority=0 actions=drop
 cookie=0x0, duration=3050.248s, table=37, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,39)
 cookie=0x0, duration=3050.248s, table=38, n_packets=0, n_bytes=0, priority=10,reg10=0x1/0x1 actions=resubmit(,40)
 cookie=0x0, duration=3050.248s, table=38, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,39)
 cookie=0x0, duration=3050.248s, table=39, n_packets=0, n_bytes=0, priority=150,reg10=0x2/0x3 actions=resubmit(,40)
 cookie=0x0, duration=3050.248s, table=39, n_packets=0, n_bytes=0, priority=150,reg10=0x10/0x10 actions=resubmit(,40)
 cookie=0x0, duration=3050.248s, table=39, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,40)
 cookie=0x0, duration=3050.248s, table=40, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x0, duration=3050.248s, table=41, n_packets=0, n_bytes=0, priority=0 actions=load:0->NXM_NX_REG0[],load:0->NXM_NX_REG1[],load:0->NXM_NX_REG2[],load:0->NXM_NX_REG3[],load:0->NXM_NX_REG4[],load:0->NXM_NX_REG5[],load:0->NXM_NX_REG6[],load:0->NXM_NX_REG7[],load:0->NXM_NX_REG8[],load:0->NXM_NX_REG9[],resubmit(,42)
 cookie=0x0, duration=3050.248s, table=64, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,65)
 cookie=0x0, duration=3050.248s, table=65, n_packets=0, n_bytes=0, priority=0 actions=drop
```

ブリッジ br-provider のフローのエントリを確認する。

```sh
ovs-ofctl dump-flows br-provider
```

```
 cookie=0x0, duration=3068.460s, table=0, n_packets=595, n_bytes=89906, priority=0 actions=NORMAL
```

ブリッジ br-mgmt のフローのエントリを確認する。

```sh
ovs-ofctl dump-flows br-mgmt
```

```
 cookie=0x0, duration=3079.440s, table=0, n_packets=671, n_bytes=98386, priority=0 actions=NORMAL
```

## サブネットの作成

サブネットを作成する。

```sh
openstack subnet create \
    --network selfservice \
    --gateway 192.168.101.254 \
    --subnet-range 192.168.101.0/24 \
    selfservice
```

```
+----------------------+--------------------------------------+
| Field                | Value                                |
+----------------------+--------------------------------------+
| allocation_pools     | 192.168.101.1-192.168.101.253        |
| cidr                 | 192.168.101.0/24                     |
| created_at           | 2024-05-26T15:29:45Z                 |
| description          |                                      |
| dns_nameservers      |                                      |
| dns_publish_fixed_ip | None                                 |
| enable_dhcp          | True                                 |
| gateway_ip           | 192.168.101.254                      |
| host_routes          |                                      |
| id                   | 27a80619-0344-4b90-9df3-3f8f4b10ec76 |
| ip_version           | 4                                    |
| ipv6_address_mode    | None                                 |
| ipv6_ra_mode         | None                                 |
| name                 | selfservice                          |
| network_id           | f5e24ee8-2bcb-43a4-91d5-0abad0c58197 |
| project_id           | bccf406c045d401b91ba5c7552a124ae     |
| revision_number      | 0                                    |
| segment_id           | None                                 |
| service_types        |                                      |
| subnetpool_id        | None                                 |
| tags                 |                                      |
| updated_at           | 2024-05-26T15:29:45Z                 |
+----------------------+--------------------------------------+
```

メタデータサービスのポートを確認する。

```sh
openstack port list --network selfservice
```

```
+--------------------------------------+------+-------------------+------------------------------------------------------------------------------+--------+
| ID                                   | Name | MAC Address       | Fixed IP Addresses                                                           | Status |
+--------------------------------------+------+-------------------+------------------------------------------------------------------------------+--------+
| 47a2be8a-c336-42e0-a8a6-044ff5df6bf2 |      | fa:16:3e:71:9e:ad | ip_address='192.168.101.1', subnet_id='27a80619-0344-4b90-9df3-3f8f4b10ec76' | DOWN   |
+--------------------------------------+------+-------------------+------------------------------------------------------------------------------+--------+
```

```sh
openstack port show 47a2be8a-c336-42e0-a8a6-044ff5df6bf2
```

```
+-------------------------+------------------------------------------------------------------------------+
| Field                   | Value                                                                        |
+-------------------------+------------------------------------------------------------------------------+
| admin_state_up          | UP                                                                           |
| allowed_address_pairs   |                                                                              |
| binding_host_id         |                                                                              |
| binding_profile         |                                                                              |
| binding_vif_details     |                                                                              |
| binding_vif_type        | unbound                                                                      |
| binding_vnic_type       | normal                                                                       |
| created_at              | 2024-05-26T15:06:24Z                                                         |
| data_plane_status       | None                                                                         |
| description             |                                                                              |
| device_id               | ovnmeta-f5e24ee8-2bcb-43a4-91d5-0abad0c58197                                 |
| device_owner            | network:distributed                                                          |
| device_profile          | None                                                                         |
| dns_assignment          | None                                                                         |
| dns_domain              | None                                                                         |
| dns_name                | None                                                                         |
| extra_dhcp_opts         |                                                                              |
| fixed_ips               | ip_address='192.168.101.1', subnet_id='27a80619-0344-4b90-9df3-3f8f4b10ec76' |
| hardware_offload_type   | None                                                                         |
| hints                   |                                                                              |
| id                      | 47a2be8a-c336-42e0-a8a6-044ff5df6bf2                                         |
| ip_allocation           | None                                                                         |
| mac_address             | fa:16:3e:71:9e:ad                                                            |
| name                    |                                                                              |
| network_id              | f5e24ee8-2bcb-43a4-91d5-0abad0c58197                                         |
| numa_affinity_policy    | None                                                                         |
| port_security_enabled   | False                                                                        |
| project_id              | bccf406c045d401b91ba5c7552a124ae                                             |
| propagate_uplink_status | None                                                                         |
| resource_request        | None                                                                         |
| revision_number         | 2                                                                            |
| qos_network_policy_id   | None                                                                         |
| qos_policy_id           | None                                                                         |
| security_group_ids      |                                                                              |
| status                  | DOWN                                                                         |
| tags                    |                                                                              |
| trunk_details           | None                                                                         |
| updated_at              | 2024-05-26T15:29:46Z                                                         |
+-------------------------+------------------------------------------------------------------------------+
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

```
_uuid               : bee7f632-da16-4b76-91b0-09b8379bc492
addresses           : ["fa:16:3e:71:9e:ad 192.168.101.1"]
dhcpv4_options      : []
dhcpv6_options      : []
dynamic_addresses   : []
enabled             : true
external_ids        : {"neutron:cidrs"="192.168.101.1/24", "neutron:device_id"=ovnmeta-f5e24ee8-2bcb-43a4-91d5-0abad0c58197, "neutron:device_owner"="network:distributed", "neutron:mtu"="", "neutron:network_name"=neutron-f5e24ee8-2bcb-43a4-91d5-0abad0c58197, "neutron:port_capabilities"="", "neutron:port_name"="", "neutron:project_id"=bccf406c045d401b91ba5c7552a124ae, "neutron:revision_number"="2", "neutron:security_group_ids"="", "neutron:subnet_pool_addr_scope4"="", "neutron:subnet_pool_addr_scope6"="", "neutron:vnic_type"=normal}
ha_chassis_group    : []
mirror_rules        : []
name                : "47a2be8a-c336-42e0-a8a6-044ff5df6bf2"
options             : {}
parent_name         : []
port_security       : []
tag                 : []
tag_request         : []
type                : localport
up                  : false
```

#### Southbound データベース

フローを確認する。

```sh
ovn-sbctl lflow-list
```

```
Datapath: "neutron-f5e24ee8-2bcb-43a4-91d5-0abad0c58197" aka "selfservice" (a6c6f842-4aea-48ea-b485-1a363553a876)  Pipeline: ingress
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
  table=21(ls_in_arp_rsp      ), priority=100  , match=(arp.tpa == 192.168.101.1 && arp.op == 1 && inport == "47a2be8a-c336-42e0-a8a6-044ff5df6bf2"), action=(next;)
  table=21(ls_in_arp_rsp      ), priority=50   , match=(arp.tpa == 192.168.101.1 && arp.op == 1), action=(eth.dst = eth.src; eth.src = fa:16:3e:71:9e:ad; arp.op = 2; /* ARP reply */ arp.tha = arp.sha; arp.sha = fa:16:3e:71:9e:ad; arp.tpa = arp.spa; arp.spa = 192.168.101.1; outport = inport; flags.loopback = 1; output;)
  table=21(ls_in_arp_rsp      ), priority=0    , match=(1), action=(next;)
  table=22(ls_in_dhcp_options ), priority=0    , match=(1), action=(next;)
  table=23(ls_in_dhcp_response), priority=0    , match=(1), action=(next;)
  table=24(ls_in_dns_lookup   ), priority=0    , match=(1), action=(next;)
  table=25(ls_in_dns_response ), priority=0    , match=(1), action=(next;)
  table=26(ls_in_external_port), priority=0    , match=(1), action=(next;)
  table=27(ls_in_l2_lkup      ), priority=110  , match=(eth.dst == $svc_monitor_mac && (tcp || icmp || icmp6)), action=(handle_svc_check(inport);)
  table=27(ls_in_l2_lkup      ), priority=70   , match=(eth.mcast), action=(outport = "_MC_flood"; output;)
  table=27(ls_in_l2_lkup      ), priority=50   , match=(eth.dst == fa:16:3e:71:9e:ad), action=(outport = "47a2be8a-c336-42e0-a8a6-044ff5df6bf2"; output;)
  table=27(ls_in_l2_lkup      ), priority=0    , match=(1), action=(outport = get_fdb(eth.dst); next;)
  table=28(ls_in_l2_unknown   ), priority=50   , match=(outport == "none"), action=(drop;)
  table=28(ls_in_l2_unknown   ), priority=0    , match=(1), action=(output;)
Datapath: "neutron-f5e24ee8-2bcb-43a4-91d5-0abad0c58197" aka "selfservice" (a6c6f842-4aea-48ea-b485-1a363553a876)  Pipeline: egress
  table=0 (ls_out_pre_acl     ), priority=110  , match=(eth.src == $svc_monitor_mac), action=(next;)
  table=0 (ls_out_pre_acl     ), priority=0    , match=(1), action=(next;)
  table=1 (ls_out_pre_lb      ), priority=110  , match=(eth.mcast), action=(next;)
  table=1 (ls_out_pre_lb      ), priority=110  , match=(eth.src == $svc_monitor_mac), action=(next;)
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

```
_uuid               : 9b5b3836-fae0-4746-9cbf-948916b44321
additional_chassis  : []
additional_encap    : []
chassis             : []
datapath            : a6c6f842-4aea-48ea-b485-1a363553a876
encap               : []
external_ids        : {"neutron:cidrs"="192.168.101.1/24", "neutron:device_id"=ovnmeta-f5e24ee8-2bcb-43a4-91d5-0abad0c58197, "neutron:device_owner"="network:distributed", "neutron:mtu"="", "neutron:network_name"=neutron-f5e24ee8-2bcb-43a4-91d5-0abad0c58197, "neutron:port_capabilities"="", "neutron:port_name"="", "neutron:project_id"=bccf406c045d401b91ba5c7552a124ae, "neutron:revision_number"="2", "neutron:security_group_ids"="", "neutron:subnet_pool_addr_scope4"="", "neutron:subnet_pool_addr_scope6"="", "neutron:vnic_type"=normal}
gateway_chassis     : []
ha_chassis_group    : []
logical_port        : "47a2be8a-c336-42e0-a8a6-044ff5df6bf2"
mac                 : ["fa:16:3e:71:9e:ad 192.168.101.1"]
mirror_rules        : []
nat_addresses       : []
options             : {}
parent_port         : []
port_security       : []
requested_additional_chassis: []
requested_chassis   : []
tag                 : []
tunnel_key          : 1
type                : localport
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

### Open vSwitch

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
        Port patch-br-int-to-provnet-ed5b0e1e-fa10-4dd3-ac28-afec8481da62
            Interface patch-br-int-to-provnet-ed5b0e1e-fa10-4dd3-ac28-afec8481da62
                type: patch
                options: {peer=patch-provnet-ed5b0e1e-fa10-4dd3-ac28-afec8481da62-to-br-int}
    Bridge br-mgmt
        Port eth3
            Interface eth3
                type: system
    Bridge br-provider
        Port eth2
            Interface eth2
                type: system
        Port patch-provnet-ed5b0e1e-fa10-4dd3-ac28-afec8481da62-to-br-int
            Interface patch-provnet-ed5b0e1e-fa10-4dd3-ac28-afec8481da62-to-br-int
                type: patch
                options: {peer=patch-br-int-to-provnet-ed5b0e1e-fa10-4dd3-ac28-afec8481da62}
    ovs_version: "3.3.1"
```

データパスを確認する。

```sh
ovs-dpctl show
```

```
system@ovs-system:
  lookups: hit:989 missed:441 lost:0
  flows: 6
  masks: hit:1783 total:2 hit/pkt:1.25
  cache: hit:584 hit-rate:40.84%
  caches:
    masks-cache: size:256
  port 0: ovs-system (internal)
  port 1: eth2
  port 2: eth3
  port 3: br-int (internal)
```

ブリッジ br-int のフローのエントリを確認する。

```sh
ovs-ofctl dump-flows br-int
```

```
 cookie=0x0, duration=3738.650s, table=0, n_packets=687, n_bytes=106442, priority=0 actions=drop
 cookie=0x0, duration=3738.650s, table=37, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,39)
 cookie=0x0, duration=3738.650s, table=38, n_packets=0, n_bytes=0, priority=10,reg10=0x1/0x1 actions=resubmit(,40)
 cookie=0x0, duration=3738.650s, table=38, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,39)
 cookie=0x0, duration=3738.650s, table=39, n_packets=0, n_bytes=0, priority=150,reg10=0x2/0x3 actions=resubmit(,40)
 cookie=0x0, duration=3738.650s, table=39, n_packets=0, n_bytes=0, priority=150,reg10=0x10/0x10 actions=resubmit(,40)
 cookie=0x0, duration=3738.650s, table=39, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,40)
 cookie=0x0, duration=3738.650s, table=40, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x0, duration=3738.650s, table=41, n_packets=0, n_bytes=0, priority=0 actions=load:0->NXM_NX_REG0[],load:0->NXM_NX_REG1[],load:0->NXM_NX_REG2[],load:0->NXM_NX_REG3[],load:0->NXM_NX_REG4[],load:0->NXM_NX_REG5[],load:0->NXM_NX_REG6[],load:0->NXM_NX_REG7[],load:0->NXM_NX_REG8[],load:0->NXM_NX_REG9[],resubmit(,42)
 cookie=0x0, duration=3738.650s, table=64, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,65)
 cookie=0x0, duration=3738.650s, table=65, n_packets=0, n_bytes=0, priority=0 actions=drop
```

ブリッジ br-provider のフローのエントリを確認する。

```sh
ovs-ofctl dump-flows br-provider
```

```
 cookie=0x0, duration=3758.946s, table=0, n_packets=689, n_bytes=106834, priority=0 actions=NORMAL
```

ブリッジ br-mgmt のフローのエントリを確認する。

```sh
ovs-ofctl dump-flows br-mgmt
```

```
 cookie=0x0, duration=3767.737s, table=0, n_packets=771, n_bytes=116006, priority=0 actions=NORMAL
```
