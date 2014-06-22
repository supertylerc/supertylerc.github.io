---
layout: post
title: "TACACS+ [2/3] - Junos Configuration"
date: 2014-01-20 20:33:09 -0700
comments: true
categories:
  - tacacs
  - authentication
  - oss
---

> Note: This is Part 2 in a series of posts on TACACS+ installation and
> deployment.

> You should start with [the special precursor][1], followed by [Part 1:
> Downloading and Compiling][2].

One of major vendors that supports TACACS+ is Juniper Networks.
Enabling TACACS+ for a Juniper Networks device is simple.  It's made
even easier by the fact that the configuration is the same across their
major routing, switching, and security platforms.

> Note: Some platforms, such as the MAG series (which runs IVE), do not
> support TACACS+ authentication.

## TACACS+ Configuration

We'll start with the TACACS+ configuration.  We already have the base
configuration from [part 1][1].  We're only going to be modifying the
sections of the configuration labeled as `group`.  This is for
scalability reasons--as your users grow, you can simply add new users to
the appropriate group instead of redefining their permissions for every
user.

Let's start with the `engineers` group.  We're going to add a simple
section to the group that will grant members the same permissions as a
local Junos user called "**admin**".

<!-- more -->

Start by opening the file:

``` bash
cd /opt/oss-conf
vim tac_plus/tac_plus.conf
```

Now, insert the lines below.

``` bash
group = engineers {
  default service = permit
  service = junos-exec {
    local-user-name = admin
  }
}
```

> Remember: press `i` to enter insert mode, the `[escape]` key to exit
> insert mode, and `:wq` to save and close the file.

Next we'll add and commit the changes to our fancy configuration
revision repository:

``` bash
git add tac_plus/tac_plus.conf
git commit -m 'add junos permissions for engineers'
```

Now, let's fix up the `noc` group, which will map to a local Junos user
claled "**noc**".  Once again, open the file:

``` bash
vim tac_plus/tac_plus.conf
```

Add the following lines:

``` bash
 group = noc {
  default service = permit
  service = junos-exec {
    local-user-name = noc
  }
 }
```

And add the changes and commit them in git:

``` bash
git add tac_plus/tac_plus.conf
git commit -m 'add junos permissions for engineers'
```

> You could have just done this all together, but when you work with
> revision control, you generally want to break changes up into the
> smallest logical blocks possible.  This makes rolling back changes
> much easier and less invasive.

That's all there is to it for the TACACS+ configuration!  We'll cirlce
back later to show ways to allow specific commands later--after we show
the Junos configuration

## Configuring Junos

The Junos configuration is shown below.  Note that you'll need to
replace IP addresses with your own relevant values.

``` bash
root@lab01# show | compare
[edit system]
+  authentication-order [ tacplus password ];
+  tacplus-server {
+      192.168.1.254 {
+          port 49;
+          secret "$9$J0GUjTQntuO.P0BRhvMJGUHTF9A0RcyKMZUHkTQSrlMWL"; ## SECRET-DATA
+          single-connection;
+          source-address 192.168.1.1;
+      }
+  }
+  accounting {
+      events [ login interactive-commands ];
+      destination {
+          tacplus;
+      }
+  }
+  login {
+      class admin {
+          login-alarms;
+          permissions all;
+      }
+      class noc {
+          permissions [ view view-configuration ];
+      }
+      user engineers {
+          full-name "TACACS+ - Engineers";
+          class admin;
+          authentication {
+              encrypted-password "$1$2FA/QHsx$Os2dyH/PdLiD9t96flygS/"; ## SECRET-DATA
+          }
+      }
+      user noc {
+          full-name "TACACS+ - NOC Techs";
+          class noc;
+          authentication {
+              encrypted-password "$1$4wt52bTA$niTEoQJBfp3maQkVTvCI60"; ## SECRET-DATA
+          }
+      }
+  }

{master:0}[edit]
```

> The configuration above is in patch form.  You can load it by copying
> everything form `[edit system]` to the last line with a `+`, then
> entering configuration mode in your Juniper device, and typing `load
> patch terminal`, then pasting the patch and pressing `^D` (control+D).

## Restart TACACS+

The next step is to restart TACACS+ if it's running.  Check to see if
it's running:

``` bash
sudo status tac_plus
```

If it's not running:

``` bash
sudo start tac_plus
```

If it is running:

``` bash
sudo stop tac_plus
sudo start tac_plus
```

Now, test your new configuration:

``` bash
ssh jim@192.168.1.1
```

Junos should show you logged in as the user "**jim**".  You should be
able to use show commands, but not change configuration.

``` bash
jim@lab02> show vlans summary

VLANs summary:
    Total: 27,  Configured VLANs: 26
    Internal VLANs: 1,  Temporary VLANs: 0

Dot1q VLANs summary:
    Total: 27, Tagged VLANs: 26, Untagged VLANs: 1

{master:0}
jim@lab02> show configuration routing-options
static {
    route 0.0.0.0/0 next-hop 192.168.1.1;
}

{master:0}
jim@lab02> ping
           ^
unknown command.
jim@lab02>
```

## Extending Permissions

You'll probably notice that although the "**jim**" user can run some
show commands and view the configuration, he can't use commands such as
`ping` and `traceroute`.  Let's fix that now.  Let's also prevent
members of the "**noc**" group from viewing the configuration under the
`system` hierarchy.

To start, let's follow the same steps we've been following.  Don't
forget to see the note above for a refresher on `vim` if necessary!

``` bash
cd /opt/oss-conf
vim tac_plus/tac_plus.conf
```

Now, find the "**noc**" group and make it look like this:

``` bash
group = noc {
  default service = permit
  service = junos-exec {
    local-user-name = noc
    allow-commands1="ping.*"
    allow-commands2="traceroute.*"
    deny-configuration1="system.*"
  }
}
```

Add and commit to your repository:

``` bash
git add tac_plus/tac_plus.conf
git commit -m 'fix command auth for noc group'
```

And restart tac_plus:

``` bash
sudo stop tac_plus
sudo start tac_plus
```

### The results

``` bash
 jim@lab01> ping 192.168.1.1 rapid
 PING 192.168.1.1 (192.168.1.1): 56 data bytes
 !!!!!
 --- 192.168.1.1 ping statistics ---
 5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max/stddev = 0.305/1.768/4.718/1.598 ms

{master:0}
jim@lab02> traceroute 192.168.1.1
traceroute to 192.168.1.1 (192.168.1.1), 30 hops max, 40 byte packets
 1  192.168.2.1 (192.168.2.1)  91.231 ms  90.658 ms  95.165 ms
 2  192.168.1.1 (192.168.1.1)  95.484 ms  94.621 ms  103.436 ms

{master:0}
jim@lab02> show configuration system
                                     ^
permission denied.

{master:0}
jim@lab02> show cli authorization
Current user: 'noc' login: 'rancid' class 'noc'
Permissions:
    view        -- Can view current values and statistics
    view-configuration-- Can view all configuration (not including secrets)
Individual command authorization:
    Allow regular expression: (ping.*|traceroute.*)
    Deny regular expression: none
    Allow configuration regular expression: none
    Deny configuration regular expression: (system.*)

{master:0}
jim@lab02>
```

## The End <small>For now...</small>

Once again, we've reached the end of another part of the TACACS+ series.
The final piece will focus on doing everything we did here, but for
Cisco's NXOS platform.  But don't worry!  There will be additional,
supplemental posts in the future to show off other features.

[1]: /blog/tacacs-introduction-to-aaa/ "TACACS+ [0/3] - Introduction to AAA"
[2]: /blog/tacacs-downloading-and-compiling/ "TACACS+ [1/3] - Downloading and Compiling"
