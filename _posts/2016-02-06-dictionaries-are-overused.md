---
layout: post
title: "Hashes (or Dictionaries) are Overused"
description: "An explanation of why you may want to consider classes."
category: code
tags: [python, ruby, code]
---

I am officially a professional software engineer. Well, maybe not
professional. But my day job is writing software. I write in Python by
day and Ruby by night. During the past several months of doing this,
I’ve felt more and more that I’ve been overusing hashes (known as
dictionaries to Pythonistas). Why? Let’s have a look.

> This post’s examples are modeled after hypothetical networking
> scenarios, but they can be applied generically to any sort of similar
> situation.

# Modeling Interfaces

We’ll start off easy. Let’s think about interfaces and the properties
they could have. For the sake of simplicity, we’ll assume that we’re
discussing only layer 3 interfaces. Here’s a list of some physical and
logical properties a layer 3 interface _might_ have:

* Name
* IP Address
* Subnet Mask
* IP MTU
* Physical MTU
* Description
* Line Rate
* Physical Status
* Administrative Status

> Obviously, this list could go on (nearly) forever. We’re going to stick
> to this list for now, though, again for the sake of simplicity.

How might we model this using a hash/dictionary? Here is how in Python:

{% highlight python %}
interface = {
    'name': 'gi1/1',
    'ip_address': '10.0.0.1',
    'subnet_mask': 24,
    'ip_mtu': 1500,
    'physical_mtu': 1514,
    'description': '10G Core to dtx1.example.com',
    'line_rate': 10000,
    'physical_status': 'up',
    'administrative_status': 'up'
}
{% endhighlight %}

And in Ruby:

{% highlight ruby %}
interface = {
  name: 'gi1/1',
  ip_address: '10.0.0.1',
  subnet_mask: 24,
  ip_mtu: 1500,
  physical_mtu: 1514,
  description: '10G Core to dtx1.example.com',
  line_rate: 10000,
  physical_status: 'up',
  administrative_status: 'up'
}
{% endhighlight %}

> The MTU values above aren’t important, but they assume the layer 2
> encapsulation is Ethernet II.

# Working with the Data

This looks fine. But what if we want to do something simple, like
concatenate the subnet mask to the IP address? Well, that’s not too
difficult. We could do something like this in Python:

{% highlight python %}
ip_address = "%s/%s" % (interface['ip_address'], interface['subnet_mask'])
{% endhighlight %}

Or in Ruby:

{% highlight ruby %}
ip_address = "#{interface[:ip_address]}/#{interface[:subnet_mask]}"
{% endhighlight %}

Okay, so it’s not terrible, but it’s a lot of typing and a lot of
duplicate work every time you need to do it, which introduces the
potential to forget the all-important `/` once or twice. Okay, so you
_might_ write a function to do it for you so that you don’t need to
worry about the slash anymore. Let’s go to another example, then.

Our hash/dictionary above has a `line_rate` key. This key is represented
as an integer, and its value is in bits per second. What if we need to
get the value represented in gigabits per second (which, arguably, is
slightly more readable)? Our implementation might look like this in
Python:

{% highlight python %}
speed = "%sg" % (interfaces['line_rate'] / 1000)
{% endhighlight %}

This time in Ruby:

{% highlight ruby %}
speed = "#{interface[:line_rate] / 1000}g"
{% endhighlight %}

> Those aren’t great implementations. Okay enough for examples, though.

Okay, so we’re back to error-prone duplication of math. We might write a
utility function again. This is starting to get annoying, though.

Now, let’s assume that we’re using interfaces in a lot of places
throughout our codebase. And let’s assume that we have multiple
engineers working on our codebase. What are the default values for the
various keys, and how do you enforce them? Take the description, for
example. Should it be `None` (Python) or `nil` (Ruby) or `''` (empty
string) if the interface doesn’t have a description set? Or should the
key just not be present in the hash/dictionary? Now that you’ve decided,
how do you enforce it? What about IP addresses and subnet masks? Should
they be strings or should they be an object from a super helpful library
(such as Python's `ipaddress` or Ruby's `ipaddr`)?  Again, how do you
enforce that decision?

> Yes, I know you could just as easily change a variable’s type in both
> Python and Ruby.

A lot of typing, a lot of duplication, and a lot of error-prone work.
And really, at the end of the day, an interface is an object, right? And
what we’re actually doing above is working with the object, right?

# A Better Way

How can we write better software that’s less error-prone and more
object-oriented? Stop using hashes/dictionaries to represent objects.
We’ll use all of the same examples as above to accomplish the same goals
in a much better form. First, Python:

{% highlight python %}
from __future__ import print_function


class Interface(object):
    def __init__(self, name=None, ip_address=None, subnet_mask=None, ip_mtu=0,
                 physical_mtu=0, description='', line_rate=0,
                 physical_status='down', administrative_status='down'):
        self.name = name
        self.ip_address = ip_address
        self.subnet_mask = subnet_mask
        self.ip_mtu = ip_mtu
        self.physical_mtu = physical_mtu
        self.description = description
        self.line_rate = line_rate
        self.physical_status = physical_status
        self.administrative_status = administrative_status

    def ip_addr(self):
        return "%s/%s" % (self.ip_address, self.subnet_mask)

    def speed(self):
        """We consider the speed to be represented as a string in Gbps"""
        return "%sg" % (self.line_rate / 1000)


intf = Interface(name='gi1/1', ip_address='10.0.0.1', subnet_mask=24,
                 ip_mtu=1500, physical_mtu=1514,
                 description='10G Core to dtx1.example.com',
                 line_rate=10000, physical_status='up',
                 administrative_status='down')

print(intf.ip_addr())
print(intf.speed())
{% endhighlight %}

> You could just as easily have made `ip_addr` and `speed` properties.

And in Ruby:

{% highlight ruby %}
class Interface
  attr_accessor :name, :ip_address, :subnet_mask, :ip_mtu, :physical_mtu,
                :description, :line_rate, :physical_status,
                :administrative_status
  def initialize(name: nil, ip_address: nil, subnet_mask: nil, ip_mtu: 0,
                 physical_mtu: 0, description: '', line_rate: 0,
                 physical_status: 'down', administrative_status: 'down')
    @name = name
    @ip_address = ip_address
    @subnet_mask = subnet_mask
    @ip_mtu = ip_mtu
    @physical_mtu = physical_mtu
    @description = description
    @line_rate = line_rate
    @physical_status = physical_status
    @administrative_status = administrative_status
  end

  def ip_addr
    "#{@ip_address}/#{@subnet_mask}"
  end

  def speed
    # We consider the speed to be represented as a string in Gbps
    "#{@line_rate / 1000}g"
  end
end

intf = Interface.new(name: 'gi1/1', ip_address: '10.0.0.1', subnet_mask: 24,
                     ip_mtu: 1500, physical_mtu: 1514,
                     description: '10G Core to dtx1.example.com',
                     line_rate: 10_000, physical_status: 'up',
                     administrative_status: 'down')

puts intf.ip_addr
puts intf.speed
{% endhighlight %}

Now, we have a class that represents our interfaces. This class has
default values for each of its attributes/properties. There are methods
to convert or otherwise display data in a way that we like. This does
everything we did before, but in a more portable and less error-prone
fashion.

# Extra Credit

We could even define a custom `__str__` (Python) or `to_s` (Ruby) method
for the objects. Let’s do that and see how it looks, just as a fun
little exercise.

In Python:

{% highlight python %}
# Add the following to the Interface class
def admin_state(self):
    if self.administrative_status == 'down':
        return 'shutdown'
    return 'no shutdown'

def __str__(self):
    return "\n".join([
        'interface %s' % self.name, '  ip address %s' % self.ip_addr(),
        '  mtu %s' % self.physical_mtu, '  ip mtu %s' % self.ip_mtu,
        '  description %s' % self.description, '  speed %s' % self.line_rate,
        '  %s' % self.admin_state()
    ])
{% endhighlight %}

And in Ruby:

{% highlight ruby %}
# Add the following to the Interface class
def admin_state
  @administrative_status == 'down' ? 'shutdown' : 'no shutdown'
end

def to_s
  [
    "interface #{@name}", "  ip address #{ip_addr}",
    "  mtu #{@physical_mtu}", "  ip mtu #{@ip_mtu}",
    "  description #{@description}", "  speed #{@line_rate}",
    "  #{admin_state}"
  ].join("\n")
end
{% endhighlight %}

Now, if you call `str(interface)` (Python) or `interface.to_s` (Ruby)
you'll get something like this:

```
interface gi1/1
  ip address 10.0.0.1/24
  mtu 1514
  ip mtu 1500
  description 10G Core to dtx1.example.com
  speed 10000
  shutdown
```

And one more exercise, just for fun: determining if an interface is
admin up. Python is up first:

{% highlight python %}
# Add this to your Interface class
def is_enabled(self):
    if self.administrative_status == 'down':
        return False
    return True

# And later call it on the intf object
print(intf.is_enabled())
{% endhighlight %}

And Ruby:

{% highlight ruby %}
def enabled?
  self.administrative_status == 'down' ? false : true
end

# And later call it on the intf object
puts intf.enabled?
{% endhighlight %}

You should get `False` in Python and `false` in Ruby.  Neat, huh? How
would you ask your hash above if it’s enabled? You would have to resort
to another helper method or just duplicating your `if` statement
throughout your codebase.

# Conclusion

I hope you can see how useful classes can be and how overused hashes or
dictionaries can be. Of course, this is all up to interpretation, and
there is bound to be someone who disagrees. Keep this under your
toolbelt, though. There are plenty of other things that can be done when
you examine dictionaries vs. classes for implementing your objects.
