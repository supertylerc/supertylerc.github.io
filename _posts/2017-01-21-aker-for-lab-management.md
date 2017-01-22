---
layout: post
title: "Aker Gateway for Lab Management"
description: |
  - "Using Aker SSH Gateway to connect to/from network devices in "
  - "the lab."
category: labs
tags: [labs, docker, python, docker-compose]
---

As I started preparing to lab for the CCNP SP, I found myself wanting a
decent way to manage access to my lab devices without remembering IP
addresses or port numbers or spending money on a terminal server or any
of that.  I don't use the graphical frontend to GNS3, so that wasn't an
option either.

# Aker for Lab Management

One day, my boss ([@joshobrien77][1]) dropped a link into chat for
something called [Aker][2].  Intrigued about its purposes for network
device management, I began hacking on it to extend it from just
supporting servers and SSH keys to supporting username and password
combinations.  The maintainer, thankfully, was amenable to the changes
and graciously merged them.

The next step for me was simple packing and deployment for my lab in
case the topology ever changes.  While I did this and submitted the
changes, the pull request was ultimately closed.  That's probably my
fault, because I fell out of the loop with the maintainer due to the
holidays and crazy work stuff.


My changes live, though, and they're packaged up as a Docker container
to boot.  Add in `docker-compose` and we're off to the races!  The work
is in a repository [here][3].  Below you can see examples of how I use
this in a lab environment, followed by instructions on how to get
started.

# Examples

## Connecting to Aker

Here's what it looks like when I want to log into a lab device:

![Aker Gateway](/assets/img/aker-gateway.png)

## Searching for Devices

Another cool feature if you just have a ridiculous amount of lab
devices is that you can type to search.  Check it out!

![Aker Search](/assets/img/aker-search.png)

## Reliving the Session

I can then attach to the container and get all of the details of that
session!  Here's how:

* First, from the computer running the Aker container, I type
  `docker exec -i -t dockeraker_aker_1 d/bin/bash`
* Next, I figure out which file has the data I'm interested in based
  on the filename, which embeds a timestamp, with `ls /home/supertylerc`
* Finally, I use `cat` to display the contents of the file

Here's an example of a session:

```
root@df773f8ac027:/opt# cat /home/supertylerc/supertylerc-cust-b-ce1-20170121-212542_63577_a4136159-3886-4b34-8b7e-4ff944de9f8f.log

**************************************************************************
* IOSv is strictly limited to use for evaluation, demonstration and IOS  *
* education. IOSv is provided as-is and is not supported by Cisco's      *
* Technical Advisory Center. Any use or disclosure, in whole or in part, *
* of the IOSv Software or Documentation to any third party for any       *
* purposes is expressly prohibited except as otherwise authorized by     *
* Cisco in writing.                                                      *
**************************************************************************
cust-b-ce1>en
cust-b-ce1#sh ip int br
Interface                  IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0         192.168.0.220   YES NVRAM  up                    up
GigabitEthernet0/1         unassigned      YES NVRAM  administratively down down
GigabitEthernet0/2         unassigned      YES NVRAM  administratively down down
cust-b-ce1#exit
root@df773f8ac027:/opt#
```

> Note that, unless you built a container based on the original, the
> username should be `lab` instead of `supertylerc`.

# Running Aker for Lab Management

First thing's first: you need Docker.  My instructions would suck
compared to the official documentation, so go read it [here][4] if you
don't have Docker running already.

Next, you'll want to clone [this repository][3]:

```
git clone https://github.com/supertylerc/docker-aker
```

Now, change to that directory:

```
cd docker-aker
```

Copy the sample config file:

```
cp aker.ini.sample aker.ini
```

And then edit it:

```
vim aker.ini
```

> If you aren't comfortable with `vim`, just use your text editor of
> choice.  I won't judge you.

This is another one of those places where the official documentation is
better than anything I could write, so check it out [here][2].

Once you've edited your `aker.ini`, it's time to edit the
`docker-compose.yml`:

```
vim docker-compose.yml
```

> Again, just use your text editor of choice here.

You'll want to edit the `extra_hosts` part to properly map your
hostnames to IP addresses.  It's pretty inconvenient to have to manually
edit the `docker-compose.yml` file.  I'm all ears on how to make that
not required anymore.

Finally, start the container using `docker-compose`:

```
docker-compose up -d
```

Now, you should be able to log into Aker with the username `lab` and
password `demo`:

```
ssh lab@$CONTAINER_HOST_IP -p2222
```

> Replace `$CONTAINER_HOST_IP` with the IP address or hostname of the
> server running the Aker container.

All done!  Enjoy!

[1]: https://twitter.com/@joshobrien77 "Josh O'Brien"
[2]: https://github.com/Aker-Gateway/Aker "Aker SSH Gateway"
[3]: https://github.com/supertylerc/docker-aker "Docker Compose and Aker"
[4]: https://docs.docker.com/engine/installation/ "Docker installation"
