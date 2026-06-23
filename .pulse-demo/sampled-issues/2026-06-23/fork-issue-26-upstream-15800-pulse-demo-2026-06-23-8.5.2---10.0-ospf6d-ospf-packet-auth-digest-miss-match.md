# Pulse Demo Issue Artifact

Fork issue: https://github.com/MsysTechnologiesllc/frr/issues/26
Fork issue number: #26
Imported upstream issue: FRRouting/frr#15800
Seed date: 2026-06-23
Label: pulse-demo-seed

## Title

[Pulse demo][2026-06-23] 8.5.2 <-> 10.0  ospf6d OSPF packet auth digest miss-match

## Imported Issue Body

        Pulse demo seed imported this issue from `FRRouting/frr` on `2026-06-23`.

        Original issue: https://github.com/FRRouting/frr/issues/15800
        Original number: #15800
        Original author: @herrin
        Original labels: triage
        Original created: 2024-04-20T05:35:12Z
        Original updated: 2024-10-18T04:23:18Z

        ---

        ### Description

auth digest in IPv6 OSPFv3 is not working for me between FRR 8.5.2 and FRR 10.0. 

Identical configuration worked fine between 8.5.2 and 8.5.4.
Identical configuration worked fine between 8.5.2 and 9.0.2
Identical configuration failed between 8.5.2 and 9.1

If I remove auth digest from both sides, OSPFv3 works.

IPv4 OSPFv2 comes up normally with auth digest.

### Version

```text
sh ver
FRRouting 8.5.2 () on Linux(6.1.0-10-amd64).
Copyright 1996-2005 Kunihiro Ishiguro, et al.
configured with:
    '--build=x86_64-linux-gnu' '--prefix=/usr' '--includedir=${prefix}/include' '--mandir=${prefix}/share/man' '--infodir=${prefix}/share/info' '--sysconfdir=/etc' '--localstatedir=/var' '--disable-option-checking' '--disable-silent-rules' '--libdir=${prefix}/lib/x86_64-linux-gnu' '--libexecdir=${prefix}/lib/x86_64-linux-gnu' '--disable-maintainer-mode' '--localstatedir=/var/run/frr' '--sbindir=/usr/lib/frr' '--sysconfdir=/etc/frr' '--with-vtysh-pager=/usr/bin/pager' '--libdir=/usr/lib/x86_64-linux-gnu/frr' '--with-moduledir=/usr/lib/x86_64-linux-gnu/frr/modules' '--disable-dependency-tracking' '--enable-rpki' '--disable-scripting' '--enable-pim6d' '--with-libpam' '--enable-doc' '--enable-doc-html' '--enable-snmp' '--enable-fpm' '--disable-protobuf' '--disable-zeromq' '--enable-ospfapi' '--enable-bgp-vnc' '--enable-multipath=256' '--enable-user=frr' '--enable-group=frr' '--enable-vty-group=frrvty' '--enable-configfile-mask=0640' '--enable-logfile-mask=0640' 'build_alias=x86_64-linux-gnu' 'PYTHON=python3'

sh ver
FRRouting 10.0 () on Linux(6.1.86-deb12).
Copyright 1996-2005 Kunihiro Ishiguro, et al.
configured with:
    '--build=x86_64-linux-gnu' '--prefix=/usr' '--includedir=${prefix}/include' '--mandir=${prefix}/share/man' '--infodir=${prefix}/share/info' '--sysconfdir=/etc' '--localstatedir=/var' '--disable-option-checking' '--disable-silent-rules' '--libdir=${prefix}/lib/x86_64-linux-gnu' '--libexecdir=${prefix}/lib/x86_64-linux-gnu' '--disable-maintainer-mode' '--sbindir=/usr/lib/frr' '--with-vtysh-pager=/usr/bin/pager' '--libdir=/usr/lib/x86_64-linux-gnu/frr' '--with-moduledir=/usr/lib/x86_64-linux-gnu/frr/modules' '--disable-dependency-tracking' '--enable-rpki' '--disable-scripting' '--enable-pim6d' '--with-libpam' '--enable-doc' '--enable-doc-html' '--enable-snmp' '--enable-fpm' '--disable-protobuf' '--disable-zeromq' '--enable-ospfapi' '--enable-bgp-vnc' '--enable-multipath=256' '--enable-user=frr' '--enable-group=frr' '--enable-vty-group=frrvty' '--enable-configfile-mask=0640' '--enable-logfile-mask=0640' 'build_alias=x86_64-linux-gnu' 'PYTHON=python3'
```


### How to reproduce

Create an OpenVPN tun (ethernet) link with a MTU of 1460. Configure with IPv6 addresses and confirm ping. Install FRR version 8.5.2 on one side of the link and version 10.0 on the other.

interface name
 ipv6 ospf6 area 0.0.0.0
 ipv6 ospf6 authentication key-id 1 hash-algo hmac-sha-256 key testkey
router ospf6
 ospf6 router-id a.b.c.d
 redistribute connected

### Expected behavior

"show ipv6 ospf neighbor" reports the neighbor on the other end of the OpenVPN link.

### Actual behavior

"show ipv6 ospf neighbor" reports no neighbors.

debug ospf6 message hello recv
debug ospf6 authentication rx

in frr.log:
Apr 19 21:07:23 server ospf6d[15596]: [ZN6JB-XGJFW] RECV[otherserver]: OSPF packet auth digest miss-match on Hello

### Additional context

IPv4 OSPF with auth digest works over the same OpenVPN link in 10.0.

If auth digest is removed from both sides. IPv6 OSPF works.

Same result with a GRE tunnel and an MTU of 1400. Haven't tried other virtual interface types.

### Checklist

- [X] I have searched the open issues for this bug.
- [X] I have not included sensitive information in this report.

