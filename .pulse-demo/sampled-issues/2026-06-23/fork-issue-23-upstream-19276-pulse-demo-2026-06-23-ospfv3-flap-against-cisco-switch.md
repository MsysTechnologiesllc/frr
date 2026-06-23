# Pulse Demo Issue Artifact

Fork issue: https://github.com/MsysTechnologiesllc/frr/issues/23
Fork issue number: #23
Imported upstream issue: FRRouting/frr#19276
Seed date: 2026-06-23
Label: pulse-demo-seed

## Title

[Pulse demo][2026-06-23] OSPFv3 flap against Cisco switch

## Imported Issue Body

        Pulse demo seed imported this issue from `FRRouting/frr` on `2026-06-23`.

        Original issue: https://github.com/FRRouting/frr/issues/19276
        Original number: #19276
        Original author: @sesse
        Original labels: triage, ospfv3
        Original created: 2025-07-26T12:37:34Z
        Original updated: 2026-01-26T12:16:47Z

        ---

        ### Description

Hi,

I've got FRR 10.3, talking OSPFv2 and OSPFv3 (I don't think FRR supports running IPv4 over OSPFv3, otherwise I'd run both over the same protocol) to a Cisco 3650 with IOS-XE. (The FRR machine is connected directly into the switch over an 802.1q trunk, and they talk on a link VLAN not used by anything else. There are no other routers involved right now.) The OSPFv2 connection is stable, but the OSPFv3 connection flaps every minute or so, with repeated messages like this:

```
Jul 26 12:26:40.733: %OSPFv3-5-ADJCHG: Process 1, IPv6, Nbr 10.0.0.1 on Vlan102 from LOADING to FULL, Loading Done
```

I.e., it goes from LOADING to FULL without having really ever specified that it goes from FULL to LOADING. Enabling debugging on the Cisco says that what is happening is “Non-duplicate DBD packet from Nbr after exchange has finished”:

```
Jul 26 12:28:12.598: OSPFv3-1-IPv6 HELLO Vl102: Rcv hello from 10.0.0.1 area 0 from FE80::ACEE:A1FF:FE94:2F7E interface ID 10
Jul 26 12:28:14.551: OSPFv3-1-IPv6 HELLO Vl101: Send hello to FF02::5 area 0 from FE80::EA65:49FF:FE12:6641 interface ID 38
Jul 26 12:28:21.245: OSPFv3-1-IPv6 HELLO Vl102: Send hello to FF02::5 area 0 from FE80::EA65:49FF:FE12:664D interface ID 39
Jul 26 12:28:22.599: OSPFv3-1-IPv6 HELLO Vl102: Rcv hello from 10.0.0.1 area 0 from FE80::ACEE:A1FF:FE94:2F7E interface ID 10
Jul 26 12:28:23.716: OSPFv3-1-IPv6 HELLO Vl101: Send hello to FF02::5 area 0 from FE80::EA65:49FF:FE12:6641 interface ID 38
Jul 26 12:28:30.537: OSPFv3-1-IPv6 HELLO Vl102: Send hello to FF02::5 area 0 from FE80::EA65:49FF:FE12:664D interface ID 39
Jul 26 12:28:30.539: OSPFv3-1-IPv6 ADJ   Vl102: Rcv DBD from 10.0.0.1 seq 0x40A1CA opt 0x413 flag 0x7 len 28 mtu 1500 state FULL
Jul 26 12:28:30.540: OSPFv3-1-IPv6 ADJ   Vl102: Non-duplicate DBD packet from Nbr 10.0.0.1 after exchange has finished
Jul 26 12:28:30.540: OSPFv3-1-IPv6 ADJ   Vl102: Bad seq received from 10.0.0.1
Jul 26 12:28:30.540: OSPFv3-1-IPv6 ADJ   Vl102: Nbr 10.0.0.1: Prepare dbase exchange
Jul 26 12:28:30.540: OSPFv3-1-IPv6 ADJ   Vl102: Send DBD to 10.0.0.1 seq 0x649B0C9 opt 0x13 flag 0x7 len 28
Jul 26 12:28:30.543: OSPFv3-1-IPv6 ADJ   Vl102: Rcv DBD from 10.0.0.1 seq 0x649B0C9 opt 0x413 flag 0x0 len 388 mtu 1500 state EXSTART
Jul 26 12:28:30.543: OSPFv3-1-IPv6 ADJ   Vl102: NBR Negotiation Done. We are the MASTER
Jul 26 12:28:30.543: OSPFv3-1-IPv6 ADJ   Vl102: Nbr 10.0.0.1: Summary list built, size 18
Jul 26 12:28:30.543: OSPFv3-1-IPv6 ADJ   Vl102: Send DBD to 10.0.0.1 seq 0x649B0CA opt 0x13 flag 0x1 len 68
Jul 26 12:28:30.544: OSPFv3-1-IPv6 ADJ   Vl102: Send LS REQ to 10.0.0.1 length 28
Jul 26 12:28:30.546: OSPFv3-1-IPv6 ADJ   Vl102: Rcv LS REQ from 10.0.0.1 length 28 LSA count 1
Jul 26 12:28:30.547: OSPFv3-1-IPv6 ADJ   Vl102: Send LS UPD to FE80::ACEE:A1FF:FE94:2F7E length 60 LSA count 1
Jul 26 12:28:30.547: OSPFv3-1-IPv6 ADJ   Vl102: Rcv DBD from 10.0.0.1 seq 0x649B0CA opt 0x413 flag 0x0 len 28 mtu 1500 state EXCHANGE
Jul 26 12:28:30.547: OSPFv3-1-IPv6 ADJ   Vl102: Exchange Done with 10.0.0.1
Jul 26 12:28:30.548: OSPFv3-1-IPv6 ADJ   Vl102: Rcv LS UPD from Nbr ID 10.0.0.1 length 44 LSA count 1
Jul 26 12:28:30.548: OSPFv3-1-IPv6 ADJ   Vl102: Synchronized with 10.0.0.1, state FULL
Jul 26 12:28:30.548: %OSPFv3-5-ADJCHG: Process 1, IPv6, Nbr 10.0.0.1 on Vlan102 from LOADING to FULL, Loading Done
```

Corresponding debug logs from FRR at this time:

```
2025-07-26 14:28:30.534 [DEBG] ospf6d: [HEXF0-KA35R] Hello received on vlan102
2025-07-26 14:28:30.534 [DEBG] ospf6d: [WT04E-NR4JP]     src: fe80::ea65:49ff:fe12:664d
2025-07-26 14:28:30.534 [DEBG] ospf6d: [K2TW0-CYAQX]     dst: ff02::5
2025-07-26 14:28:30.534 [DEBG] ospf6d: [GNVTE-A64WG]     OSPFv3 Type:1 Len:40 Router-ID:10.0.110.1
2025-07-26 14:28:30.534 [DEBG] ospf6d: [RA6WJ-XZB2C]     Area-ID:0.0.0.0 Cksum:0 Instance-ID:0
2025-07-26 14:28:30.534 [DEBG] ospf6d: [K6N0H-3Q3W7]     I/F-Id:39 Priority:1 Option:AT|-|--|-|-|--|R|-|--|E|V6
2025-07-26 14:28:30.534 [DEBG] ospf6d: [MFC3G-8KW9N]     HelloInterval:10 DeadInterval:40
2025-07-26 14:28:30.534 [DEBG] ospf6d: [SMPBA-YB7BT]     DR:0.0.0.0 BDR:0.0.0.0
2025-07-26 14:28:30.534 [DEBG] ospf6d: [TKWAM-D4JS9]     Neighbor: 10.0.0.1
2025-07-26 14:28:30.534 [DEBG] ospf6d: [YS8BA-ZHSRC] DbDesc send on vlan102
2025-07-26 14:28:30.534 [DEBG] ospf6d: [WT04E-NR4JP]     src: fe80::acee:a1ff:fe94:2f7e
2025-07-26 14:28:30.534 [DEBG] ospf6d: [K2TW0-CYAQX]     dst: ff02::5
2025-07-26 14:28:30.534 [DEBG] ospf6d: [GNVTE-A64WG]     OSPFv3 Type:2 Len:28 Router-ID:10.0.0.1
2025-07-26 14:28:30.534 [DEBG] ospf6d: [RA6WJ-XZB2C]     Area-ID:0.0.0.0 Cksum:0 Instance-ID:0
2025-07-26 14:28:30.534 [DEBG] ospf6d: [X6WJ4-NN478]     MBZ: 0 Option: AT|-|--|-|-|--|R|-|--|E|V6 IfMTU: 1500
2025-07-26 14:28:30.534 [DEBG] ospf6d: [Q0N2C-MRJQ6]     MBZ: 0 Bits: IMm SeqNum: 0x40a1ca
2025-07-26 14:28:30.538 [DEBG] ospf6d: [HEXF0-KA35R] DbDesc received on vlan102
2025-07-26 14:28:30.538 [DEBG] ospf6d: [WT04E-NR4JP]     src: fe80::ea65:49ff:fe12:664d
2025-07-26 14:28:30.538 [DEBG] ospf6d: [K2TW0-CYAQX]     dst: fe80::acee:a1ff:fe94:2f7e
2025-07-26 14:28:30.538 [DEBG] ospf6d: [GNVTE-A64WG]     OSPFv3 Type:2 Len:28 Router-ID:10.0.110.1
2025-07-26 14:28:30.538 [DEBG] ospf6d: [RA6WJ-XZB2C]     Area-ID:0.0.0.0 Cksum:0 Instance-ID:0
2025-07-26 14:28:30.538 [DEBG] ospf6d: [X6WJ4-NN478]     MBZ: 0 Option: AT|-|--|-|-|--|R|-|--|E|V6 IfMTU: 1500
2025-07-26 14:28:30.538 [DEBG] ospf6d: [Q0N2C-MRJQ6]     MBZ: 0 Bits: IMm SeqNum: 0x649b0c9
2025-07-26 14:28:30.538 [DEBG] ospf6d: [YS8BA-ZHSRC] DbDesc send on vlan102
2025-07-26 14:28:30.538 [DEBG] ospf6d: [WT04E-NR4JP]     src: fe80::acee:a1ff:fe94:2f7e
2025-07-26 14:28:30.538 [DEBG] ospf6d: [K2TW0-CYAQX]     dst: ff02::5
2025-07-26 14:28:30.538 [DEBG] ospf6d: [GNVTE-A64WG]     OSPFv3 Type:2 Len:388 Router-ID:10.0.0.1
2025-07-26 14:28:30.538 [DEBG] ospf6d: [RA6WJ-XZB2C]     Area-ID:0.0.0.0 Cksum:0 Instance-ID:0
2025-07-26 14:28:30.538 [DEBG] ospf6d: [X6WJ4-NN478]     MBZ: 0 Option: AT|-|--|-|-|--|R|-|--

[truncated for Pulse demo seed]

