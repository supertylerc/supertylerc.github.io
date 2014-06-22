---
layout: post
title: "Using Python to Retrieve Port Status"
date: 2014-01-24 22:11:31 -0700
comments: true
categories:
  - python
  - coding
  - juniper
---

> This post is based loosely on [@networkjanitor's][1] (Kurt Bales) post
> on his experience [starting with Python programming][2].  You should
> read his experience as well.

In this post, we're going to explore how we can construct a list of
routers and get the port status and the last state change of those ports
very quickly and easily.  This post won't go into the details of how to
use the [py-junos-eznc][4] API, nor will it discuss [Python][5]
fundamentals.  Those are both left as an exercise to the reader (and
perhaps to future posts).

## Router Setup

Before we start, you need to ensure your router has `netconf` enabled.
This is easy.  Just log into your router and type:

``` bash
configure private
set system services netconf ssh
show | compare
commit check
commit comment "Enable NetConf"
```

## py-junos-eznc <small>A Python library to automate Junos</small>

[@nwkautomaniac][3] (Jeremy Schulman) has done some excellent work
automating Junos.  The currently-supported automation library is
[py-junos-eznc][4], which we're going to install and use.

<!-- more -->

Open up your terminal and let's get started!

``` bash
sudo apt-get install -y python python-pip python-yaml
cd ~
mkdir -p scripts/junos && cd $_
pip install git+https://github.com/Juniper/ncclient.git
pip install junos-eznc
touch port_stats
touch config.json
chmod 700 port_stats
chmod 600 config.json
```

> This assumes the use of Ubuntu 12.04.  Adjust the package installation
> line (the first line above) if you're using a different distribution.

That was easy.  Let's (very quickly) test our success by opening the
Python shell and importing a module:

``` python
Python 2.7.3 (default, Sep 26 2013, 20:03:06)
[GCC 4.6.3] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> from jnpr.junos.op.phyport import *
>>> from jnpr.junos import Device
>>> import json
>>>
```

> To open a Python shell, just type `python` at the command line.

> If you get an error about "**python2.yml**" not existing, see issues
> [#86][6] and [#62][7] on the project's GitHub page for a temporary
> fix.

You shouldn't get any errors, and we should be off to the races.

## Configuration <small>config.json</small>

Before we get started, we should define a configuration hierarchy and
methodology.  For this example, we're going to use [JSON, the JavaScript
Object Notation][8].  It is easy to read and write, and it's implemented
across a large variety of languages.

There are, essentially, three configuration-level variables we want: a
**user name**, a **password**, and a **hostname** or **IP address**.  We
should think about the future, though, and take this one step further:
what if we want to get port status for _multiple_ devices at the same
time?  So we need to change our **hostname** requirement to a **list of
hostnames**.

Now that we know what we need, let's set up our configuration file with
`vim`:

> If you're not familiar with `vim` or Linux in general, you may find my
> [crash course on Linux][9] helpful.

``` javascript
{
  "user": "rancid",
  "pass": "rancid",
  "hosts": [
             "192.168.1.1",
             "192.168.1.2"
           ]
}
```

Pretty simple, right?  Two routers, a user name, and a password.

> I'm using the "**rancid**" user here just for simplicity.  I recommend
> you use the lowest-privileged user that is capable of the task
> required in the event of a system compromise.

## The Meat of It <small>The `python` script</small>

Now open up `port_stats` and enter the following:

``` python
#!/usr/bin/python
# Author  : Tyler Christiansen
# Purpose : Get port state for multiple routers
# Date    : 2013-01-21

from jnpr.junos import Device
from jnpr.junos.op.phyport import *
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
  ports = PhyPortTable(rtr).get()
  print("Host: %s" % host)
  print("%s%s%s" % (
                     "Interface".ljust(12),
                     "Status".ljust(8),
                     "Time Since Last Flap".ljust(45)
                   )
       )
  print '-' * 65
  print
  for port in ports:
    print("%s%s%s" % (
                         port.name.ljust(12),
                         port.oper.ljust(8),
                         port.flapped.ljust(45)
                     )
         )
```

There's quite a bit going on in there.

> The verbosity of `python` is one the reasons I'm not particularly
> enchanted with it--but this is the best tool for the job right now.

### Making Sense <small>An explanation of the script</small>

Assume that **line 1** actually starts with the first line of code,
which is `from jnpr.junos.op.phyport import *`.

Lines `1-3` import modules and code that are necessary for our
script--modules to interact with the device, the port, and with our JSON
configuration file.

Lines `5-7` open the configuration file, read its contents, and parse
the JSON, making a Python object that we can work with later.

Line `9` starts the real magic: it begins to loop through all of the
hosts we defined under the `hosts` key in `config.json`, selecting one
value of the list at a time and giving it a temporary name of `host`,
and continues until the end of the loop is reached.

Lines `10-14` use data from our `config.json` file and the `host` loop
variable to build a `Device` object.  This is what will actually build
the connection, and it is necessary to gather information.

Line `16` opens the `netconf` session with the router, and line `17`
gets all of the port information.

Lines `18-26` build a header, making use of the `ljust()` function to
build padding to the right and left-justify the text.

The rest of the lines loop through all of the ports on the device,
giving a temporary variable name of `port` to each, and then printing
out the port name, operation status, and last state change time
following the same length constraints as the header.

## The Result <small>An example in action</small>

``` bash
tylerc@mon [07:54:42] [~/scripts/junos]
-> % ./port_stats
Host: 192.168.1.1
Interface   Status  Time Since Last Flap
-----------------------------------------------------------------

xe-0/0/0    up      2013-11-01 06:37:15 UTC (11w5d 01:17 ago)
xe-0/0/1    up      2013-12-13 06:00:45 UTC (5w5d 01:54 ago)
xe-0/0/2    up      2013-11-01 06:42:30 UTC (11w5d 01:12 ago)
xe-0/0/3    down    2013-11-01 06:14:30 UTC (11w5d 01:40 ago)



Host: 192.168.1.2
Interface   Status  Time Since Last Flap
-----------------------------------------------------------------

xe-0/0/0    up      2013-12-13 06:14:28 UTC (5w5d 01:40 ago)
xe-0/0/1    up      2013-12-13 06:14:20 UTC (5w5d 01:40 ago)
xe-0/0/2    up      2013-12-18 22:39:35 UTC (4w6d 09:15 ago)
xe-0/0/3    down    2013-12-13 06:14:20 UTC (5w5d 01:40 ago)



tylerc@mon [07:55:02] [~/scripts/junos]
-> %
```

## The End

This script is licensed under the [BSD 2-Clause license][10], while the
[py-junos-eznc][4] library is licensed under the [Apache license][11].
You're free to modify my code as long as you retain notice that the
original codebase was mine.

Otherwise, enjoy it!

> Note: I'm not a programmer.  I'm really not even a systems guy.  To be
> honest, I'm not sure what you would call me, but this script could be
> a whole lot better.  More features, command-line arguments, and so on.
> **PLEASE** make every effort to improve it--and then share your
> contributions with the rest of the community.


[1]: https://twitter.com/networkjanitor "Kurt Bales @networkjanitor"
[2]: http://www.network-janitor.net/2013/11/on-python-networks-and-the-py-junos-eznc-library/ "ON PYTHON, NETWORKS AND THE PY-JUNOS-EZNC LIBRARY"
[3]: https://twitter.com/nwkautomaniac "Jeremy Schulman @nwkautomaniac"
[4]: https://github.com/Juniper/py-junos-eznc "py-junos-eznc"
[5]: http://www.python.org/ "Python"
[6]: https://github.com/Juniper/py-junos-eznc/issues/86 "Issue #86"
[7]: https://github.com/Juniper/py-junos-eznc/issues/62 "Issue #62"
[8]: http://www.json.org/ "JSON"
[9]: /blog/linux-crash-course/ "Crash Course - Linux"
[10]: http://opensource.org/licenses/BSD-2-Clause "BSD 2-Clause License"
[11]: http://opensource.org/licenses/Apache-2.0 "Apache v2.0 License"
