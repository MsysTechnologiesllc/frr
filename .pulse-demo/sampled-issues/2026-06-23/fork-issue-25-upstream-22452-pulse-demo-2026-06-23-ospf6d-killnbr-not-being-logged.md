# Pulse Demo Issue Artifact

Fork issue: https://github.com/MsysTechnologiesllc/frr/issues/25
Fork issue number: #25
Imported upstream issue: FRRouting/frr#22452
Seed date: 2026-06-23
Label: pulse-demo-seed

## Title

[Pulse demo][2026-06-23] ospf6d: KillNbr not being logged

## Imported Issue Body

        Pulse demo seed imported this issue from `FRRouting/frr` on `2026-06-23`.

        Original issue: https://github.com/FRRouting/frr/issues/22452
        Original number: #22452
        Original author: @subsecond
        Original labels: triage
        Original created: 2026-06-23T09:52:58Z
        Original updated: 2026-06-23T09:52:58Z

        ---

        ### Description

I was troubleshooting a small FRR setup and noticed that – when a link is taken down – we only see the expected `KillNbr` log for OSPFv2 (ospf) and not OSPFv3 (ospf6), despite both having set `log-adjacency-changes`.

### Version

```text
FRRouting 10.6.1 (frr-r2) on Linux(6.12.94+deb13-amd64).
Copyright 1996-2005 Kunihiro Ishiguro, et al.
configured with:
    '--build=x86_64-linux-gnu' '--prefix=/usr' '--includedir=${prefix}/include' '--mandir=${prefix}/share/man' '--infodir=${prefix}/share/info' '--sysconfdir=/etc' '--localstatedir=/var' '--disable-option-checking' '--disable-silent-rules' '--libdir=${prefix}/lib/x86_64-linux-gnu' '--libexecdir=${prefix}/lib/x86_64-linux-gnu' '--disable-maintainer-mode' '--sbindir=/usr/lib/frr' '--with-vtysh-pager=/usr/bin/pager' '--libdir=/usr/lib/x86_64-linux-gnu/frr' '--with-moduledir=/usr/lib/x86_64-linux-gnu/frr/modules' '--disable-dependency-tracking' '--enable-rpki' '--disable-scripting' '--enable-pim6d' '--disable-grpc' '--disable-address-sanitizer' '--with-libpam' '--enable-doc' '--enable-doc-html' '--enable-snmp' '--enable-fpm' '--disable-protobuf' '--disable-zeromq' '--enable-ospfapi' '--enable-bgp-vnc' '--enable-multipath=256' '--enable-pcre2posix' '--enable-user=frr' '--enable-group=frr' '--enable-vty-group=frrvty' '--enable-configfile-mask=0640' '--enable-logfile-mask=0640' 'build_alias=x86_64-linux-gnu' 'PYTHON=python3'
```

### How to reproduce

**Topology:**
```
frr-r1 enp7s0 ---- enp7s0 frr-r2
````

**Minimal Config:**
```
frr-r1# show run
Building configuration...

Current configuration:
!
frr version 10.6.1
frr defaults traditional
hostname frr-r1
log syslog informational
service integrated-vtysh-config
!
ip router-id 10.0.0.1
!
interface enp7s0
 ip address 10.0.0.1/32
 ip ospf area 0.0.0.0
 ip ospf network point-to-point
 ipv6 address 2001:db8:bb::1/128
 ipv6 ospf6 area 0.0.0.0
 ipv6 ospf6 network point-to-point
exit
!
interface lo
 ip address 10.0.0.1/32
 ip ospf area 0.0.0.0
 ip ospf passive
 ipv6 address 2001:db8:bb::1/128
 ipv6 ospf6 area 0.0.0.0
 ipv6 ospf6 passive
exit
!
router ospf
 log-adjacency-changes
exit
!
router ospf6
 log-adjacency-changes
exit
!
end
```
and
```
frr-r2# show run
Building configuration...

Current configuration:
!
frr version 10.6.1
frr defaults traditional
hostname frr-r2
log syslog informational
service integrated-vtysh-config
!
ip router-id 10.0.0.2
!
interface enp7s0
 ip address 10.0.0.2/32
 ip ospf area 0.0.0.0
 ip ospf network point-to-point
 ipv6 address 2001:db8:bb::2/128
 ipv6 ospf6 area 0.0.0.0
 ipv6 ospf6 network point-to-point
 shutdown
exit
!
interface lo
 ip address 10.0.0.2/32
 ip ospf area 0.0.0.0
 ip ospf passive
 ipv6 address 2001:db8:bb::2/128
 ipv6 ospf6 area 0.0.0.0
 ipv6 ospf6 passive
exit
!
router ospf
 log-adjacency-changes
exit
!
router ospf6
 log-adjacency-changes
exit
!
end
```

Then, bring down `enp7s0` on `frr-r2` as follows:
```
frr-r2# conf t
frr-r2(config)# interface enp7s0
frr-r2(config-if)# shutdown 
frr-r2(config-if)# 
```

### Expected behavior

We also would like to see the log for OSPFv3, so something like this should (additionally) show up in the log:
```
Jun 23 09:24:39 frr-r2 ospf6d[1365]: [GHN28-AMRE7] AdjChg: Nbr 10.0.0.2(default) on 10.0.0.2%enp7s0: Full -> Down (KillNbr)
```

### Actual behavior

Once I take down the interface `enp7s0` on `frr-r2`, I see the all of the expected logs on `frr-r1`:
```
Jun 23 09:25:12 frr-r1 ospfd[1362]: [Y05P2-YJVXY] AdjChg: Nbr 10.0.0.2, NbrIP 10.0.0.2 (default) on enp7s0:10.0.0.1: Full -> Deleted (InactivityTimer)
Jun 23 09:25:15 frr-r1 ospf6d[1365]: [GHN28-AMRE7] AdjChg: Nbr 10.0.0.2(default) on 10.0.0.2%enp7s0: Full -> Down (InactivityTimer)
```
The reason that these logs get triggered is that we have reached `InactivityTimer` and therefore, we take down the OSPFv2 and OSPFv3 neighbor.

However, on `frr-r2`, where we explicitly took down the interface `enp7s0`, we only see the log from OSPFv2 and not OSPFv3:
```
Jun 23 09:24:39 frr-r2 systemd-networkd[553]: enp7s0: Link DOWN
Jun 23 09:24:39 frr-r2 systemd-networkd[553]: enp7s0: Lost carrier
Jun 23 09:24:39 frr-r2 ospfd[1307]: [Y05P2-YJVXY] AdjChg: Nbr 10.0.0.1, NbrIP 10.0.0.1 (default) on enp7s0:10.0.0.2: Full -> Deleted (KillNbr)
```

### Additional context

While I have identified the respective code, where this *should* get triggered, I am not a software engineer and therefore cannot contribute a fix here. But it might help as a starting point:
* OSPFv2: https://github.com/FRRouting/frr/blob/master/ospfd/ospf_nsm.c#L643
* OSPFv3: https://github.com/FRRouting/frr/blob/master/ospf6d/ospf6_neighbor.c#L239

### Checklist

- [x] I have searched the open issues for this bug.
- [x] I have not included sensitive information in this report.

