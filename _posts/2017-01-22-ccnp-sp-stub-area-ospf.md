---
layout: post
title: "OSPF Stub Area in IOS and IOS XR"
description: |
  - Adding an OSPF stub area in IOS and IOS XR.
category: ccnp-sp
tags: [ccnp-sp, ios, ios-xe, ios-xr, ospf, labs, sproute]
---

> This post is a lab that I'm using as part of my CCNP SP preparations.
> It follows a scenario format, and sample configurations are provided
> throughout the various stages.

# Summary

Your company has started converting smaller PoPs into their own OSPF
areas in order to reduce the load on the smaller routers.  You need to
convert one such PoP.  The core P routers are already configured
correctly, but you will need to reconfigure `P2` to be an ABR for area 1
and reconfigure `PE1` to be completely within area 1.

# Topology

![OSPF Stub Area Topology](/assets/img/ospf-abr-topology.png)

# Design Requirements

In order to finish this phase of the deployment, you must have the
following requirements satisfied:

* The link between `P2` and `PE1` must be in area `0.0.0.1`
* Area `0.0.0.1` must be a stub area
* `PE1` should not have `P1`'s Loopback1-4 addresses in its routing
  table
* Loopback IP addresses should be reachable from all three routers
* `PE1` should have a default route via `P2`
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
  ip ospf 1 area 0.0.0.0
  ipv6 ospf 1 area 0.0.0.0
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
  passive-interface default
  no passive-interface GigabitEthernet4
  no passive-interface GigabitEthernet6
!
router ospfv3 1
  router-id 2.2.2.2
  area 0.0.0.0 authentication ipsec spi 256 md5 ABCDEF0123456789ABCDEF0123456789
  !
  address-family ipv6 unicast
    passive-interface default
    no passive-interface GigabitEthernet4
    no passive-interface GigabitEthernet6
  exit-address-family
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
  area 0.0.0.0
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
  authentication ipsec spi 256 md5 ABCDEF0123456789ABCDEF0123456789
  area 0.0.0.0
    interface Loopback0
      passive
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
interface GigabitEthernet4
  no ip ospf 1 area 0.0.0.0
  ip ospf 1 area 0.0.0.1
  no ipv6 ospf 1 area 0.0.0.0
  ipv6 ospf 1 area 0.0.0.1
!
router ospf 1
  area 0.0.0.1 authentication message-digest
  area 0.0.0.1 stub
!
router ospfv3 1
  area 0.0.0.1 stub
  area 0.0.0.1 authentication ipsec spi 257 md5 ABCDEF0123456789ABCDEF0123456789
!
```

## PE1

```
router ospf 1
  no area 0.0.0.0
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
  no authentication ipsec spi 256 md5 ABCDEF0123456789ABCDEF0123456789
  authentication ipsec spi 257 md5 ABCDEF0123456789ABCDEF0123456789
  no area 0.0.0.0
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
```

# Verification

* Log into `pe1` and ping every other device's loopback IPv4 and IPv6
  addresses.
* Log into `pe1` and verify that its `GigabitEthernet0/0/0/1` interface
  is in OSPF area `0.0.0.1`
* Log into `p2` and verify that its `GigabitEthernet4` interface is in
  OSPF area `0.0.0.1`
* Log into `p2` and `pe1` and verify that OSPF area `0.0.0.1` is a
  `stub` area
* Log into `pe1` and verify it has a default route via `p2`

> In a future post, we'll look into how you can use Ansible to perform
> verification for you.

### `p2`

```
p2#show ip ospf interface GigabitEthernet 4 | include Area
  Internet Address 10.0.0.4/31, Area 0.0.0.1, Attached via Interface Enable
p2#show ipv6 ospf interface GigabitEthernet 4 | include Are
  Area 0.0.0.1, Process ID 1, Instance ID 0, Router ID 2.2.2.2
  MD5 authentication (Area) SPI 257, secure socket UP (errors: 0)
p2#show ip ospf topology-info | include [Aa]rea
 Number of areas transit capable is 0
    Area BACKBONE(0.0.0.0)
  Area ranges are
    Area 0.0.0.1
        It is a stub area
  Area ranges are
p2#show ip ospf topology-info | include [Aa]rea
 Number of areas transit capable is 0
    Area BACKBONE(0.0.0.0)
  Area ranges are
    Area 0.0.0.1
        It is a stub area
  Area ranges are
```

### `pe1`

```
RP/0/0/CPU0:pe1#show ospf interface GigabitEthernet 0/0/0/1 | include Area
Sat Jan 21 19:45:44.643 UTC
  Internet Address 10.0.0.5/31, Area 0.0.0.1
RP/0/0/CPU0:pe1#show ospfv3 interface GigabitEthernet 0/0/0/1 | include Area
Sat Jan 21 19:45:57.513 UTC
  Area 0.0.0.1, Process ID 1, Instance ID 0, Router ID 3.3.3.3
RP/0/0/CPU0:pe1#show ospf 1 | include [Aa]rea
Sat Jan 21 19:50:12.665 UTC
 Adjacency stagger enabled; initial (per area): 2, maximum: 64
 Number of areas in this router is 1. 0 normal 1 stub 0 nssa
    Area 0.0.0.1
        Number of interfaces in this area is 2
        It is a stub area
RP/0/0/CPU0:pe1#show ospfv3 1 | include [Aa]rea
Sat Jan 21 19:50:34.284 UTC
 Number of areas in this router is 1. 0 normal 1 stub 0 nssa
    Area 0.0.0.1
        Number of interfaces in this area is 2
        It is a stub area
RP/0/0/CPU0:pe1#show route ipv4 0.0.0.0/0
Sat Jan 21 19:52:51.544 UTC

Routing entry for 0.0.0.0/0
  Known via "ospf 1", distance 110, metric 2, candidate default path, type inter area
  Installed Jan 21 19:07:06.402 for 00:45:45
  Routing Descriptor Blocks
    10.0.0.4, from 2.2.2.2, via GigabitEthernet0/0/0/1
      Route metric is 2
  No advertising protos.
RP/0/0/CPU0:pe1#show route ipv6 ::/0
Sat Jan 21 19:52:55.984 UTC

Routing entry for ::/0
  Known via "ospf 1", distance 110, metric 2, candidate default path, type inter area
  Installed Jan 21 19:14:01.884 for 00:38:54
  Routing Descriptor Blocks
    fe80::26f:e5ff:fe3e:9c03, from ::, via GigabitEthernet0/0/0/1
      Route metric is 2
  No advertising protos.
RP/0/0/CPU0:pe1#show route ipv4 192.168.255.0/24
Sat Jan 21 19:53:23.342 UTC

% Network not in table

RP/0/0/CPU0:pe1#show route ipv6 2001:db8:4::10:0:0:1/64
Sat Jan 21 19:53:45.201 UTC

% Network not in table

RP/0/0/CPU0:pe1#ping 192.168.255.1
Sat Jan 21 19:54:00.759 UTC
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.255.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/4/9 ms
RP/0/0/CPU0:pe1#ping 192.168.254.1
Sat Jan 21 19:54:05.369 UTC
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.254.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/4/9 ms
RP/0/0/CPU0:pe1#ping 192.168.253.1
Sat Jan 21 19:54:08.199 UTC
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.253.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/4/19 ms
RP/0/0/CPU0:pe1#ping 192.168.252.1
Sat Jan 21 19:54:14.089 UTC
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.252.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/4/9 ms
RP/0/0/CPU0:pe1#ping 2001:db8:1::10:0:0:1
Sat Jan 21 19:54:50.776 UTC
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:db8:1:0:10::1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/4/9 ms
RP/0/0/CPU0:pe1#ping 2001:db8:2::10:0:0:1
Sat Jan 21 19:54:55.276 UTC
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:db8:2:0:10::1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/4/9 ms
RP/0/0/CPU0:pe1#ping 2001:db8:3::10:0:0:1
Sat Jan 21 19:54:59.415 UTC
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:db8:3:0:10::1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/4/9 ms
RP/0/0/CPU0:pe1#ping 2001:db8:4::10:0:0:1
Sat Jan 21 19:55:06.385 UTC
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:db8:4:0:10::1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/4/9 ms
```
