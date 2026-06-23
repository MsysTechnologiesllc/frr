# Pulse Demo Issue Artifact

Fork issue: https://github.com/MsysTechnologiesllc/frr/issues/15
Fork issue number: #15
Imported upstream issue: FRRouting/frr#9185
Seed date: 2026-06-23
Label: pulse-demo-seed

## Title

[Pulse demo][2026-06-23] Kernel routes missed in case of no gateway IP address 

## Imported Issue Body

        Pulse demo seed imported this issue from `FRRouting/frr` on `2026-06-23`.

        Original issue: https://github.com/FRRouting/frr/issues/9185
        Original number: #9185
        Original author: @vgrebenschikov
        Original labels: zebra
        Original created: 2021-07-26T13:31:11Z
        Original updated: 2025-12-19T20:22:51Z

        ---

        **Describe the bug**
[x] Did you check if this is a duplicate issue?
[x] Did you test it on the latest FRRouting/frr master branch?


**To Reproduce**
Kernel routes are not added into zebra list if route has device as gateway:

before:
```
srv# show ip route kernel
...
K>* 172.22.4.0/24 [0/0] via 172.22.4.2, 00:04:42
K>* 172.23.0.0/16 [0/0] via 172.22.4.2, 00:04:42
K>* 172.24.0.0/23 [0/0] via 172.22.4.2, 00:04:42
srv#
```

Then add route (in fact it is as wireguard scripts add it):
```
# route add 172.22.9.0/24 -iface wg0
add net 172.22.9.0: gateway wg0
```

And it is pretty usual for ptp/tunnel interfaces to add route just to interface, not on gateway IP

After that - nothing changed, route did not appeared:
```
srv# show ip route kernel
...
K>* 172.22.4.0/24 [0/0] via 172.22.4.2, 00:07:47
K>* 172.23.0.0/16 [0/0] via 172.22.4.2, 00:07:47
K>* 172.24.0.0/23 [0/0] via 172.22.4.2, 00:07:47
srv#
```

While that route monitor shows route message:
```
#  route -n monitor

got message of size 240 on Mon Jul 26 16:24:03 2021
RTM_ADD: Add Route: len 240, pid: 4356, seq 1, errno 0, flags:<UP,DONE,STATIC>
locks:  inits:
sockaddrs: <DST,GATEWAY,NETMASK>
 172.22.9.0 wg0 255.255.255.0 
```

One can notice that interface name (wg0) is sent in GATEWAY part of message

If we take a look at https://github.com/FRRouting/frr/blob/master/zebra/kernel_socket.c#L778
It is clear that for gateway we expect here only IP address, not interface name 

**Expected behavior**
Route appeared as kernel and then available for redistribution


**Versions**
- FRR: frr7-7.5.1_1
- OS: FreeBSD 12.2-STABLE r369379 amd64
- Kernel: 12.2-STABLE r369379 amd64


**Additional context**
Pretty basic PR which fixes the problem in Quagga
https://github.com/Quagga/quagga/pull/5


