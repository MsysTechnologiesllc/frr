# Pulse Demo Issue Artifact

Fork issue: https://github.com/MsysTechnologiesllc/frr/issues/14
Fork issue number: #14
Imported upstream issue: FRRouting/frr#17550
Seed date: 2026-06-23
Label: pulse-demo-seed

## Title

[Pulse demo][2026-06-23] Weird inactive interface in route table on frr-10.2

## Imported Issue Body

        Pulse demo seed imported this issue from `FRRouting/frr` on `2026-06-23`.

        Original issue: https://github.com/FRRouting/frr/issues/17550
        Original number: #17550
        Original author: @bocmanpy
        Original labels: triage
        Original created: 2024-12-03T06:33:15Z
        Original updated: 2026-05-05T14:05:59Z

        ---

        ### Description

On new version frr-10.2 i getting weird case.
Interface po1 is UP and you can see it from `ip link` and `show interface command`.
But in `show ip route`  routes belonging that interface saying it's linkdown/inactive.
<img width="300" height="300" alt="frr_10 2_weird_status" src="https://github.com/user-attachments/assets/ae0f5b7c-e3d8-44e9-8ac7-72a3af389d68">
I'm downgraded to frr-10.1 and that problem won't reproducing.
Restarting frr-10.2 helping interface become normal in route table.

### Version

```text
FRRouting 10.2 (hostname) on Linux(4.18.0-532.el8.x86_64).
Copyright 1996-2005 Kunihiro Ishiguro, et al.
configured with:
    '--prefix=/usr' '--sbindir=/usr/sbin' '--sysconfdir=/etc' '--localstatedir=/run' '--libdir=/usr/lib64/frr' '--libexecdir=/usr/libexec/frr' '--with-moduledir=/usr/lib64/frr/modules' '--with-crypto=openssl' '--with-libpam' '--enable-multipath=256' '--enable-user=frr' '--enable-group=frr' '--enable-vty-group=frrvty' '--enable-configfile-mask=0640' '--enable-logfile-mask=0640' '--enable-config-rollbacks' '--enable-pcre2posix' '--enable-rpki' '--enable-fpm' '--disable-babeld' '--disable-pimd' '--disable-pim6d' '--disable-isisd' '--disable-ripd' '--disable-ripngd' '--disable-ldpd' '--disable-nhrpd' '--disable-eigrpd' 'runstatedir=/run' '--disable-static' 'PKG_CONFIG_PATH=:/usr/lib64/pkgconfig:/usr/share/pkgconfig'
```


### How to reproduce

1. On frr 10.2 use similar `frr.conf`
<details>
  <summary>frr.conf</summary>

```
!
ip route 10.10.0.0/16 10.10.3.254 po1
ip route 10.24.0.0/16 10.24.3.254 vlan24
ip route 10.25.0.0/16 10.25.3.254 vlan25
!
interface po1
 pbr-policy po1
exit
!
interface vlan24
 pbr-policy vlan24
exit
!
interface vlan25
 pbr-policy vlan25
exit
!
pbr-map po1 seq 5
 match src-ip 10.10.3.24/32
 set nexthop-group po1
exit
!
pbr-map vlan24 seq 5
 match src-ip 10.24.3.24/32
 set nexthop-group vlan24
exit
!
pbr-map vlan25 seq 5
 match src-ip 10.25.3.16/32
 set nexthop-group vlan25
exit
!
nexthop-group po1
 nexthop 10.10.3.254 po1
exit
!
nexthop-group vlan24
 nexthop 10.24.3.254 vlan24
exit
!
nexthop-group vlan25
 nexthop 10.25.3.254 vlan25
exit
!
access-list vty remark Disable connections to vtysh from non localhost
access-list vty seq 5 permit 127.0.0.1/8
access-list vty seq 10 deny any
!
line vty
 access-class vty
exit
!
```

</details

### Expected behavior

Interface should be active in route table and related routes was added.

### Actual behavior

Interface in route table marked as linkdown/inactive and related routes doesn't adding to route table.
Here is debug logs from 10.1 and 10.2 versions.
[frr_10.1_debug.log](https://github.com/user-attachments/files/17988357/frr_10.1_debug.log)
[frr_10.2_debug.log](https://github.com/user-attachments/files/17988363/frr_10.2_debug.log)

### Additional context

The main problem is that is not 100% reproducible.
I have a same(only subnets different) configuration that is working fine with frr-10.2

### Checklist

- [X] I have searched the open issues for this bug.
- [X] I have not included sensitive information in this report.

