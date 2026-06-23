# Pulse Demo Issue Artifact

Fork issue: https://github.com/MsysTechnologiesllc/frr/issues/19
Fork issue number: #19
Imported upstream issue: FRRouting/frr#21245
Seed date: 2026-06-23
Label: pulse-demo-seed

## Title

[Pulse demo][2026-06-23] VRF directive not cleaned up when the configuration file is reloaded through the reloader script

## Imported Issue Body

        Pulse demo seed imported this issue from `FRRouting/frr` on `2026-06-23`.

        Original issue: https://github.com/FRRouting/frr/issues/21245
        Original number: #21245
        Original author: @jcaamano
        Original labels: triage
        Original created: 2026-03-18T15:05:36Z
        Original updated: 2026-06-16T14:44:53Z

        ---

        ### Description

When you are running with a configuration that contains a VRF directive and then  you reload a configuration that does not have it, the VRF directive remains with no content. With enough churn, all these VRF directives left behing make the the configuration file be so big that cascades to issues on downstream components reading the configuration.

[Originally reported in https://github.com/FRRouting/frr/issues/17430](https://github.com/metallb/frr-k8s/issues/417)

### Version

```text
FRRouting 10.4.1_git (frr-k8s-worker) on Linux(6.18.8-200.fc43.x86_64).
```

### How to reproduce

Apply this configuration
```
vrf bgp203
 vni 3241440
exit-vrf
!
router bgp 64512
 no bgp ebgp-requires-policy
 no bgp default ipv4-unicast
 bgp graceful-restart preserve-fw-state
 no bgp network import-check
 neighbor 172.18.0.5 remote-as 64512
 neighbor fc00:f853:ccd:e793::5 remote-as 64512
 !
 address-family ipv4 unicast
  neighbor 172.18.0.5 activate
  neighbor 172.18.0.5 route-map 172.18.0.5-in in
  neighbor 172.18.0.5 route-map 172.18.0.5-out out
 exit-address-family
 !
 address-family ipv6 unicast
  neighbor fc00:f853:ccd:e793::5 activate
  neighbor fc00:f853:ccd:e793::5 route-map fc00:f853:ccd:e793::5-in in
  neighbor fc00:f853:ccd:e793::5 route-map fc00:f853:ccd:e793::5-out out
 exit-address-family
 !
 address-family l2vpn evpn
  neighbor 172.18.0.5 activate
  advertise-all-vni
 exit-address-family
exit
!
router bgp 64512 vrf bgp203
 no bgp ebgp-requires-policy
 no bgp enforce-first-as
 no bgp hard-administrative-reset
 no bgp default ipv4-unicast
 no bgp graceful-restart notification
 bgp graceful-restart preserve-fw-state
 no bgp network import-check
 !
 address-family ipv4 unicast
  network 10.206.176.0/24
 exit-address-family
 !
 address-family ipv6 unicast
  network fd00:ceb::/64
 exit-address-family
 !
 address-family l2vpn evpn
  advertise ipv4 unicast
  advertise ipv6 unicast
  rd 64512:3241440
  route-target import 64512:3241440
  route-target export 64512:3241440
 exit-address-family
exit
```
then use reloader script to apply this configuration
```
router bgp 64512
 no bgp ebgp-requires-policy
 no bgp default ipv4-unicast
 bgp graceful-restart preserve-fw-state
 no bgp network import-check
 neighbor 172.18.0.5 remote-as 64512
 neighbor fc00:f853:ccd:e793::5 remote-as 64512
 !
 address-family ipv4 unicast
  neighbor 172.18.0.5 activate
  neighbor 172.18.0.5 route-map 172.18.0.5-in in
  neighbor 172.18.0.5 route-map 172.18.0.5-out out
 exit-address-family
 !
 address-family ipv6 unicast
  neighbor fc00:f853:ccd:e793::5 activate
  neighbor fc00:f853:ccd:e793::5 route-map fc00:f853:ccd:e793::5-in in
  neighbor fc00:f853:ccd:e793::5 route-map fc00:f853:ccd:e793::5-out out
 exit-address-family
exit
```
the resulting configuration will be
```
vrf bgp203
exit-vrf
!
router bgp 64512
 no bgp ebgp-requires-policy
 no bgp default ipv4-unicast
 bgp graceful-restart preserve-fw-state
 no bgp network import-check
 neighbor 172.18.0.5 remote-as 64512
 neighbor fc00:f853:ccd:e793::5 remote-as 64512
 !
 address-family ipv4 unicast
  neighbor 172.18.0.5 activate
  neighbor 172.18.0.5 route-map 172.18.0.5-in in
  neighbor 172.18.0.5 route-map 172.18.0.5-out out
 exit-address-family
 !
 address-family ipv6 unicast
  neighbor fc00:f853:ccd:e793::5 activate
  neighbor fc00:f853:ccd:e793::5 route-map fc00:f853:ccd:e793::5-in in
  neighbor fc00:f853:ccd:e793::5 route-map fc00:f853:ccd:e793::5-out out
 exit-address-family
exit
```

Note the leftover
```
vrf bgp203
exit-vrf
!
```

### Expected behavior

I expect the resulting configuration to be 
```
router bgp 64512
 no bgp ebgp-requires-policy
 no bgp default ipv4-unicast
 bgp graceful-restart preserve-fw-state
 no bgp network import-check
 neighbor 172.18.0.5 remote-as 64512
 neighbor fc00:f853:ccd:e793::5 remote-as 64512
 !
 address-family ipv4 unicast
  neighbor 172.18.0.5 activate
  neighbor 172.18.0.5 route-map 172.18.0.5-in in
  neighbor 172.18.0.5 route-map 172.18.0.5-out out
 exit-address-family
 !
 address-family ipv6 unicast
  neighbor fc00:f853:ccd:e793::5 activate
  neighbor fc00:f853:ccd:e793::5 route-map fc00:f853:ccd:e793::5-in in
  neighbor fc00:f853:ccd:e793::5 route-map fc00:f853:ccd:e793::5-out out
 exit-address-family
exit
```

### Actual behavior

The resulting configuration has an empty vrf directive
```
vrf bgp203
exit-vrf
!
router bgp 64512
 no bgp ebgp-requires-policy
 no bgp default ipv4-unicast
 bgp graceful-restart preserve-fw-state
 no bgp network import-check
 neighbor 172.18.0.5 remote-as 64512
 neighbor fc00:f853:ccd:e793::5 remote-as 64512
 !
 address-family ipv4 unicast
  neighbor 172.18.0.5 activate
  neighbor 172.18.0.5 route-map 172.18.0.5-in in
  neighbor 172.18.0.5 route-map 172.18.0.5-out out
 exit-address-family
 !
 address-family ipv6 unicast
  neighbor fc00:f853:ccd:e793::5 activate
  neighbor fc00:f853:ccd:e793::5 route-map fc00:f853:ccd:e793::5-in in
  neighbor fc00:f853:ccd:e793::5 route-map fc00:f853:ccd:e793::5-out out
 exit-address-family
exit
```

### Additional context

Might be a reloader issue 
https://github.com/FRRouting/frr/blob/e119526f953c05ed4d54be3836b96cd5e5b412df/tools/frr-reload.py#L1808-L1810
```
            # We cannot do 'no vrf' in FRR, and so deal with it
            elif running_ctx_keys[0].startswith("vrf") or running_ctx_keys[
                0
```
However I found no apparent issues running `no vrf` on my side
```
ovn-worker(config)# no vrf bgp2886
ovn-worker(config)# exit
ovn-worker# write
Note: this version of vtysh never writes vtysh.conf
Building Configuration...
Integrated configuration saved to /etc/frr/frr.conf
[OK]
ovn-worker# show vrf
ovn-worker#
```

### Checklist

- [x] I have searched the open issues for this bug.
- [x] I have not included sensitive information in this report.

