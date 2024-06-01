# vlan ネットワーク (Open Virtual Network)

Open Viirtual Network を利用した vlan ネットワークを作成する。

## 前提条件

* [](../../installation/controller/neutron_ovn/vlan) を設定していること。

## 外部ネットワークの作成

eth0 に繋がる外部ネットワークに vlan ネットワークを作成する。

| オプション                  | 説明                         |
| --------------------------- | ---------------------------- |
| --provider-segment          | VLAN ID                      |

```sh
openstack network create \
    --share \
    --external \
    --provider-physical-network provider \
    --provider-network-type vlan \
    --provider-segment 100 \
    provider-100
```

```
+---------------------------+--------------------------------------+
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| admin_state_up            | UP                                   |
| availability_zone_hints   |                                      |
| availability_zones        |                                      |
| created_at                | 2024-05-25T23:31:46Z                 |
| description               |                                      |
| dns_domain                | None                                 |
| id                        | 626d80b2-c036-4dc4-90df-954fb2591869 |
| ipv4_address_scope        | None                                 |
| ipv6_address_scope        | None                                 |
| is_default                | False                                |
| is_vlan_transparent       | None                                 |
| mtu                       | 1500                                 |
| name                      | provider-100                         |
| port_security_enabled     | True                                 |
| project_id                | be94f4411bd74f249f5e25f642209b82     |
| provider:network_type     | vlan                                 |
| provider:physical_network | provider                             |
| provider:segmentation_id  | 100                                  |
| qos_policy_id             | None                                 |
| revision_number           | 1                                    |
| router:external           | External                             |
| segments                  | None                                 |
| shared                    | True                                 |
| status                    | ACTIVE                               |
| subnets                   |                                      |
| tags                      |                                      |
| tenant_id                 | be94f4411bd74f249f5e25f642209b82     |
| updated_at                | 2024-05-25T23:31:46Z                 |
+---------------------------+--------------------------------------+
```

メタデータサービスのポートを確認する。

```sh
openstack port list
```

```
+--------------------------------------+------+-------------------+--------------------+--------+
| ID                                   | Name | MAC Address       | Fixed IP Addresses | Status |
+--------------------------------------+------+-------------------+--------------------+--------+
| fddff5b2-0987-4644-bae9-b915f9b572aa |      | fa:16:3e:1e:cf:67 |                    | DOWN   |
+--------------------------------------+------+-------------------+--------------------+--------+
```

```sh
openstack port show fddff5b2-0987-4644-bae9-b915f9b572aa
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
| created_at              | 2024-05-25T23:31:47Z                         |
| data_plane_status       | None                                         |
| description             |                                              |
| device_id               | ovnmeta-626d80b2-c036-4dc4-90df-954fb2591869 |
| device_owner            | network:distributed                          |
| device_profile          | None                                         |
| dns_assignment          | None                                         |
| dns_domain              | None                                         |
| dns_name                | None                                         |
| extra_dhcp_opts         |                                              |
| fixed_ips               |                                              |
| hardware_offload_type   | None                                         |
| hints                   |                                              |
| id                      | fddff5b2-0987-4644-bae9-b915f9b572aa         |
| ip_allocation           | None                                         |
| mac_address             | fa:16:3e:1e:cf:67                            |
| name                    |                                              |
| network_id              | 626d80b2-c036-4dc4-90df-954fb2591869         |
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
| updated_at              | 2024-05-25T23:31:47Z                         |
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
_uuid               : 22f3af2e-bd51-4193-a67f-4cc600ab6070
acls                : []
copp                : []
dns_records         : []
external_ids        : {"neutron:availability_zone_hints"="", "neutron:mtu"="1500", "neutron:network_name"=provider-100, "neutron:revision_number"="1"}
forwarding_groups   : []
load_balancer       : []
load_balancer_group : []
name                : neutron-626d80b2-c036-4dc4-90df-954fb2591869
other_config        : {fdb_age_threshold="0", mcast_flood_unregistered="false", mcast_snoop="false", vlan-passthru="false"}
ports               : [5e20e112-a7d4-438f-81ff-e6540effedd8, f196fd5b-b0b8-4b9d-be4c-8e93011ea20f]
qos_rules           : []
```

ポートを確認する。

```sh
ovn-nbctl list Logical_Switch_Port
```

```
_uuid               : 5e20e112-a7d4-438f-81ff-e6540effedd8
addresses           : ["fa:16:3e:1e:cf:67"]
dhcpv4_options      : []
dhcpv6_options      : []
dynamic_addresses   : []
enabled             : true
external_ids        : {"neutron:cidrs"="", "neutron:device_id"=ovnmeta-626d80b2-c036-4dc4-90df-954fb2591869, "neutron:device_owner"="network:distributed", "neutron:mtu"="", "neutron:network_name"=neutron-626d80b2-c036-4dc4-90df-954fb2591869, "neutron:port_capabilities"="", "neutron:port_name"="", "neutron:project_id"=be94f4411bd74f249f5e25f642209b82, "neutron:revision_number"="1", "neutron:security_group_ids"="", "neutron:subnet_pool_addr_scope4"="", "neutron:subnet_pool_addr_scope6"="", "neutron:vnic_type"=normal}
ha_chassis_group    : []
mirror_rules        : []
name                : "fddff5b2-0987-4644-bae9-b915f9b572aa"
options             : {}
parent_name         : []
port_security       : []
tag                 : []
tag_request         : []
type                : localport
up                  : false

_uuid               : f196fd5b-b0b8-4b9d-be4c-8e93011ea20f
addresses           : [unknown]
dhcpv4_options      : []
dhcpv6_options      : []
dynamic_addresses   : []
enabled             : []
external_ids        : {}
ha_chassis_group    : []
mirror_rules        : []
name                : provnet-ed5b0e1e-fa10-4dd3-ac28-afec8481da62
options             : {localnet_learn_fdb="false", mcast_flood="false", mcast_flood_reports="true", network_name=provider}
parent_name         : []
port_security       : []
tag                 : 100
tag_request         : []
type                : localnet
up                  : false
```

セキュリティグループの設定を確認する。

```sh
ovn-nbctl list ACL
```

```
_uuid               : 31bacda3-d31c-4739-8637-f30b4c1b47ac
action              : allow-related
direction           : from-lport
external_ids        : {"neutron:security_group_rule_id"="a0ec8f7c-0c17-496d-86c5-6bbd545f9b03"}
label               : 0
log                 : false
match               : "inport == @pg_158f2c45_1393_46e4_8093_6e82e4d876c9 && ip6"
meter               : []
name                : []
options             : {}
priority            : 1002
severity            : []
tier                : 0

_uuid               : a5e40279-0698-4d4e-82c0-349b3f5e368f
action              : allow-related
direction           : to-lport
external_ids        : {"neutron:security_group_rule_id"="6e1cb864-a5a8-49e9-ba76-4d47fb265ee2"}
label               : 0
log                 : false
match               : "outport == @pg_158f2c45_1393_46e4_8093_6e82e4d876c9 && ip4 && ip4.src == 0.0.0.0/0 && tcp && tcp.dst == 22"
meter               : []
name                : []
options             : {}
priority            : 1002
severity            : []
tier                : 0

_uuid               : 655423fa-e375-41de-b063-40507c2ef83d
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

_uuid               : 17be7dac-406b-4fb3-a036-238248cbcff7
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

_uuid               : 6af3a7fd-e126-4b94-8f13-35061fbc1291
action              : allow-related
direction           : from-lport
external_ids        : {"neutron:security_group_rule_id"="12dab7f7-5c98-436d-9ff1-fb9ab179f5c4"}
label               : 0
log                 : false
match               : "inport == @pg_158f2c45_1393_46e4_8093_6e82e4d876c9 && ip4"
meter               : []
name                : []
options             : {}
priority            : 1002
severity            : []
tier                : 0

_uuid               : 7b75e62e-e06f-4ea3-b4ad-6204168b58f8
action              : allow-related
direction           : to-lport
external_ids        : {"neutron:security_group_rule_id"="d84d5d71-ad24-4072-a5c1-fa950ac6fdc9"}
label               : 0
log                 : false
match               : "outport == @pg_158f2c45_1393_46e4_8093_6e82e4d876c9 && ip4 && ip4.src == 0.0.0.0/0 && icmp4"
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
Datapath: "neutron-626d80b2-c036-4dc4-90df-954fb2591869" aka "provider-100" (7869b546-90fb-40f4-adbc-026b46b278f4)  Pipeline: ingress
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
  table=5 (ls_in_pre_lb       ), priority=110  , match=(ip && inport == "provnet-ed5b0e1e-fa10-4dd3-ac28-afec8481da62"), action=(next;)
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
  table=21(ls_in_arp_rsp      ), priority=100  , match=(inport == "provnet-ed5b0e1e-fa10-4dd3-ac28-afec8481da62"), action=(next;)
  table=21(ls_in_arp_rsp      ), priority=0    , match=(1), action=(next;)
  table=22(ls_in_dhcp_options ), priority=0    , match=(1), action=(next;)
  table=23(ls_in_dhcp_response), priority=0    , match=(1), action=(next;)
  table=24(ls_in_dns_lookup   ), priority=0    , match=(1), action=(next;)
  table=25(ls_in_dns_response ), priority=0    , match=(1), action=(next;)
  table=26(ls_in_external_port), priority=0    , match=(1), action=(next;)
  table=27(ls_in_l2_lkup      ), priority=110  , match=(eth.dst == $svc_monitor_mac && (tcp || icmp || icmp6)), action=(handle_svc_check(inport);)
  table=27(ls_in_l2_lkup      ), priority=70   , match=(eth.mcast), action=(outport = "_MC_flood"; output;)
  table=27(ls_in_l2_lkup      ), priority=50   , match=(eth.dst == fa:16:3e:1e:cf:67), action=(outport = "fddff5b2-0987-4644-bae9-b915f9b572aa"; output;)
  table=27(ls_in_l2_lkup      ), priority=0    , match=(1), action=(outport = get_fdb(eth.dst); next;)
  table=28(ls_in_l2_unknown   ), priority=50   , match=(outport == "none"), action=(outport = "_MC_unknown"; output;)
  table=28(ls_in_l2_unknown   ), priority=0    , match=(1), action=(output;)
Datapath: "neutron-626d80b2-c036-4dc4-90df-954fb2591869" aka "provider-100" (7869b546-90fb-40f4-adbc-026b46b278f4)  Pipeline: egress
  table=0 (ls_out_pre_acl     ), priority=110  , match=(eth.src == $svc_monitor_mac), action=(next;)
  table=0 (ls_out_pre_acl     ), priority=0    , match=(1), action=(next;)
  table=1 (ls_out_pre_lb      ), priority=110  , match=(eth.mcast), action=(next;)
  table=1 (ls_out_pre_lb      ), priority=110  , match=(eth.src == $svc_monitor_mac), action=(next;)
  table=1 (ls_out_pre_lb      ), priority=110  , match=(ip && outport == "provnet-ed5b0e1e-fa10-4dd3-ac28-afec8481da62"), action=(ct_clear; next;)
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
_uuid               : 25e1c74a-7e38-48a3-b2c9-265048727cf7
datapath            : 7869b546-90fb-40f4-adbc-026b46b278f4
name                : _MC_mrouter_flood
ports               : [d56c6207-93d8-4f3b-873b-1589e983d6e4]
tunnel_key          : 32770

_uuid               : 93ab4c5b-87c3-43d2-a827-4e61a01f3605
datapath            : 7869b546-90fb-40f4-adbc-026b46b278f4
name                : _MC_flood_l2
ports               : [aa90ca59-9fb3-4be0-8315-fac94312915b, d56c6207-93d8-4f3b-873b-1589e983d6e4]
tunnel_key          : 32772

_uuid               : 179a43fc-33c3-4126-bde1-0debf133a4c8
datapath            : 7869b546-90fb-40f4-adbc-026b46b278f4
name                : _MC_unknown
ports               : [d56c6207-93d8-4f3b-873b-1589e983d6e4]
tunnel_key          : 32769

_uuid               : d1bdde16-1c51-4050-a3c6-d5ddab5cb4d8
datapath            : 7869b546-90fb-40f4-adbc-026b46b278f4
name                : _MC_flood
ports               : [aa90ca59-9fb3-4be0-8315-fac94312915b, d56c6207-93d8-4f3b-873b-1589e983d6e4]
tunnel_key          : 32768
```

データパスを確認する。

```sh
ovn-sbctl list Datapath_Binding
```

```
_uuid               : 7869b546-90fb-40f4-adbc-026b46b278f4
external_ids        : {logical-switch="22f3af2e-bd51-4193-a67f-4cc600ab6070", name=neutron-626d80b2-c036-4dc4-90df-954fb2591869, name2=provider-100}
load_balancers      : []
tunnel_key          : 2
```

ポートを確認する。

```sh
ovn-sbctl list Port_Binding
```

```
_uuid               : aa90ca59-9fb3-4be0-8315-fac94312915b
additional_chassis  : []
additional_encap    : []
chassis             : []
datapath            : 7869b546-90fb-40f4-adbc-026b46b278f4
encap               : []
external_ids        : {"neutron:cidrs"="", "neutron:device_id"=ovnmeta-626d80b2-c036-4dc4-90df-954fb2591869, "neutron:device_owner"="network:distributed", "neutron:mtu"="", "neutron:network_name"=neutron-626d80b2-c036-4dc4-90df-954fb2591869, "neutron:port_capabilities"="", "neutron:port_name"="", "neutron:project_id"=be94f4411bd74f249f5e25f642209b82, "neutron:revision_number"="1", "neutron:security_group_ids"="", "neutron:subnet_pool_addr_scope4"="", "neutron:subnet_pool_addr_scope6"="", "neutron:vnic_type"=normal}
gateway_chassis     : []
ha_chassis_group    : []
logical_port        : "fddff5b2-0987-4644-bae9-b915f9b572aa"
mac                 : ["fa:16:3e:1e:cf:67"]
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

_uuid               : d56c6207-93d8-4f3b-873b-1589e983d6e4
additional_chassis  : []
additional_encap    : []
chassis             : []
datapath            : 7869b546-90fb-40f4-adbc-026b46b278f4
encap               : []
external_ids        : {}
gateway_chassis     : []
ha_chassis_group    : []
logical_port        : provnet-ed5b0e1e-fa10-4dd3-ac28-afec8481da62
mac                 : [unknown]
mirror_rules        : []
nat_addresses       : []
options             : {localnet_learn_fdb="false", mcast_flood="false", mcast_flood_reports="true", network_name=provider}
parent_port         : []
port_security       : []
requested_additional_chassis: []
requested_chassis   : []
tag                 : 100
tunnel_key          : 1
type                : localnet
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
        Port tap7b43909d-8a
            Interface tap7b43909d-8a
                error: "could not open network device tap7b43909d-8a (No such device)"
        Port patch-br-int-to-provnet-698fbbf6-da6f-4d59-8259-60d87eb5e697
            Interface patch-br-int-to-provnet-698fbbf6-da6f-4d59-8259-60d87eb5e697
                type: patch
                options: {peer=patch-provnet-698fbbf6-da6f-4d59-8259-60d87eb5e697-to-br-int}
    Bridge br-provider
        Port eth2
            Interface eth2
                type: system
        Port patch-provnet-698fbbf6-da6f-4d59-8259-60d87eb5e697-to-br-int
            Interface patch-provnet-698fbbf6-da6f-4d59-8259-60d87eb5e697-to-br-int
                type: patch
                options: {peer=patch-br-int-to-provnet-698fbbf6-da6f-4d59-8259-60d87eb5e697}
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

```
system@ovs-system:
  lookups: hit:683 missed:434 lost:0
  flows: 0
  masks: hit:1460 total:0 hit/pkt:1.31
  cache: hit:389 hit-rate:34.83%
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
 cookie=0x0, duration=3467.200s, table=0, n_packets=548, n_bytes=95280, priority=0 actions=drop
 cookie=0x0, duration=3467.200s, table=37, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,39)
 cookie=0x0, duration=3467.200s, table=38, n_packets=0, n_bytes=0, priority=10,reg10=0x1/0x1 actions=resubmit(,40)
 cookie=0x0, duration=3467.200s, table=38, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,39)
 cookie=0x0, duration=3467.200s, table=39, n_packets=0, n_bytes=0, priority=150,reg10=0x2/0x3 actions=resubmit(,40)
 cookie=0x0, duration=3467.200s, table=39, n_packets=0, n_bytes=0, priority=150,reg10=0x10/0x10 actions=resubmit(,40)
 cookie=0x0, duration=3467.200s, table=39, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,40)
 cookie=0x0, duration=3467.200s, table=40, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x0, duration=3467.200s, table=41, n_packets=0, n_bytes=0, priority=0 actions=load:0->NXM_NX_REG0[],load:0->NXM_NX_REG1[],load:0->NXM_NX_REG2[],load:0->NXM_NX_REG3[],load:0->NXM_NX_REG4[],load:0->NXM_NX_REG5[],load:0->NXM_NX_REG6[],load:0->NXM_NX_REG7[],load:0->NXM_NX_REG8[],load:0->NXM_NX_REG9[],resubmit(,42)
 cookie=0x0, duration=3467.200s, table=64, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,65)
 cookie=0x0, duration=3467.200s, table=65, n_packets=0, n_bytes=0, priority=0 actions=drop
```

ブリッジ br-provider のフローのエントリを確認する。

```sh
ovs-ofctl dump-flows br-provider
```

```
 cookie=0x0, duration=3485.340s, table=0, n_packets=548, n_bytes=95280, priority=0 actions=NORMAL
```

ブリッジ br-mgmt のフローのエントリを確認する。

```sh
ovs-ofctl dump-flows br-mgmt
```

```
 cookie=0x0, duration=3494.157s, table=0, n_packets=569, n_bytes=97410, priority=0 actions=NORMAL
```

## サブネットの作成

サブネットを作成する。

```sh
openstack subnet create \
    --network provider-100 \
    --allocation-pool start=192.168.100.1,end=192.168.100.253 \
    --gateway 192.168.100.254 \
    --subnet-range 192.168.100.0/24 \
    provider-100
```

```
+----------------------+--------------------------------------+
| Field                | Value                                |
+----------------------+--------------------------------------+
| allocation_pools     | 192.168.100.1-192.168.100.253        |
| cidr                 | 192.168.100.0/24                     |
| created_at           | 2024-05-25T23:56:06Z                 |
| description          |                                      |
| dns_nameservers      |                                      |
| dns_publish_fixed_ip | None                                 |
| enable_dhcp          | True                                 |
| gateway_ip           | 192.168.100.254                      |
| host_routes          |                                      |
| id                   | f399a277-8df8-4d77-9b07-0858cdc4a36e |
| ip_version           | 4                                    |
| ipv6_address_mode    | None                                 |
| ipv6_ra_mode         | None                                 |
| name                 | provider-100                         |
| network_id           | 626d80b2-c036-4dc4-90df-954fb2591869 |
| project_id           | be94f4411bd74f249f5e25f642209b82     |
| revision_number      | 0                                    |
| segment_id           | None                                 |
| service_types        |                                      |
| subnetpool_id        | None                                 |
| tags                 |                                      |
| updated_at           | 2024-05-25T23:56:06Z                 |
+----------------------+--------------------------------------+
```

メタデータサービスのポートを確認する。

```sh
openstack port list
```

```
+--------------------------------------+------+-------------------+------------------------------------------------------------------------------+--------+
| ID                                   | Name | MAC Address       | Fixed IP Addresses                                                           | Status |
+--------------------------------------+------+-------------------+------------------------------------------------------------------------------+--------+
| fddff5b2-0987-4644-bae9-b915f9b572aa |      | fa:16:3e:1e:cf:67 | ip_address='192.168.100.1', subnet_id='f399a277-8df8-4d77-9b07-0858cdc4a36e' | DOWN   |
+--------------------------------------+------+-------------------+------------------------------------------------------------------------------+--------+
```

```sh
openstack port show fddff5b2-0987-4644-bae9-b915f9b572aa
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
| created_at              | 2024-05-25T23:31:47Z                                                         |
| data_plane_status       | None                                                                         |
| description             |                                                                              |
| device_id               | ovnmeta-626d80b2-c036-4dc4-90df-954fb2591869                                 |
| device_owner            | network:distributed                                                          |
| device_profile          | None                                                                         |
| dns_assignment          | None                                                                         |
| dns_domain              | None                                                                         |
| dns_name                | None                                                                         |
| extra_dhcp_opts         |                                                                              |
| fixed_ips               | ip_address='192.168.100.1', subnet_id='f399a277-8df8-4d77-9b07-0858cdc4a36e' |
| hardware_offload_type   | None                                                                         |
| hints                   |                                                                              |
| id                      | fddff5b2-0987-4644-bae9-b915f9b572aa                                         |
| ip_allocation           | None                                                                         |
| mac_address             | fa:16:3e:1e:cf:67                                                            |
| name                    |                                                                              |
| network_id              | 626d80b2-c036-4dc4-90df-954fb2591869                                         |
| numa_affinity_policy    | None                                                                         |
| port_security_enabled   | False                                                                        |
| project_id              | be94f4411bd74f249f5e25f642209b82                                             |
| propagate_uplink_status | None                                                                         |
| resource_request        | None                                                                         |
| revision_number         | 2                                                                            |
| qos_network_policy_id   | None                                                                         |
| qos_policy_id           | None                                                                         |
| security_group_ids      |                                                                              |
| status                  | DOWN                                                                         |
| tags                    |                                                                              |
| trunk_details           | None                                                                         |
| updated_at              | 2024-05-25T23:56:07Z                                                         |
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
_uuid               : 5e20e112-a7d4-438f-81ff-e6540effedd8
addresses           : ["fa:16:3e:1e:cf:67 192.168.100.1"]
dhcpv4_options      : []
dhcpv6_options      : []
dynamic_addresses   : []
enabled             : true
external_ids        : {"neutron:cidrs"="192.168.100.1/24", "neutron:device_id"=ovnmeta-626d80b2-c036-4dc4-90df-954fb2591869, "neutron:device_owner"="network:distributed", "neutron:mtu"="", "neutron:network_name"=neutron-626d80b2-c036-4dc4-90df-954fb2591869, "neutron:port_capabilities"="", "neutron:port_name"="", "neutron:project_id"=be94f4411bd74f249f5e25f642209b82, "neutron:revision_number"="2", "neutron:security_group_ids"="", "neutron:subnet_pool_addr_scope4"="", "neutron:subnet_pool_addr_scope6"="", "neutron:vnic_type"=normal}
ha_chassis_group    : []
mirror_rules        : []
name                : "fddff5b2-0987-4644-bae9-b915f9b572aa"
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
Datapath: "neutron-626d80b2-c036-4dc4-90df-954fb2591869" aka "provider-100" (7869b546-90fb-40f4-adbc-026b46b278f4)  Pipeline: ingress
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
  table=5 (ls_in_pre_lb       ), priority=110  , match=(ip && inport == "provnet-ed5b0e1e-fa10-4dd3-ac28-afec8481da62"), action=(next;)
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
  table=21(ls_in_arp_rsp      ), priority=100  , match=(arp.tpa == 192.168.100.1 && arp.op == 1 && inport == "fddff5b2-0987-4644-bae9-b915f9b572aa"), action=(next;)
  table=21(ls_in_arp_rsp      ), priority=100  , match=(inport == "provnet-ed5b0e1e-fa10-4dd3-ac28-afec8481da62"), action=(next;)
  table=21(ls_in_arp_rsp      ), priority=50   , match=(arp.tpa == 192.168.100.1 && arp.op == 1), action=(eth.dst = eth.src; eth.src = fa:16:3e:1e:cf:67; arp.op = 2; /* ARP reply */ arp.tha = arp.sha; arp.sha = fa:16:3e:1e:cf:67; arp.tpa = arp.spa; arp.spa = 192.168.100.1; outport = inport; flags.loopback = 1; output;)
  table=21(ls_in_arp_rsp      ), priority=0    , match=(1), action=(next;)
  table=22(ls_in_dhcp_options ), priority=0    , match=(1), action=(next;)
  table=23(ls_in_dhcp_response), priority=0    , match=(1), action=(next;)
  table=24(ls_in_dns_lookup   ), priority=0    , match=(1), action=(next;)
  table=25(ls_in_dns_response ), priority=0    , match=(1), action=(next;)
  table=26(ls_in_external_port), priority=0    , match=(1), action=(next;)
  table=27(ls_in_l2_lkup      ), priority=110  , match=(eth.dst == $svc_monitor_mac && (tcp || icmp || icmp6)), action=(handle_svc_check(inport);)
  table=27(ls_in_l2_lkup      ), priority=70   , match=(eth.mcast), action=(outport = "_MC_flood"; output;)
  table=27(ls_in_l2_lkup      ), priority=50   , match=(eth.dst == fa:16:3e:1e:cf:67), action=(outport = "fddff5b2-0987-4644-bae9-b915f9b572aa"; output;)
  table=27(ls_in_l2_lkup      ), priority=0    , match=(1), action=(outport = get_fdb(eth.dst); next;)
  table=28(ls_in_l2_unknown   ), priority=50   , match=(outport == "none"), action=(outport = "_MC_unknown"; output;)
  table=28(ls_in_l2_unknown   ), priority=0    , match=(1), action=(output;)
Datapath: "neutron-626d80b2-c036-4dc4-90df-954fb2591869" aka "provider-100" (7869b546-90fb-40f4-adbc-026b46b278f4)  Pipeline: egress
  table=0 (ls_out_pre_acl     ), priority=110  , match=(eth.src == $svc_monitor_mac), action=(next;)
  table=0 (ls_out_pre_acl     ), priority=0    , match=(1), action=(next;)
  table=1 (ls_out_pre_lb      ), priority=110  , match=(eth.mcast), action=(next;)
  table=1 (ls_out_pre_lb      ), priority=110  , match=(eth.src == $svc_monitor_mac), action=(next;)
  table=1 (ls_out_pre_lb      ), priority=110  , match=(ip && outport == "provnet-ed5b0e1e-fa10-4dd3-ac28-afec8481da62"), action=(ct_clear; next;)
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
_uuid               : aa90ca59-9fb3-4be0-8315-fac94312915b
additional_chassis  : []
additional_encap    : []
chassis             : []
datapath            : 7869b546-90fb-40f4-adbc-026b46b278f4
encap               : []
external_ids        : {"neutron:cidrs"="192.168.100.1/24", "neutron:device_id"=ovnmeta-626d80b2-c036-4dc4-90df-954fb2591869, "neutron:device_owner"="network:distributed", "neutron:mtu"="", "neutron:network_name"=neutron-626d80b2-c036-4dc4-90df-954fb2591869, "neutron:port_capabilities"="", "neutron:port_name"="", "neutron:project_id"=be94f4411bd74f249f5e25f642209b82, "neutron:revision_number"="2", "neutron:security_group_ids"="", "neutron:subnet_pool_addr_scope4"="", "neutron:subnet_pool_addr_scope6"="", "neutron:vnic_type"=normal}
gateway_chassis     : []
ha_chassis_group    : []
logical_port        : "fddff5b2-0987-4644-bae9-b915f9b572aa"
mac                 : ["fa:16:3e:1e:cf:67 192.168.100.1"]
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

_uuid               : d56c6207-93d8-4f3b-873b-1589e983d6e4
additional_chassis  : []
additional_encap    : []
chassis             : []
datapath            : 7869b546-90fb-40f4-adbc-026b46b278f4
encap               : []
external_ids        : {}
gateway_chassis     : []
ha_chassis_group    : []
logical_port        : provnet-ed5b0e1e-fa10-4dd3-ac28-afec8481da62
mac                 : [unknown]
mirror_rules        : []
nat_addresses       : []
options             : {localnet_learn_fdb="false", mcast_flood="false", mcast_flood_reports="true", network_name=provider}
parent_port         : []
port_security       : []
requested_additional_chassis: []
requested_chassis   : []
tag                 : 100
tunnel_key          : 1
type                : localnet
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
        Port tap7b43909d-8a
            Interface tap7b43909d-8a
                error: "could not open network device tap7b43909d-8a (No such device)"
        Port patch-br-int-to-provnet-698fbbf6-da6f-4d59-8259-60d87eb5e697
            Interface patch-br-int-to-provnet-698fbbf6-da6f-4d59-8259-60d87eb5e697
                type: patch
                options: {peer=patch-provnet-698fbbf6-da6f-4d59-8259-60d87eb5e697-to-br-int}
    Bridge br-provider
        Port eth2
            Interface eth2
                type: system
        Port patch-provnet-698fbbf6-da6f-4d59-8259-60d87eb5e697-to-br-int
            Interface patch-provnet-698fbbf6-da6f-4d59-8259-60d87eb5e697-to-br-int
                type: patch
                options: {peer=patch-br-int-to-provnet-698fbbf6-da6f-4d59-8259-60d87eb5e697}
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

```
system@ovs-system:
  lookups: hit:783 missed:494 lost:0
  flows: 0
  masks: hit:1651 total:0 hit/pkt:1.29
  cache: hit:448 hit-rate:35.08%
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
 cookie=0x0, duration=4205.731s, table=0, n_packets=628, n_bytes=109864, priority=0 actions=drop
 cookie=0x0, duration=4205.731s, table=37, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,39)
 cookie=0x0, duration=4205.731s, table=38, n_packets=0, n_bytes=0, priority=10,reg10=0x1/0x1 actions=resubmit(,40)
 cookie=0x0, duration=4205.731s, table=38, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,39)
 cookie=0x0, duration=4205.731s, table=39, n_packets=0, n_bytes=0, priority=150,reg10=0x2/0x3 actions=resubmit(,40)
 cookie=0x0, duration=4205.731s, table=39, n_packets=0, n_bytes=0, priority=150,reg10=0x10/0x10 actions=resubmit(,40)
 cookie=0x0, duration=4205.731s, table=39, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,40)
 cookie=0x0, duration=4205.731s, table=40, n_packets=0, n_bytes=0, priority=0 actions=drop
 cookie=0x0, duration=4205.731s, table=41, n_packets=0, n_bytes=0, priority=0 actions=load:0->NXM_NX_REG0[],load:0->NXM_NX_REG1[],load:0->NXM_NX_REG2[],load:0->NXM_NX_REG3[],load:0->NXM_NX_REG4[],load:0->NXM_NX_REG5[],load:0->NXM_NX_REG6[],load:0->NXM_NX_REG7[],load:0->NXM_NX_REG8[],load:0->NXM_NX_REG9[],resubmit(,42)
 cookie=0x0, duration=4205.731s, table=64, n_packets=0, n_bytes=0, priority=0 actions=resubmit(,65)
 cookie=0x0, duration=4205.731s, table=65, n_packets=0, n_bytes=0, priority=0 actions=drop
```

ブリッジ br-provider のフローのエントリを確認する。

```sh
ovs-ofctl dump-flows br-provider
```

```
 cookie=0x0, duration=4225.529s, table=0, n_packets=636, n_bytes=111032, priority=0 actions=NORMAL
```

ブリッジ br-mgmt のフローのエントリを確認する。

```sh
ovs-ofctl dump-flows br-mgmt
```

```
 cookie=0x0, duration=4233.485s, table=0, n_packets=660, n_bytes=113616, priority=0 actions=NORMAL
```
