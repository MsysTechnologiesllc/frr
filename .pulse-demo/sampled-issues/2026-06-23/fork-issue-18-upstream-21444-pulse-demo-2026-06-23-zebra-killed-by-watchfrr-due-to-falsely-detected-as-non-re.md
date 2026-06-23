# Pulse Demo Issue Artifact

Fork issue: https://github.com/MsysTechnologiesllc/frr/issues/18
Fork issue number: #18
Imported upstream issue: FRRouting/frr#21444
Seed date: 2026-06-23
Label: pulse-demo-seed

## Title

[Pulse demo][2026-06-23] zebra killed by watchfrr due to (falsely) detected as non-responsive

## Imported Issue Body

        Pulse demo seed imported this issue from `FRRouting/frr` on `2026-06-23`.

        Original issue: https://github.com/FRRouting/frr/issues/21444
        Original number: #21444
        Original author: @jkroonza
        Original labels: triage
        Original created: 2026-04-01T19:19:51Z
        Original updated: 2026-04-01T21:29:55Z

        ---

        ### Description

I did find #18706 - we saw similar this morning on frr 10.1.3, 10.3.3 and 10.5.3.

In all three cases there was (we suspect) a heavy fluctuation in routes available from bgpd.  bgpd itself saw very high CPU usage (~3m ipv4 routes, 1m from each of three other peers).

At the same time zebra was running 100% of a core CPU for extended periods.  Once this triggered the first time we could not get things started up as zebra needs to load ~1m IPv4 routes (and several hundred thousand v6) routes into the kernel (6.9.3 which is also due an upgrade at some point soon), would be flatlining at 100% CPU (presumably trying to catch up with bgpd loading routes into the kernel) for a while, and then all (but zebra) frr processes would terminate, only restarting once zebra is killed with -9.

The message we saw that eventually got us onto the actual issue:

```
Apr  1 13:44:45 kerberos watchfrr[30587]: [T58XM-TP956][EC 268435457] zebra state -> unresponsive : no response yet to ping sent 90 seconds ago
```

We subsequently pushed -t 600 to watchfrr, but it just ook longer (about 20 minutes):

```
Apr  1 14:39:13 kerberos watchfrr[13171]: [T58XM-TP956][EC 268435457] zebra state -> unresponsive : no response yet to ping sent 600 seconds ago
```

It must be noted watchfrr then sends (via watchfrr.sh) SIGINT to *all* monitored processes.

watchfrr command:

```/usr/lib/frr/watchfrr -d -F traditional zebra staticd mgmtd bgpd ospfd ospf6d -t 600```

zebra takes quite a while to come down, presumably first cathing up with it's message queue.

I'm not sure the monitoring process between watchfrr and the relevant processes it monitors, but zebra was most certainly still alive (albeit struggling to keep up quite a bit) in all cases.

We ended up having to discard bulk of ingress routes at our edge on IPTs, which now results in sub-optimal routing, but at least everything is stable.  We're looking to use communities combined with "router bgp ???; address family ???; table-map no-internal" to not install full routing tables into the linux kernel - but time did not permit this afternoon to implement that.

Wondering if there is perhaps a monitoring bug here between watchfrr and zebra, or possibly some tweaking required in zebra in order to speed up loading/unloading routes into/from the kernel.

### Version

```text
FRRouting 10.3.3-gentoo (kerberos) on Linux(6.9.3-uls).
Copyright 1996-2005 Kunihiro Ishiguro, et al.
configured with:
    '--prefix=/usr' '--build=x86_64-pc-linux-gnu' '--host=x86_64-pc-linux-gnu' '--mandir=/usr/share/man' '--infodir=/usr/share/info' '--datadir=/usr/share' '--sysconfdir=/etc' '--localstatedir=/var/lib' '--datarootdir=/usr/share' '--disable-dependency-tracking' '--disable-silent-rules' '--disable-static' '--docdir=/usr/share/doc/frr-10.3.3' '--htmldir=/usr/share/doc/frr-10.3.3/html' '--with-sysroot=/' 'ac_cv_prog_VALGRIND_CHECK=no' 'LEX=flex' '--with-pkg-extra-version=-gentoo' '--enable-configfile-mask=0640' '--enable-logfile-mask=0640' '--libdir=/usr/lib/frr' '--sbindir=/usr/lib/frr' '--libexecdir=/usr/lib/frr' '--sysconfdir=/etc/frr' '--localstatedir=/run/frr' '--with-moduledir=/usr/lib/frr/modules' '--enable-user=frr' '--enable-group=frr' '--enable-vty-group=frr' '--enable-multipath=64' '--disable-doc' '--disable-fpm' '--disable-grpc' '--enable-realms' '--disable-nhrpd' '--disable-rpki' '--disable-snmp' 'build_alias=x86_64-pc-linux-gnu' 'host_alias=x86_64-pc-linux-gnu' 'PKG_CONFIG_PATH=/var/tmp/portage/net-misc/frr-10.3.3/temp/python3.13/pkgconfig' 'PYTHON=/usr/bin/python3.13'


FRRouting 10.5.3-gentoo (cerberus) on Linux(6.9.3-uls).
Copyright 1996-2005 Kunihiro Ishiguro, et al.
configured with:
    '--prefix=/usr' '--build=x86_64-pc-linux-gnu' '--host=x86_64-pc-linux-gnu' '--mandir=/usr/share/man' '--infodir=/usr/share/info' '--datadir=/usr/share' '--sysconfdir=/etc' '--localstatedir=/var/lib' '--datarootdir=/usr/share' '--disable-dependency-tracking' '--disable-silent-rules' '--disable-static' '--docdir=/usr/share/doc/frr-10.5.3' '--htmldir=/usr/share/doc/frr-10.5.3/html' '--with-sysroot=/' 'ac_cv_prog_VALGRIND_CHECK=no' 'LEX=flex' '--with-pkg-extra-version=-gentoo' '--enable-configfile-mask=0640' '--enable-logfile-mask=0640' '--libdir=/usr/lib/frr' '--sbindir=/usr/lib/frr' '--libexecdir=/usr/lib/frr' '--sysconfdir=/etc/frr' '--localstatedir=/run/frr' '--with-moduledir=/usr/lib/frr/modules' '--enable-user=frr' '--enable-group=frr' '--enable-vty-group=frr' '--enable-multipath=64' '--disable-doc' '--disable-fpm' '--disable-grpc' '--enable-realms' '--disable-nhrpd' '--disable-rpki' '--disable-snmp' 'build_alias=x86_64-pc-linux-gnu' 'host_alias=x86_64-pc-linux-gnu' 'PKG_CONFIG_PATH=/var/tmp/portage/net-misc/frr-10.5.3/temp/python3.13/pkgconfig' 'PYTHON=/usr/bin/python3.13'
```

### How to reproduce

Ensure that many, many route updates are queued from bgpd to zebra.

### Expected behavior

Don't kill routing processes unless they are actually dead.

### Actual behavior

watchfrr causes all routing processes to be killed with SIGINT, zebra takes long to tear down, before watchfrr.sh restarts routing processes.

### Additional context

_No response_

### Checklist

- [x] I have searched the open issues for this bug.
- [x] I have not included sensitive information in this report.

