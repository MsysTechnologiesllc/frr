# Pulse Demo Issue Artifact

Fork issue: https://github.com/MsysTechnologiesllc/frr/issues/24
Fork issue number: #24
Imported upstream issue: FRRouting/frr#20627
Seed date: 2026-06-23
Label: pulse-demo-seed

## Title

[Pulse demo][2026-06-23] Feature Request: (i)BGP fall-over support with selective address tracking

## Imported Issue Body

        Pulse demo seed imported this issue from `FRRouting/frr` on `2026-06-23`.

        Original issue: https://github.com/FRRouting/frr/issues/20627
        Original number: #20627
        Original author: @vom513
        Original labels: feature-request
        Original created: 2026-01-29T16:18:01Z
        Original updated: 2026-02-03T16:11:16Z

        ---

        In summary - to track the neighbor address and evaluate it against a route-map.  This enables tracking /32 + /128 for example to very quickly tear down sessions when the loopback IP is lost due to intermediate path failure.

[Cisco IOS docs / config example](https://www.cisco.com/c/en/us/td/docs/routers/ios/config/17-x/ip-routing/b-ip-routing/m_irg-neighbor-0.html#GUID-221B3E94-DC9B-4B20-B227-EF33CAEE0AE3)

[Blog post explanation](https://networkop.co.uk/blog/2015/06/11/ibgp-fallover-trick/)

