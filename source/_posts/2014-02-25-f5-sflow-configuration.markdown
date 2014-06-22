---
layout: post
title: "Snippet: F5 sFlow Configuration"
date: 2014-02-25 22:18:23 -0700
comments: true
categories:
  - f5
  - monitoring
  - snippet
---

# F5 sFlow Configuration

In the first of a new series of snippets, we cover configuring an sFlow
collector on an F5 Networks Big IP LTM 7200v!

> While the target platform is an LTM 7200v, this procedure should work
> on any F5 Networks platform running Big IP v11 or greater.  It may
> work with other versions of Big IP if they support the `tmsh`
> interface.

## Necessary Information

Before we get started, let's just cover some necessary information.
First, I assume you have a collector somewhere capable of receiving
sFlow data and that you know how to configure it properly.  In the
examples below, my collector has the IP address `192.168.0.1` and
expects to receive sFlow data on UDP port `9993`.

<!-- more -->

## Configuration

> IP addresses and port numbers have been changed to protect the
> innocent.

### Snippet
``` bash
create receiver nms address 192.168.0.1 port 9993 state enabled
```

### Example
``` bash
[admin@7200v:Active:In Sync] ~ # tmsh
admin@(7200v)(cfg-sync In Sync)(Active)(/Common)(tmos)# sys sflow
admin@(7200v)(cfg-sync In Sync)(Active)(/Common)(tmos.sys.sflow)# create receiver noms address 192.168.0.1 port 9993 state enabled
admin@(7200v)(cfg-sync In Sync)(Active)(/Common)(tmos.sys.sflow)# list
sys sflow global-settings http { }
sys sflow global-settings interface { }
sys sflow global-settings system { }
sys sflow global-settings vlan { }
sys sflow receiver noms {
    address 192.168.0.1
    port palace-2
    state enabled
}
admin@(7200v)(cfg-sync In Sync)(Active)(/Common)(tmos.sys.sflow)# quit
```

> `port palace-2` is a "well-known" port that the F5 translates to a
> name.  It is port `9993`.

### Verification

``` bash
[admin@7200v:Active:In Sync] ~ # tcpdump -c 5 -nni any host 192.168.0.1 and udp and port 9993 -vv
tcpdump: listening on any, link-type EN10MB (Ethernet), capture size 96 bytes
20:08:44.683361 IP (tos 0x0, ttl 255, id 54233, offset 0, flags [DF], proto: UDP (17), length: 208) 192.168.0.2.46343 > 192.168.0.1.9993: UDP, length 180
20:08:44.696217 IP (tos 0x0, ttl 255, id 37826, offset 0, flags [DF], proto: UDP (17), length: 208) 192.168.0.2.39833 > 192.168.0.1.9993: UDP, length 180
20:08:44.706779 IP (tos 0x0, ttl 255, id 37896, offset 0, flags [DF], proto: UDP (17), length: 228) 192.168.0.2.39833 > 192.168.0.1.9993: UDP, length 200
20:08:44.720309 IP (tos 0x0, ttl 255, id 37994, offset 0, flags [DF], proto: UDP (17), length: 272) 192.168.0.2.39833 > 192.168.0.1.9993: UDP, length 244
20:08:44.731756 IP (tos 0x0, ttl 255, id 38056, offset 0, flags [DF], proto: UDP (17), length: 208) 192.168.0.2.39833 > 192.168.0.1.9993: UDP, length 180
5 packets captured
5 packets received by filter
0 packets dropped by kernel
[admin@7200v:Active:In Sync] ~ #
```

> In the `tcpdump` example, feel free to go a step further and add `-X`
> or `-XX` to get the actual contents of the packet.  These options were
> left off in the interest of brevity.

## THE END!
