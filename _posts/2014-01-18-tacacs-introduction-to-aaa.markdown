---
layout: post
title: "TACACS+ [0/3] - Introduction to AAA"
date: 2014-01-18 11:58:54 -0700
comments: true
categories:
  - tacacs
  - authentication
  - oss
---

> Note: this is a special precursor to a series of posts related to AAA
> (authentication, authorization, and accounting) in general and TACACS+
> specifically.

These days, it seems that everyone is using a centralized authentication
mechanism.  There are certainly a lot of ways to do it, too--scripts
that push SSH keys to every device, tried-and-true services like RADIUS
and TACACS+, or just throwing any possible form of auditing out the door
and handing everyone the same generic credentials.  Let's just pretend
that the last one in that list doesn't happen in the real world.

## AAA <small>Authentication, Authorization, Accounting</small>

AAA is all about one thing: accountability.  You can argue the
"authentication" and "authorization" parts, but lets face it: those
exist only to hold technicians and engineers accountable for what they
do.

### Why You Want It

If you're not using a AAA solution already, you're probably Doing It
Wrong&trade;.  If you have 5 devices and one person accessing those
devices, you can get away with skipping out on AAA.  If your environment
is any more complex, you should seriously consider a AAA solution.

AAA solutions allow you to centralize authentication and authorization,
as well as log accounting information (such as who entered which
commands on which devices at what times).  This is a powerful
combination--being able to quickly determine who caused an outage can
greatly reduce the time it takes to determine a reason for an outage,
and it has the added benefit of having concrete proof when you coach
your employees on their actions.

<!-- more -->

### RADIUS <small>VS</small> TACACS+

There's a pretty good argument between some people on the merits of
RADIUS versus TACACS+.  The biggest case for RADIUS that I've seen is
that it's been around longer and has more integration with various other
services.  For example, Apache has easy-to-use RADIUS modules for
authentication, but the TACACS+ module that's out there is poorly
documented and may not even work depending on your version of Apache.  

Why is this even important?  Well, remember that this is called the "OSS
Stack," and the whole idea is to consolidate and integrate systems as
much as possible.  This means that applications such as Cacti that can
take advantage of Apache's authentication mechanisms can be tied into
your RADIUS deployment.  This is moderately defeated by the fact that
permissions must be configured in Cacti, which sort of duplicates effort
and work required, but it has its merits.

TACACS+, on the other hand, gives you extreme flexibility.  I define a
set of roles on a device, then configure TACACS+ to bind a user to those
roles.  TACACS+ can be further integrated with Active Directory,
OpenLDAP, `/etc/passwd`, MySQL, or a plain text file.  RADIUS can do
this as well, but it's much more difficult and involved.

Need more reason to go with TACACS+ instead of RADIUS?  Here's a few
quick ones:

* RADIUS uses UDP; TACACS+ uses TCP
* Because of the use of TCP, TACACS+:
  * Scales higher
  * Handles congestion better
  * Handles latency better
* RADIUS encrypts the header, but leaves all other packet contents
  unencrypted
* TACACS+ encrypts the entire packet, leaving a common header
  unencrypted to indicate if the packet is encrypted or not
* TACACS+ was designed to separate the components of AAA
  * This makes using certain components alone or in conjunction with
    other services easier than with RADIUS
* TACACS+ can control which commands a user can utilize in addition to
  that user's role
  * RADIUS cannot do this

For the above reasons, we're going to be going with TACACS+.  Most
articles at The OSS Stack will stick with TACACS+, although RADIUS will
be explored in certain posts as it is necessary for certain
infrastructure (notably 802.1x).

### Integrating with Authentication Infrastructure

One of the great things you can do with TACACS+ is integrate it with
your existing authentication infrastructure.  This means if you already
have Active Directory or OpenLDAP deployments, you can configure TACACS+
to pass the authentication off to those services.

We will <strong>NOT</strong> be doing that in this series.  This type of
integration, while good and highly sought after, introduces a terrible
point of failure.  Equipment lockout has been observed under certain
conditions where the TACACS+ server is available, but the backend AD or
LDAP server is not.  This causes a router or switch to reach the TACACS+
server but fail to authenticate (because the backend Active Directory,
OpenLDAP, or RSA server is down), resulting in all users being unable to
log into the device.

However, as this is a common desire amongst administrators, it will be
covered in a future supplemental article with a very large and in-depth
disclaimer.

## Other Options

There are, of course, other options.  Routers, switches, and firewalls
from Juniper Networks can be configured to log all commands to a
separate logfile, and an engineer can write a script that copies SSH
keys to devices that support key-based authentication.  There are a few
problems with this, namely that not all devices support key-based
authentication.  There is also a problem when logging in from a laptop
or iPad that does not have its public key loaded on the network device.
It's also an administrative nightmare when users need to be removed on
all devices.

For this reason, these types of "centralized" authentication and/or
logging will not be explored here at OSS Stack.
