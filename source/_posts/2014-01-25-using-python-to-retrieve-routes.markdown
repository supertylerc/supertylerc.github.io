---
layout: post
title: "Using Python to Retrieve Routes"
date: 2014-01-25 22:15:00 -0700
comments: true
categories:
  - python
  - juniper
  - coding
---

> You should read [this post on retrieving port status][1] first.  It
> goes into the details of setting up [py-junos-eznc][2] and getting
> your environment going.

We've previously tackled retrieving port status for multiple devices
using Python; now, let's take a look at ensuring we have similar route
entries on multiple devices.

## The Script

We're going to use the same design principles as we did in the previous
script, so I won't bother going through the explanation here.

<!-- more -->

Here's the script:

``` python
#!/usr/bin/python
# Author  : Tyler Christiansen
# Purpose : Get route info for multiple routers
# Date    : 2013-01-21

from jnpr.junos import Device
from jnpr.junos.op.routes import *
import json

config_file = open('config.json')
config = json.load(config_file)
config_file.close();

for host in config['hosts']:
  rtr = Device(
                user=config['user'],
                password=config['pass'],
                host=host
        )

  rtr.open()
  print("Host: %s" % host)
  print("%s%s%s" % (
                     "Route".ljust(20),
                     "Protocol".ljust(10),
                     "Next Hop"
                   )
       )
  print '-' * 65
  print
  for route in config['routes']:
    results = RouteTable(rtr).get(route)
    for route_data in results:
      print("%s%s" % (
                       route.ljust(20),
                       route_data.protocol.ljust(10)
                     )
           ),
      if isinstance(route_data.nexthop, list):
        for index, value in enumerate(route_data.nexthop):
          if index == 0:
            print("%s, via %s" % (
                                   route_data.nexthop[index],
                                   route_data.via[index]
                                 )
                 )
          else:
            print(" " * 31 + "%s, via %s" % (
                                              route_data.nexthop[index],
                                              route_data.via[index]
                                           )
                 )
      else:
        print( "%s, via %s" % (route_data.nexthop, route_data.via) )
    print("\n\n")
```

## Config <small>The JSON config</small>

We're going to use JSON again as in the last post, but we're adding a
new property: `routes`.  This is an array (also known as a `list` in
`python`) of routes.  Be careful how you assign the routes--you could
get more than you bargained for!

``` javascript
{
  "user": "rancid",
  "pass": "rancid",
  "hosts": [
             "192.168.1.1",
             "192.168.1.2"
           ],
  "routes": [
              "8.8.8.0/24",
              "8.8.4.0/24"
            ]
}
```

## Example <small>The /24s to Google's public DNS</small>

``` bash
tylerc@mon [08:58:11] [~/scripts/junos]
-> % ./route_stats
Host: 192.168.1.1
Route               Protocol  Next Hop
-----------------------------------------------------------------

8.8.8.0/24          BGP        38.105.223.211, via ae0.0
                               198.17.53.3, via ae0.0
                               207.86.160.130, via ae0.0
                               207.86.166.98, via ae0.0



8.8.4.0/24          BGP        38.105.223.211, via ae0.0
                               198.17.53.3, via ae0.0
                               207.86.160.130, via ae0.0
                               207.86.166.98, via ae0.0



Host: 192.168.1.2
Route               Protocol  Next Hop
-----------------------------------------------------------------

8.8.8.0/24          BGP        209.220.18.17, via ae0.0



8.8.4.0/24          BGP        209.220.18.17, via ae0.0



tylerc@mon [08:58:29] [~/scripts/junos]
-> %
```

## The End

As usual, this can be heavily improved.  There are a lot of "hacky"
things, and since I just learned the majority of the Python I know
today, I'm not really up on its stylings.  It seems like a language
that's impossible to style well, but that's for another time.

This is released under the [BSD 2-Clause License][3], so feel free to
modify within the guidlines of the license.

[1]: /blog/using-python-to-retrieve-port-status/ "Using Python to Retrieve Port Status"
[2]: https://github.com/Juniper/py-junos-eznc "py-junos-eznc"
[3]: http://opensource.org/licenses/BSD-2-Clause "BSD 2-Clause License"
