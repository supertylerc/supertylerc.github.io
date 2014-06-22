---
layout: post
title: "TACACS+ [3/3] - Nexus Configuration"
date: 2014-01-21 20:43:00 -0700
comments: true
categories:
  - tacacs
  - authentication
  - oss
---

> Note: This is Part 3 in a series of posts on TACACS+ installation and
> deployment.

> You should read these posts in order if you're not familiar with
> TACACS+:

> * [TACACS+ [0/3] - Introduction to AAA][1]
> * [TACACS+ [1/3] - Downloading and Compiling][2]
> * [TACACS+ [2/3] - Junos Configuration][3]

In [Part 2][3], we covered the configuration of a Juniper Networks
device to authenticate against a TACACS+ server.  In this part, we'll
cover the configuration necessary for a Cisco Nexus switch running NXOS.

## TACACS+ Configuration

Just like in [Part 2][3], we're going to be starting with the TACACS+
configuration.  Most of the work is already done for us--we'll be adding
another service the same way we did previously and the configuration
will be complete.

Start by opening the TACACS+ configuration file:

``` bash
cd /opt/oss-conf
vim tac_plus/tac_plus.conf
```

Next, find the `group` section for "**engineers**".  You'll want to make
the section look like it does below by adding the new `service` stanza:

``` bash
group = engineers {
  default service = permit
  service = junos-exec {
    local-user-name = admin
  }
  service = exec {
    shell:roles="\"network-admin\""
  }
}
```

> Remember: press `i` to enter insert mode, the `[escape]` key to exit
> insert mode, and `:wq` to save and close the file.

<!-- more -->

Next, we'll add and commit the changes to our configuration repository:

``` bash
git add tac_plus/tac_plus.conf
git commit -m 'add nxos permissions for engineers'
```

> Remember that we're adding commits for each minor thing we do (that
> adds complete functionality for a given need).  This is generally
> considered a best practice.

Next, let's set the `noc` group to have read-only access:

``` bash
vim tac_plus/tac_plus.conf
```

Add the necessary lines below to match your configuration to this
example:

``` bash
group = noc {
  default service = permit
  service = junos-exec {
    local-user-name = noc
  }
  service = exec {
    shell:roles="\"network-operator\""
  }
}
```

And add and commit the changes:

``` bash
git add tac_plus/tac_plus.conf
git commit -m 'add nxos permissions for noc techs'
```

## Configuring NXOS

The NXOS configuration is shown below. Note that you'll need to replace
IP addresses with your own relevant values.

``` bash
lab03(config)# sh run tacacs+

!Command: show running-config tacacs+
!Time: Sat Jan 18 22:51:01 2014

version 5.2(1)N1(3)
feature tacacs+

ip tacacs source-interface mgmt0
tacacs-server host 192.168.1.254 key 7 "!e-q0v3_l4M4s5#"
aaa group server tacacs+ lab
    server 192.168.1.254
    use-vrf management


lab03(config)#
```

> Start under the `version 5.2(1)N1(3)` line.  Since we're using
> preexisting groups, you don't need to create them like we did with
> Junos.

## Restart TACACS+

``` bash
sudo stop tac_plus
sudo start tac_plus
```

### Test the NOC Tech <small>His name is Jim</small>

``` bash
ssh jim@192.168.1.3
```

You should be able to run commands such as `show run`, `ping`, and
`traceroute`.

``` bash
lab03# show vlan summary

Number of existing VLANs           : 20
Number of existing user VLANs      : 20
Number of existing extended VLANs  : 0


lab03# show running-config tacacs+

!Command: show running-config tacacs+
!Time: Sat Jan 18 23:45:22 2014

version 5.2(1)N1(3)
feature tacacs+

ip tacacs source-interface mgmt0
tacacs-server host 192.168.1.254 key 7 "!e-q0v3_l4M4s5#"
aaa group server tacacs+ lab
    server 192.168.1.254
    use-vrf management


lab03# ping 192.168.1.1 vrf management
PING 192.168.1.1 (192.168.1.1): 56 data bytes
64 bytes from 192.168.1.1: icmp_seq=0 ttl=63 time=1.494 ms
64 bytes from 192.168.1.1: icmp_seq=1 ttl=63 time=2.047 ms
64 bytes from 192.168.1.1: icmp_seq=2 ttl=63 time=1.099 ms
64 bytes from 192.168.1.1: icmp_seq=3 ttl=63 time=1.29 ms
64 bytes from 192.168.1.1: icmp_seq=4 ttl=63 time=1.1 ms

--- 192.168.1.1 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 1.099/1.405/2.047 ms
lab03# traceroute 192.168.1.1 vrf management
traceroute to 192.168.1.1 (192.168.1.1), 30 hops max, 40 byte packets
 1  192.168.1.1 (192.168.1.1)  1.567 ms  1.713 ms  1.323 ms
lab03#
```

## Fine-grained Control

Nexus switches don't seem to allow fine-grained control of which
commands can be executed, unlike Junos and IOS.  That or I haven't found
the special sauce yet.

## The End <small>Finally!</small>

This post concludes the three-part series on TACACS+, but don't worry!
There will be more posts exploring other features and additional vendor
integration.

[1]: /blog/tacacs-introduction-to-aaa/ "TACACS+ [0/3] - Introduction to AAA"
[2]: /blog/tacacs-downloading-and-compiling/ "TACACS+ [1/3] - Downloading and Compiling"
[3]: /blog/tacacs-junos-configuration/ "TACACS+ [2/3] - Junos Configuration"
