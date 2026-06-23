# Pulse Demo Issue Artifact

Fork issue: https://github.com/MsysTechnologiesllc/frr/issues/30
Fork issue number: #30
Imported upstream issue: FRRouting/frr#22320
Seed date: 2026-06-23
Label: pulse-demo-seed

## Title

[Pulse demo][2026-06-23] topotests: provide a migration path from ExaBGP 4.2 to 5.x

## Imported Issue Body

        Pulse demo seed imported this issue from `FRRouting/frr` on `2026-06-23`.

        Original issue: https://github.com/FRRouting/frr/issues/22320
        Original number: #22320
        Original author: @higebu
        Original labels: none
        Original created: 2026-06-11T01:05:13Z
        Original updated: 2026-06-12T23:31:08Z

        ---

        The topotests currently require ExaBGP 4.2: the in-tree installs
(tests/topotests/Dockerfile, docker/ubuntu-ci/Dockerfile,
doc/developer/topotests.rst) pin 4.2.21, and the NetDEF CI agents
provision their own 4.2.11-frr build. The 4.2 branch no longer
receives new features; newer address families such as BGP-MUP
(used by the topotest in #22311) are only available in ExaBGP 5.x.

Upstream documents the migration in
https://github.com/Exa-Networks/exabgp/wiki/From-4.x-to-5.x.
The impact on the topotests is small: only the launch code in
lib/topogen.py uses 4.x-only forms (CLI options, env file handling,
API fifo setup). The per-test configurations work on both versions
as is.

Proposal, following the exabgp 3 -> 4 migration (#14890):

1. Make lib/topogen.py detect the ExaBGP version and use the
   matching launch form, keeping the 4.x path unchanged.
2. Update the in-tree pins to ExaBGP 5.0.9.
3. Update the NetDEF CI agent provisioning. With 1. in place this
   can happen at any time, since both versions keep working.

A PR implementing 1. and 2. will follow.


