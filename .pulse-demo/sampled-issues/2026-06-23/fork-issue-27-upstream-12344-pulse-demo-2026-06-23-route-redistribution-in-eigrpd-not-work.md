# Pulse Demo Issue Artifact

Fork issue: https://github.com/MsysTechnologiesllc/frr/issues/27
Fork issue number: #27
Imported upstream issue: FRRouting/frr#12344
Seed date: 2026-06-23
Label: pulse-demo-seed

## Title

[Pulse demo][2026-06-23] route redistribution in EIGRPD not work

## Imported Issue Body

        Pulse demo seed imported this issue from `FRRouting/frr` on `2026-06-23`.

        Original issue: https://github.com/FRRouting/frr/issues/12344
        Original number: #12344
        Original author: @Churrofighter
        Original labels: triage
        Original created: 2022-11-18T19:29:08Z
        Original updated: 2023-12-02T06:47:30Z

        ---

        tested on master commit [33dfcd7](https://github.com/FRRouting/frr/commit/33dfcd73978647961b578a2348cd6eb931d40713)

When redistribution for PROTOCOL(BGP, ISIS, rip,...) configure in EIGRP, EIGRP requests PROTOCOL routes from Zebra and then Zebra sends the routes. At this moment a ZEBRA_REDISTRIBUTE_ROUTE_ADD message receives from zebra and EIGRP handles this message in the eigrp_zebra_read_route function in the eigrp_zebra.c file.

because this function didn't compliment completely, these routes don't add and don't send for peers.

