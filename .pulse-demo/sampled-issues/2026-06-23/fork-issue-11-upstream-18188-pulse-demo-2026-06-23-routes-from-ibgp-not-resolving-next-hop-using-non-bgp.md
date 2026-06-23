# Pulse Demo Issue Artifact

Fork issue: https://github.com/MsysTechnologiesllc/frr/issues/11
Fork issue number: #11
Imported upstream issue: FRRouting/frr#18188
Seed date: 2026-06-23
Label: pulse-demo-seed

## Title

[Pulse demo][2026-06-23] routes from iBGP not resolving next-hop using non-BGP

## Imported Issue Body

        Pulse demo seed imported this issue from `FRRouting/frr` on `2026-06-23`.

        Original issue: https://github.com/FRRouting/frr/issues/18188
        Original number: #18188
        Original author: @jkroonza
        Original labels: triage
        Original created: 2025-02-16T18:22:48Z
        Original updated: 2025-08-24T10:25:59Z

        ---

        ### Description

Hi,

All info trimmed, and IP addresses obfuscated where needed.

On router A:

```
kerberos# sh ip bgp sum
...
a.b.c.2     4     65512    566386    564281  2358673    0    0 5d19h36m           20   190980 cerberus
kerberos# sh ip route a.b.c.2
Routing entry for a.b.c.2/32
  Known via "ospf", distance 110, metric 30, best
  Last update 5d19h39m ago
  * 172.31.255.2, via bond0.2, weight 1
kerberos# sh ip bgp a.b.c.2
BGP routing table entry for a.b.c.2/32, version 0
Paths: (1 available, no best path)
  Not advertised to any peer
  Local
    a.b.c.2 (inaccessible) from a.b.c.2 (a.b.c.2)
      Origin IGP, metric 0, localpref 100, invalid, internal
      Community: .....
      Last update: Tue Feb 11 00:31:21 2025
```
As a result of a.b.c.2 being inaccessible, this prefix is not being advertised where it should be:

```
kerberos# sh ip bgp neigh a.b.c.137 advertised-routes 
BGP table version is 2358883, local router ID is a.b.c.1, vrf id 0
Default local pref 100, local AS 65512
Status codes:  s suppressed, d damped, h history, * valid, > best, = multipath,
               i internal, r RIB-failure, S Stale, R Removed
Nexthop codes: @NNN nexthop's vrf id, < announce-nh-self
Origin codes:  i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

    Network          Next Hop            Metric LocPrf Weight Path
 *> a.b.c.1/32   0.0.0.0                  0         65512 i

Total number of prefixes 1
```

a.b.c.137 is an eBGP "downstream" peer.

### Version

```text
kerberos# show version
FRRouting 10.0.2-gentoo (kerberos) on Linux(...).
Copyright 1996-2005 Kunihiro Ishiguro, et al.
configured with:
    '--prefix=/usr' '--build=x86_64-pc-linux-gnu' '--host=x86_64-pc-linux-gnu' '--mandir=/usr/share/man' '--infodir=/usr/share/info' '--datadir=/usr/share' '--sysconfdir=/etc' '--localstatedir=/var/lib' '--datarootdir=/usr/share' '--disable-dependency-tracking' '--disable-silent-rules' '--disable-static' '--docdir=/usr/share/doc/frr-10.0.2' '--htmldir=/usr/share/doc/frr-10.0.2/html' '--with-sysroot=/' 'LEX=flex' '--with-pkg-extra-version=-gentoo' '--enable-configfile-mask=0640' '--enable-logfile-mask=0640' '--libdir=/usr/lib/frr' '--sbindir=/usr/lib/frr' '--libexecdir=/usr/lib/frr' '--sysconfdir=/etc/frr' '--localstatedir=/run/frr' '--with-moduledir=/usr/lib/frr/modules' '--enable-user=frr' '--enable-group=frr' '--enable-vty-group=frr' '--enable-multipath=64' '--disable-doc' '--disable-fpm' '--disable-grpc' '--enable-realms' '--disable-nhrpd' '--disable-rpki' '--disable-snmp' 'build_alias=x86_64-pc-linux-gnu' 'host_alias=x86_64-pc-linux-gnu' 'PKG_CONFIG=/usr/bin/pkg-config' 'PKG_CONFIG_PATH=/var/tmp/portage/net-misc/frr-10.0.2/temp/python3.12/pkgconfig' 'PYTHON=/usr/bin/python3.12'

An upgrade to 10.0.3 is queued for our next change-window later in the week, but I don't think that will fix this.
```

### How to reproduce

Peer two iBGP routers via loopbacks, connected to the same subnet.  Have both also advertise their loopback via iBGP so that when they're up these loopbacks can (if route maps permit) be advertised to over eBGP peerings.

### Expected behavior

Since the remote loopback is reachable in the FIB, I expect the network originated loopbacks to be forward advertised on eBGP peers as determined by route-maps.  For that to happen, the route has to be marked as valid and not inaccessible.

### Actual behavior

Valid routes are marked inaccessible, preventing them from being forward advertised.

This works if the routers peer using their ethernet interface addresses rather than their loopback addresses.  Since there are a number of fail-over paths available all our iBGP peerings uses loopbacks.

### Additional context

a.b.c.1 and a.b.c.2 peers via loopbacks, which is exchanged using OSPF.

disable-connected-check seems like a sensible candidate for the problem, but the route here is received on iBGP not eBGP, and as such it does not seem to relate.

### Checklist

- [x] I have searched the open issues for this bug.
- [x] I have not included sensitive information in this report.

