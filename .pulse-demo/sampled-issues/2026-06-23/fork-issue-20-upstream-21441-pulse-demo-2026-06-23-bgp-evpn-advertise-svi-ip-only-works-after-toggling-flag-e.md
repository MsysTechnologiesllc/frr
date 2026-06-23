# Pulse Demo Issue Artifact

Fork issue: https://github.com/MsysTechnologiesllc/frr/issues/20
Fork issue number: #20
Imported upstream issue: FRRouting/frr#21441
Seed date: 2026-06-23
Label: pulse-demo-seed

## Title

[Pulse demo][2026-06-23] bgp/evpn: advertise-svi-ip only works after toggling flag, evpn_set_advertise_svi_macip happens too soon (before Zebra connection is ready)

## Imported Issue Body

        Pulse demo seed imported this issue from `FRRouting/frr` on `2026-06-23`.

        Original issue: https://github.com/FRRouting/frr/issues/21441
        Original number: #21441
        Original author: @robinchrist
        Original labels: triage
        Original created: 2026-04-01T16:57:34Z
        Original updated: 2026-04-01T17:41:25Z

        ---

        ### Description

It appears like `evpn_set_advertise_svi_macip` is happening too soon, before the Zebra connection is ready.
`advertise-svi-ip` then only works after toggling the flag (off and then on)


### Version

```text
FRRouting 10.6.0 (infra-pve-muc-01) on Linux(6.17.13-2-pve).
Copyright 1996-2005 Kunihiro Ishiguro, et al.
configured with:
    '--build=x86_64-linux-gnu' '--prefix=/usr' '--includedir=${prefix}/include' '--mandir=${prefix}/share/man' '--infodir=${prefix}/share/info' '--sysconfdir=/etc' '--localstatedir=/var' '--disable-option-checking' '--disable-silent-rules' '--libdir=${prefix}/lib/x86_64-linux-gnu' '--runstatedir=/run' '--disable-maintainer-mode' '--sbindir=/usr/lib/frr' '--with-vtysh-pager=/usr/bin/pager' '--libdir=/usr/lib/x86_64-linux-gnu/frr' '--with-moduledir=/usr/lib/x86_64-linux-gnu/frr/modules' '--disable-dependency-tracking' '--enable-rpki' '--disable-scripting' '--enable-pim6d' '--disable-grpc' '--disable-address-sanitizer' '--with-libpam' '--enable-doc' '--enable-doc-html' '--enable-snmp' '--enable-fpm' '--disable-protobuf' '--disable-zeromq' '--enable-ospfapi' '--enable-bgp-vnc' '--enable-multipath=256' '--enable-pcre2posix' '--enable-user=frr' '--enable-group=frr' '--enable-vty-group=frrvty' '--enable-configfile-mask=0640' '--enable-logfile-mask=0640' 'build_alias=x86_64-linux-gnu' 'PYTHON=python3'
```

### How to reproduce

Configure multiple VRFs and EVPN, configure `advertise-svi-ip`


### Expected behavior

`advertise-svi-ip` should work after fresh FRR start

### Actual behavior

`advertise-svi-ip` only works after toggling off and on again

### Additional context

I added some debug logging:
```
2026/04/01 19:02:13 BGP: [YC02E-VF9N6] VRF Created: tvrfDEMO_100G01(4294967295)
2026/04/01 19:02:13 ZEBRA: [S680K-WDKR1] vrf tvrfDEMO_100G01 vni 15000002 ADD
2026/04/01 19:02:13 ZEBRA: [Q1QMJ-RDEZ4] zebra_vxlan_process_vrf_vni_cmd: l3vni 15000002 svi_if vrfDEMO_100G_l3 mac_vlan_if NIL
2026/04/01 19:02:13 ZEBRA: [S680K-WDKR1] vrf tvrfDEMO_25G01 vni 15000001 ADD
2026/04/01 19:02:13 ZEBRA: [Q1QMJ-RDEZ4] zebra_vxlan_process_vrf_vni_cmd: l3vni 15000001 svi_if vrfDEMO_25G_l3 mac_vlan_if NIL
2026/04/01 19:02:13 ZEBRA: [S680K-WDKR1] vrf tvrfDEMO_25GP01 vni 15000003 ADD
2026/04/01 19:02:13 ZEBRA: [Q1QMJ-RDEZ4] zebra_vxlan_process_vrf_vni_cmd: l3vni 15000003 svi_if vrfDEMO_25GP_l3 mac_vlan_if NIL
2026/04/01 19:02:13 BGP: [YC02E-VF9N6] VRF Created: tvrfDEMO_25G01(4294967295)
2026/04/01 19:02:13 BFD: [VTVCM-Y2NW3] Configuration Read in Took: 00:00:00
2026/04/01 19:02:13 BFD: [G6NKK-8C6DV] end_config: VTY:0x5ef2d8857fb0, pending SET-CFG: 0
2026/04/01 19:02:13 BGP: [YC02E-VF9N6] VRF Created: tvrfDEMO_25GP01(4294967295)
2026/04/01 19:02:13 BGP: [Z1RP6-0X3QJ] Creating Default VRF, AS 64505.1101
2026/04/01 19:02:13 BGP: [ZZKY3-FX5JH] bgp_get: Registering BGP instance VRF default to zebra
2026/04/01 19:02:13 BGP: [TBNSW-XXXBM] sendmsg_zebra_rnh: We have not connected yet, cannot send nexthops
2026/04/01 19:02:13 BGP: [HXW3G-K1M2A] sendmsg_zebra_rnh: sending cmd ZEBRA_NEXTHOP_REGISTER for 10.151.8.11/32 (vrf VRF default)
2026/04/01 19:02:13 BGP: [YTHK0-FSPPJ][EC 33554500] sendmsg_nexthop: zclient_send_message() failed
2026/04/01 19:02:13 BGP: [TBNSW-XXXBM] sendmsg_zebra_rnh: We have not connected yet, cannot send nexthops
2026/04/01 19:02:13 BGP: [HXW3G-K1M2A] sendmsg_zebra_rnh: sending cmd ZEBRA_NEXTHOP_REGISTER for 10.151.9.11/32 (vrf VRF default)
2026/04/01 19:02:13 BGP: [YTHK0-FSPPJ][EC 33554500] sendmsg_nexthop: zclient_send_message() failed
2026/04/01 19:02:13 BGP: [TBNSW-XXXBM] sendmsg_zebra_rnh: We have not connected yet, cannot send nexthops
2026/04/01 19:02:13 BGP: [HXW3G-K1M2A] sendmsg_zebra_rnh: sending cmd ZEBRA_NEXTHOP_REGISTER for 10.151.10.11/32 (vrf VRF default)
2026/04/01 19:02:13 BGP: [YTHK0-FSPPJ][EC 33554500] sendmsg_nexthop: zclient_send_message() failed
2026/04/01 19:02:13 BGP: [TBNSW-XXXBM] sendmsg_zebra_rnh: We have not connected yet, cannot send nexthops
2026/04/01 19:02:13 BGP: [HXW3G-K1M2A] sendmsg_zebra_rnh: sending cmd ZEBRA_NEXTHOP_REGISTER for 10.151.11.11/32 (vrf VRF default)
2026/04/01 19:02:13 BGP: [YTHK0-FSPPJ][EC 33554500] sendmsg_nexthop: zclient_send_message() failed
2026/04/01 19:02:13 BGP: [XADNE-H0J09] bgp_evpn_advertise_svi_ip_magic: Debug: bgp_evpn_advertise_svi_ip start
2026/04/01 19:02:13 BGP: [TVY7M-RSXKQ] evpn_set_advertise_svi_macip: Debug: evpn_set_advertise_svi_macip start, set=1
2026/04/01 19:02:13 BGP: [MFT5V-438WW] evpn_set_advertise_svi_macip: Debug: !vpn
2026/04/01 19:02:13 BGP: [ZYR8Y-BG69Q] bgp_zebra_advertise_svi_macip: Debug: bgp_zebra_advertise_svi_macip: Start
2026/04/01 19:02:13 BGP: [YMGRM-X6CDT] bgp_zebra_advertise_svi_macip: Debug: !bgp_zclient || bgp_zclient->sock < 0
2026/04/01 19:02:13 BGP: [NTAZ6-NXSGN] Creating VRF tvrfDEMO_25G01, AS 64505.1101
2026/04/01 19:02:13 BGP: [NTAZ6-NXSGN] Creating VRF tvrfDEMO_100G01, AS 64505.1101
2026/04/01 19:02:13 BGP: [NTAZ6-NXSGN] Creating VRF tvrfDEMO_25GP01, AS 64505.1101
2026/04/01 19:02:13 BGP: [VTVCM-Y2NW3] Configuration Read in Took: 00:00:00
2026/04/01 19:02:13 BGP: [G6NKK-8C6DV] end_config: VTY:0x561ab20d8b50, pending SET-CFG: 0
2026/04/01 19:02:14 BFD: [YMS0T-Z7S3B] zclient_connect is called
2026/04/01 19:02:14 BFD: [XD2SA-H7D64] zclient_start is called
2026/04/01 19:02:14 BFD: [KRMMR-X7JNT] zclient connect success with socket [24]
2026/04/01 19:02:14 ZEBRA: [V98V0-MTWPF] client 42 says hello and bids fair to announce only bfd routes vrf=0
2026/04/01 19:02:14 BFD: [HHE4V-7Z3WB] zclient 0x5ef2d87cfa70 command ZEBRA_CAPABILITIES VRF 0
2026/04/01 19:02:14 BFD: [HHE4V-7Z3WB] zclient 0x5ef2d87cfa70 command ZEBRA_VRF_ADD VRF 0
2026/04/01 19:02:14 BFD: [HHE4V-7Z3WB] zclient 0x5ef2d87cfa70 command ZEBRA_VRF_ADD VRF 912
2026/04/01 19:02:14 BFD: [ZY156-WR37J] zclient_send_reg_requests: send register messages for VRF 912
2026/04/01 19:02:14 BFD: [HHE4V-7Z3WB] zclient 0x5ef2d87cfa70 command ZEBRA_VRF_ADD VRF 919
2026/04/01 19:02:14 BFD: [ZY156-WR37J] zclient_send_reg_requ

[truncated for Pulse demo seed]

