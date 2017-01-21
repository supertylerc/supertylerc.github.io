---
layout: post
title: "Beyond junos-eznc (PyEZ)"
description: "Examining junos-eznc and how to make it better."
category: code
tags: [automation, networking, python, code]
---

### Disclaimer

I hesitated on this post for a long time.  I have a great deal of respect for
its primary author ([Jeremy Schulman](https://twitter.com/nwkautomaniac)), and
I have a place in Juniper's Ambassador program.  I have been a little concerned
that what I have to say here could be taken as a slight against either Jeremy
or Juniper (it isn't--not at all!), but ultimately I've decided that this post
will move people forward more than it will hurt anyone's feelings.

# Good, but not Great

Juniper's [junos-eznc](https://github.com/Juniper/py-junos-eznc) (which is
somehow known as PyEZ, which doesn't make it very obvious what it is or does)
is a Python library designed to help you write code to interact with their
devices.  This is good!  Lowering the barrier to entry is an awesome thing.
But it isn't _great_.  Let's take a look at a few things I don't like about
junos-eznc.

## Tables and Views

Okay, this is actually a pretty awesome concept and makes it easier for a
beginner to get his or her feet wet.  The problem is that once you progress
from beginner, you have to throw it all away.  Why?

Tables and Views don't provide all of the data you need*.  They don't allow you
to retrieve specific data*.  They don't easily allow for Pythonic code.

> * This can be overcome by writing your own tables and views.

### Getting Data

Tables and Views are basic.  Very basic.  And good luck getting any
contributions merged into the upstream (more on this later).  Take the ARP
table and view as an example:

{% highlight yaml %}
---
ArpTable:
  rpc: get-arp-table-information
  item: arp-table-entry
  key: mac-address
  view: ArpView

ArpView:
  fields:
    mac_address: mac-address
    ip_address: ip-address
    interface_name: interface-name
{% endhighlight %}

Do you notice anything missing there?  It may not be immediately noticeable,
and it probably doesn't matter unless you're a service provider.  Give up?

There's no way to specify a VPN.  This is pretty important in service provider
environments, and maybe even in some enterprise environments that utilize
virtual routers.  This isn't 100% `junos-eznc`'s fault, though.  The XML
response from Junos doesn't actually include this data.  See below:

{% highlight xml %}
<arp-table-information xmlns="http://xml.juniper.net/junos/12.1X47/junos-arp" junos:style="no-resolve">
    <arp-table-entry>
        <mac-address>52:54:00:12:35:02</mac-address>
        <ip-address>10.0.2.2</ip-address>
        <interface-name>ge-0/0/0.0</interface-name>
        <arp-table-entry-flags>
            <none/>
        </arp-table-entry-flags>
    </arp-table-entry>
    <arp-table-entry>
        <mac-address>52:54:00:12:35:03</mac-address>
        <ip-address>10.0.2.3</ip-address>
        <interface-name>ge-0/0/0.0</interface-name>
        <arp-table-entry-flags>
            <none/>
        </arp-table-entry-flags>
    </arp-table-entry>
    <arp-table-entry>
        <mac-address>60:03:08:a6:d5:e2</mac-address>
        <ip-address>10.66.172.16</ip-address>
        <interface-name>ge-0/0/2.0</interface-name>
        <arp-table-entry-flags>
            <none/>
        </arp-table-entry-flags>
    </arp-table-entry>
    <arp-table-entry>
        <mac-address>00:ff:85:7f:78:03</mac-address>
        <ip-address>10.66.172.169</ip-address>
        <interface-name>ge-0/0/1.0</interface-name>
        <arp-table-entry-flags>
            <permanent/>
            <published/>
        </arp-table-entry-flags>
    </arp-table-entry>
    <arp-entry-count>4</arp-entry-count>
</arp-table-information>
{% endhighlight %}

Junos does, however, let you do something that the ARP table doesn't let you
do.  It let's you specify a VPN instance (or a logical system, if that's your
thing):

{% highlight xml %}
<rpc>
    <get-arp-table-information>
            <no-resolve/>
            <vpn>vpn1</vpn>
    </get-arp-table-information>
</rpc>
{% endhighlight %}

And the response:

{% highlight xml %}
<arp-table-information xmlns="http://xml.juniper.net/junos/12.1X47/junos-arp" junos:style="no-resolve">
    <arp-table-entry>
        <mac-address>00:ff:85:7f:78:03</mac-address>
        <ip-address>10.66.172.169</ip-address>
        <interface-name>ge-0/0/1.0</interface-name>
        <arp-table-entry-flags>
            <permanent/>
            <published/>
        </arp-table-entry-flags>
    </arp-table-entry>
</arp-table-information>
{% endhighlight %}

Can you modify the ARP table to add this functionality?  YES!  See below:

{% highlight yaml %}
---
 ArpTable:
   rpc: get-arp-table-information
   args:
     vpn: default
   item: arp-table-entry
   key: mac-address
   view: ArpView
{% endhighlight %}

All we did was add the `args` key with `vpn` set to `default`.  Easy enough,
right?  But now you have to override something that `junos-eznc` provides by
default.  And then you still have to interact with it in a non-Pythonic way!
How can you remedy this?  We'll look into specifics later, but the short
version: just use RPC calls directly.  `junos-eznc` makes this easy for you,
and you don't have to worry about Tables and Views anymore (nor will you have
to worry about potentially breaking changes in the upstream tables and views).

### Non-Pythonic

Chances are that if you're writing code that has `.get()` in it, you're doing
it wrong.  Not wrong in the sense that it won't work, but wrong in the sense
that it isn't really Pythonic.  Unfortunately, the guides and recommendations
for using `junos-eznc` universally use `.get()` methods as ways to retrieve
data from a device.  Let's see an example of that.

{% highlight python %}
>>> from jnpr.junos import Device
>>> from jnpr.junos.op.ethport import EthPortTable
>>> dev = Device('10.66.172.151', user='tyler', passwd='harbl1234')
>>> dev.open()
Device(10.66.172.151)
>>> eths = EthPortTable(dev)
>>> for eth in eths:
...     print eth
...
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/usr/local/Cellar/python/2.7.9/Frameworks/Python.framework/Versions/2.7/lib/python2.7/site-packages/jnpr/junos/factory/table.py", line 259, in __iter__
    self._assert_data()
  File "/usr/local/Cellar/python/2.7.9/Frameworks/Python.framework/Versions/2.7/lib/python2.7/site-packages/jnpr/junos/factory/table.py", line 87, in _assert_data
    raise RuntimeError("Table is empty, use get()")
RuntimeError: Table is empty, use get()
>>> eths.get()
EthPortTable:10.66.172.151: 4 items
>>> for eth in eths:
...     print eth
...
EthPortView:ge-0/0/0
EthPortView:ge-0/0/1
EthPortView:ge-0/0/2
EthPortView:ge-0/0/3
>>>
{% endhighlight %}

That's actually painful for me.  The ports on a network element really should
be a property of that element.  We'll get into this later when we look at how
to make `junos-eznc` great.

## Dormancy and Response

Juniper has been pretty focused on core bug fixes and better error handling.
This should be commended!

However, they don't typically merge pull requests.  Most of the pull requests
that have been merged have been opened by Juniper employees.  There are 13
open pull requests at the time of this writing, all from community members.
The newest one is from 25 days ago.  The oldest?  July 10, 2014.  Their
response in this area isn't particularly great.  They neither merge nor close
these pull requests, so they just sit there in limbo.  Most of these are
updates to tables and views--the things that will help 90% of network engineers
get engaged in this coding craze.  Unfortunately, there isn't much
communication on why these are never merged.  I have a few ideas on why this is
happening, but they're just conjecture so I'll refrain.  If anyone from Juniper
reads this, please just update the repository with a message that says "We're
not interested in community contributions for tables and/or views."  If you
want to take it a step further, create a "community repository", the purpose of
which would be to house tables and views from the community that could easily
be pulled in and used.

I consider `junos-eznc` to be a good, functional library that is essentially in
"maintenance mode" now.

# A Better Way

First, let's resolve the first of my complaints: the information (or lack
thereof) in tables and views.  As I mentioned, the easiest way to deal with
this is to manually make the RPC calls however you need to make them.

## RPC Calls

This one is pretty easy.  Let's make the RPC call for the `ArpTable` example
above.

{% highlight python %}
>>> from jnpr.junos import Device
>>> from lxml import etree
>>> dev = Device('10.169.81.190', user='tyler', passwd='harbl1234')
>>> dev.open()
Device(10.169.81.190)
>>> arp_table = dev.rpc.get_arp_table_information()
>>> etree.dump(arp_table)
<arp-table-information style="normal">
    <arp-table-entry>
        <mac-address>
            52:54:00:12:35:02
        </mac-address>
        <ip-address>
            10.0.2.2
        </ip-address>
        <hostname>
            10.0.2.2
        </hostname>
        <interface-name>
            ge-0/0/0.0
        </interface-name>
        <arp-table-entry-flags>
            <none/>
        </arp-table-entry-flags>
    </arp-table-entry>
    <arp-table-entry>
        <mac-address>
            52:54:00:12:35:03
        </mac-address>
        <ip-address>
            10.0.2.3
        </ip-address>
        <hostname>
            10.0.2.3
        </hostname>
        <interface-name>
            ge-0/0/0.0
        </interface-name>
        <arp-table-entry-flags>
            <none/>
        </arp-table-entry-flags>
    </arp-table-entry>
    <arp-table-entry>
        <mac-address>
            00:ff:85:7f:78:03
        </mac-address>
        <ip-address>
            10.66.172.169
        </ip-address>
        <hostname>
            10.66.172.169
        </hostname>
        <interface-name>
            ge-0/0/1.0
        </interface-name>
        <arp-table-entry-flags>
            <permanent/>
            <published/>
        </arp-table-entry-flags>
    </arp-table-entry>
    <arp-table-entry>
        <mac-address>
            40:6c:8f:3a:ec:21
        </mac-address>
        <ip-address>
            10.169.81.143
        </ip-address>
        <hostname>
            10.169.81.143
        </hostname>
        <interface-name>
            ge-0/0/2.0
        </interface-name>
        <arp-table-entry-flags>
            <none/>
        </arp-table-entry-flags>
    </arp-table-entry>
    <arp-entry-count>
        4
    </arp-entry-count>
</arp-table-information>

>>>
{% endhighlight %}

Great, but how do we solve the problem we had before?  What if we want to get
only the ARP entries that correspond to a VPN called `vpn1`?  It's easy!

{% highlight python %}
>>> vpn_arp_table = dev.rpc.get_arp_table_information(vpn='vpn1')
>>> etree.dump(vpn_arp_table)
<arp-table-information style="normal">
    <arp-table-entry>
        <mac-address>
            00:ff:85:7f:78:03
        </mac-address>
        <ip-address>
            10.66.172.169
        </ip-address>
        <hostname>
            10.66.172.169
        </hostname>
        <interface-name>
            ge-0/0/1.0
        </interface-name>
        <arp-table-entry-flags>
            <permanent/>
            <published/>
        </arp-table-entry-flags>
    </arp-table-entry>
</arp-table-information>

>>>
{% endhighlight %}

## Pythonic Code

None of this addresses the issue of writing Pythonic code, though.  I don't
consider this to be Pythonic.  Let's take our ARP table example and see how
we can transform it into something beautiful.

We're going to accomplish this by wrapping up some of these details into a neat
little class that we can then use to instantiate objects.  Create a file called
`router.py`.  If you're using a terminal in a Unix-like OS (Mac OS X, Linux,
BSD, etc.), then you can just type `touch router.py`.

{% highlight python %}
from jnpr.junos import Device
from os import getenv


class Junos(object):
    """Base class for Junos devices.

    :attr:`hostname`: router hostname
    :attr:`user`: user for logging into `hostname`
    :attr:`password`: password for logging into `hostname`
    :attr:`timeout`: time to wait for a response
    :attr:`connection`: connection to `hostname`
    """
    @property
    def arp_table(self):
        """A list of ARP entries.

        :returns: ARP entries
        :rtype: list
        """
        table = []
        old_table = self.connection.rpc.get_arp_table_information(vpn=self.vpn)
        for old_entry in old_table:
            if old_entry.tag != 'arp-table-entry':
                continue
            entry = dict(ip_address=old_entry.findtext('ip-address').strip(),
                         interface=old_entry.findtext('interface-name').strip(),
                         hostname=self.hostname.strip(),
                         vpn=self.vpn,
                         mac_address=old_entry.findtext('mac-address').strip())
            table.append(entry)
        return table

    def __init__(self, *args, **kwargs):
        self.hostname = args[0] if len(args) else kwargs.get('host')
        self.user = kwargs.get('user', getenv('USER'))
        self.password = kwargs.get('password')
        self.timeout = kwargs.get('timeout')
        self.vpn = kwargs.get('vpn', 'default')

    def _connect(self):
        """Connect to a device.

        :returns: a connection to a Juniper Networks device.
        :rtype: ``Device``
        """
        dev = Device(self.hostname, user=self.user, password=self.password)
        dev.open()
        dev.timeout = self.timeout
        return dev
{% endhighlight %}

Now, let's see how we can use this to create a new instance of our object and
make the ARP table interactions a little more Pythonic!

{% highlight python %}
>>> from pprint import pprint
>>> import router
>>> rtr = router.Junos('192.168.0.151', password='harbl1234')
>>> rtr._connect()
Device(192.168.0.151)
>>> default_arp_table = rtr.arp_table
>>> rtr.vpn = 'vpn1'
>>> vpn1_arp_table = rtr.arp_table
>>> pprint(default_arp_table)
[{'hostname': '192.168.0.151',
  'interface': 'ge-0/0/0.0',
  'ip_address': '10.0.2.2',
  'mac_address': '52:54:00:12:35:02',
  'vpn': 'default'},
 {'hostname': '192.168.0.151',
  'interface': 'ge-0/0/2.0',
  'ip_address': '192.168.0.1',
  'mac_address': '88:f7:c7:95:91:96',
  'vpn': 'default'},
 {'hostname': '192.168.0.151',
  'interface': 'ge-0/0/2.0',
  'ip_address': '192.168.0.7',
  'mac_address': '60:03:08:a6:d5:e2',
  'vpn': 'default'}]
>>> pprint(vpn1_arp_table)
[{'hostname': '192.168.0.151',
  'interface': 'ge-0/0/1.0',
  'ip_address': '192.168.0.169',
  'mac_address': '00:ff:85:7f:78:03',
  'vpn': 'vpn1'}]
>>>
{% endhighlight %}

The `arp_table` property uses the objects `vpn` property, so whenever you want
or need the ARP table for a specific VPN, just set the objects `vpn` property!

> Sometimes, when working on a script or tool, I use base64 to encode the
> password.  There's an example below.  It provides only the most basic
> protection from prying eyes.  `base64` does not hash the value; it merely
> encodes it.  It can just as easily be decoded!

{% highlight python %}
>>> import base64
>>> import router
>>> rtr = router.Junos('192.168.0.151', password='harbl1234')
>>> rtr._connect()
Device(192.168.0.151)
>>> rtr.password
'aGFyYmwxMjM0'
>>> base64.b64decode(rtr.password)
'harbl1234'
>>>
{% endhighlight %}

> A better practice may be to create a property that returns `None` for the
> password, but this will really depend on if your application will need to
> later reference the password.  If it does, you won't be able to set a
> property to return `None`.  Password security in code is way outside of the
> scope of this article, though!

So far, so good.  But there are a few problems with our code as-is.  First, we
don't cleanly terminate the connection to the router!  Oops!  That's pretty
easy to overlook, actually.  Besides that, writing the same lines over and over
to connect and disconnect in various places tends to get old.  How can we fix
that?  Context managers!

Let's modify our `Junos` class to add the following:

{% highlight python %}
class Junos(object):
...
    def __enter__(self):
        self.connection = self._connect()
        return self

    def __exit__(self, exctype, excisnt, exctb):
        if self.connection:
            self.connection.close()
...
{% endhighlight %}

Now, we can use a context manager to handle the connection stand up and
teardown logic.  Let's see how the previous example works out under this new
workflow.

{% highlight python %}
>>> from pprint import pprint
>>> import router
>>> with router.Junos('192.168.0.151', password='harbl1234') as rtr:
...     default_arp_table = rtr.arp_table
...     rtr.vpn = 'vpn1'
...     vpn1_arp_table = rtr.arp_table
...     pprint(default_arp_table)
...     pprint(vpn1_arp_table)
...
[{'hostname': '192.168.0.151',
  'interface': 'ge-0/0/0.0',
  'ip_address': '10.0.2.2',
  'mac_address': '52:54:00:12:35:02',
  'vpn': 'default'},
 {'hostname': '192.168.0.151',
  'interface': 'ge-0/0/2.0',
  'ip_address': '192.168.0.1',
  'mac_address': '88:f7:c7:95:91:96',
  'vpn': 'default'},
 {'hostname': '192.168.0.151',
  'interface': 'ge-0/0/2.0',
  'ip_address': '192.168.0.7',
  'mac_address': '60:03:08:a6:d5:e2',
  'vpn': 'default'}]
[{'hostname': '192.168.0.151',
  'interface': 'ge-0/0/1.0',
  'ip_address': '192.168.0.169',
  'mac_address': '00:ff:85:7f:78:03',
  'vpn': 'vpn1'}]
>>>
{% endhighlight %}

Neat!  It works exactly as expected.  And a nice benefit: it reduced the number
of lines by one!  In reality, if we had remembered to disconnect the session,
it would've reduced the number of lines by two.  It also saves us from
forgetting to connect or disconnect, as well as reducing repetitive code.

# END!

Okay, so there's actually a lot more I could say about this topic.  The `Junos`
class, for example, should probably be subclassed from a more generic class,
perhaps titled `Router` or `NetworkElement`.  Exceptions should be implemented
at some point.  The password should be handled a little better internally.
Credentials in general should be sourced from somewhere else.  There's a lot
more to say about all of this, but it's all out of scope.

This post was really about going beyond `junos-eznc` and some of the practices
and limitations around it.  Hopefully you were able to glean something useful
from it.  Find me [on the Twitters](https://twitter.com/oss_stack) if you'd
like to discuss this post.
