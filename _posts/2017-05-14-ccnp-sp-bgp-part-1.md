---
layout: post
title: "Dissecting the CCNP SPADVROUTE Syllabus: BGP, Part 1"
description: |
  First in a series on the BGP section of the CCNP SP Syllabus.
category: ccnp-sp
tags: [ccnp-sp, ios, ios-xe, ios-xr, bgp, advsproute]
---

> This post is the first in a series on BGP in the CCNP-SP SPADVROUTE
> exam.

# Introduction

This post covers part of `1.0 BGP Routing Features in a Service
Provider IP NGN Environment`.  Specifically, it covers the following:

* 1.1 Describe the BGP routing processes in IOS-XR
* 1.2 Configure the BGP timers on IOS-XR and IOS-XE
* 1.3 Describe the need for BGP confederations in BGP transit backbones

# 1.1 BGP Routing Processes in IOS-XR

> For some reason, the BGP routing processes in IOS-XE are not called
> out in the syllabus.  As a result, this section focuses exclusively
> on IOS-XR.

## BGP Process Manager

The BGP Process Manager is responsible for parsing and ensuring the
validity of the BGP configuration in IOS-XR.  It determines the BGP
Router ID and checks `clear bgp` commands before forwarding them on to
the BGP process.

## BGP Speaker Process

The BGP Speaker Process is the intermediary between a neighbor and the
BGP RIB Process.  It receives and performs partial path calculations on
prefixes received from neighbors before sending them to the BGP RIB
Process.  This is because the BGP Speaker Process can run multiple
instances in a distributed mode, while the BGP RIB Process runs in a
single process.

Note that distributed BGP Speakers is platform-dependent and is most
useful when there is more than one route processor.  In distributed
mode, the BGP Speaker Processes are isolated from each other.  A failure
in one is isolated from a failure in another.  This generally increases
stability.  In a single route processor platform or configuration, the
end result is higher resource utilization to achieve potentially greater
stability.

## BGP RIB Process

The BGP RIB Process receives prefixes from the BGP Speaker Processes and
performs path calculations on them.  It also handles redistribution
directly related to BGP.  The results of the path calculations are sent
back to the BGP Speaker Processes and the IP RIB Process.

> One BGP RIB Process is run per AFI.

# Configure BGP Timers

## Keepalive and Hold Timers

The default keepalive is 60 seconds.  The default hold timer is 180
seconds.  You can only specify the keepalive and hold timers together.
You can also specify a minimum acceptable hold timer advertised by a
neighbor.

> You can workaround the inability to specify the hold time and minimum
> hold time values individually by simply using the defaults of the
> keepalive and hold timers.  In fact, chances are you shouldn't modify
> these timers at all.  There are better mechanisms to handle
> convergence, such as BFD.

It's worth noting that the syntax for configuring these timers is
_almost_ identical in both IOS-XE and IOS-XR.

### IOS-XE

```
ios-xe#configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
ios-xe(config)#router bgp 65000
ios-xe(config-router)#timers bgp ?
  <0-65535>  Keepalive interval

ios-xe(config-router)#timers bgp 90 ?
  <0-65535>  Holdtime

ios-xe(config-router)#timers bgp 90 180 ?
  <0-65535>  Minimum hold time from neighbor
  <cr>

ios-xe(config-router)#timers bgp 90 180 90
ios-xe(config-router)#neighbor 1.2.3.4 remote-as 65001
ios-xe(config-router)#neighbor 1.2.3.4 timers 100 200 130
```

### IOS-XR

```
RP/0/0/CPU0:ios-xr#configure
Mon May 15 03:13:31.718 UTC
RP/0/0/CPU0:ios-xr(config)#router bgp 65001
RP/0/0/CPU0:ios-xr(config-bgp)#timers bgp 90 180 90
RP/0/0/CPU0:ios-xr(config-bgp)#neighbor 1.2.3.4
RP/0/0/CPU0:ios-xr(config-bgp-nbr)#remote-as 65001
RP/0/0/CPU0:ios-xr(config-bgp-nbr)#timers 100 200 130
```

> The above values are really, really, really astonishingly bad.  Don't
> use them ever.

## BGP Scanner Timers

The BGP scanner runs, by default, every 60 seconds.  It is configurable,
and the configuration is the same on both IOS-XE and IOS-XR.  They are
both shown below.

> You should probably never ever change the BGP scanner timer unless
> directed to do so by TAC, and even then you should take extreme care.

### IOS-XE

```
ios-xe#configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
ios-xe(config)#router bgp 65000
ios-xe(config-router)#bgp scan-time ?
  <5-60>  Scanner interval (seconds)

ios-xe(config-router)#bgp scan-time 5
```

### IOS-XR

```
RP/0/0/CPU0:ios-xr#configure
Mon May 15 03:19:14.295 UTC
RP/0/0/CPU0:ios-xr(config)#router bgp 65001
RP/0/0/CPU0:ios-xr(config-bgp)#bgp scan-time ?
  <5-3600>  Scanner interval (seconds)
RP/0/0/CPU0:ios-xr(config-bgp)#bgp scan-time 5
```

> Again, this is such an incredibly horrible idea.  Never do this.
> Ever.

## Other Timers

The timers below are probably not covered by the blueprint, but they're
noted here just in case.

### Next Hop Tracking

Next Hop Tracking helps reduce some of the convergence problems that
the BGP Scanner alone couldn't handle.  Next Hop Tracking checks the
next hop table to determine if a next hop changed, and it updates BGP
accordingly.

#### IOS-XE

In IOS-XE, the timer can only be configured in seconds:

```
ios-xe#configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
ios-xe(config)#router bgp 65000
ios-xe(config-router)#bgp nexthop trigger enable
% BGP: Scan interval configuration reset to default because nexthop tracking has been enabled
ios-xe(config-router)#bgp nexthop trigger delay 1
```

> In this version of IOS-XE, NHT is not enabled by default.
> Additionally, the BGP scan timer gets reset by enabling NHT.

#### IOS-XR

In IOS-XR, there are two configurable timers: critical changes and
non-critical changes.  Non-critical changes are IGP metric changes.
Critical changes are everything else.  In addition, the timers are
configured in miliseconds in IOS-XR instead of seconds.

```
RP/0/0/CPU0:ios-xr#configure
Mon May 15 03:29:51.271 UTC
RP/0/0/CPU0:ios-xr(config)#router bgp 65001
RP/0/0/CPU0:ios-xr(config-bgp)#address-family ipv4 unicast
RP/0/0/CPU0:ios-xr(config-bgp-af)#nexthop trigger-delay critical 5
RP/0/0/CPU0:ios-xr(config-bgp-af)#nexthop trigger-delay non-critical 5
```

### Advertisement Interval

This interval depends upon whether the neighbor is an eBGP or an iBGP
neighbor.  If the neighbor is an eBGP neighbor, then the default timer
is 30 seconds.  If it is iBGP, it is 0 seconds.  This interval is
configured on a per-neighbor basis.

#### IOS-XE

```
ios-xe#configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
ios-xe(config)#router bgp 65000
ios-xe(config-router)#neighbor 1.2.3.4 advertisement-interval 0
```

#### IOS-XR

```
RP/0/0/CPU0:ios-xr#configure
Mon May 15 03:36:44.043 UTC
RP/0/0/CPU0:ios-xr(config)#router bgp 65001
RP/0/0/CPU0:ios-xr(config-bgp)#neighbor 1.2.3.4
RP/0/0/CPU0:ios-xr(config-bgp-nbr)#advertisement-interval 0
```

> There are other timers, but it's even less like that they are needed
> for the SPADVROUTE exam.

# Describe the Need for BGP Confederations

Short answer: they're never necessary for any deployment anywhere.  You
should feel bad if you've ever recommended them.

Okay, fine, the real answer is below.

Full iBGP meshes are not scalable beyond even just a few routers.  A
full iBGP mesh requires `N(N-1)/2` sessions.  Even for just four
routers, this is 6 sessions.  For six routers, this becomes 15 sessions.
That's a lot of sessions and a lot of typing.

Confederations ease this burden by splitting a single autonomous system
into multiple smaller systems.  These smaller systems present a unified
Autonomous System Number externally, but internally they reference each
other with a private ASN.  The sub-autonomous systems know that they are
part of the same confederation by agreeing on a confederation identifier.
Any confederation identifying information is stripped on true eBGP
connections.  Some BGP attributes--MED, Next Hop, and Local
Preference--are maintained between cBGP ("confederation BGP" peers)
sessions.

Although not explicitly called out by the syllabus, a basic
confederation configuration is presented below.

## IOS-XE

```
ios-xe#configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
ios-xe(config)#router bgp 65000
ios-xe(config-router)#bgp confederation identifier ?
  <1-4294967295>  AS number
  <1.0-XX.YY>     AS number

ios-xe(config-router)#bgp confederation identifier 1234
ios-xe(config-router)#bgp confederation peers 65001 65002
```

## IOS-XR

```
RP/0/0/CPU0:ios-xr#configure
Mon May 15 03:46:44.442 UTC
RP/0/0/CPU0:ios-xr(config)#router bgp 65001
RP/0/0/CPU0:ios-xr(config-bgp)#bgp confederation identifier ?
  <1-65535>           2-byte AS number
  <1-65535>.          4-byte AS number in asdot (X.Y) format - first half (X)
  <65536-4294967295>  4-byte AS number in asplain format
RP/0/0/CPU0:ios-xr(config-bgp)#bgp confederation identifier 1234
RP/0/0/CPU0:ios-xr(config-bgp)#bgp confederation peers
RP/0/0/CPU0:ios-xr(config-bgp-confed-peers)#65001
RP/0/0/CPU0:ios-xr(config-bgp-confed-peers)#65002
```

# Next Time

In the next post, we'll look at BGP Route Reflection in IOS-XE and
IOS-XR.