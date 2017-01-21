---
layout: post
title: "TACACS+ [1/3] - Downloading and Compiling"
date: 2014-01-19 12:05:17 -0700
comments: true
categories:
  - tacacs
  - oss
  - authentication
---

> Note: This is Part 1 in a series of posts on TACACS+ installation and
deployment.

> There is a special precursor post [here][1] that you can read if you
> don't have experience with TACACS+.

> [Part 2][4] is available [here][4].

This post is all about installing TACACS+.  For this guide, we're going
to use [Shrubbery Networks' excellent TACACS+ daemon][2].  We're also
going to start laying the foundations for a centralized configuration
control system using `git`.  Finally, we'll write a few scripts to wrap
the daemon up nice and tight to make it a bit easier to manage.

## Installing the Daemon

There may or may not be a package available for the Shrubbery TACACS+
daemon, but for a variety of reasons, it's best to build the most recent
version from source.  This will ensure you get the best new features and
bug fixes.  Plus, building the package is pretty easy.

> Note: This post's examples are all from Ubuntu 12.04, but the concepts
> apply to any Linux distribution or BSD derivative.

### Prerequisites

You'll need a few things to get going.  Here's a list:

* Root on the target Linux or BSD system
* Essential compilation tools
* `tcp_wrappers` library

Here's a quick line to get everything you need from an Ubuntu 12.04
system for which you have the ability to run all commands under `sudo`:

``` bash
sudo apt-get update
sudo apt-get upgrade -y
sudo apt-get install -y vim curl build-essential flex bison tcpd
libwrap0-dev git
```

<!-- more -->

### Get the Source!

Getting the source code for the Shrubbery Networks distribution of
TACACS+ is pretty simple.

``` bash
cd /usr/src
sudo curl -O
ftp://ftp.shrubbery.net/pub/tac_plus/tacacs+-F4.0.4.27a.tar.gz
sudo tar xzf tacacs+-F4.0.4.27a.tar.gz
rm tacacs+-F4.0.4.27a.tar.gz
sudo chown -R $(whoami) tacacs+-F4.0.4.27a
cd tacacs+-F4.0.4.27a
```

> Note: As of this post (17 January 2014), F4.0.4.27a is the most recent
> version of the Shrubbery Networks TACACS+ daemon.  You'll probably
> want to [look here][3] to ensure you're getting the most recent
> version.

### If You Build It...

The next part is cake.  We'll create the directory where the daemon will
live, configure the environment, compile the source, and install the
final result.  We'll create the configuration directory, too, but hold
off on populating it for now.

``` bash
sudo mkdir -p /opt/tac_plus
./configure --prefix=/opt/tac_plus
make
sudo make install
sudo mkdir -p /opt/tac_plus/etc
```

> Note: you shouldn't encounter any errors, but if you do and need a
> hand, feel free to
> <a href="mailto:tyler@oss-stack.io?Subject=TACACS%20Installation%20Issues">
> e-mail me: tyler (at) oss-stack [dot] io.</a>

### Wrap It Up

Next, open up `/etc/init/tac_plus.conf` with `vim`:

``` bash
vim /etc/init/tac_plus.conf
```

Now enter the below into the file.

``` bash
# TACACS+ Upstart Job

description "Shrubbery Networks tac_plus Daemon"
author "Tyler Christiansen <tyler@oss-stack.io>"

start on startup
stop on shutdown
respawn
expect daemon
exec /opt/tac_plus/bin/tac_plus -C /opt/tac_plus/etc/tac_plus.conf
```

> Note: if you're not familiar with `vim`, just press `i` to enter
> _insert_ mode, paste the above contents, press the `[escape]` key, and
> type `:wq`.

So what does the above script do?  Well, when your system starts, the
`tac_plus` daemon starts.  When you shut your system down, `tac_plus`
stops.  The service will restart automatically if it crashes, and
upstart should expect the service to daemonize itself.  This line
(`expect daemon`) is really the key to the `upstart` script working
properly: without it, `upstart` will lose track of the process, and you
won't be able to get status of the daemon, stop it, restart it, or
automatically restart when it crashes.

### State of the Daemon <small>Take a break!</small>

So far, we've installed the Shrubbery Networks `tac_plus` daemon in a
custom directory and written an `upstart` script to control it.  Doesn't
sound like much, does it?  Take a quick break--basic configuration is
next!

## Configuration

We're going to do something a bit different here.  This is in
preparation for a centralized configuration repository that tracks
changes.  This is why you needed to install `git` earlier, even though
it isn't required by `tac_plus`.  We're going to create a new directory
structure under `/opt`, initialize a `git` repository, and create the
initial configuration file that we'll build on in the next two posts.

``` bash
sudo mkdir -p /opt/oss-conf/tac_plus
sudo chown -R $(whoami) /opt/oss-conf
cd /opt/oss-conf
git init
vim tac_plus/tac_plus.conf
```

Recall the `vim` crash-course above and enter the below contents into
your `tac_plus.conf` file:

> Note: Don't worry too much about what the below information actually
> means--we'll cover it as we go through the next two posts.

``` bash
key = "!i-l0v3_t4C4c5#"
accounting file = /var/log/tac_plus
group = engineers {
  default service = permit
}
group = noc {
  default service = permit
}
 user = tyler {
  member = engineers
  login = des $6$nKlz6YCf5d$5jYg3kSMcIpnUP74XOreUiTDlgyFFxP5BivOH4uRl5pc8idyToRtJoe7b.D.CnhGR//R8jmkkW2N4/u9L3l7M1
  enable = des des $6$nKlz6YCf5d$5jYg3kSMcIpnUP74XOreUiTDlgyFFxP5BivOH4uRl5pc8idyToRtJoe7b.D.CnhGR//R8jmkkW2N4/u9L3l7M1
}
user = jim {
  member = noc
  login = des $6$QKH4BA/kj$t/ZI5pXRzRfH0WK/EBye5eZG4twusPTjQPCgF/ZvXD/nfP2f5LqlnY1oVIB1AItoElsSN6quKuf3mrCCkgm7p.
  enable = des $6$QKH4BA/kj$t/ZI5pXRzRfH0WK/EBye5eZG4twusPTjQPCgF/ZvXD/nfP2f5LqlnY1oVIB1AItoElsSN6quKuf3mrCCkgm7p.
}
```

> Note: the password for the user `tyler` is `harbl`, and the password
> for the user `jim` is `blah`.

> Also note that these passwords are encrypted.  They should _always_
> have `des` before the hash, and the hash should _always_ indicate the
> hash type.  In this case, it is `SHA-512`.

> To generate a hashed password, use the `mkpasswd --method=sha-512`
> command.  Note that your own internal policies may require to use a
> different hashing method.

Now add the configuration file to the repository and commit it.

``` bash
git add tac_plus/tac_plus.conf
git commit -m 'add initial tac_plus configuration'
```

Finally, link this to the file your `upstart` script expects:

``` bash
sudo ln -s /opt/oss-conf/tac_plus/tac_plus.conf
/opt/tac_plus/etc/tac_plus.conf
```

## The End <small>For now...</small>

A lot of stuff happened.  Downloading, building, installing,
configuring, revision control, upstart job creation, mini crash
courses...whew!

Some of the items that were briefly covered will have their own posts
(`git` and Linux basics especially) while others will be expanded upon
later in the TACACS+ series.

> [Part 2][4] is now available [here][4].

[1]: /blog/tacacs-introduction-to-aaa/ "TACACS+ [0/3] - Introduction to AAA"
[2]: http://www.shrubbery.net/tac_plus/ "Shrubbery Networks' TACACS+ Daemon"
[3]: ftp://ftp.shrubbery.net/pub/tac_plus "Shrubbery Networks TACACS+ Daemon FTP"
[4]: /blog/tacacs-junos-configuration/ "TACACS+ [2/3] - Junos Configuration"
