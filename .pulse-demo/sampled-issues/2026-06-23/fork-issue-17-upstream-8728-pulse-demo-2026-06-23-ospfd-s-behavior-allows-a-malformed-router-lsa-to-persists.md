# Pulse Demo Issue Artifact

Fork issue: https://github.com/MsysTechnologiesllc/frr/issues/17
Fork issue number: #17
Imported upstream issue: FRRouting/frr#8728
Seed date: 2026-06-23
Label: pulse-demo-seed

## Title

[Pulse demo][2026-06-23] ospfd's behavior allows a malformed Router LSA to persists in the routing domain

## Imported Issue Body

        Pulse demo seed imported this issue from `FRRouting/frr` on `2026-06-23`.

        Original issue: https://github.com/FRRouting/frr/issues/8728
        Original number: #8728
        Original author: @tromai
        Original labels: triage, ospf, security
        Original created: 2021-05-25T01:04:14Z
        Original updated: 2021-06-08T15:18:33Z

        ---

        <!--

*** ATTENTION ***

YOU MUST READ THIS TO HAVE YOUR ISSUE ADDRESSED

PLEASE READ AND FILL OUT THIS TEMPLATE

NEGLECTING TO PROVIDE INFORMATION REQUESTED HERE WILL RESULT IN
SIGNIFICANT DELAYS ADDRESSING YOUR ISSUE

ALWAYS PROVIDE:
- FRR VERSION
- OPERATING SYSTEM VERSION
- KERNEL VERSION

FAILURE TO PROVIDE THIS MAY RESULT IN YOUR ISSUE BEING IGNORED

FOLLOW THESE GUIDELINES:

- When reporting a crash, provide a backtrace
- When pasting configs, logs, shell output, backtraces, and other large chunks
  of text, surround this text with triple backtics

  ```
  like this
  ```

- Include the FRR version; if you built from Git, please provide the commit
  hash
- Write your issue in English

-->

---------------

**Describe the bug**
ospfd rejects any LSA that has sequence number = `MaxSeqNumber` (0x7fffffff) and age different from `MaxAge` (3600). This is to addressed the vulnerability CVE-2017-3224. The code implementing this feature is located [here](https://github.com/FRRouting/frr/blob/978f4b32ebd8081ae95b1d8ac6a2994d48a1719d/ospfd/ospf_packet.c#L2090).
However, when a router want to generate a fight-back LSA with sequence number of `MaxSeqNumber - 1` (age obviously != `MaxAge`), this fight-back will be rejected by its neighbor routers. Therefore, the fight-back LSA is rendered useless.

[x] Did you check if this is a duplicate issue?
[] Did you test it on the latest FRRouting/frr master branch?


**To Reproduce**
When I discovered the vulnerability, this is the topology that I used
![topology_labeled](https://user-images.githubusercontent.com/33509400/119423534-d42cd380-bd46-11eb-99e5-97c21a15d954.png)

With the following configurations:
![topo_config](https://user-images.githubusercontent.com/33509400/119423653-1fdf7d00-bd47-11eb-85d4-af1e6fecd33a.png)

The vulnerability is replicated by having the attacker flood a malformed Router LSA (modified from the current `r010_1`'s router LSA) to the routing domain. This malformed Router LSA will have sequence number of MaxSeqNumber – 1 (0x7ffffffe) and age != 3600. 
When the router that originated this LSA (`r010_1`) receives the malformed LSA, it will issue a fight-back LSA. This fight-back LSA will have sequence number of `MaxSeqNumber `and age != `MaxAge`. 
However, because FRRouting’s ospfd rejects LSA with MaxSeqNumber but Age != MaxAge, this fight-back instance is rejected and will be rendered useless.

**Expected behavior**
The malformed router LSA sent by the attacker will persists in the Link State Database (LSDB) of all other router except `r010_1`. `r010_1` will keep sending the fight-back LSA because it's not accepted and acked by `r010_5`.

The attack impacts depend on the content of the malformed LSA (e.g bring down connection to `r010_1`)

**Screenshots**
The log file of `r010_5` indicating that it rejects the fight-back LSA.
```
2021/03/16 01:45:19 OSPF: DR-Election[1st]: Backup 10.255.0.14
2021/03/16 01:45:19 OSPF: DR-Election[1st]: DR     10.255.0.14
2021/03/16 01:45:19 OSPF: DR-Election[2nd]: Backup 10.255.0.13
2021/03/16 01:45:19 OSPF: DR-Election[2nd]: DR     10.255.0.14
2021/03/16 01:45:19 OSPF: interface 10.255.0.14 [2] join AllDRouters Multicast group.
2021/03/16 01:45:19 OSPF: DR-Election[1st]: Backup 10.255.0.17
2021/03/16 01:45:19 OSPF: DR-Election[1st]: DR     10.255.0.17
2021/03/16 01:45:19 OSPF: DR-Election[2nd]: Backup 10.255.0.18
2021/03/16 01:45:19 OSPF: DR-Election[2nd]: DR     10.255.0.17
2021/03/16 01:45:19 OSPF: interface 10.255.0.17 [4] join AllDRouters Multicast group.
2021/03/16 01:45:19 OSPF: Packet[DD]: Neighbor 10.10.0.4 Negotiation done (Master).
2021/03/16 01:45:19 OSPF: Packet[DD]: Neighbor 10.10.0.1 Negotiation done (Master).
2021/03/16 01:45:29 OSPF: DR-Election[1st]: Backup 10.255.0.18
2021/03/16 01:45:29 OSPF: DR-Election[1st]: DR     10.255.0.17
2021/03/16 01:45:29 OSPF: DR-Election[1st]: Backup 10.255.0.13
2021/03/16 01:45:29 OSPF: DR-Election[1st]: DR     10.255.0.14
2021/03/16 02:00:33 OSPF: Link State Update[Type1,id(10.10.0.1),ar(10.10.0.1)]: has Max Seq but not MaxAge. Dropping it
2021/03/16 02:00:40 OSPF: Link State Update[Type1,id(10.10.0.1),ar(10.10.0.1)]: has Max Seq but not MaxAge. Dropping it
2021/03/16 02:00:45 OSPF: Link State Update[Type1,id(10.10.0.1),ar(10.10.0.1)]: has Max Seq but not MaxAge. Dropping it
2021/03/16 02:00:50 OSPF: Link State Update[Type1,id(10.10.0.1),ar(10.10.0.1)]: has Max Seq but not MaxAge. Dropping it
```

`r010_1` keeps sending the rejected fight-back LSA (note that no LS Ack is issued).

![r010_1_traffic](https://user-images.githubusercontent.com/33509400/119424165-49e56f00-bd48-11eb-9e05-45ba2f927c8a.png)

**Versions**
<!-- e.g. Fedora 24, Debian 10] -->
 - OS Version: Ubuntu 18.04 LTS
<!-- [e.g. Linux 5.4, OpenBSD 6.6] -->
 - Kernel: 5.4.0-53-generic
<!-- e.g. 6.0, 7.4 -->
 - FRR Version: 7.5.1

**Additional context**
The topology is emulated with Mininet 2.3.0


