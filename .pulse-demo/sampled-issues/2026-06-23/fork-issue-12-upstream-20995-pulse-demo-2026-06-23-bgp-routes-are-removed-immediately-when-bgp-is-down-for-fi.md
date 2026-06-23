# Pulse Demo Issue Artifact

Fork issue: https://github.com/MsysTechnologiesllc/frr/issues/12
Fork issue number: #12
Imported upstream issue: FRRouting/frr#20995
Seed date: 2026-06-23
Label: pulse-demo-seed

## Title

[Pulse demo][2026-06-23] bgp: routes are removed immediately when BGP is down for first time on the peer side without honoring the GR restart timer. The subsequent BGP DOWN/UP is working fine with GR settings

## Imported Issue Body

        Pulse demo seed imported this issue from `FRRouting/frr` on `2026-06-23`.

        Original issue: https://github.com/FRRouting/frr/issues/20995
        Original number: #20995
        Original author: @mobalakr
        Original labels: none
        Original created: 2026-03-03T09:45:37Z
        Original updated: 2026-03-25T23:01:42Z

        ---

        I am testing GR functionality with FRR version 10.4.1.

I have a test case where I bring down the BGP on Peer A and the expectation is Peer B should start the restart timer and should hold the routes till the timer expiry. But the routes are removed immediately on peer B instead of honoring the GR timer.

FRR version:
===========
vtysh -c "show version"
FRRouting 10.4.1

BGP config:
==========
router bgp 65100
 bgp router-id 192.0.1.1
 bgp log-neighbor-changes
 no bgp default ipv4-unicast
 bgp graceful-restart restart-time 300
 bgp graceful-restart
 bgp graceful-restart preserve-fw-state
 bgp bestpath compare-routerid
 neighbor G1 peer-group
 neighbor G1 remote-as 65100
 neighbor G1 timers 3 9
 neighbor QT1 peer-group
 neighbor QT1 remote-as 65200
 neighbor QT1 timers 3 9
 neighbor 10.1.0.9 peer-group G1
 neighbor 10.1.0.9 description default|10.1.0.9
 neighbor 10.1.0.9 update-source 10.1.0.8
 neighbor 10.1.0.11 peer-group G1
 neighbor 10.1.0.11 description default|10.1.0.11
 neighbor 10.1.0.11 update-source 10.1.0.10
 neighbor 10.1.0.1 peer-group QT1
 neighbor 10.1.0.1 description default|10.1.0.1
 neighbor 10.1.0.1 update-source 10.1.0.0
 neighbor 10.1.0.3 peer-group QT1
 neighbor 10.1.0.3 description default|10.1.0.3
 neighbor 10.1.0.3 update-source 10.1.0.2
 !


Before BGP down on peer A:
======================
 show ip route
Codes: K - kernel route, C - connected, L - local, S - static,
       I - IS-IS, B - BGP, E - EIGRP, N - NHRP, T - Table,
       v - VNC, V - VNC-Direct, A - Babel, F - PBR,
       f - OpenFabric, t - Table-Direct,
       > - selected route, * - FIB route, q - queued, r - rejected, b - backup
       t - trapped, o - offload failure

IPv4 unicast VRF default:
C>* 10.1.0.0/31 is directly connected, Ethernet256, weight 1, 00:02:47
C>* 10.1.0.2/31 is directly connected, Ethernet264, weight 1, 00:02:47
B>* 10.1.0.4/31 [20/0] via 10.1.0.1, Ethernet256, weight 1, 00:02:23
B>* 10.1.0.6/31 [20/0] via 10.1.0.3, Ethernet264, weight 1, 00:02:36
C>* 10.1.0.8/31 is directly connected, Ethernet0, weight 1, 00:02:47
C>* 10.1.0.10/31 is directly connected, Ethernet8, weight 1, 00:02:47
B>* 10.1.0.12/31 [20/0] via 10.1.0.1, Ethernet256, weight 1, 00:02:23
  *                     via 10.1.0.3, Ethernet264, weight 1, 00:02:23
B>* 10.1.0.14/31 [20/0] via 10.1.0.1, Ethernet256, weight 1, 00:02:23
  *                     via 10.1.0.3, Ethernet264, weight 1, 00:02:23
B>* 192.0.2.2/32 [20/0] via 10.1.0.1, Ethernet256, weight 1, 00:02:23
B>* 192.0.2.3/32 [20/0] via 10.1.0.3, Ethernet264, weight 1, 00:02:36
C>* 192.0.2.4/32 is directly connected, Loopback0, weight 1, 00:02:47
B>* 192.0.2.5/32 [20/0] via 10.1.0.1, Ethernet256, weight 1, 00:02:23
  *                     via 10.1.0.3, Ethernet264, weight 1, 00:02:23

After BGP down on Peer A
=====================

show ip route
Codes: K - kernel route, C - connected, L - local, S - static,
       I - IS-IS, B - BGP, E - EIGRP, N - NHRP, T - Table,
       v - VNC, V - VNC-Direct, A - Babel, F - PBR,
       f - OpenFabric, t - Table-Direct,
       > - selected route, * - FIB route, q - queued, r - rejected, b - backup
       t - trapped, o - offload failure

IPv4 unicast VRF default:
C>* 10.1.0.0/31 is directly connected, Ethernet256, weight 1, 00:03:13
C>* 10.1.0.2/31 is directly connected, Ethernet264, weight 1, 00:03:13
B>* 10.1.0.4/31 [20/0] via 10.1.0.3, Ethernet264, weight 1, 00:00:16
B>* 10.1.0.6/31 [20/0] via 10.1.0.3, Ethernet264, weight 1, 00:03:02
C>* 10.1.0.8/31 is directly connected, Ethernet0, weight 1, 00:03:13
C>* 10.1.0.10/31 is directly connected, Ethernet8, weight 1, 00:03:13
B>* 10.1.0.12/31 [20/0] via 10.1.0.3, Ethernet264, weight 1, 00:00:16
B>* 10.1.0.14/31 [20/0] via 10.1.0.3, Ethernet264, weight 1, 00:00:16
B>* 192.0.2.3/32 [20/0] via 10.1.0.3, Ethernet264, weight 1, 00:03:02
C>* 192.0.2.4/32 is directly connected, Loopback0, weight 1, 00:03:13
B>* 192.0.2.5/32 [20/0] via 10.1.0.3, Ethernet264, weight 1, 00:00:16


This issue happens only for the first time when BGP is brought down on peer A. For the next iterations of BGP DOWN/UP, the routes are not removed as expected with GR timer.

Any reason why this issue happens? Any help resolving/debugging the issue?

