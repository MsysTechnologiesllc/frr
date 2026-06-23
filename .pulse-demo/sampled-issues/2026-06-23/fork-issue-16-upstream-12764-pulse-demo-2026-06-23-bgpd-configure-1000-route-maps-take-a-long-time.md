# Pulse Demo Issue Artifact

Fork issue: https://github.com/MsysTechnologiesllc/frr/issues/16
Fork issue number: #16
Imported upstream issue: FRRouting/frr#12764
Seed date: 2026-06-23
Label: pulse-demo-seed

## Title

[Pulse demo][2026-06-23] bgpd：configure 1000 route-maps take a long time

## Imported Issue Body

        Pulse demo seed imported this issue from `FRRouting/frr` on `2026-06-23`.

        Original issue: https://github.com/FRRouting/frr/issues/12764
        Original number: #12764
        Original author: @guoguojia2021
        Original labels: performance, triage
        Original created: 2023-02-08T07:53:09Z
        Original updated: 2024-07-30T08:43:58Z

        ---

        **Describe the bug**
When I test _**"route-map xxx permit xx"**_. If configure 1000 _**route-maps**_ and 1000 _**match local-perference xx**_, it will take 22m 37s. while, If I configure 1000 _**route-maps**_ and 1000 _**match ipv6 address prefix-len xx**_, it only takes 44s.  

|  route-map(1000) | match(1000)  | spend time |
|  ----  | ----  | ----  |
| route-map xxx permit x  | match local-perference xx |22m37s |
| route-map xxx permit x   | match ipv6 address prefix-len x|44s |

- [x]  Did you check if this is a duplicate issue?
- [x] Did you test it on the latest FRRouting/frr master 

**Versions**
- OS Version: Ubuntu 20.04 LTS
- Kernel: 5.4.0-135-generic
- FRR Version: v8.4.1 docker

**Additional context**
 _**route-map xxx permit x**_ and _**match local-preference x**_
```
root@guoguo:~/Documents/frrlab# date
Wed 08 Feb 2023 05:33:34 AM UTC
root@guoguo:~/Documents/frrlab# sudo docker exec -it clab-frr01-router2 vtysh
Hello, this is FRRouting (version 8.4.1_git).
Copyright 1996-2005 Kunihiro Ishiguro, et al.

router2# config
router2(config)# route-map TEST_MAP_VRF1 permit 6501 
router2(config-route-map)# match local-preference 1
router2(config-route-map)# route-map TEST_MAP_VRF2 permit 6502 
router2(config-route-map)# match local-preference 2
router2(config-route-map)# route-map TEST_MAP_VRF3 permit 6503 
router2(config-route-map)# match local-preference 3
Trouter2(config-route-map)# route-map TEST_MAP_VRF4 permit 6504 
router2(config-route-map)# match local-preference 4
router2(config-route-map)# route-map TEST_MAP_VRF5 permit 6505 
router2(config-route-map)# match local-preference 5
router2(config-route-map)# route-map TEST_MAP_VRF6 permit 6506 
router2(config-route-map)# match local-preference 6
router2(config-route-map)# route-map TEST_MAP_VRF7 permit 6507 
router2(config-route-map)# match local-preference 7
router2(config-route-map)# route-map TEST_MAP_VRF8 permit 6508 
router2(config-route-map)# match local-preference 8
router2(config-route-map)# route-map TEST_MAP_VRF9 permit 6509 
router2(config-route-map)# match local-preference 9
router2(config-route-map)# route-map TEST_MAP_VRF10 permit 6510 
router2(config-route-map)# match local-preference 10
router2(config-route-map)# route-map TEST_MAP_VRF11 permit 6511 
router2(config-route-map)# match local-preference 11
router2(config-route-map)# route-map TEST_MAP_VRF12 permit 6512 
router2(config-route-map)# match local-preference 12
.
.
.
router2(config-route-map)# route-map TEST_MAP_VRF992 permit 7492 
router2(config-route-map)# match local-preference 224
router2(config-route-map)# route-map TEST_MAP_VRF993 permit 7493 
router2(config-route-map)# match local-preference 225
router2(config-route-map)# route-map TEST_MAP_VRF994 permit 7494 
router2(config-route-map)# match local-preference 226
router2(config-route-map)# route-map TEST_MAP_VRF995 permit 7495 
router2(config-route-map)# match local-preference 227
router2(config-route-map)# route-map TEST_MAP_VRF996 permit 7496 
router2(config-route-map)# match local-preference 228
router2(config-route-map)# route-map TEST_MAP_VRF997 permit 7497 
router2(config-route-map)# match local-preference 229
router2(config-route-map)# route-map TEST_MAP_VRF998 permit 7498 
router2(config-route-map)# match local-preference 230
router2(config-route-map)# route-map TEST_MAP_VRF999 permit 7499 
router2(config-route-map)# match local-preference 231
router2(config-route-map)# route-map TEST_MAP_VRF1000 permit 7500 
router2(config-route-map)# match local-preference 232
router2(config-route-map)# end
router2# exit
root@guoguo:~/Documents/frrlab# date
Wed 08 Feb 2023 05:56:11 AM UTC
root@guoguo:~/Documents/frrlab# 
```

_**route-map xxx permit x**_ and _**match  ipv6 address prefix-len x**_
```
root@guoguo:~/Documents/frrlab# 
root@guoguo:~/Documents/frrlab# date
Wed 08 Feb 2023 06:00:33 AM UTC
root@guoguo:~/Documents/frrlab# sudo docker exec -it clab-frr01-router2 vtysh
Hello, this is FRRouting (version 8.4.1_git).
Copyright 1996-2005 Kunihiro Ishiguro, et al.

router2# config
router2(config)# route-map TEST_MAP_VRF1 permit 6501 
router2(config-route-map)# match ipv6 address prefix-len 1
router2(config-route-map)# route-map TEST_MAP_VRF2 permit 6502 
router2(config-route-map)# match ipv6 address prefix-len 2
router2(config-route-map)# route-map TEST_MAP_VRF3 permit 6503 
router2(config-route-map)# match ipv6 address prefix-len 3
router2(config-route-map)# route-map TEST_MAP_VRF4 permit 6504 
router2(config-route-map)# match ipv6 address prefix-len 4
router2(config-route-map)# route-map TEST_MAP_VRF5 permit 6505 
router2(config-route-map)# match ipv6 address prefix-len 5
router2(config-route-map)# route-map TEST_MAP_VRF6 permit 6506 
router2(config-route-map)# match ipv6 address prefix-len 6
router2(config-route-map)# route-map TEST_MAP_VRF7 permit 6507 
router2(config-route-map)# match ipv6 address prefix-len 7
router2(config-route-map)# route-map TEST_MAP_VRF8 permit 6508 
router2(config-route-map)# match ipv6 address prefix-len 8
router2(config-route-map)# route-map TEST_MAP_VRF9 permit 6509 
.
.
.
router2(config-route-map)# route-map TEST_MAP_VRF988 permit 7488 
router2(config-route-map)# match ipv6 address prefix-len 92
router2(config-route-map)# route-map TEST_MAP_VRF989 permit 7489 
router2(config-route-map)# match ipv6 address prefix-len 93
router2(config-route-map)# route-map TEST_MAP_VRF990 permit 7490 
router2(config-route-map)# match ipv6 address prefix-len 94
router2(config-route-map)# route-map TEST_MAP_VRF991 permit 7491 
router2(config-route-map)# match ipv6 address prefix-len 95
router2(config-route-map)# route-map TEST_MAP_VRF992 permit 7492 
router2(config-route-map)# match ipv6 address prefix-len 96
router2(config-route-map)# route-map TEST_MAP_VRF993 permit 7493 
router2(config-route-map)# match ipv6 address prefix-len 97
router2(config-route-map)# route-map TE

[truncated for Pulse demo seed]

