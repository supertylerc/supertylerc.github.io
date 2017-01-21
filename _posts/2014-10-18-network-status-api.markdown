---
layout: post
title: "Network Status API"
date: 2014-10-18 23:17:03 -0700
comments: true
categories:
  - automation
  - api
---

> This post was written after completing a basic POC.  It was written in
> haste and may be missing information or have crazy grammar issues or
> completely incorrect facts.  Reader beware!

Tonight, I started working on a Proof of Concept for something I've
wanted to do for a year: an API you can query to get the network status
for any network device in your organization.  I've finished the PoC, and
I really want to move forward.

## The API

The Proof of Concept has two routes: `/api/v1/interfaces` and
`/api/v1/protocols/ospf`.  These essentially do what you would expect:
pull data from a database and send it to the client formatted as JSON.

### Example

Here's the API in action for the `/api/v1/protocols/ospf` route.

```bash
> curl -s http://localhost:3000/api/v1/protocols/ospf | json_pp
{
   "neighbors" : [
      {
         "ospf_neighbor_state" : "Full",
         "neighbor_priority" : "128",
         "activity_timer" : "38",
         "created_at" : "2014-10-19T05:57:42.333Z",
         "updated_at" : "2014-10-19T05:57:42.333Z",
         "id" : 1,
         "interface_name" : "reth1.0",
         "neighbor_id" : "10.49.63.3",
         "neighbor_address" : "10.49.49.250"
      },
      {
         "interface_name" : "reth1.0",
         "neighbor_id" : "10.49.63.3",
         "neighbor_address" : "10.10.1.30",
         "updated_at" : "2014-10-19T05:57:42.361Z",
         "id" : 2,
         "activity_timer" : "36",
         "created_at" : "2014-10-19T05:57:42.361Z",
         "ospf_neighbor_state" : "Full",
         "neighbor_priority" : "128"
      },
      {
         "id" : 3,
         "updated_at" : "2014-10-19T05:57:42.388Z",
         "neighbor_address" : "169.254.100.3",
         "neighbor_id" : "10.49.95.8",
         "interface_name" : "st0.0",
         "ospf_neighbor_state" : "Full",
         "neighbor_priority" : "128",
         "created_at" : "2014-10-19T05:57:42.388Z",
         "activity_timer" : "37"
      },
      {
         "updated_at" : "2014-10-19T05:57:42.411Z",
         "id" : 4,
         "interface_name" : "st0.6",
         "neighbor_id" : "10.49.31.4",
         "neighbor_address" : "169.254.100.1",
         "neighbor_priority" : "128",
         "ospf_neighbor_state" : "Full",
         "activity_timer" : "30",
         "created_at" : "2014-10-19T05:57:42.411Z"
      }
   ]
}
```

### How It Works

Conceptually, every 5 minutes, a NETCONF session is established to the
router.  It pulls the data it needs, checks to see if a record exists,
and then either creates a new record or updates the existing record.
This action is independent of the API, though to make prototyping
quicker, I elected to manage this portion through a Rake task and
manually run the task.

The API itself, though, simply reads from the database that was
populated by the Rake task.  It renders the database contents as JSON
and returns every record from that particular table.  A collection is
used to send `/api/v1/protocols/ospf` to the ospf method of the
Protocols controller.

### Things Missing

This PoC was performed against a single device from a single vendor and
does not take into account multiple devices or vendors.  This is an
"easy" issue to fix.  The "multi-vendor" aspect is where things get
tricky.  Still, if a script can gather the data and update the database,
then it can easily be done.

In truth, the API is independent of the data collection.  It's almost
a plugin architecture, where anyone can write plugins for a vendor that
updates the database in the expected format.

There is plenty more missing, but this is off the top of my head.

## Collaboration

E-mail me.  I'd like to get this off of the ground.
