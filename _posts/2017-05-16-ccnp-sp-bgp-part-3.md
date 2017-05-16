---
layout: post
title: "Dissecting the CCNP SPADVROUTE Syllabus: BGP, Part 3"
description: |
  Third in a series on the BGP section of the CCNP SPADVROUTE Syllabus.
category: ccnp-sp
tags: [ccnp-sp, ios, ios-xe, ios-xr, bgp, advsproute]
---

> This post is the third in a series on BGP in the CCNP-SP SPADVROUTE
> exam.

# Introduction

This post covers part of `1.0 BGP Routing Features in a Service
Provider IP NGN Environment`.  Specifically, it covers the following:

* 1.5 Implement BGP in SP IP NGN IOS-XR and IOS-XE PE routers to support
  multi-homed BGP Customers

# 1.5 Implement BGP for Multi-homed Customers

This part of the syllabus is a little odd.  Most of the best practices
for providing BGP to a customer are the same regardless of their BGP
topology.  Some best practices:

* Inbound BGP filters to ensure customers only advertise _their_ space
  with _their_ ASN.
* Advertise a default route
* Optionally advertise your more specific prefixes
* Remove private ASNs
* Allow policy modifications with BGP communities

## Multihop

Multihop may be necessary if the router to which a customer is connected
cannot advertise a full BGP feed due to hardware constraints.

### IOS-XE

```
ios-xe#configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
ios-xe(config)#router bgp 65000
ios-xe(config-router)#neighbor 1.2.3.4 remote-as 65001
ios-xe(config-router)#neighbor 1.2.3.4 ebgp-multihop 3
```

### IOS-XR

```
RP/0/0/CPU0:ios-xr#configure
Mon May 15 22:29:51.505 UTC
RP/0/0/CPU0:ios-xr(config)#router bgp 65000
RP/0/0/CPU0:ios-xr(config-bgp)#neighbor 1.2.3.4
RP/0/0/CPU0:ios-xr(config-bgp-nbr)#ebgp-multihop 3
```

## Multipath

By default, BGP selects a single best path.  It may be desireable in
some scenarios to load balance multiple iBGP routes.

> This feature is undesireable in systems with limited memory.
> Additionally, it is not useful with route reflection when the route
> reflector advertises only the best path.

### IOS-XE

```
ios-xe#configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
ios-xe(config)#router bgp 65000
ios-xe(config-router)#address-family ipv4 unicast
ios-xe(config-router-af)#maximum-paths ibgp 4
```

### IOS-XR

```
RP/0/0/CPU0:ios-xr#configure
Mon May 15 22:33:32.790 UTC
RP/0/0/CPU0:ios-xr(config)#router bgp 65000
RP/0/0/CPU0:ios-xr(config-bgp)#address-family ipv4 unicast
RP/0/0/CPU0:ios-xr(config-bgp-af)#maximum-paths ibgp 4
```

## Signaling Intent with Communities

It can be extremely beneficial for a customer to be able to partially
influence a provider network.  Providers can enable this by exposing,
in a controlled manner, a number of routing policy actions based on a
predetermined BGP community value.

### IOS-XE

```
ios-xe(config)#ip community-list 1 permit 65000:50
ios-xe(config)#ip community-list 2 permit 65000:4
ios-xe(config-route-map)#match community 1
ios-xe(config-route-map)#set local-preference 50
ios-xe(config-route-map)#exit
ios-xe(config)#route-map CUSTOMER_SIGNALED_PREPEND 10
ios-xe(config-route-map)#match community 2
ios-xe(config-route-map)#set as-path prepend 65000 4
ios-xe(config)#router bgp 65000
ios-xe(config-router)#neighbor 1.2.3.4 remote-as 65001
ios-xe(config-router)#neighbor 1.2.3.4 description "Customer"
ios-xe(config-router)#neighbor 4.3.2.1 remote-as 65002
ios-xe(config-router)#neighbor 4.3.2.1 description "Upstream Provider"
ios-xe(config-router)#address-family ipv4 unicast
ios-xe(config-router-af)#neighbor 1.2.3.4 route-map CUSTOMER_SIGNALED_LP in
ios-xe(config-router-af)#neighbor 1.2.3.4 route-map CUSTOMER_SIGNALED_PREPEND out
```

### IOS-XR

```
RP/0/0/CPU0:ios-xr#configure
Mon May 15 22:38:48.429 UTC
RP/0/0/CPU0:ios-xr(config)#community-set LPREF_50
RP/0/0/CPU0:ios-xr(config-comm)#65000:50
RP/0/0/CPU0:ios-xr(config-comm)#exit
RP/0/0/CPU0:ios-xr(config)#community-set PREPEND_4
RP/0/0/CPU0:ios-xr(config-comm)#65000:4
RP/0/0/CPU0:ios-xr(config-comm)#exit
RP/0/0/CPU0:ios-xr(config)#route-policy CUSTOMER_SIGNALED_LP
RP/0/0/CPU0:ios-xr(config-rpl)#if community matches-any LPREF_50 then
RP/0/0/CPU0:ios-xr(config-rpl-if)#set local-preference 50
RP/0/0/CPU0:ios-xr(config-rpl-if)#endif
RP/0/0/CPU0:ios-xr(config-rpl)#end-policy
RP/0/0/CPU0:ios-xr(config)#route-policy CUSTOMER_SIGNALED_PREPEND
RP/0/0/CPU0:ios-xr(config-rpl)#if community matches-any PREPEND_4 then
RP/0/0/CPU0:ios-xr(config-rpl-if)#prepend as-path 65000 4
RP/0/0/CPU0:ios-xr(config-rpl-if)#endif
RP/0/0/CPU0:ios-xr(config-rpl)#end-policy
RP/0/0/CPU0:ios-xr(config)#router bgp 65000
RP/0/0/CPU0:ios-xr(config-bgp)#neighbor 1.2.3.4
RP/0/0/CPU0:ios-xr(config-bgp-nbr)#remote-as 65001
RP/0/0/CPU0:ios-xr(config-bgp-nbr)#description "Customer"
RP/0/0/CPU0:ios-xr(config-bgp-nbr)#address-family ipv4 unicast
RP/0/0/CPU0:ios-xr(config-bgp-nbr-af)#route-policy CUSTOMER_SIGNALED_LP in
RP/0/0/CPU0:ios-xr(config-bgp-nbr-af)#exit
RP/0/0/CPU0:ios-xr(config-bgp-nbr)#exit
RP/0/0/CPU0:ios-xr(config-bgp)#neighbor 4.3.2.1
RP/0/0/CPU0:ios-xr(config-bgp-nbr)#remote-as 65002
RP/0/0/CPU0:ios-xr(config-bgp-nbr)#description "Upstream Provider"
RP/0/0/CPU0:ios-xr(config-bgp-nbr)#address-family ipv4 unicast
RP/0/0/CPU0:ios-xr(config-bgp-nbr-af)#route-policy CUSTOMER_SIGNALED_PREPEND out
```

# Next Time

In the next post, we'll look at the security sides of the BGP section
of the SPADVROUTE syllabus.  This includes prefix limits, TTL security,
and remote triggered blackhole filtering.