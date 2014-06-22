---
layout: post
title: "Exploring Junos Traffic Monitoring"
date: 2014-02-27 22:20:40 -0700
comments: true
categories:
  - juniper
  - monitoring
---

While preparing for another post, I ran into an interesting Junos
"feature."  First, the problem statement: I needed to monitor traffic
for all interfaces simultaneously for a specific set of conditions.  In
Junos, this is not possible.  Let's explore this below.

## The Attempt

Below, I'll step through the various attempts I made to monitor all
traffic on the device.

First, I made the attempt from the Junos shell with the `monitor
traffic` command.  I utilized the `matching` knob, which takes a
`tcpdump`-like filter expression as an argument.

``` bash
tylerc@lab01> monitor traffic matching "host 192.168.0.1 && udp && port 9997"
verbose output suppressed, use <detail> or <extensive> for full protocol decode
Address resolution is ON. Use <no-resolve> to avoid any reverse lookup delay.
Address resolution timeout is 4s.
Listening on fxp0, capture size 96 bytes

^C
4 packets received by filter
0 packets dropped by kernel

{master:0}
```

<!-- more -->

After I noted that this listens on the management interface (`fxp0` in
this case) by default, I proceeded to drop to the shell and attempt to
utilize `tcpdump` directly.

``` bash
tylerc@lab01> start shell
% tcpdump -n host 192.168.0.1 and udp and port 9997
(no devices found) /dev/bpf0: Permission denied
% tcpdump
(no devices found) /dev/bpf0: Permission denied
%
```

After receiving a "**Permission denied**" error, I made myself the root
user and tried using `tcpdump` again.

``` bash
% whoami
tylerc
% su
Password:
root@core:RE:0% whoami
root
root@lab01:RE:0% tcpdump -n host 192.168.0.1 and udp and port 9997
verbose output suppressed, use <detail> or <extensive> for full protocol decode
Address resolution is OFF.
Listening on fxp0, capture size 96 bytes

^C
1 packets received by filter
0 packets dropped by kernel
root@lab01:RE:0% exit
exit
% exit
exit

{master:0}
tylerc@lab01>
```

> The `monitor traffic` operational command utilizes `tcpdump` under the
> hood.  The attempt here was to see if Junos might have abstracted
> certain capabilities.

As you can see above, there is no way to listen to all interfaces.
After some research, I've discovered that this is a BSD limitation.
There is no work around.

> You can read about it
> [here](https://www.mail-archive.com/tcpdump-workers@lists.tcpdump.org/msg03923.html).
> Although dated, the post is accurate.

## The Solution

The solution is actually pretty obvious: discover the outbound interface
and monitor traffic for it.  See the example below for determining the
route to our destination (**192.168.0.1**) and then listening for the
same details.

``` bash
tylerc@lab01> show route 192.168.0.1

inet.0: 477454 destinations, 952821 routes (477454 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

192.168.0.0/16      *[Static/5] 22w1d 06:04:53
                    > to 192.168.1.1 via fxp0.0

tylerc@lab01> monitor traffic matching "host 192.168.0.1 && udp && port 9997" count 5 no-resolve
verbose output suppressed, use <detail> or <extensive> for full protocol decode
Address resolution is OFF.
Listening on fxp0, capture size 96 bytes

08:20:28.123704 Out IP truncated-ip - 1384 bytes missing! 192.168.1.2.51925 > 192.168.0.1.9997: UDP, length 1416
08:20:28.123820 Out IP truncated-ip - 1384 bytes missing! 192.168.1.2.51925 > 192.168.0.1.9997: UDP, length 1416
08:20:28.123936 Out IP truncated-ip - 1384 bytes missing! 192.168.1.2.51925 > 192.168.0.1.9997: UDP, length 1416
08:20:28.124048 Out IP truncated-ip - 1384 bytes missing! 192.168.1.2.51925 > 192.168.0.1.9997: UDP, length 1416
08:20:28.124159 Out IP truncated-ip - 1384 bytes missing! 192.168.1.2.51925 > 192.168.0.1.9997: UDP, length 1416

tylerc@lab01>
```

Finally, you can see the output I was after.  Unfortunately, listening
on all interfaces isn't possible.  A combination of `show route
<address|network>` and `show route forwarding-table destination
<address>` will show you the appropriate information, though!

> In this particular example, the route I was interested in just
> happened to be via the management interface (the default for `monitor
> traffic`.  For other interesting data that you need to capture,
> however, this probably won't be the case--that's why the `show route`
> output is there.  Furthermore, you'll probably want to do a `show
> route forwarding-table destination <address>` in order to ensure
> you're monitoring the appropriate interface.

# A Word of Caution

I originally attempted this on an EX series switch, but there appears to
be a bug.  The switch _never_ captured any output for the port in which
I was interested, but I was able to confirm that it was sending the data
by monitoring the other side.  A case with JTAC will be opened to
explore that particular issue closer.
