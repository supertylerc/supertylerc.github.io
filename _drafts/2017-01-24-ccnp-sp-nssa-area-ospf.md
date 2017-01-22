---
layout: post
title: "OSPF Not So Stubby Area in IOS and IOS XR"
description: |
  - Adding an OSPF NSSA in IOS and IOS XR.
category: ccnp-sp
tags: [ccnp-sp, ios, ios-xe, ios-xr, ospf, labs, sproute]
---

> This post is a lab that I'm using as part of my CCNP SP preparations.
> It follows a scenario format, and sample configurations are provided
> throughout the various stages.

# Summary

Requirements are always changing!  One of the PoPs that was recently
converted to a stub area needs to be able to redistribute routes into
OSPF.  You must fix this PoP to resolve an outage.

# Topology

![OSPF Stub Area Topology](/assets/img/ospf-abr-topology.png)

# Design Requirements

In order to finish this trouble, the following criteria must be met:

* `P1` should be able to ping `PE1`'s loopback addresses
* `P1`'s route table should have one entry for the `192.168.254.0/23`
  network.
* Area `0.0.0.1` must be a Not So Stubby Area
* Both IPv4 and IPv6 connectivity should work

# Initial Configurations

## P1

```
hostname p1
interface Loopback0
  ipv4 address 1.1.1.1 255.255.255.255
  ipv6 address 2001:db8::1:1:1:1/128
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
interface Loopback1
  ipv4 address 192.168.254.1 255.255.255.0
  ipv6 address 2001:db8:1234:aba2::/64
!
interface Loopback2
  ipv4 address 192.168.255.1 255.255.255.0
  ipv6 address 2001:db8:1234:abc3::/64
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
  no area 0.0.0.1 stub
  area 0.0.0.1 nssa
!
router ospfv3 1
  no area 0.0.0.1 stub
  area 0.0.0.1 nssa
!
```

## PE1

```
router ospf 1
  summary-prefix 192.168.254.0/23
  redistribute connected
  area 0.0.0.1
    no stub
    nssa
  !
!
router ospfv3 1
  summary-prefix 2001:db8:1234:ab80::/57
  redistribute connected
  area 0.0.0.1
    no stub
    nssa
  !
!
```

# Verification

* Log into `p2` and `pe1` and verify that OSPF area `0.0.0.1` is a NSSA
* Log into `p1` and verify the `192.168.254.0/23` and
  `2001:db8:1234:ab80::/57` prefixes are in the route table but the more
  specifics are not
* Log into `pe1` and verify connectivity to `1.1.1.1` sourcing each of
  IPv4 loopback IPs
* Log into `pe1` and verify connectivity to `2001:db8::1:1:1:1` sourcing
  each of the IPv6 loopback IPs
* Log into `pe1` and verify that there is no default route for IPv4 or
  IPv6.

> In a future post, we'll look into how you can use Ansible to perform
> verification for you.

### `p1`

```
RP/0/0/CPU0:p1#show route ipv4 longer-prefixes 192.168.254.0/23 | include Gigabit
Sun Jan 22 07:15:20.969 UTC
O E2 192.168.254.0/23 [110/20] via 10.0.0.1, 00:07:10, GigabitEthernet0/0/0/4
RP/0/0/CPU0:p1#show route ipv6 longer-prefixes 2001:db8:1234:ab80::/57
Sun Jan 22 07:15:56.527 UTC

Codes: C - connected, S - static, R - RIP, B - BGP, (>) - Diversion path
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2, E - EGP
       i - ISIS, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, su - IS-IS summary null, * - candidate default
       U - per-user static route, o - ODR, L - local, G  - DAGR, l - LISP
       A - access/subscriber, a - Application route
       M - mobile route, r - RPL, (!) - FRR Backup path

O E2 2001:db8:1234:ab80::/57
      [110/20] via fe80::26f:e5ff:fe3e:9c05, 00:07:39, GigabitEthernet0/0/0/4
```

### `p2`

```
p2#show ip ospf topology-info | include [Aa]rea
 Number of areas transit capable is 0
    Area BACKBONE(0.0.0.0)
  Area ranges are
    Area 0.0.0.1
        It is a NSSA area
  Area ranges are
p2#show ipv6 ospf | include [Aa]rea
 It is an area border and autonomous system boundary router
 Number of areas in this router is 2. 1 normal 0 stub 1 nssa
    Area BACKBONE(0.0.0.0)
  Number of interfaces in this area is 2
    Area 0.0.0.1
  Number of interfaces in this area is 1
        It is a NSSA area
```

### `pe1`

```
RP/0/0/CPU0:pe1#show ospf 1 | include [Aa]rea
Sun Jan 22 07:18:36.556 UTC
 Adjacency stagger enabled; initial (per area): 2, maximum: 64
 Number of areas in this router is 1. 0 normal 0 stub 1 nssa
    Area 0.0.0.1
        Number of interfaces in this area is 2
        It is a NSSA area
RP/0/0/CPU0:pe1#show ospfv3 1 | include [Aa]rea
Sun Jan 22 07:18:48.885 UTC
 Number of areas in this router is 1. 0 normal 0 stub 1 nssa
    Area 0.0.0.1
        Number of interfaces in this area is 2
        It is a NSSA area
RP/0/0/CPU0:pe1#ping 1.1.1.1 source 3.3.3.3
Sun Jan 22 07:19:46.211 UTC
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 1.1.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/4/9 ms
RP/0/0/CPU0:pe1#ping 1.1.1.1 source 192.168.254.1
Sun Jan 22 07:19:54.160 UTC
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 1.1.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/4/9 ms
RP/0/0/CPU0:pe1#ping 1.1.1.1 source 192.168.255.1
Sun Jan 22 07:19:59.830 UTC
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 1.1.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/4/9 ms
RP/0/0/CPU0:pe1#ping 2001:db8::1:1:1:1 source 2001:db8::3:3:3:3
Sun Jan 22 07:21:50.362 UTC
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:db8::1:1:1:1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/4/9 ms
RP/0/0/CPU0:pe1#ping 2001:db8::1:1:1:1 source 2001:db8:1234:aba2::1
Sun Jan 22 07:21:56.202 UTC
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:db8::1:1:1:1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/4/9 ms
RP/0/0/CPU0:pe1#ping 2001:db8::1:1:1:1 source 2001:db8:1234:abc3::1
Sun Jan 22 07:22:03.821 UTC
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:db8::1:1:1:1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/4/9 ms
RP/0/0/CPU0:pe1#show route ipv4 0.0.0.0/0
Sun Jan 22 07:22:17.300 UTC

% Network not in table

RP/0/0/CPU0:pe1#show route ipv6 ::/0
Sun Jan 22 07:22:21.570 UTC

% Network not in table
```
