# Pulse Demo Issue Artifact

Fork issue: https://github.com/MsysTechnologiesllc/frr/issues/29
Fork issue number: #29
Imported upstream issue: FRRouting/frr#20712
Seed date: 2026-06-23
Label: pulse-demo-seed

## Title

[Pulse demo][2026-06-23] `show bgp l2vpn evpn vni` shows wrong import-rt for L2VNIs with automatic RT

## Imported Issue Body

        Pulse demo seed imported this issue from `FRRouting/frr` on `2026-06-23`.

        Original issue: https://github.com/FRRouting/frr/issues/20712
        Original number: #20712
        Original author: @robinchrist
        Original labels: triage
        Original created: 2026-02-05T13:04:00Z
        Original updated: 2026-02-28T15:15:41Z

        ---

        ### Description

The `show bgp l2evpn evpn vni` commands shows wrong import-rts for L2VNIs with automatic RT:

```
pve-sdn-test-01# show bgp l2vpn evpn vni
Advertise Gateway Macip: Disabled
Advertise SVI Macip: Disabled
Advertise All VNI flag: Enabled
BUM flooding: Head-end replication
VXLAN flooding: Enabled
Number of L2 VNIs: 5
Number of L3 VNIs: 2
Flags: * - Kernel
  VNI        Type RD                    Import RT                 Export RT                 MAC-VRF Site-of-Origin    Tenant VRF                           
* 1002       L2   10.X.X.X:4        65000:1002                65000:1002                                          vrf_Elim01                           
* 1001       L2   10.X.X.X:2        65000:1001                65000:1001                                          vrf_Elim01                           
* 1003       L2   10.X.X.X:5        65000:1003                65000:1003                                          vrf_Elim01                           
* 2001       L2   10.X.X.X:6        65000:2001                65000:2001                                          vrf_Elim02                           
* 2002       L2   10.X.X.X:8        65000:2002                65000:2002                                          vrf_Elim02                           
* 10001      L3   10.X.X.X:3        65000:10001               65000:10001                                         vrf_Elim01                           
* 10002      L3   10.X.X.X:7        65000:10002               65000:10002                                         vrf_Elim02                          
```

The configured AS is 65000.

According to a comment in `bgp_evpn.c`:
```
/* If using "automatic" RT, we only care about the
 * local-admin sub-field.
 * This is to facilitate using VNI as the RT for EBGP
 * peering too.
 */
```

If you do `show bgp l2vpn evpn import-rt ` it is shown "correctly" with `0`
```
pve-sdn-test-01# show bgp l2vpn evpn import-rt 
Route-target: 0:2002
List of VNIs importing routes with this route-target:
  2002
Route-target: 0:2001
List of VNIs importing routes with this route-target:
  2001
Route-target: 0:1002
List of VNIs importing routes with this route-target:
  1002
Route-target: 0:1003
List of VNIs importing routes with this route-target:
  1003
Route-target: 0:1001
List of VNIs importing routes with this route-target:
  1001
```

### Version

```text
FRRouting 10.4.1 (pve-sdn-test-01) on Linux(6.17.4-2-pve).
Copyright 1996-2005 Kunihiro Ishiguro, et al.
configured with:
    '--build=x86_64-linux-gnu' '--prefix=/usr' '--includedir=${prefix}/include' '--mandir=${prefix}/share/man' '--infodir=${prefix}/share/info' '--sysconfdir=/etc' '--localstatedir=/var' '--disable-option-checking' '--disable-silent-rules' '--libdir=${prefix}/lib/x86_64-linux-gnu' '--runstatedir=/run' '--disable-maintainer-mode' '--sbindir=/usr/lib/frr' '--with-vtysh-pager=/usr/bin/pager' '--libdir=/usr/lib/x86_64-linux-gnu/frr' '--with-moduledir=/usr/lib/x86_64-linux-gnu/frr/modules' '--disable-dependency-tracking' '--enable-rpki' '--disable-scripting' '--enable-pim6d' '--disable-grpc' '--with-libpam' '--enable-doc' '--enable-doc-html' '--enable-snmp' '--enable-fpm' '--disable-protobuf' '--disable-zeromq' '--enable-ospfapi' '--enable-bgp-vnc' '--enable-multipath=256' '--enable-pcre2posix' '--enable-user=frr' '--enable-group=frr' '--enable-vty-group=frrvty' '--enable-configfile-mask=0640' '--enable-logfile-mask=0640' 'build_alias=x86_64-linux-gnu' 'PYTHON=python3'
```

### How to reproduce

Configure normal EVPN with L2VNIs

### Expected behavior

Expected output:
```
...
  VNI        Type RD                    Import RT                 Export RT                 MAC-VRF Site-of-Origin    Tenant VRF                           
* 1002       L2   10.X.X.X:4        *:1002                        65000:1002                                          vrf_Elim01                           
* 1001       L2   10.X.X.X:2        *:1001                        65000:1001                                          vrf_Elim01                           
* 1003       L2   10.X.X.X:5        *:1003                        65000:1003                                          vrf_Elim01                           
* 2001       L2   10.X.X.X:6        *:2001                        65000:2001                                          vrf_Elim02                           
* 2002       L2   10.X.X.X:8        *:2002                        65000:2002                                          vrf_Elim02                           
* 10001      L3   10.X.X.X:3        *:10001                     65000:10001                                         vrf_Elim01                           
* 10002      L3   10.X.X.X:7        *:10002                     65000:10002                                         vrf_Elim02                          
```

Wildcard import-RTs should generally be formatted with `*:<Local Admin>`! The alternative `0:<Local Admin>` as in `show bgp l2vpn evpn import-rt` formatting should be avoided as the meaning is not obvious and rather an internal implementation detail (that the Global Admin is set to 0)

The output of `show bgp l2vpn evpn import-rt ` should also be adjusted:

```
pve-sdn-test-01# show bgp l2vpn evpn import-rt 
Route-target: *:2002
List of VNIs importing routes with this route-target:
  2002
Route-target: *:2001
List of VNIs importing routes with this route-target:
  2001
Route-target: *:1002
List of VNIs importing routes with this route-target:
  1002
Route-target: *:1003
List of VNIs importing routes with this route-target:
  1003
Route-target: *:1001
List of VNIs importing routes with this route-target:
  1001
```
Note that `0:<Local Admin>` was replaced with `*:<Local Admin>`

### Actual behavior

See above

### Additional context

_No response_

### Checklist

- [x] I have searched the open issues for this bug.
- [x] I have not included sensitive information in this report.

