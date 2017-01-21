---
layout: post
title: "Gathering Troubleshooting Data Every N Seconds in Junos"
description: "Gathering Troubleshooting Data - Programmatically and Not"
category: junos
tags: [ technical, junos, 30in30 ]
---

First, this is _not_ the way I would approach this task.  My preferred approach
is a programmatic, but sometimes you just don't have access to the things you
need (or you don't yet have the necessary skills).  Maybe you just need
something now instead of spending time hacking up a Python or Ruby script.

# Neat FreeBSD Shell Trick

Here's something you might now have known.  I sure didn't until I decided
I wanted to write this post and find the way to make it happen without
a programmatic approach.

You know how you type `cli` to start the Junos shell when you're in the FreeBSD
shell?  Did you know you can pass arguments to it?  Check it out!

{% highlight bash %}
tyler@fw.example.com> start shell
% cli show version
Hostname: fw.example.com
Model: srx5400
JUNOS Software Release [12.1X46-D25.7]
%
{% endhighlight %}

Let's take it a step farther and (crudely) parse the output:

{% highlight bash %}
% cli show version | grep -i junos | awk '{print $4}' 
[12.1X46-D25.7]
%
{% endhighlight %}

Now, there are obviously _much_ better ways to get this type of information,
but there are some things for which SNMP just isn't a fit (or it's too painful
to parse).

What other interesting things can we do?  How about getting the input rate of
an interface every five seconds for 30 seconds?

{% highlight bash %}
% sh
$ i=0; while [ $i -le 5 ]; do cli show interfaces xe-2/0/0 | grep -i rate; true $((i=i+1)); sleep 5; done
Input rate     : 0 bps (0 pps)
Output rate    : 0 bps (0 pps)
Input rate     : 0 bps (0 pps)
Output rate    : 0 bps (0 pps)
Input rate     : 0 bps (0 pps)
Output rate    : 0 bps (0 pps)
Input rate     : 0 bps (0 pps)
Output rate    : 0 bps (0 pps)
Input rate     : 0 bps (0 pps)
Output rate    : 0 bps (0 pps)
Input rate     : 0 bps (0 pps)
Output rate    : 0 bps (0 pps)
$
{% endhighlight %}

You can't see it here because this is a non-live lab, but rest assured that it
works (try for yourself!).

> There's some overhead in this, so it won't be _entirely_ accurate.  There are
> other commands (cprod, for example) but their syntax isn't publicly
> documented.

# Going a Step Further

With the knowledge we know have, we can implement a script such as this:

{% highlight bash %}
#!/bin/sh

INTERFACE=$1
COUNTER=0
MAX=6

while true; do
  date
  cli show interfaces xe-2/0/0 | grep -i rate
  sleep 5
  true $((COUNTER=COUNTER+1))
  if [ $COUNTER -eq $MAX ]; then
    exit
  fi
done
{% endhighlight %}

Create that file and SCP it to the target machine (or create it manually using
`vi` if you are comfortable with doing so).  I named my script `get_rates`, so
to run the script, I type `sh get_rates xe-2/0/0`.  Here's an example:

{% highlight bash %}
root@fw% sh get_rates xe-2/0/0
Fri Oct 31 02:49:40 UTC 2014
Input rate     : 0 bps (0 pps)
Output rate    : 0 bps (0 pps)
Fri Oct 31 02:49:55 UTC 2014
Input rate     : 0 bps (0 pps)
Output rate    : 0 bps (0 pps)
Fri Oct 31 02:50:10 UTC 2014
Input rate     : 0 bps (0 pps)
Output rate    : 0 bps (0 pps)
Fri Oct 31 02:50:25 UTC 2014
Input rate     : 0 bps (0 pps)
Output rate    : 0 bps (0 pps)
Fri Oct 31 02:50:40 UTC 2014
Input rate     : 0 bps (0 pps)
Output rate    : 0 bps (0 pps)
Fri Oct 31 02:50:55 UTC 2014
Input rate     : 0 bps (0 pps)
Output rate    : 0 bps (0 pps)
root@fw%
{% endhighlight %}

With this, you should be able to go forth and conquer!

# Extra: The Programmatic Way

You can programmatically obtain this data, too.  Below are two scripts, one in
Ruby, and one in Python, that accomplish the same task with significantly more
accuracy with regards to the original requirements.

## The Ruby Way

{% highlight ruby %}
#!/usr/bin/env ruby

require 'net/netconf/jnpr'
require 'junos-ez/stdlib'

intf = ARGV[0]
login = {
  target: '192.168.0.1',
  username: 'tyler',
  password: 'secretsauce'
}

session = Netconf::SSH.new(login)
session.open
Junos::Ez::Provider(session)

current = 0
max = 6

while current < max do
  interface = session.rpc.get_interface_information :interface_name => intf
  interface = interface.xpath('physical-interface')
  puts "Output bps: #{interface.xpath('traffic-statistics/output-bps').text}"
  puts "Output pps: #{interface.xpath('traffic-statistics/output-pps').text}"
  puts "Input bps: #{interface.xpath('traffic-statistics/input-bps').text}"
  puts "Input pps: #{interface.xpath('traffic-statistics/input-pps').text}"
  sleep 5
  current += 1
end
{% endhighlight %}

> This code could _definitely_ be cleaned up.  It's very sloppy, and uses
> `ARGV` instead of something nicer like [`trollop`][2].  You'll also need the
> [`junos-ez-stdlib`][3] gem.

## The Python Way

{% highlight python %}
import sys
import time
from jnpr.junos import Device

intf = sys.argv[1]
dev = Device('192.168.0.1', user='tyler', password='secretsauce')
current = 0
max = 5

dev.open()

while (current < max):
    interface = dev.rpc.get_interface_information(interface_name=intf)
    interface = interface.find('physical-interface')
    print("Output bps: " + interface.findtext('traffic-statistics/output-bps'))
    print("Output pps: " + interface.findtext('traffic-statistics/output-pps'))
    print("Input bps: " + interface.findtext('traffic-statistics/input-bps'))
    print("Input pps: " + interface.findtext('traffic-statistics/input-pps'))
    time.sleep(5)
    current = current + 1
{% endhighlight %}

> If the Ruby code seems a bit more verbose, that's because it is.
> I parameterized a bit more in the Ruby script than in the Python script.

# End!

I tried to maintain a decent amount of feature parity between everything
without going overboard.  The examples above are all extremely quick and didn't
take much though.  I probably could have done a much better job, but I'll leave
refinement as an exercise to the reader (I'm lazy).

I hope this was at least moderately informative.  Drop me a line if you have
any questions.

This post is part of the [#30in30 challenge][1].

[1]: http://etherealmind.com/challenge-30-blogs-30-days/ "30 Blogs in 30 Days Challenge"
[2]: https://rubygems.org/gems/trollop "trollop ruby gem"
[3]: https://github.com/Juniper/ruby-junos-ez-stdlib "junos-ez-stdlib gem"

