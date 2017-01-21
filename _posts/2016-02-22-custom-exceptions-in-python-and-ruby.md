---
layout: post
title: "Custom Exceptions in Ruby and Python"
description: "The case for custom exceptions."
category: code
tags: [python, ruby, code]
---
Creating your own exceptions is a good thing. It lets you catch
something very specific without the need to inspect more generic
exceptions for the right messages every time.

# Catching All Exceptions

Catching all exceptions is bad. Why? Have a look at the following:

#### Python

{% highlight python %}
#!/usr/bin/env python3

try:
    f = open('/tmp/potatoes.txt', 'r')
    f.write('I really love potatoes!')
except Exception as exc:
    print(exc.message)
{% endhighlight %}

#### Ruby

{% highlight ruby %}
#!/usr/bin/env ruby

begin
  f = File.open('/tmp/potatoes.txt', 'r')
  f.write('I really love potatoes!')
rescue Exception => e
  puts e.message
end
{% endhighlight %}

Given the code above, it’s nearly impossible to know why your code
fails. At best, there are two reasons for the failure: the file doesn’t
exist; or you’re attempting to write to a file when you’ve opened it as
read-only. In reality, there are probably many more exceptions that
could be generated, and you probably want to handle them all
differently.

# Custom Exceptions in Python

Let’s look at Python’s `ncclient` library. What if we want to catch
unauthorized RPCs?  `ncclient` doesn’t raise an exception for this. To
see how we can catch an unauthorized command, let’s create a small
miniature library.

{% highlight python %}
#!/usr/bin/env python2

from ncclient import manager
from ncclient.operations import RPCError


class UnauthorizedError(Exception):
    pass


class Device(object):
    def __init__(self, host, port, username, password):
        self.device_manager = manager.connect(host=host, port=port,
                                              username=username,
                                              password=password,
                                              hostkey_verify=None)

    def send_config(self, xml_config):
        try:
            return self.device_manager.edit_config(target='running',
                                                   config=xml_config)
        except RPCError as error:
            if error.tag == 'access-denied':
                raise UnauthorizedError('User is not authorized for this RPC.')
            raise
{% endhighlight %}

Now if a user uses our library, they’ll get the appropriate error for
trying to configure something for which they are not authorized. After
the `if` statement, we re-raise the exception with a bare `raise` so
that we don’t skip over something other than an unauthorized command.

> You could catch many other types of exceptions at the same time, but
> I’ve ommitted them for brevity.

# Custom Exceptions in Ruby

Now let’s look at custom exceptions in Ruby. We’ll do this example as a
simple script instead of assuming it’s a library. Let’s assume that a
user `bob` with the password `bob` exists on the switch but is not
allowed to change configuration on the device. This example assumes a
Brocade VDX is the target device, but the principles remain the same
regardless of vendor.

{% highlight ruby %}
#!/usr/bin/env ruby

require 'net/netconf'


class UnauthorizedError < StandardError
end


login = {target: '10.0.0.1', username: 'bob', password: 'bob'}
Netconf::SSH.new(login) do |dev|
  begin
    dev.rpc.edit_config do |x|
      x.interface(xmlns: 'urn:brocade.com:mgmt:brocade-interface') {
        x.tengigabitethernet {
          x.name '200/0/18'
          x.description "to sw37 te 201/0/1"
        }
      }
    end
  rescue Netconf::EditError => e
    if e.rsp.xpath('//error-tag').first.content == 'access-denied'
      raise UnauthorizedError, 'User is not authorized for this RPC.'
    end
    raise
  end
end
{% endhighlight %}

If we were to execute this script, this is what we’d get:

```
$ ruby custom_exception.rb
custom_exception.rb:21:in `rescue in block in <main>': User is not authorized for this RPC. (UnauthorizedError)
        from harbl.rb:10:in `block in <main>'
        from /home/tyler/.rvm/gems/ruby-2.3.0@netconf/gems/net-netconf-0.4.3/lib/net/netconf/transport.rb:27:in `initialize'
        from /home/tyler/.rvm/gems/ruby-2.3.0@netconf/gems/net-netconf-0.4.3/lib/net/netconf/ssh.rb:21:in `initialize'
        from custom_exception.rb:9:in `new'
        from custom_exception.rb:9:in `<main>'
```

# Conclusion

I hope you can see how useful creating your own exceptions can be. It
makes it much easier to diagnose and debug when a user (inevitably)
reports a bug in your software.
