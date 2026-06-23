# Pulse Demo Issue Artifact

Fork issue: https://github.com/MsysTechnologiesllc/frr/issues/21
Fork issue number: #21
Imported upstream issue: FRRouting/frr#12929
Seed date: 2026-06-23
Label: pulse-demo-seed

## Title

[Pulse demo][2026-06-23] debug commands in config mode not in running configuration

## Imported Issue Body

        Pulse demo seed imported this issue from `FRRouting/frr` on `2026-06-23`.

        Original issue: https://github.com/FRRouting/frr/issues/12929
        Original number: #12929
        Original author: @mruprich
        Original labels: triage
        Original created: 2023-03-02T09:50:06Z
        Original updated: 2023-07-04T15:17:56Z

        ---

        **Describe the bug**
Various debug options can be entered either in the main configuration shell OR in the global config mode. The problem is that only commands entered in the global config show up in the running configuration. That means that when entering the debug options in the main shell, issuing 'copy run start' or 'write terminal' will not save these options in the frr.conf file. At the same time, if I enter the debug command in the global config mode and then disable it in the main shell, debugging is disabled but the command stays in running config.

This could result in two states:
1. Either I enable debug in the main shell, save the running config, restart frr and the debugging is not enabled anymore because it is not in the running configuration.
OR
2. I disable the debugging in the main shell, the command actually stays in the running config, I save the config and restart and the debugging is still on even when I disabled it.

I am not sure if this is the case for all the debug options but I can check that if there is a will to look into this. I noticed this while using debug commands for bgp so I am using it as an example.

- [x] Did you check if this is a duplicate issue?
- [x] Did you test it on the latest FRRouting/frr master branch?

Tested with 8.4.2, not the master version.

**To Reproduce**

1. I am using debug bgp neighbor-events commands as an example:
```
R1# sh run | include debug  <--- no debugging enabled at this point
R1# debug bgp neighbor-events 
BGP neighbor-events debugging is on
R1# sh debugging | include BGP
BGP debugging status:
  BGP neighbor-events debugging is on
R1# sh run | include debug
R1#  <---- no debug command in the running configuration
```
2. Let's move to the global config and enter the same debug command:
```
R1# conf t
R1(config)# debug bgp neighbor-events <--- btw. no output saying that debugging has been enabled
R1(config)# do sh run | include debug
debug bgp neighbor-events
R1(config)# do sh debugging | include BGP  <--- debugging still on, this is ok
BGP debugging status:
  BGP neighbor-events debugging is on
R1(config)# no debug bgp neighbor-events <--- again, no message about disabling debugging
R1(config)# do sh debugging | include BGP
BGP debugging status:
R1(config)# do sh run | include debug <--- removed from the running config
R1(config)#
```
3. Let's enable debugging in global config and disable it in the main shell:
```
R1(config)# debug bgp neighbor-events
R1(config)# do sh run | include debug
debug bgp neighbor-events
R1(config)# do sh debugging | include BGP
BGP debugging status:
  BGP neighbor-events debugging is on
R1(config)# end
R1# no debug bgp neighbor-events 
BGP neighbor-events debugging is off
R1# sh debugging | include BGP 
BGP debugging status: <--- empty, that is expected
R1# sh run | include debug
debug bgp neighbor-events  <--- debug option still in the running configuration
R1#
```

**Expected behavior**

I would probably expect some consistency. Either both command modes should do the same or the debug options should be entered just in one of the modes.

**Versions**

<!-- e.g. Fedora 24, Debian 10] -->
 - OS Version: Fedora 37
<!-- [e.g. Linux 5.4, OpenBSD 6.6] -->
 - Kernel: 6.1.14-200.fc37.x86_64
<!-- e.g. 6.0, 7.4 -->
 - FRR Version: 8.4.2

