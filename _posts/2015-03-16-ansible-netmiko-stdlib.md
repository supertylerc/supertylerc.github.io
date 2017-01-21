---
layout: post
title: "ansible-netmiko-stdlib"
description: "Using the ansible-netmiko-stdlib module"
category: automation
tags: [ automation, cisco, ansible, ios ]
---

I hacked together an [Ansible](http://www.ansible.com/home) module called
`ansible-netmiko-stdlib`.  This post discusses how it can be used, what its
limitations are, and shows an example of the module in action.  You can follow
along by grabbing the source for this post's example
[here](https://github.com/supertylerc/ios-deploy).

## Setting Up

To set up to follow along, you should clone the above repository.

{% highlight bash %}
mkdir ~/git
cd ~/git
git clone https://github.com/supertylerc/ios-deploy
{% endhighlight %}

You should have two Cisco IOS devices that are connected to each other.
They can be connected via any type of interface--just make a note of it.  You
should edit the `~/git/ios-deploy/hosts` file with a name that will resolve to
an IP address for each of the two hosts.  If you don't have FQDNs fro them, you
can modify your `/etc/hosts` file and add two lines similar to below:

{% highlight bash %}
192.168.0.100 cr1
192.168.0.101 cr2
{% endhighlight %}

Next, edit the two files in `~/git/ios-deploy/host_vars/`.  You should only need
to change the interface that is used.  Abbreviated interface names (such as
`gi3`) will work, but ensure that they are unique enough for IOS to decipher the
interface.

You should also configure your IOS devices with some basic configuration.  At a
minimum, each device should have the following configured:

* Interface: admin up and IP address
* User: ansible/Ansible123, privilege 15
* SSH: SSHv2 for the ansible user

Almost finished!  You'll need the actual Ansible module.  Follow the
instructions for the module
[here](https://github.com/supertylerc/ansible-netmiko-stdlib).

Finally, run the playbook: `ansible-playbook -i hosts deploy.yml`.  That command
needs to be run from the repository `ios-deploy`.

# Module in Action

I wrote a role and playbook to show off the module.  It's not terribly complex,
but it does show off the features of the module fairly well.  I have two IOS
devices connected to each other.  Their interfaces are configured, and there
is a user called `ansible` that has been configured.

> By "their interfaces are configured", I mean that they have IP addresses and
> they are administratively up.

## Running the Playbook

Now, let's see the playbook in action!

{% highlight bash %}
 ❯ ansible-playbook -i hosts deploy.yml

PLAY [One-Time Setup] *********************************************************

TASK: [Get Timestamp] *********************************************************
changed: [localhost]

PLAY [Base Directory Setup] ***************************************************

TASK: [Create Log Directory] **************************************************
changed: [cr1]
changed: [cr2]

TASK: [Create Diff Directory] *************************************************
changed: [cr1]
changed: [cr2]

TASK: [Clean Build Directory] *************************************************
ok: [cr1]
ok: [cr2]

TASK: [Clean Config Directory] ************************************************
ok: [cr1]
ok: [cr2]

TASK: [Create Build Directory] ************************************************
changed: [cr1]
changed: [cr2]

TASK: [Create Config Directory] ***********************************************
changed: [cr1]
changed: [cr2]

PLAY [Build Config] ***********************************************************

TASK: [common | Router ID Configuration] **************************************
changed: [cr2]
changed: [cr1]

TASK: [common | OSPF Configuration] *******************************************
changed: [cr2]
changed: [cr1]

PLAY [Compile Config] *********************************************************

TASK: [Assemble Fragments] ****************************************************
changed: [cr2]
changed: [cr1]

PLAY [Deploy Config] **********************************************************

TASK: [Deploy Cisco Systems] **************************************************
changed: [cr1]
changed: [cr2]

PLAY RECAP ********************************************************************
cr1                        : ok=10   changed=8    unreachable=0    failed=0
cr2                        : ok=10   changed=8    unreachable=0    failed=0
localhost                  : ok=1    changed=1    unreachable=0    failed=0

>>> elapsed time 34s

 ❯
{% endhighlight %}

## Understanding What's Going On

Looks like a _lot_ going on just to deploy OSPF, right?

Not really.  A lot of it is housekeeping.  Ensure build and config directories are
clean.  Ensure all of the necessary directories exist.  Get the current time.
Use [Jinja2](http://jinja.pocoo.org) to template the OSPF configuration (by way
of the Ansible [`template`](http://docs.ansible.com/template_module.html) module).
Use the `netmiko_install_config` module to push the config, diff the configs, and
log all of the work.

## Reviewing the Results

First, I can tell you that the deploy was successful by logging into the box.
Here's the output of `show ip ospf neighbor detail` after I ran the playbook:

#### cr1

{% highlight bash %}
cr1#show ip ospf neighbor detail
 Neighbor 192.168.0.101, interface address 172.16.0.1
    In the area 0.0.0.0 via interface GigabitEthernet3
    Neighbor priority is 0, State is FULL, 6 state changes
    DR is 0.0.0.0 BDR is 0.0.0.0
    Options is 0x12 in Hello (E-bit, L-bit)
    Options is 0x52 in DBD (E-bit, L-bit, O-bit)
    LLS Options is 0x1 (LR)
    Dead timer due in 00:00:33
    Neighbor is up for 00:18:04
    Index 1/1/1, retransmission queue length 0, number of retransmission 0
    First 0x0(0)/0x0(0)/0x0(0) Next 0x0(0)/0x0(0)/0x0(0)
    Last retransmission scan length is 0, maximum is 0
    Last retransmission scan time is 0 msec, maximum is 0 msec
cr1#
{% endhighlight %}

#### cr2

{% highlight bash %}
cr2#show ip ospf neighbor detail
 Neighbor 192.168.0.100, interface address 172.16.0.0
    In the area 0.0.0.0 via interface GigabitEthernet3
    Neighbor priority is 0, State is FULL, 6 state changes
    DR is 0.0.0.0 BDR is 0.0.0.0
    Options is 0x12 in Hello (E-bit, L-bit)
    Options is 0x52 in DBD (E-bit, L-bit, O-bit)
    LLS Options is 0x1 (LR)
    Dead timer due in 00:00:32
    Neighbor is up for 00:18:48
    Index 1/1/1, retransmission queue length 0, number of retransmission 0
    First 0x0(0)/0x0(0)/0x0(0) Next 0x0(0)/0x0(0)/0x0(0)
    Last retransmission scan length is 0, maximum is 0
    Last retransmission scan time is 0 msec, maximum is 0 msec
cr2#
{% endhighlight %}

> Although manually logging in and verifying the expected results is fine
> (and a good idea!), it isn't scalable.  Look into automated testing of
> your changes.  I presented recently on this, and I've written a post
> with more to follow about automated network testing and verification.

### Logs

The module will also write logfiles if you pass it the `log_file` parameter.
Here's the output of the logs for each router.

#### cr1

{% highlight bash %}
 ❯ cat /tmp/cr1/log/2015-03-14_154936.log
2015-03-14 15:49:37,323:CONFIG:cr1:connecting to cr1
2015-03-14 15:49:37,510:paramiko.transport:Connected (version 1.99, client Cisco-1.25)
2015-03-14 15:49:38,071:paramiko.transport:Authentication (password) successful!
2015-03-14 15:49:43,613:CONFIG:cr1:loading /tmp/cr1/config/compiled.cnf

 ❯
{% endhighlight %}

#### cr2

{% highlight bash %}
 ❯ cat /tmp/cr2/log/2015-03-14_154936.log
2015-03-14 15:49:37,325:CONFIG:cr2:connecting to cr2
2015-03-14 15:49:37,509:paramiko.transport:Connected (version 1.99, client Cisco-1.25)
2015-03-14 15:49:38,067:paramiko.transport:Authentication (password) successful!
2015-03-14 15:49:43,624:CONFIG:cr2:loading /tmp/cr2/config/compiled.cnf

 ❯
{% endhighlight %}

They aren't extremely descriptive, and the logging will be improved over time, but
you can at least get a basic idea of where things might have gone wrong (if, for
example, the authentication failed).

### Diffs

This is a feature that's particularly powerful: diffing the configs.  This is
handled by the module automatically if you pass the `diff_file` parameter.  Let's
see what it looks like!

#### cr1

{% highlight diff %}
 ❯ cat /tmp/cr1/diff/2015-03-14_154936.log
--- old config
+++ new config
@@ -1,8 +1,8 @@
 Building configuration...

-Current configuration : 1253 bytes
+Current configuration : 1355 bytes
 !
-! Last configuration change at 22:27:36 UTC Sat Mar 14 2015
+! Last configuration change at 22:50:05 UTC Sat Mar 14 2015 by ansible
 !
 version 15.5
 service timestamps debug datetime msec
@@ -90,12 +90,17 @@
 interface GigabitEthernet3
  description to cr2
  ip address 172.16.0.0 255.255.255.254
+ ip ospf network point-to-point
+ ip ospf 1 area 0.0.0.0
  negotiation auto
 !
 interface GigabitEthernet4
  no ip address
  shutdown
  negotiation auto
+!
+router ospf 1
+ router-id 1.1.1.1
 !
 !
 virtual-service csr_mgmt

 ❯
{% endhighlight %}

#### cr2

{% highlight diff %}
 ❯ cat /tmp/cr2/diff/2015-03-14_154936.log
--- old config
+++ new config
@@ -1,8 +1,8 @@
 Building configuration...

-Current configuration : 1181 bytes
+Current configuration : 1283 bytes
 !
-! Last configuration change at 22:27:53 UTC Sat Mar 14 2015
+! Last configuration change at 22:50:04 UTC Sat Mar 14 2015 by ansible
 !
 version 15.5
 service timestamps debug datetime msec
@@ -90,7 +90,12 @@
 interface GigabitEthernet3
  description to cr1
  ip address 172.16.0.1 255.255.255.254
+ ip ospf network point-to-point
+ ip ospf 1 area 0.0.0.0
  negotiation auto
+!
+router ospf 1
+ router-id 2.2.2.2
 !
 !
 virtual-service csr_mgmt

 ❯
{% endhighlight %}

### Configs

I've shown you everything except for the actual lines that were sent to the remote
IOS devices.  The playbook also records that data.  Note that it's the playbook
that saves a copy of the final config, not the module, and the playbook will
clear out the config directory on its next run--so be sure to review before that
happens!

#### cr1

{% highlight bash %}
 ❯ cat /tmp/cr1/config/compiled.cnf
router ospf 1
!
interface gi3
    ip ospf network point-to-point
    ip ospf 1 area 0.0.0.0
!
router ospf 1
  router-id 1.1.1.1

 ❯
{% endhighlight %}

#### cr2

{% highlight bash %}
 ❯ cat /tmp/cr2/config/compiled.cnf
router ospf 1
!
interface gi3
    ip ospf network point-to-point
    ip ospf 1 area 0.0.0.0
!
router ospf 1
  router-id 2.2.2.2

 ❯
{% endhighlight %}

And now you know a lot about everything this playbook just did and can do!  What's
next?

# Next Steps

This module _is_ multi-vendor, so if [Netmiko](https://github.com/ktbyers/netmiko)
supports a vendor, then so does this module.  There's a caveat, however: the
`diff_file` parameter only supports devices for which the command to show the
current configuration is `show running-config`.  This will change in future
versions, though, so keep an eye on the changelog!

Another issue that will change in future versions is that this module does not make
changes to the configuration permanent.  In other words, it doesn't run the
`copy running-config startup-config` or `write memory` commands.

The above issues will be addressed in the near future.  If you want to contribute,
you can open pull requests and I will review and merge them when appropriate to
do so.

# Thanks!

A very big thank you to [Ed Henry](https://twitter.com/NetworkN3rd) who reviewed
this post prior to its publication.
