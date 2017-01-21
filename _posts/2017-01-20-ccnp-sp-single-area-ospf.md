---
layout: post
title: "Single Area OSPF in IOS (XE) and IOS XR"
description: |
  "Building a single area OSPF topology for CCNP SP preparation."
category: ccnp-sp
tags: [ccnp-sp, ios, ios-xe, ios-xr, ospf, labs, sproute]
---

> This post is a lab that I'm using as part of my CCNP SP preparations.
> It follows a scenario format, and sample configurations are provided
> throughout the various stages.

# Summary

You've recently taken on a project to deploy a small Point of Presence
for a a small service provider.  The company operates a single OSPF
topology, and you must deploy two new P routers and one new PE router in
this PoP.

# Topology

![OSPF Single Area Topology](/assets/img/ospf-single-area.png)

# Design Requirements

In order to finish this phase of the deployment, you must have the
following requirements satisfied:

* All three routers in the same OSPF area
* Loopback IP addresses reachable from all three routers
* Router IDs should be the loopback IPs
* PE1 should always prefer the path to P1 due to a current trouble with
  the connection to P2
* PE1 should always have a default route learned via OSPF from both P1
  and P2
* MD5 authentication must be used for the IPv4 OSPF process
* IPv6 connectivity is required
* Due to interoperability concerns, OSPFv3 with multiple AFIs should not
  be used for the IPv4 IGP
* The OSPF process used for IPv6 should be secured with IPSec using MD5

# Initial Configurations

## P1

```
hostname p1
interface Loopback0
  ipv4 address 1.1.1.1/32
  ipv6 address 2001:db8::1:1:1:1/128
!
interface GigabitEthernet0/0/0/2
  description "to pe1_g3"
  ipv4 address 10.0.0.2/31
  ipv6 address 2001:db8::10:0:0:2/127
!
interface GigabitEthernet0/0/0/4
  description "to p2_g6"
  ipv4 address 10.0.0.0/31
  ipv6 address 2001:db8::10:0:0:0/127
!
```

## P2

```
hostname p2
!
ipv6 unicast-routing
!
interface Loopback0
  ip address 2.2.2.2 255.255.255.255
  ipv6 address 2001:db8::2:2:2:2/128
!
interface GigabitEthernet2
  description "to pe1_g2"
  ip address 10.0.0.4 255.255.255.254
  ipv6 address 2001:db8::10:0:0:4/127
!
interface GigabitEthernet6
  description "to p1_g0/0/0/4"
  ip address 10.0.0.1 255.255.255.254
  ipv6 address 2001:db8::10:0:0:1/127
!
```

## PE1

```
hostname pe1
!
ipv6 unicast-routing
!
interface Loopback0
  ip address 3.3.3.3 255.255.255.255
  ipv6 address 2001:db8::3:3:3:3/128
!
interface GigabitEthernet2
  description "to p2_g2"
  ip address 10.0.0.5 255.255.255.254
  ipv6 address 2001:db8::10:0:0:5/127
!
interface GigabitEthernet3
  description "to p1_g0/0/0/2"
  ip address 10.0.0.3 255.255.255.254
  ipv6 address 2001:db8::10:0:0:3/127
!
```

# Solutions

## P1

```
router ospf 1
  router-id 1.1.1.1
  authentication message-digest
  message-digest-key 1 md5 cisco
  default-information originate always
  area 0.0.0.0
    interface Loopback0
      passive enable
    !
    interface GigabitEthernet0/0/0/2
      network point-to-point
    !
    interface GigabitEthernet0/0/0/4
      network point-to-point
    !
  !
!
router ospfv3 1
  router-id 1.1.1.1
  authentication ipsec spi 256 md5 ABCDEF0123456789ABCDEF0123456789
  default-information originate always
  area 0.0.0.0
    interface Loopback0
      passive
    !
    interface GigabitEthernet0/0/0/2
      network point-to-point
    !
    interface GigabitEthernet0/0/0/4
      network point-to-point
    !
  !
!
```

## P2

```
router ospf 1
  router-id 2.2.2.2
  area 0.0.0.0 authentication message-digest
  passive-interface default
  no passive-interface GigabitEthernet2
  no passive-interface GigabitEthernet6
  default-information originate always
!
router ospfv3 1
  router-id 2.2.2.2
  area 0.0.0.0 authentication ipsec spi 256 md5 ABCDEF0123456789ABCDEF0123456789
  !
  address-family ipv6 unicast
    passive-interface default
    no passive-interface GigabitEthernet3
    no passive-interface GigabitEthernet2
    default-information originate always
  exit-address-family
!
interface Loopback0
  ip ospf 1 area 0.0.0.0
  ipv6 ospf 1 area 0.0.0.0
!
interface GigabitEthernet2
  ip ospf message-digest-key 1 md5 cisco
  ip ospf network point-to-point
  ip ospf 1 area 0.0.0.0
  ip ospf cost 1000
  ipv6 ospf 1 area 0.0.0.0
  ipv6 ospf network point-to-point
  ipv6 ospf cost 1000
!
interface GigabitEthernet6
  ip ospf message-digest-key 1 md5 cisco
  ip ospf network point-to-point
  ip ospf 1 area 0.0.0.0
  ipv6 ospf 1 area 0.0.0.0
  ipv6 ospf network point-to-point
!
```

## PE1

```
router ospf 1
  router-id 3.3.3.3
  area 0.0.0.0 authentication message-digest
  passive-interface default
  no passive-interface GigabitEthernet2
  no passive-interface GigabitEthernet3
!
router ospfv3 1
  router-id 3.3.3.3
  area 0.0.0.0 authentication ipsec spi 256 md5 ABCDEF0123456789ABCDEF0123456789
  !
  address-family ipv6 unicast
    passive-interface default
    no passive-interface GigabitEthernet3
    no passive-interface GigabitEthernet2
  exit-address-family
!
interface Loopback0
  ip ospf 1 area 0.0.0.0
  ipv6 ospf 1 area 0.0.0.0
!
interface GigabitEthernet2
  ip ospf message-digest-key 1 md5 cisco
  ip ospf network point-to-point
  ip ospf 1 area 0.0.0.0
  ip ospf cost 1000
  ipv6 ospf 1 area 0.0.0.0
  ipv6 ospf network point-to-point
  ipv6 ospf cost 1000
!
interface GigabitEthernet3
  ip ospf message-digest-key 1 md5 cisco
  ip ospf network point-to-point
  ip ospf 1 area 0.0.0.0
  ipv6 ospf 1 area 0.0.0.0
  ipv6 ospf network point-to-point
!
```

# Verification

There are two ways that you can verify your lab:

* [Ansible](#Ansible)
* [Manually](#Manually)

## Ansible {#Ansible}

> Expect a future blog post detailing how this is performed.

```
$ ./test.sh ospf_single_area

PLAY [p1,p2,pe1] ***************************************************************

TASK [test_ospf_single_area : Include Vendor-Specific Tasks] *******************
p1 | SUCCESS
p2 | SUCCESS
pe1 | SUCCESS

TASK [test_ospf_single_area : iosxr_command] ***********************************
p1 | SUCCESS

TASK [test_ospf_single_area : iosxr_command] ***********************************
p1 | SUCCESS

TASK [test_ospf_single_area : iosxr_command] ***********************************
p1 | SUCCESS

TASK [test_ospf_single_area : iosxr_command] ***********************************
p1 | SUCCESS

TASK [test_ospf_single_area : ios_command] *************************************
p2 | SUCCESS
pe1 | SUCCESS

TASK [test_ospf_single_area : ios_command] *************************************
pe1 | SUCCESS
p2 | SUCCESS

TASK [test_ospf_single_area : ios_command] *************************************
pe1 | SUCCESS
p2 | SUCCESS

TASK [test_ospf_single_area : ios_command] *************************************
pe1 | SUCCESS
p2 | SUCCESS

TASK [test_ospf_single_area : Validate OSPFv2 Neighbor Count] ******************
p1 | SUCCESS
p2 | SUCCESS
pe1 | SUCCESS

TASK [test_ospf_single_area : Validate OSPFv3 Neighbor Count] ******************
p1 | SUCCESS
p2 | SUCCESS
pe1 | SUCCESS

TASK [test_ospf_single_area : Validate IPv4 Reachability] **********************
p1 | SUCCESS
p2 | SUCCESS
pe1 | SUCCESS

TASK [test_ospf_single_area : Validate IPv6 Reachability] **********************
pe1 | SUCCESS
p1 | SUCCESS
p2 | SUCCESS
```

## Manually {#Manually}

* Log into each device and ping every other device's loopback IPv4 and
  IPv6 addresses.
* Log into each device and verify they each have two IPv4 and two IPv6
  OSPF neighbors
* Log into `pe1` and verify it has a default route via `p1` and `p2`

### `p1`

```
RP/0/0/CPU0:p1#ping 2.2.2.2
Sat Jan 21 11:48:19.386 UTC
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2.2.2.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
RP/0/0/CPU0:p1#ping 3.3.3.3
Sat Jan 21 11:48:25.075 UTC
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 3.3.3.3, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
RP/0/0/CPU0:p1#ping ipv6 2001:db8::2:2:2:2
Sat Jan 21 11:48:38.745 UTC
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:db8::2:2:2:2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/3/9 ms
RP/0/0/CPU0:p1#ping ipv6 2001:db8::3:3:3:3
Sat Jan 21 11:48:48.014 UTC
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:db8::3:3:3:3, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/3/9 ms
RP/0/0/CPU0:p1#show ospf neighbor
Sat Jan 21 11:49:01.423 UTC

* Indicates MADJ interface
# Indicates Neighbor awaiting BFD session up

Neighbors for OSPF 1

Neighbor ID     Pri   State           Dead Time   Address         Interface
3.3.3.3         1     FULL/  -        00:00:39    10.0.0.3        GigabitEthernet0/0/0/2
    Neighbor is up for 03:41:24
2.2.2.2         1     FULL/  -        00:00:37    10.0.0.1        GigabitEthernet0/0/0/4
    Neighbor is up for 04:14:27

Total neighbor count: 2
RP/0/0/CPU0:p1#show ospfv3 neighbor
Sat Jan 21 11:49:06.573 UTC

# Indicates Neighbor awaiting BFD session up

Neighbors for OSPFv3 1

Neighbor ID     Pri   State           Dead Time   Interface ID    Interface
3.3.3.3         1     FULL/  -        00:00:31    9               GigabitEthernet0/0/0/2
    Neighbor is up for 03:44:25
2.2.2.2         1     FULL/  -        00:00:38    12              GigabitEthernet0/0/0/4
    Neighbor is up for 03:43:48

Total neighbor count: 2
RP/0/0/CPU0:p1#
```

### `p2`

```
p2#ping 1.1.1.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 1.1.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 2/8/17 ms
p2#ping 3.3.3.3
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 3.3.3.3, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 2/9/17 ms
p2#ping ipv6 2001:db8::1:1:1:1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:DB8::1:1:1:1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 2/5/16 ms
p2#ping ipv6 2001:db8::3:3:3:3
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:DB8::3:3:3:3, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 3/9/17 ms
p2#show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
1.1.1.1           0   FULL/  -        00:00:34    10.0.0.0        GigabitEthernet6
3.3.3.3           0   FULL/  -        00:00:30    10.0.0.5        GigabitEthernet2
p2#show ipv6 ospf neighbor

            OSPFv3 Router with ID (2.2.2.2) (Process ID 1)

Neighbor ID     Pri   State           Dead Time   Interface ID    Interface
1.1.1.1           0   FULL/  -        00:00:33    7               GigabitEthernet6
3.3.3.3           0   FULL/  -        00:00:33    8               GigabitEthernet2
p2#
```

### `pe1`

```
pe1#ping 1.1.1.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 1.1.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 2/5/9 ms
pe1#ping 2.2.2.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2.2.2.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 2/8/17 ms
pe1#ping ipv6 2001:db8::1:1:1:1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:DB8::1:1:1:1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 2/8/17 ms
pe1#ping ipv6 2001:db8::2:2:2:2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:DB8::2:2:2:2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 3/5/12 ms
pe1#show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
1.1.1.1           0   FULL/  -        00:00:30    10.0.0.2        GigabitEthernet3
2.2.2.2           0   FULL/  -        00:00:33    10.0.0.4        GigabitEthernet2
pe1#show ipv6 ospf neighbor

            OSPFv3 Router with ID (3.3.3.3) (Process ID 1)

Neighbor ID     Pri   State           Dead Time   Interface ID    Interface
1.1.1.1           0   FULL/  -        00:00:39    5               GigabitEthernet3
2.2.2.2           0   FULL/  -        00:00:31    8               GigabitEthernet2
pe1#show ip route 0.0.0.0 0.0.0.0
Routing entry for 0.0.0.0/0, supernet
  Known via "ospf 1", distance 110, metric 1, candidate default path
  Tag 1, type extern 2, forward metric 1
  Last update from 10.0.0.2 on GigabitEthernet3, 03:40:43 ago
  Routing Descriptor Blocks:
  * 10.0.0.2, from 1.1.1.1, 03:40:43 ago, via GigabitEthernet3
      Route metric is 1, traffic share count is 1
      Route tag 1
pe1#
```
