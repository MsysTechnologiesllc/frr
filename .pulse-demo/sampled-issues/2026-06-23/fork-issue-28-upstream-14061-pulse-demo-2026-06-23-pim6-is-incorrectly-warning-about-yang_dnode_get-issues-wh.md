# Pulse Demo Issue Artifact

Fork issue: https://github.com/MsysTechnologiesllc/frr/issues/28
Fork issue number: #28
Imported upstream issue: FRRouting/frr#14061
Seed date: 2026-06-23
Label: pulse-demo-seed

## Title

[Pulse demo][2026-06-23] pim6 is incorrectly warning about yang_dnode_get issues when issuing `ipv6 pim rp ...` commands

## Imported Issue Body

        Pulse demo seed imported this issue from `FRRouting/frr` on `2026-06-23`.

        Original issue: https://github.com/FRRouting/frr/issues/14061
        Original number: #14061
        Original author: @donaldsharp
        Original labels: triage, pim, pimv6
        Original created: 2023-07-19T20:36:34Z
        Original updated: 2024-05-15T01:53:01Z

        ---

        pim6 is complaining about yang_dnode_get whenever a single rp get's multiple ranges of multicast addresses to handle:

```sharpd@eva ~/p/r/s/test_modify_mld_max_query_response_timer_p0> sudo /usr/lib/frr/pim6d --log stdout --log-level debug
2023/07/19 16:27:12.805646 PIM6: [JBNZN-EVZ5D] VRF Created: default(0)
2023/07/19 16:27:12.805657 PIM6: [QF5NQ-EJ8X3] pim_vrf_enable: for default 0
2023/07/19 16:27:12.806818 PIM6: [WGYFP-JK6RV] zclient_lookup_sched_now: zclient lookup immediate connection scheduled
2023/07/19 16:27:12.806821 PIM6: [PK8MV-NK1RX] zclient_lookup_new: zclient lookup socket initialized
2023/07/19 16:27:12.806947 PIM6: [T83RR-8SM5G] pim6d 9.1-dev starting: vty@2622
2023/07/19 16:27:36.950885 PIM6: [YXPK3-1W6CV][EC 100663326] yang_dnode_get: found 2 elements (expected 0 or 1) [xpath /frr-routing:routing/control-plane-protocols/control-plane-protocol[type='frr-pim:pimd'][name='pim'][vrf='default']/frr-pim:pim/address-family[address-family='frr-routing:ipv6']/frr-pim-rp:rp/static-rp/rp-list[rp-address='2001:db8:f::4:17']/group-list]
^Zfish: Job 3, 'sudo /usr/lib/frr/pim6d --log s…' has stopped
sharpd@eva ~/p/r/s/test_modify_mld_max_query_response_timer_p0> bg
Send job 3 “sudo /usr/lib/frr/pim6d --log stdout --log-level debug” to background
sharpd@eva ~/p/r/s/test_modify_mld_max_query_response_timer_p0> sudo /usr/lib/frr/pimd --log stdout --log-level debug --daemon
2023/07/19 16:29:27.236401 PIM: [JBNZN-EVZ5D] VRF Created: default(0)
2023/07/19 16:29:27.236414 PIM: [QF5NQ-EJ8X3] pim_vrf_enable: for default 0
2023/07/19 16:29:27.238995 PIM: [WGYFP-JK6RV] zclient_lookup_sched_now: zclient lookup immediate connection scheduled
2023/07/19 16:29:27.238999 PIM: [PK8MV-NK1RX] zclient_lookup_new: zclient lookup socket initialized
2023/07/19 16:29:27.239552 PIM: [T83RR-8SM5G] pimd 9.1-dev starting: vty@2611
sharpd@eva ~/p/r/s/test_modify_mld_max_query_response_timer_p0> bg
bg: There are no suitable jobs
sharpd@eva ~/p/r/s/test_modify_mld_max_query_response_timer_p0 [1]> 2023/07/19 16:32:51.853457 PIM6: [YXPK3-1W6CV][EC 100663326] yang_dnode_get: found 3 elements (expected 0 or 1) [xpath /frr-routing:routing/control-plane-protocols/control-plane-protocol[type='frr-pim:pimd'][name='pim'][vrf='default']/frr-pim:pim/address-family[address-family='frr-routing:ipv6']/frr-pim-rp:rp/static-rp/rp-list[rp-address='2001:db8:f::4:17']/group-list]


```

v4 does not

```
eva(config)# do show run
Building configuration...

Current configuration:
!
frr version 9.1-dev
frr defaults traditional
hostname eva
log timestamp precision 6
no ip forwarding
no ipv6 forwarding
ip pim rp 1.2.3.4 229.1.1.1/32
ip pim rp 1.2.3.4 229.1.1.2/32
ipv6 pim rp 2001:db8:f::4:17 ffbb::4/128
ipv6 pim rp 2001:db8:f::4:17 ffbb::5/128
ipv6 pim rp 2001:db8:f::4:17 ffbb::6/128
service integrated-vtysh-config
!
end
```


