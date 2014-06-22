---
layout: post
title: "Things You Don't Know About Your Network"
date: 2014-03-20 22:27:13 -0700
comments: true
categories:
  - ddos
  - pentesting
---

Think your network is secure?  Guess again.  Chances are you have
glaring security holes in your network.  I'm not talking about random
holes that no one would think to check--I'm talking about very simple
things that give attackers all of the information they could possibly
want on your network equpiment.

> **DISCLAIMER**: Please do **NOT** attempt to perform any attacks or
> active reconnaissance based on the information in this article unless
> it is on your own network and you have permission to do so!  You could
> end up in jail, and I'm not responsible for your actions.  If you
> can't resist, just stop reading here.

## Selecting a Target Platform

For this very simple reconnaissance mission, we're going to target a
Junos platform that responds to public SNMP requests.  This allows us to
gain as much information as we could possibly want about our potential
targets and gives us an at-a-glance view of our potential target base.

I'll even be a bit more specific--we're looking for Juniper EX platforms
on the public Internet responding to public SNMP requests which may be
using a default community (or one easily brute forced) and most likely
running SNMPv2c.

<!-- more -->

## The Tool

Hop over to [ShodanHQ](http://shodanhq.com/) and poke around a bit.
When you're relatively satisfied, come back here and we'll keep going.

## The Query

Our search string is actually pretty simple.  It is just `ex junos
port:161 country:us`.  Enter that in the search box and hit enter.  Or
you could just point your browser
[here](http://www.shodanhq.com/search?q=ex+junos+port%3A161+country%3Aus).

> Note: You need an account (it's free) to search by country.  Feel free
> to leave the country off, though, and see your global target base.

The results may surprise you.  You don't even have to run the SNMP
reconnassaince yourself, really, as Shodan gives you the results of
querying the `sysDescr` MIB.  Now, here's the trick: it appears that
when Shodan is doing its poke at things, it's actually brute-forcing the
community--with a great deal of success, I might add.  Some of these may
be using a community string of **public**.  Others may be using
something equally simple but easily guessed.

To be honest, depending on the type of attack you'd like to ultimately
launch, you could just stop here.  But if you want more details about
the system--IP addresses, hardware installed, anything that sends its
details via SNMP--you could use SNMP as your only tool for gathering
information and have everything you need.

## Finding an Exploit

This is also trivial.  Let's say you knew you wanted to target Juniper
EX switches, but you didn't know what attacks were out there or what the
severity of those attacks are.  Enter Shodan once again, this time via
their [Shodan Exploits](https://exploits.shodan.io/) site.  Enter a
search query of `junos 10.4`, or follow [this
link](https://exploits.shodan.io/?q=junos+10.4).

> Note: there seems to be an issue with the links for exploits that
> results in 404s for all of the exploits.  Since they're CVEs, though,
> you can just grab the CVE number (it's the last part of the URL for
> the **exploit-db** link) and look up the info on the [Official CVE
> site](http://cve.mitre.org/).

Voila.  You have a list of exploits that telling you what versions are
affected and what the attack vector is.  You also have a list of
affected devices.  If you make use of the [Shodan
API](https://developer.shodan.io/), it's _even easier_.

## Wrap Up

To be fair, you actually have to have a way to perform the exploit.  But
hackers are pretty smart people.  There's probably an exploit out there
already that they just have to fish up.  Or, if the target is important
or large enough, they'll just write their own.

The point of this article has been to point out how extremely easy it is
to perform reconnaissance on a network.  And--chances are--you're
affected.  I spoke with a few engineers last night and was able to find
exposed machines in their networks just by searching by company name.
They had no clue they were as exposed as they are.
