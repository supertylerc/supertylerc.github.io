---
layout: post
title: "Snippet: Juniper NTP Configuration and Protection"
date: 2014-03-03 22:22:36 -0700
comments: true
categories:
  - juniper
  - ddos
  - snippet
---

Below, you'll see a quick way to configure NTP on a Junos device,
including how to secure the device from the current NTP amplification
attack that's floating around.

## Configuration

Below, I show a sample configuration in a lab environment.

> The firewall filter below should not be used in production
> environments.  The "**NTP**" section is secure; the rest is not.  Take
> the appropriate "**NTP**" section, but do not use the rest.  This is a
> lab environment!

<!-- more -->

``` bash
root@jnpr04-m7i# show | compare
[edit system]
+   ntp {
+       server 192.168.0.1;
+       server 192.168.0.5;
+   }
[edit interfaces]
+   lo0 {
+       unit 0 {
+           family inet {
+               filter {
+                   input RE_PROTECT;
+               }
+               address 4.4.4.4/32;
+           }
+       }
+   }
[edit policy-options]
    prefix-list RFC1918 { ... }
+   prefix-list NTP {
+       4.4.4.4/32;
+       apply-path "system ntp server <*>";
+   }
[edit]
+  firewall {
+      family inet {
+          filter RE_PROTECT {
+              term if-ntp {
+                  from {
+                      prefix-list {
+                          NTP;
+                      }
+                      protocol udp;
+                      port ntp;
+                  }
+                  then accept;
+              }
+              term if-ssh {
+                  from {
+                      protocol tcp;
+                      port ssh;
+                  }
+                  then accept;
+              }
+              term else {
+                  then {
+                      reject;
+                  }
+              }
+          }
+      }
+  }

[edit]
```

> It's important to note that you need to allow your loopback IP in the
> firewall filter, otherwise you won't be able to query yourself (which
> is what `show ntp associations` does).

## Verification

``` bash
root@jnpr04-m7i> show ntp associations
     remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
*192.168.0.1     17.168.198.149   2 -  186 1024  377   70.734    1.999   0.105
+192.168.0.5     108.71.253.18    2 -  216 1024  377   12.510   -0.580   0.356

{primary:node0}
```
