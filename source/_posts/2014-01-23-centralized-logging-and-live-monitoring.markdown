---
layout: post
title: "Centralized Logging and Live Monitoring"
date: 2014-01-23 21:23:05 -0700
comments: true
categories:
  - oss
  - logging
  - monitoring
---

How great would it be to read all of your router logs from one location?
And to be able to watch the messages scroll by in real time?  This post
will show you not only how to send your router logs to a centralized
location, but it will also show you how to organize the logfiles on your
centralized server.  We'll take it one step further by writing a shell
script that finds the latest logfiles and prints them out.

## Junos Configuration

I suspect that you're pretty familiar with Junos and probably already
know the configuration necessary to send logfiles to a server, but just
in case, here's the necessary configuration:

``` bash
tyler@lab01# show | compare
[edit system]
+   syslog {
+       host 192.168.1.254 {
+           any any;
+           match "!(.*rancid.*)";
+           port 514;
+       }
+   }

{master:0}[edit]
```

> You can copy from `[edit system]` to the line above `{master:0}[edit]`
> and paste it into your Junos session if you enter the `load patch
> terminal` command from configuration mode.  Just press `^D`
> (control+D) to tell the system you've finished loading the patch.

The above will send every log message, except for messages contianing
the line "**rancid**", to the specified server.  I always exclude
"**rancid**" from log messages as my RANCID configuration is set to pull
configs from equipment every five minutes.

<!-- more -->

## Server Setup

`rsyslog` is the tool of choice for this article.  It's worth noting
that you can likely obtain the same effect with other syslog servers,
but you'll need to consult the relevant documentation.

> In this article, the author assumes the use of Ubuntu (and
> specifically uses version 12.04).  Other distributions will work but
> may use different commands, particularly for package installation.

> In addition, if your company already uses a syslog server, you should
> continue to use that particular server instead of trying to run
> `rsyslog` next to it.

### Installing and Configuring <small>`rsyslog` to the rescue</small>

To get started, you'll need to install `rsyslog`.  Do this with the
following command:

``` bash
sudo apt-get install -y rsyslog
```

Once installed, open `/etc/rsyslog.conf` with `vim` and delete whatever
exists and paste the below configuration:

``` bash
$ModLoad imuxsock
$ModLoad imudp
$ModLoad imklog
$UDPServerRun 514
$IncludeConfig /etc/rsyslog.d/*.conf
```

> If you don't know how to use `vim`, I recommend that you read [my
> article on the basics of Linux][1].

The above configuration makes it possible to listen on UDP port
"**514**", the port specified in the Junos configuration above, and
includes configuration files in the `/etc/rsyslog.d/` directory (if it
ends in `.conf`).  This makes it easier to write configuration files in
a distributed manner.

The next step is to configure a template.  Open up
`/etc/rsyslog.d/10-routers.conf` and paste the below configuration:

``` bash
$template
DynFile,"/var/log/netops/%HOSTNAME%/%timegenerated:1:10:date-rfc3339%"
$filegroup noc
:source , !isequal , "localhost" ?DynFile
:source , !isequal , "localhost" ~
```

> You may have to change `localhost` to the hostname portion of your
> monitoring server's FQDN.  For example, my server's FQDN (as revealed
> by the `hostname` command) is "**mon.example.com**".  The hostname
> portion of that is "**mon**", so I had to change `localhost` to `mon`.

The above configuration defines a template, `DynFile`, and essentially
says, "If the source of the log message I receive is anything EXCEPT
"**localhost**", put a file with the format "**YYYY-MM-DD**" in a
directory `/var/log/netops/%HOSTNAME%/`, where %HOSTNAME% is the name of
the message source.  When I create the file, it will have a group of
"**noc**"."  This results in your logfile being something such as
"**/var/log/netops/lab01/2014-01-21**" and being readable by members of
the "**noc**" group.

> The "**noc**" group does not exist yet.  Neither does the folder
> "**/var/log/netops/**".  We're getting to that.

### Groups and Folders <small>Final preparations</small>

To get everything up and going, we need to create the "**noc**" group
and the "**/var/log/netops/**" folder.  We'll have to add the
appropriate permissions for it as well.  This can all be done relatively
quickly with the following commands:

``` bash
sudo -s
mkdir /var/log/netops
groupadd noc
usermod -a -G noc tyler
chgrp -R noc /var/log/netops
chmod 750 /var/log/netops
exit
```

> Don't forget to change "**tyler**" to your username!

### Testing

After a few minutes, you should be able to see logfiles in your new
directory structure.  See the example output below:

``` bash
tylerc@mon [02:04:30] [~]
-> % ls -R /var/log/netops
/var/log/netops:
lab01

/var/log/netops/lab01:
2014-01-22

tylerc@mon [02:04:56] [~]
-> % cat /var/log/netops/lab01/2013-01-22
2014-01-22T00:00:00+00:00 lab01 /usr/sbin/cron[10072]: (root) CMD (newsyslog)
2014-01-22T00:00:00+00:00 lab01 /usr/sbin/cron[10073]: (root) CMD (/usr/libexec/atrun)
2014-01-22T00:00:04+00:00 lab01 sshd[10077]: TAC_AUTHEN_STATUS_GETPASS
2014-01-22T00:00:04+00:00 lab01 sshd[10077]: TAC_AUTHEN_STATUS_PASS
2014-01-22T00:00:04+00:00 lab01 sshd[10077]: Accepted password for jim from 192.168.1.50 port 53460 ssh2
2014-01-22T00:00:05+00:00 lab01 mgd[10081]: UI_AUTH_EVENT: Authenticated user 'noc' at permission level 'j-ops-ro'
```

## Unified Live Log Messages <small>`multitail` and `bash` are
superheroes</small>

The magic of an excellent utility called `multitail` will let you read
multiple log files at the same time and split them up into multiple
sub-windows if you so desire.  We'll use a simple bash script to
automagically pull all of the current router/switch/firewall/etc logs
in, aggregates them into a single feed and window, and "follows" them
(outputs their contents to the screen whenever they're data is written
to them).  We'll also add the ability to filter this information so that
you can get only OSPF events or only BGP events or any combination of
any matched phrase.

### Multitail

First, grab multitail:

``` bash
sudo apt-get install -y multitail
```

And test it out:

``` bash
tylerc@mon [02:55:14] [~]
-> % multitail /var/log/netops/lab01/2014-01-22 -I
/var/log/netops/lab02/2014-01-22
2014-01-22T02:31:19+00:00 lab01 xntpd[1223]: NTP Server Unreachable
2014-01-22T02:31:33+00:00 lab01 last message repeated 7 times
2014-01-22T02:31:35+00:00 lab01 xntpd[1223]: NTP Server Unreachable
2014-01-22T02:35:00+00:00 lab01 /usr/sbin/cron[70084]: (root) CMD (/usr/libexec/atrun)
2014-01-22T02:40:00+00:00 lab01 /usr/sbin/cron[70088]: (root) CMD (/usr/libexec/atrun)
2014-01-22T02:45:00+00:00 lab01 /usr/sbin/cron[70093]: (root) CMD (/usr/libexec/atrun)
2014-01-22T02:45:00+00:00 lab01 /usr/sbin/cron[70094]: (root) CMD (newsyslog)
2014-01-22T02:48:36+00:00 lab01 xntpd[1223]: NTP Server Unreachable
2014-01-22T02:48:52+00:00 lab01 last message repeated 8 times
2014-01-22T02:50:00+00:00 lab01 /usr/sbin/cron[70099]: (root) CMD (/usr/libexec/atrun)
2014-01-22T02:55:00+00:00 lab01 /usr/sbin/cron[70103]: (root) CMD (/usr/libexec/atrun)
2014-01-22T02:55:18+00:00 lab01 mgd[11303]: UI_CHILD_STATUS: Cleanup child '/usr/sbin/lrmuxd', PID 11321, status 0
2014-01-22T02:55:18+00:00 lab02 mgd[11303]: UI_CHILD_START: Starting child '/usr/sbin/pgmd'
2014-01-22T02:55:18+00:00 lab02 mgd[11303]: UI_CHILD_STATUS: Cleanup child '/usr/sbin/pgmd', PID 11322, status 0
2014-01-22T02:55:19+00:00 lab02 mgd[11303]: UI_CHILD_START: Starting child '/usr/sbin/sdxd'
2014-01-22T02:55:19+00:00 lab02 mgd[11303]: UI_CHILD_STATUS: Cleanup child '/usr/sbin/sdxd', PID 11323, status 0
2014-01-22T02:55:19+00:00 lab02 mgd[11303]: UI_JUNOSCRIPT_CMD: User 'root' used JUNOScript client to run command 'request-end-session'
```

### rtail <small>The `bash` script</small>

Now, let's create a script called `rtail` (for "**r**outer **tail**")
and make it do some magic.  First, create a new text file called `rtail`
in the `/usr/bin/local/` directory.

> For a refresher, the appropriate command would be `sudo vim
> /usr/bin/local/rtail`.  See [my crash course on the basics of
> Linux][1] for more details.

Enter the following in the file and save it:

``` bash
#!/bin/bash
# Name    : rtail
# Author  : Tyler Christiansen
# Purpose : View current and future log messages for routers
# Date    : 2013-12-22

# Variable declarations
today=$(date +"%Y-%m-%d")
declare -a arg
delim=" -I "

# Search for files with today's date in the log dir
for x in $(find /var/log/netops -name *$today*); do
  # Add the full path to the $arg array
  arg=( "${arg[@]}" "$x" )
done

# Implode the array
arg=$(printf "${delim}%s" "${arg[@]}")
arg=${arg:${#delim}}

# If the user didn't supply a filter...
if [ $# -eq 0 ]; then
  # Then call multitail without flags
  multitail $arg
else
  # The user supplied a filter, so call multitail
  #   with the -E flag
  multitail -E $1 $arg
fi
```

Now, edit permissions to make it owned by the "**noc**" group and lock
down execution and read/write:

``` bash
sudo chgrp noc /usr/local/bin/rtail
sudo chmod 650 /usr/local/bin/rtail
```

Now you should be able to execute the command.  Supply filters you want
to apply (such as "**ospf**") in quotation marks.

> There is an unfortunate feature missing in `multitail`: you can't
> perform case-insensitive matches from the command invocation.

> However, if you start rtail with no filter expressions, you can easily
> perform case-insenstive searches by pressing the `/` key and entering
> your text, then selecting the `case insensitive` option by pressing
> the `tab` key.

Here's an example filtering on "**cron**":

``` bash
2014-01-22T03:53:00+00:00 lab01 /usr/sbin/cron[96133]: (root) CMD (newsyslog)
2014-01-22T03:50:00+00:00 lab03 /usr/sbin/cron[96593]: (root) CMD (/usr/libexec/atrun)
2014-09-19T06:25:00+00:00 lab07 /usr/sbin/cron[48036]: (root) CMD (/usr/libexec/atrun)
2014-09-26T03:01:00+00:00 lab06 /usr/sbin/cron[98802]: (root) CMD (adjkerntz -a)
2014-09-26T03:40:00+00:00 lab05 /usr/sbin/cron[98767]: (root) CMD (/usr/libexec/atrun)
2014-01-22T03:53:00+00:00 lab02 /usr/sbin/cron[53337]: (root) CMD (newsyslog)
2014-01-22T03:50:00+00:00 lab05 /usr/sbin/cron[39761]: (root) CMD (/usr/libexec/atrun)
2014-09-14T07:50:00+00:00 lab10 /usr/sbin/cron[98647]: (root) CMD (/usr/libexec/atrun)
2014-09-26T03:35:00+00:00 lab08 /usr/sbin/cron[98022]: (root) CMD (/usr/libexec/atrun)
2014-09-14T07:55:00+00:00 lab09 /usr/sbin/cron[98586]: (root) CMD (/usr/libexec/atrun)
2014-01-22T03:50:00+00:00 lab07 /usr/sbin/cron[11694]: (root) CMD (/usr/libexec/atrun)
2014-09-14T05:05:00+00:00 lab02 /usr/sbin/cron[97612]: (root) CMD (/usr/libexec/atrun)
2014-09-14T07:55:00+00:00 lab04 /usr/sbin/cron[98640]: (root) CMD (/usr/libexec/atrun)
2014-01-22T03:54:00+00:00 lab02 /usr/sbin/cron[53340]: (root) CMD (newsyslog)
2014-01-22T03:54:00+00:00 lab01 /usr/sbin/cron[96136]: (root) CMD (newsyslog)
```

> The output in reality looks a bit different.  The screenshot below
> shows what it would actually look like.  I've blacked out actual
> hostnames (sorry).

## The End

You should definitely check out the [multitail webpage][2] and read up
on it a little more.  The bash script above is licensed under the
[FreeBSD 2-Clause License][3], so feel free to modify--it has a lot of
room for improvement!


[1]: /blog/linux-crash-course/ "Crash Course - Linux"
[2]: http://www.vanheusden.com/multitail/ "multitail"
[3]: http://opensource.org/licenses/BSD-2-Clause "BSD 2-Clause License"
