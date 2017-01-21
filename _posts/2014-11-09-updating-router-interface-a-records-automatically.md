---
layout: post
title: "Updating Router Interface A Records"
description: "Updating Router Interface A Records Automatically"
category: code
tags: [ ruby, code, 30in30, junos ]
---

This is specifically for A records and Junos.  Similar things can be done for
other vendors and other record types.  For this script, you would need
a separate script for each device.  In a future version that I will be
releasing on GitHub, it will take into account all devices and zones.

To use the script, you need the `zonefile` and `junos-ez-stdlib` gems.

> This script will rewrite your zone file.  You are keeping your zone files in
> source control, right?  It will not restart `bind`, so the changes will not
> go into effect until you manually restart `bind`.  This is a safety feature.

{% highlight ruby %}
#!/usr/bin/env ruby

require 'net/netconf/jnpr'
require 'junos-ez/stdlib'
require 'zonefile'

login = {
  target: '192.168.0.1',
  username: 'tyler',
  password: 'secretSauce'
}
interfaces = []
zone_path = 'sfo.example.com.zone'
interface_node = 'configuration/interfaces/interface'
ip_node = 'family/inet/address/name'
hostname_node = 'configuration/system/host-name'
zone_file = Zonefile.from_file(zone_path)
zone_file_changed = false

session = Netconf::SSH.new(login)
session.open
Junos::Ez::Provider(session)
config = session.rpc.get_config
interfaces_config = config.xpath(interface_node)
hostname = config.xpath(hostname_node).text
interfaces_config.each do |intf|
  interface = intf.xpath('name').text.gsub(/\//, '-')
  intf.xpath('unit').each do |u|
    unit = u.xpath('name').text
    ip = u.xpath(ip_node).text.split('/')[0] if u.xpath(ip_node)
    interfaces << { name: "#{interface}-#{unit}-#{hostname}", host: ip }
  end
end

interfaces.each do |interface|
  interface_exists = false
  zone_file.a.each do |a_record|
    if a_record[:name] == interface[:name]
      interface_exists = true
      if a_record[:host] != interface[:host]
        a_record[:host] = interface[:host]
        zone_file_changed = true
      end
    end
  end
  unless interface_exists
    zone_file.a << interface
    zone_file_changed = true
  end
end

if zone_file_changed
  zone_file.new_serial
  File.open(zone_path, 'w') {|f| f.write(zone_file.output) }
end
{% endhighlight %}

This post is part of the [#30in30 challenge][1].

[1]: http://etherealmind.com/challenge-30-blogs-30-days/ "30 Blogs in 30 Days Challenge"
