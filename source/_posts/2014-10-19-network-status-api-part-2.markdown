---
layout: post
title: "Network Status API - Part 2"
date: 2014-10-19 15:08:10 -0700
comments: true
categories:
  - automation
  - api
---

Last night, I started working on a Proof of Concept (POC) for a backend
Network Status API.  While similar in some ways to current open source
projects designed to provide information about the network, the idea
I have for this system is quite different.  In fact, the only real
similarity is that it collects data.

## The Goal

There are dozens of tools that gather data via SNMP.  They even store
the data for historical purposes.  They normalize the data, which is
a bit unfortuante, but I'm not interested in tackling that problem.

What I'm interested in is knowing what the network looks like _now_.
I don't want to maintain historical data.  As a result, there are other
features that I don't want: trending, historical analysis, graphs, etc.

My goal, then, is to create an API which, at its core, simply queries
a database and returns the results of the query in JSON, XML, or the
next format of the future.  There must also be a component to populate
that database, but that component should operate independently of the
API.  It should be event-driven or have strong multi-threading.  Perhaps
node.js calling external scripts and gathering the return data?  Who
knows.  Whatever it is, though, it _must_ be a framework that is
agnostic to any language.  Think of the way Ansible does this.  It
allows you to extend it with any language, as long as that language
returns a result in a manner that Ansible expects.

The data is relational by nature.  Individual portions of a network
element relate to other portions on the same device, but they also
relate to one or more other network elements, with a very complex
relationship.  An OSPF relationship, for example, relates to each of the
devices on the segment, as well as the links between the devices.  It
might even relate to the protocols that rely on it--LDP, for example.
This portion of the project is _highly_ ambitious and will take dozens
of iterations to get right.  Initially, the only relationships will be
within a single device.

## Problems

This also means there are problems I need to solve that I'm introducing
as a result of the data I want.  I need a sophisticated filtering
mechanism--ElasticSearch, maybe--that allows me to specify complex
criteria.  I need a way to create relationships between devices and
protocols.  Imagine collecting all router IDs for OSPF neighbors and
building a topological database based on this information, generating
a map of the OSPF network, and showing links that are down on this map.
I need a way to reliably connect to devices and obtain the data that
I want in a somewhat transactional way.  I don't want to use SNMP--it
doesn't scale well, requires a great deal of extremely irritating
parsing, and is not always supported in the same way across vendors (or
even across platforms from the same vendor!).  For this reason, I've
elected to use NETCONF for Juniper Networks devices, which is where my
POCing is being done.  Other vendors are beginning to offer similar APIs
(or have been for a long time).  F5 has iControl, Arista has eAPI, and
even Cisco has onePK (which is little better than screen-scraping, but
it's still a thing).

Another question that's weighing me down is this: how much adoption can
I get when I'm focused on an API, something that most network
professional won't care about until there's a front end to that API?
Even if the API of my dreams bridges the vendor gap to present a common
language for obtaining status information, most network professionals
won't adopt or contribute, and even fewer will care at all unless
there's a front end to it.  I'm faced with a decision to develop a front
end in parallel or skip it and hope that I can some modicum of
assistance and/or interest.

What technology stack do I use?  Do I use a traditional relational
database (MySQL, PostgreSQL) and enforce relationships at a database
level, requiring DB migrations and reducing flexibility for
contributors?  Or do I use a "NoSQL" database (MongoDB) to increase
flexibility for contributors (although perhaps increase the barrier of
entry!) and bring the relationship logic into the code?

## The Proof

I've already done the POC using Ruby/Rails/SQLite3.  I'm using a variety
of gems that aren't particularly important, but here's the basics of
what happens:

- You can update `world` or individual components (`interfaces`, `ospf`,
  etc) using a `rake` task.
  - Although I'm using a `rake` task here (which is tied strongly to the
    Rails application in this example), the conceptual idea is to force
this task to take place outside of the API.
- You query a versioned API.  The base route is `/api/v1`, and in the
  future may become `/api/v2` and so on as the API is updated.
- You query a route that is consistent regardless of the vendor,
  bringing a unified communication language to a multi-vendor network.
  - `/api/v1/interfaces`, for example, would return the same type of
    data for any vendor.
- Once you have the data, you can do anything you want with it.  There
  is a front-end example below.

### Examples

Here's the API in action, which currently returns the data in JSON
format only.  I plan on adding the ability to request XML as a response,
but that's off a ways.

```bash
> curl -s localhost:3000/api/v1/protocols/ospf | json_pp
{
   "neighbors" : [
      {
         "device" : "10.10.1.1",
         "created_at" : "2014-10-19T20:26:08.478Z",
         "neighbor_id" : "10.49.63.3",
         "neighbor_address" : "10.49.49.250",
         "interface_name" : "reth1.0",
         "id" : 1,
         "activity_timer" : "31",
         "neighbor_priority" : "128",
         "updated_at" : "2014-10-19T22:01:44.190Z",
         "ospf_neighbor_state" : "Full"
      },
      {
         "neighbor_priority" : "128",
         "updated_at" : "2014-10-19T22:01:44.202Z",
         "ospf_neighbor_state" : "Full",
         "device" : "10.10.1.1",
         "created_at" : "2014-10-19T20:26:08.487Z",
         "neighbor_address" : "10.10.1.30",
         "neighbor_id" : "10.49.63.3",
         "interface_name" : "reth1.0",
         "id" : 2,
         "activity_timer" : "38"
      },
      {
         "id" : 3,
         "interface_name" : "st0.0",
         "activity_timer" : "39",
         "neighbor_address" : "169.254.100.3",
         "neighbor_id" : "10.49.95.8",
         "created_at" : "2014-10-19T20:26:08.495Z",
         "device" : "10.10.1.1",
         "ospf_neighbor_state" : "Full",
         "updated_at" : "2014-10-19T22:01:44.218Z",
         "neighbor_priority" : "128"
      },
      {
         "ospf_neighbor_state" : "Full",
         "updated_at" : "2014-10-19T22:01:44.230Z",
         "neighbor_priority" : "128",
         "interface_name" : "st0.6",
         "id" : 4,
         "activity_timer" : "33",
         "neighbor_address" : "169.254.100.1",
         "neighbor_id" : "10.49.31.4",
         "created_at" : "2014-10-19T20:26:08.505Z",
         "device" : "10.10.1.1"
      },
      {
         "activity_timer" : "34",
         "id" : 5,
         "interface_name" : "reth1.0",
         "neighbor_address" : "10.1.1.29",
         "neighbor_id" : "10.49.95.2",
         "created_at" : "2014-10-19T20:31:01.828Z",
         "device" : "10.1.1.1",
         "ospf_neighbor_state" : "Full",
         "updated_at" : "2014-10-19T22:01:55.225Z",
         "neighbor_priority" : "128"
      },
      {
         "neighbor_priority" : "128",
         "updated_at" : "2014-10-19T22:01:55.237Z",
         "ospf_neighbor_state" : "Full",
         "device" : "10.1.1.1",
         "created_at" : "2014-10-19T20:31:01.862Z",
         "neighbor_address" : "169.254.100.2",
         "neighbor_id" : "10.49.63.11",
         "interface_name" : "st0.0",
         "id" : 6,
         "activity_timer" : "38"
      },
      {
         "created_at" : "2014-10-19T20:31:01.883Z",
         "device" : "10.1.1.1",
         "id" : 7,
         "interface_name" : "st0.7",
         "activity_timer" : "37",
         "neighbor_id" : "10.49.31.4",
         "neighbor_address" : "169.254.100.5",
         "neighbor_priority" : "128",
         "ospf_neighbor_state" : "Full",
         "updated_at" : "2014-10-19T22:01:55.251Z"
      }
   ]
}
```

And here's a screen capture of an AngularJS application that uses this
API.

{% img /images/front_end_poc.png Network Status Front End POC %}
