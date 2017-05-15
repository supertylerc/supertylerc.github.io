---
layout: post
title: "Dissecting the CCNP SPADVROUTE Syllabus: BGP, Part 2"
description: |
  Second in a series on the BGP section of the CCNP SPADVROUTE Syllabus.
category: ccnp-sp
tags: [ccnp-sp, ios, ios-xe, ios-xr, bgp, advsproute]
---

> This post is the second in a series on BGP in the CCNP-SP SPADVROUTE
> exam.

# Introduction

This post covers part of `1.0 BGP Routing Features in a Service
Provider IP NGN Environment`.  Specifically, it covers the following:

* 1.4 Design and implement BGP route reflectors to scale IBGP in BGP
  transit backbones on IOS-XR and IOS-XE

# 1.4 Design and Implement BGP Route Reflection

In a previous post, we discussed BGP confederations to solve the iBGP
full mesh problem.  As a refresher, the problem with iBGP is that it:

* Requires a full mesh
* Requires `N(N-1)/2` BGP sessions

Confederations solve this in an operationally frustrating way.  They
don't really remove the need for a full mesh; they just break the number
of full meshes required into smaller parts.  In addition, looking at BGP
paths in a confederation environment is ugly.  This says nothing of the
more advanced BGP functions.  How do confederations affect MPLS?
Multicast?  I don't even want to know.

Route reflection solves this differently.  It moves the burden of a mesh
to a subset of routers defined by the administrator.  Route reflection,
therefore, requires a partial mesh instead of a full mesh.  The number
of BGP sessions required is thus `M(N-1)+(M-1)`, where `M` is the number of
route reflectors and `N` is the number of routers.  This means that for 6
routers with two route reflectors, you need 11 sessions compared to the 15
sessions necessary for a full mesh.  If two of your six routers double as
route reflectors, you only need 9 sessions.  The savings increase
exponentially with the number of routers deployed.

## iBGP Speakers

In a deployment consisting of route reflection, there are three basic
types of iBGP speakers:

* Route Reflector: Requires a full mesh to other reflectors and
  non-client, non-reflector routers
* Route Reflector Client: Requires a session to the route reflectors in
  the same cluster
* Non-Client, Non-Reflector: Requires a full mesh to all other
  non-client, non-reflector routers and all route reflectors

### IOS-XE

```
ios-xe#configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
ios-xe(config)#router bgp 65000
ios-xe(config-router)#neighbor 10.10.10.10 remote-as 65000
ios-xe(config-router)#neighbor 10.10.10.10 description "Route Reflector Client"
ios-xe(config-router)#neighbor 10.10.10.10 route-reflector-client
ios-xe(config-router)#neighbor 20.20.20.20 remote-as 65000
ios-xe(config-router)#neighbor 20.20.20.20 description "Non-Client"
```

### IOS-XR

```
RP/0/0/CPU0:ios-xr#configure
Mon May 15 06:52:33.743 UTC
RP/0/0/CPU0:ios-xr(config)#router bgp 65000
RP/0/0/CPU0:ios-xr(config-bgp)#neighbor 10.10.10.10
RP/0/0/CPU0:ios-xr(config-bgp-nbr)#remote-as 65000
RP/0/0/CPU0:ios-xr(config-bgp-nbr)#description "Route Reflector Client"
RP/0/0/CPU0:ios-xr(config-bgp-nbr)#address-family ipv4 unicast
RP/0/0/CPU0:ios-xr(config-bgp-nbr-af)#route-reflector-client
RP/0/0/CPU0:ios-xr(config-bgp-nbr-af)#exit
RP/0/0/CPU0:ios-xr(config-bgp-nbr)#exit
RP/0/0/CPU0:ios-xr(config-bgp)#neighbor 20.20.20.20
RP/0/0/CPU0:ios-xr(config-bgp-nbr)#remote-as 65000
RP/0/0/CPU0:ios-xr(config-bgp-nbr)#description "Non-Client"
```

> There's no special configuration necessary on a route reflector
> client.  They aren't aware that they're route reflection clients.

## Reflector Rules

A route reflector breaks a few rules.  Normally, an iBGP peer does not
advertise a route it learned via iBGP to other iBGP peers.  With route
reflection, however, a route reflector does the following:

* Advertises prefixes learned from a client to other route reflector
  clients and non-clients
* Advertises prefixes learned from non-clients to route reflector
  clients

### BGP Clusters

To improve resiliency and prevent crippling outages resulting in a
route reflector going down, two route reflectors may be configured to
be in the same cluster.  In this configuration, you manually set the
`cluster-id` attribute instead of letting the route reflector derive
its own from its Router ID.

> There is conflicting information available on whether or not route
> reflector clusters should be used or not.  Proponents aregue that
> clusters reduce resource utilization; opponents claim there are
> failure scenarios in clusters that could be ugly.  As with all things,
> weight he trade-offs.

#### IOS-XE

```
ios-xe#configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
ios-xe(config)#router bgp 65000
ios-xe(config-router)#neighbor 1.2.3.4 remote-as 65000
ios-xe(config-router)#neighbor 1.2.3.4 cluster-id 5
```

#### IOS-XR

```
RP/0/0/CPU0:ios-xr#configure
Mon May 15 06:50:14.113 UTC
RP/0/0/CPU0:ios-xr(config)#router bgp 65000
RP/0/0/CPU0:ios-xr(config-bgp)#neighbor 1.2.3.4
RP/0/0/CPU0:ios-xr(config-bgp-nbr)#cluster-id 5
```

> You should only manually configure the `cluster-id` if you plan on
> placing two route reflectors in the same cluster.

### Loop Prevention

Normally, iBGP loop prevention is accomplished because there is a rule
that does not allow iBGP peers to advertise routes learned via iBGP to
other iBGP peers.  Route reflection, however, breaks this rule, so it
needs a new loop prevention mechanism.

When two router reflectors are in the same cluster, they are both
configured with the same `cluster-id` attribute.  If they receive a
prefix with this attribute set and it's the same as the local
`cluster-id`, the prefix is rejected.

When a route reflector receives a prefix from a route reflector client,
it sets a special BGP attribute, `originator-id`, to the Router ID of
the advertising client.  If the `originator-id` is set, it is not
updated by other route reflectors that might receive the prefix.  If a
route reflector recieves a prefix that has the `originator-id` set and
it matches a prefix already in that reflector's RIB, it rejects that
prefix.  This assumes the reflectors are in the same cluster.

In a multi-tiered route reflector topology, where one route reflector
is a client of another route reflector, the route reflectors are not in
the same cluster.  In this case, the `cluster-id` cannot be used and the
`originator-id` may not be a reliable prevention mechanism.  Instead, an
attribute known as the `cluster-list` is used.  When a route reflector
reflects an update, it prepends the `cluster-id` to the `cluster-list`
attribute.  If it receives a prefix with its own `cluster-id` in the
`cluster-list` attribute, it rejects the prefix.

## Design Considerations

### Out-of-Band Reflectors

If possible, out-of-band route reflection can greatly increase
flexibility and scalability by placing the function of the route
reflectors in a virtual machine or virtual appliance.  Route reflectors
that are not in the data path have their own design considerations and
failure scenarios.  They are not a silver bullet.

### In-Band Route Reflectors

If you deploy route reflectors as part of your data path, they should
generally be deployed as close to the physical topology as possible.
In other words, route reflectors in the data path should not be several
hops, areas, zones, or tiers away from their clients.  This is to
prevent suboptimal routing and certain failure scenarios.

# Next Time

In the next post, we'll look at deploying BGP for multi-homed
enterprises.