---
layout: post
title: "OSPF Totally Stubby Area in IOS and IOS XR"
description: |
  - Adding an OSPF stub area in IOS and IOS XR.
category: ccnp-sp
tags: [ccnp-sp, ios, ios-xe, ios-xr, ospf, labs, sproute]
---

> This post is a lab that I'm using as part of my CCNP SP preparations.
> It follows a scenario format, and sample configurations are provided
> throughout the various stages.

# Summary

You recently began converting small PoPs into stub areas, but the
routers in these locations are still working too hard!  You've been
tasked with converting these areas into totally stubby areas with the
hope that this will reduce CPU load enough that it won't be a problem
anymore.

# Topology

![OSPF Stub Area Topology](/assets/img/ospf-abr-topology.png)

# Design Requirements

In order to finish this phase of the deployment, you must have the
following requirements satisfied:

* The link between `P2` and `PE1` must be in area `0.0.0.1`
* Area `0.0.0.1` must be a "totally stubby" area
* `PE1`'s only OSPF route should be a default route via `P2`
* Both IPv4 and IPv6 connectivity should work

# Initial Configurations

## P1

```
hostname p1
interface Loopback0
  ipv4 address 1.1.1.1 255.255.255.255
  ipv6 address 2001:db8::1:1:1:1/128
!
interface Loopback1
  ipv4 address 192.168.252.1 255.255.255.0
  ipv6 address 2001:db8:1:0:10::1/64
!
interface Loopback2
  ipv4 address 192.168.253.1 255.255.255.0
  ipv6 address 2001:db8:2:0:10::1/64
!
interface Loopback3
  ipv4 address 192.168.254.1 255.255.255.0
  ipv6 address 2001:db8:3:0:10::1/64
!
interface Loopback4
  ipv4 address 192.168.255.1 255.255.255.0
  ipv6 address 2001:db8:4:0:10::1/64
!
interface GigabitEthernet0/0/0/4
  description "to p2_g6"
  ipv4 address 10.0.0.0/31
  ipv6 address 2001:db8::10:0:0:0/127
!
router ospf 1
  router-id 1.1.1.1
  authentication message-digest
  message-digest-key 1 md5 cisco
  redistribute connected
  area 0.0.0.0
    interface Loopback0
      passive enable
    !
    interface GigabitEthernet0/0/0/4
      network point-to-point
    !
  !
!
router ospfv3 1
  router-id 1.1.1.1
  authentication ipsec spi 256 md5 ABCDEF0123456789ABCDEF0123456789
  redistribute connected
  area 0.0.0.0
    interface Loopback0
      passive
    !
    interface GigabitEthernet0/0/0/4
      network point-to-point
    !
  !
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
  ip ospf 1 area 0.0.0.0
  ipv6 ospf 1 area 0.0.0.0
!
interface GigabitEthernet4
  description "to pe1_g0/0/0/1"
  ip address 10.0.0.4 255.255.255.254
  ipv6 address 2001:db8::10:0:0:4/127
  ip ospf message-digest-key 1 md5 cisco
  ip ospf network point-to-point
  ip ospf 1 area 0.0.0.1
  ipv6 ospf 1 area 0.0.0.1
  ipv6 ospf network point-to-point
!
interface GigabitEthernet6
  description "to p1_g0/0/0/4"
  ip address 10.0.0.1 255.255.255.254
  ipv6 address 2001:db8::10:0:0:1/127
  ip ospf message-digest-key 1 md5 cisco
  ip ospf network point-to-point
  ip ospf 1 area 0.0.0.0
  ipv6 ospf 1 area 0.0.0.0
  ipv6 ospf network point-to-point
!
router ospf 1
  router-id 2.2.2.2
  area 0.0.0.0 authentication message-digest
  area 0.0.0.1 authentication message-digest
  area 0.0.0.1 stub
  passive-interface default
  no passive-interface GigabitEthernet4
  no passive-interface GigabitEthernet6
!
router ospfv3 1
  router-id 2.2.2.2
  area 0.0.0.0 authentication ipsec spi 256 md5 ABCDEF0123456789ABCDEF0123456789
  area 0.0.0.1 stub
  area 0.0.0.1 authentication ipsec spi 257 md5 ABCDEF0123456789ABCDEF0123456789
  !
  address-family ipv6 unicast
    passive-interface default
    no passive-interface GigabitEthernet4
    no passive-interface GigabitEthernet6
  exit-address-family
  !
!
```

## PE1

```
hostname pe1
!
interface Loopback0
  ipv4 address 3.3.3.3 255.255.255.255
  ipv6 address 2001:db8::3:3:3:3/128
!
interface GigabitEthernet0/0/0/1
  description "to p2_g4"
  ipv4 address 10.0.0.5 255.255.255.254
  ipv6 address 2001:db8::10:0:0:5/127
!
router ospf 1
  router-id 3.3.3.3
  authentication message-digest
  message-digest-key 1 md5 cisco
  area 0.0.0.1
    stub
    interface Loopback0
      passive enable
    !
    interface GigabitEthernet0/0/0/1
      network point-to-point
    !
  !
!
router ospfv3 1
  router-id 3.3.3.3
  authentication ipsec spi 257 md5 ABCDEF0123456789ABCDEF0123456789
  area 0.0.0.1
    stub
    interface Loopback0
      passive
    !
    interface GigabitEthernet0/0/0/1
      network point-to-point
    !
  !
  !
  area 0.0.0.1
    stub
    interface Loopback0
      passive enable
    !
    interface GigabitEthernet0/0/0/1
      network point-to-point
    !
  !
!
```

# Solutions

## P2

```
router ospf 1
  area 0.0.0.1 stub no-summary
!
router ospfv3 1
  area 0.0.0.1 stub no-summary
!
```

# Verification

* Log into `p2` and verify that OSPF area `0.0.0.1` is a totally stubby
  area
* Log into `pe1` and ping every other device's loopback IPv4 and IPv6
  addresses.
* Log into `pe1` and verify its only OSPF route is a default route via
  `p2`

> In a future post, we'll look into how you can use Ansible to perform
> verification for you.

### `p2`

```
p2#show ip ospf topology-info | include [Aa]rea|default
 Number of areas transit capable is 0
    Area BACKBONE(0.0.0.0)
  Area ranges are
    Area 0.0.0.1
        It is a stub area, no summary LSA in this area
        Generates stub default route with cost 1
  Area ranges are
p2#
```

> The area is a stub area with no summary LSAs, and the router is
> generating a default route.  This is what a "totally stubby" area is.

### `pe1`

```
RP/0/0/CPU0:pe1#show route ipv4 ospf
Sat Jan 21 20:47:47.608 UTC

O*IA 0.0.0.0/0 [110/2] via 10.0.0.4, 00:01:02, GigabitEthernet0/0/0/1
RP/0/0/CPU0:pe1#show route ipv6 ospf
Sat Jan 21 20:47:53.288 UTC

O*IA ::/0
      [110/2] via fe80::26f:e5ff:fe3e:9c03, 01:33:51, GigabitEthernet0/0/0/1
RP/0/0/CPU0:pe1#ping 192.168.252.1
Sat Jan 21 20:49:23.842 UTC
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.252.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/4/9 ms
RP/0/0/CPU0:pe1#ping 192.168.253.1
Sat Jan 21 20:49:28.511 UTC
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.253.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/4/9 ms
RP/0/0/CPU0:pe1#ping 192.168.254.1
Sat Jan 21 20:49:30.531 UTC
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.254.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/2/9 ms
RP/0/0/CPU0:pe1#ping 192.168.255.1
Sat Jan 21 20:49:34.311 UTC
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.255.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/4/9 ms
RP/0/0/CPU0:pe1#ping 2001:db8:1::10:0:0:1
Sat Jan 21 20:49:57.569 UTC
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:db8:1:0:10::1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/4/9 ms
RP/0/0/CPU0:pe1#ping 2001:db8:2::10:0:0:1
Sat Jan 21 20:50:01.609 UTC
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:db8:2:0:10::1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/6/9 ms
RP/0/0/CPU0:pe1#ping 2001:db8:3::10:0:0:1
Sat Jan 21 20:50:08.529 UTC
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:db8:3:0:10::1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/4/9 ms
RP/0/0/CPU0:pe1#ping 2001:db8:4::10:0:0:1
Sat Jan 21 20:50:13.128 UTC
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:db8:4:0:10::1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/4/9 ms
```
