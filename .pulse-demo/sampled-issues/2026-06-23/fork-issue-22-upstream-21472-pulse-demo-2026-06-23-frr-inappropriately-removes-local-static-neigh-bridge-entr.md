# Pulse Demo Issue Artifact

Fork issue: https://github.com/MsysTechnologiesllc/frr/issues/22
Fork issue number: #22
Imported upstream issue: FRRouting/frr#21472
Seed date: 2026-06-23
Label: pulse-demo-seed

## Title

[Pulse demo][2026-06-23] FRR inappropriately removes local static neigh/bridge entries during VM live-migration in EVPN fabrics

## Imported Issue Body

        Pulse demo seed imported this issue from `FRRouting/frr` on `2026-06-23`.

        Original issue: https://github.com/FRRouting/frr/issues/21472
        Original number: #21472
        Original author: @toreanderson
        Original labels: triage
        Original created: 2026-04-08T10:12:51Z
        Original updated: 2026-04-08T10:12:51Z

        ---

        ### Description

When a virtual machine is being live migrated from one hypervisor to another, where both hypervisors are running FRR and participating in an EVPN fabric, the virtual machine orchestration on the target hypervisor adds static sticky bridge FDB and IP neighbour entries reflecting the new location of the virtual machine in the fabric. FRR on this hypervisor will eventually advertise MACIP routes for those into the EVPN fabric.

On the source hypervisor, the static sticky bridge FDB and IP neighbour entries are simultaneously being removed, causing FRR to withdraw its MACIP routes.

However, due to factors such as the propagation delay for the withdrawals of the MACIP routes it usually happens that the target hypervisor sees the MACIP route withdrawal from the source hypervisor some time *after* the static sticky FDB/neigh entries have been added locally. From what I can tell, FRR appears to act on this MACIP route withdrawal by removing the _**local**_ FDB/neigh entries added by the virtual machine orchestration.

The virtual machine orchestration's reconciliation loop eventually notices that the FDB/neigh entries it added after the live migration have gone AWOL and adds them back, and things usually settles into a functional state. However the oscillating FDB/neigh entries adds quite a bit of delay to the VM live migration process, which is supposed to be nearly hitless.

### Version

```text
FRRouting 10.7.0-dev (frr1) on Linux(5.14.0-611.20.1.el9_7.x86_64).
Copyright 1996-2005 Kunihiro Ishiguro, et al.
configured with:
    '--build=x86_64-redhat-linux-gnu' '--host=x86_64-redhat-linux-gnu' '--program-prefix=' '--disable-dependency-tracking' '--prefix=/usr' '--exec-prefix=/usr' '--bindir=/usr/bin' '--datadir=/usr/share' '--includedir=/usr/include' '--libdir=/usr/lib64' '--libexecdir=/usr/libexec' '--sharedstatedir=/var/lib' '--mandir=/usr/share/man' '--infodir=/usr/share/info' '--sbindir=/usr/lib/frr' '--sysconfdir=/etc' '--localstatedir=/var' '--disable-static' '--disable-werror' '--enable-multipath=256' '--enable-vtysh' '--enable-ospfclient' '--enable-ospfapi' '--enable-ldpd' '--enable-pimd' '--enable-pim6d' '--enable-pbrd' '--enable-nhrpd' '--enable-eigrpd' '--enable-babeld' '--enable-vrrpd' '--enable-user=frr' '--enable-group=frr' '--enable-vty-group=frrvty' '--enable-fpm' '--enable-watchfrr' '--disable-bgp-vnc' '--enable-isisd' '--enable-doc' '--enable-rpki' '--enable-bfdd' '--enable-pathd' '--disable-grpc' '--enable-snmp' '--disable-zeromq' '--enable-pcre2posix' 'build_alias=x86_64-redhat-linux-gnu' 'host_alias=x86_64-redhat-linux-gnu' 'PKG_CONFIG_PATH=:/usr/lib64/pkgconfig:/usr/share/pkgconfig' 'CC=gcc' 'CXX=g++' 'LT_SYS_LIBRARY_PATH=/usr/lib64:'
```

### How to reproduce

Have a OpenStack deployment with (at least) two hypervisors connected with BGP to a L3 data centre fabric. Deploy EVPN on the hypervisors with an orchestration agent such as [evpn_agent](https://github.com/toreanderson/evpn_agent). Deploy a VM on a VLAN («provider network» in OpenStack nomenclature) that is connected to a VLAN-aware Linux Bridge with a L2VNI device connected and FRR set up to advertise all VNIs, ensuring there is EVPN-arranged L2 connectivity between the two hypervisors. The live migrate a VM from one hypervisor to another.

The FRR configuration in question is as follows:

```
frr version 10.7.0-dev
frr defaults datacenter
hostname frr1
log syslog
!
route-map LEAF-IN permit 1
 set community no-export additive
exit
!
route-map vrf-2-redistribute-connected permit 99
 match interface irb-99
exit
!
route-map vrf-2-redistribute-connected deny 65535
exit
!
debug zebra kernel
debug bgp updates in
!
vrf vrf-2
 vni 2
exit-vrf
!
interface lo
 ip address 10.0.0.1/32
exit
!
router bgp 65001
 bgp router-id 10.0.0.1
 bgp disable-ebgp-connected-route-check
 bgp bestpath as-path multipath-relax
 neighbor LEAF peer-group
 neighbor LEAF remote-as external
 neighbor eth4 interface peer-group LEAF
 neighbor eth5 interface peer-group LEAF
 use-underlays-nexthop-weight
 !
 address-family ipv4 unicast
  network 10.0.0.1/32
  redistribute kernel
  redistribute connected
  neighbor LEAF route-map LEAF-IN in
 exit-address-family
 !
 address-family ipv6 unicast
  redistribute kernel
  redistribute connected
  neighbor LEAF activate
  neighbor LEAF route-map LEAF-IN in
 exit-address-family
 !
 address-family l2vpn evpn
  neighbor LEAF activate
  neighbor LEAF route-map LEAF-IN in
  advertise-all-vni
 exit-address-family
exit
!
router bgp 65001 vrf vrf-2
 no bgp default ipv4-unicast
 bgp disable-ebgp-connected-route-check
 bgp bestpath as-path multipath-relax
 use-underlays-nexthop-weight
 !
 address-family ipv4 unicast
  redistribute kernel
  redistribute connected route-map vrf-2-redistribute-connected
 exit-address-family
 !
 address-family ipv6 unicast
  redistribute kernel
  redistribute connected route-map vrf-2-redistribute-connected
 exit-address-family
 !
 address-family l2vpn evpn
  advertise ipv4 unicast
  advertise ipv6 unicast
 exit-address-family
exit
!
end
```

### Expected behavior

When the orchestration agent on the target hypervisor adds the VM's FDB/neigh entries, FRR should pick up those and immediately start advertising MACIP routes for them. It should not delete them. When withdraws for MACIP routes advertised from the old hypervisor are received FRR, it should only delete the exact entries that this MACIP route previously caused to be installed, not *all* entries for the MAC and/or IP address (as those might have been added by some external process and should therefore be left alone).

### Actual behavior

I'll go through the systemd journal (with line numbers added) from when a virtual machine is being migrated to a hypervisor and comment along the lines. The log contains the debug output from FRR, from the orchestration agent (evpn_agent.service) as well as from some custom units that run `ip monitor neigh` and `bridge monitor fdb`, as well as one unit (differ.service) that reports

[truncated for Pulse demo seed]

