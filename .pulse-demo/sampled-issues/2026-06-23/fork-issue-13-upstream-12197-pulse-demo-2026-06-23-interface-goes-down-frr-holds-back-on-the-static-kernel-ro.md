# Pulse Demo Issue Artifact

Fork issue: https://github.com/MsysTechnologiesllc/frr/issues/13
Fork issue number: #13
Imported upstream issue: FRRouting/frr#12197
Seed date: 2026-06-23
Label: pulse-demo-seed

## Title

[Pulse demo][2026-06-23] Interface goes down FRR holds back on the static kernel routes

## Imported Issue Body

        Pulse demo seed imported this issue from `FRRouting/frr` on `2026-06-23`.

        Original issue: https://github.com/FRRouting/frr/issues/12197
        Original number: #12197
        Original author: @maisheri
        Original labels: triage, staticd
        Original created: 2022-10-25T13:13:18Z
        Original updated: 2023-07-23T18:15:20Z

        ---

        When an interface goes down / brought down with 4.x Linux based kernel, the static default routes associated with the respective interfaces are taken down from the kernel, while FRR / Zebra (8.3.1) holds back the static default routes.

Following are the steps followed:
1. Static default route configured
2. Interface administratively down
3. Check the kernel routes, kernel cleans up the kernel routes.
4. Check routes in FRR, FRR/Zebra holds the default routes.

**Curious to know the primary reason to holding back on to the default route entry in FRR/Zebra, when kernel route was deleted?**

**Also, if there would be any knob to make sure the kernel routes and FRR routes are synced at regular intervals without restarting FRR ?**

### Step 1 :

```
root@localhost:/# sudo ip route add default src 10.223.104.111 metric 5 nexthop via 172.31.94.40 dev p0 weight 1 nexthop via 172.31.94.168 dev p1 weight 1
root@localhost:/# sudo ip route add 10.223.104.154 src 10.223.104.111 metric 5 nexthop via 172.31.94.40 dev p0 weight 1 nexthop via 172.31.94.168 dev p1 weight 1

root@localhost:/# ip r | grep default
default src 10.223.104.111 metric 5
root@localhost:/# vtysh -c "show ip route" | grep -A 3 0.0.0.0
K>* 0.0.0.0/0 [0/5] via 172.31.94.40, p0, src 10.223.104.111, weight 1, 00:00:38
 *         via 172.31.94.168, p1, src 10.223.104.111, weight 1, 00:00:38
B  0.0.0.0/0 [20/10] via 172.31.94.40, p0, weight 1, 00:09:58
           via 172.31.94.168, p1, weight 1, 00:09:58
root@localhost:/# ip r | grep 10.223.104.154
10.223.104.154 src 10.223.104.111 metric 5
root@localhost:/# vtysh -c "show ip route" | grep -A 3 10.223.104.154
K>* 10.223.104.154/32 [0/5] via 172.31.94.40, p0, src 10.223.104.111, weight 1, 00:00:50
 *             via 172.31.94.168, p1, src 10.223.104.111, weight 1, 00:00:50
B  10.223.104.154/32 [20/0] via 172.31.94.40, p0, weight 1, 00:10:18
               via 172.31.94.168, p1, weight 1, 00:10:18
```

### Step 2:

```
root@localhost:/# ip link set p0 down
root@localhost:/# ip link set p1 down
```

### Step 3:

```
Kernel cleans up the associated route.
root@localhost:/# ip r | grep default
root@localhost:/#
```

### Step 4:

```
root@localhost:/# vtysh -c "show ip route" | grep -A 3 0.0.0.0
K * 0.0.0.0/0 [0/5] via 172.31.94.40, p0 inactive, src 10.223.104.111, weight 1, 00:01:57
 *         via 172.31.94.168, p1 inactive, src 10.223.104.111, weight 1, 00:01:57
root@localhost:/# ip r | grep 10.223.104.154
root@localhost:/# vtysh -c "show ip route" | grep -A 3 10.223.104.154
K * 10.223.104.154/32 [0/5] via 172.31.94.40, p0 inactive, src 10.223.104.111, weight 1, 00:02:18
 *             via 172.31.94.168, p1 inactive, src 10.223.104.111, weight 1, 00:02:18
```

