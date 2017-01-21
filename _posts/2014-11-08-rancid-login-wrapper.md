---
layout: post
title: "RANCID Login Wrapper"
description: ""
category: rancid
tags: [ rancid, 30in30 ]
---

[RANCID][2] is an incredibly powerful tool.  I've written about installing it
with a git backend using [Ansible][3] previously [here][4].

This time, though, I'm going to introduce you to the first of (potentially)
several scripts that utilize RANCID.  There won't be a lot of detail, and this
script is a first rough draft.  To use it, you need to have a `.cloginrc` and
the "binaries" that RANCID ships with must be in your path.

This script currently recognizes Cisco, Force10, Juniper (Junos-based), Juniper
(ScreenOS-based), and Dell (PowerConnect) devices.  Others can be added
easily--contact me with specifics or submit a patch.

To utilize this script, you can clone the git repo ([here][5]), or you can copy
and paste the two scripts below.

### rancid.cfg

{% highlight bash %}
CONF_DIR="/opt/rancid/etc"
REPO_BASE="/opt/rancid/var"
LOCATIONS=$(egrep "^LIST_OF" $CONF_DIR/rancid.conf | awk -F\" '{print
$(NF-1)}')
LOCATIONS=($LOCATIONS)
{% endhighlight %}

### l

{% highlight bash %}
#!/bin/bash

# Load configuration
source rancid.cfg

# Functions
usage () {
  echo "Usage: $0 [options]"
  echo "Options:"
  echo "  -h: (optional) This help menu"
  echo "  -r: (required) The device to connect to"
  echo "  -d: (optional) Enable debug output"
  echo "  -x: (optional) A file containing a list of commands to execute"
  echo "  -c: (optional) A command to execute"
  echo "Notes:"
  echo "  The -x and -c options are mutually exclusive."
  echo "  You can also invoke without the -d option:"
  echo "    $0 [options] - <device>"
}

error () {
  MSG="$1"
  USAGE="$2"
  echo "Fatal error: $MSG."
  [ -n "$USAGE" ] && if $USAGE; then usage; fi
}

get_script () {
  local VENDOR="$1"
  case "$VENDOR" in
    "juniper")
      echo "jlogin"
      ;;
    "netscreen")
      echo "nlogin"
      ;;
    "cisco" | "force10")
      echo "clogin"
      ;;
    "dell")
      echo "dlogin"
      ;;
    *)
      error "Vendor: $VENDOR is unknown for $DEVICE" false
      ;;
  esac
}

get_device () {
  local DEVICE="$1"
  local RESULT=""
  for LOCATION in ${LOCATIONS[@]}; do
    RESULT=$(egrep "^$DEVICE.*:up" "$REPO_BASE/$LOCATION/router.db" 2>
/dev/null)
    [ -n $"RESULT" ] && break
  done
  [ -z "$RESULT" ] && error "Host $DEVICE not found in rancid configuration"
false && exit 1
  echo "$RESULT"
}

validate_options () {
  [ -z "$DEVICE" ] && error "No device specified" true
  [ -n "$COMMANDS_FILE" ] && [ -n "$COMMANDS" ] &&
    error "-x and -c are mutually exclusive" true
}

get_option () {
  local OPTION=""
  [ -n "$COMMANDS_FILE" ] && OPTION="-x $COMMANDS_FILE"
  [ -n "$COMMANDS" ] && OPTION="-c $COMMANDS"
  echo "$OPTION"
}

# Get command line options
DEVICE=""
while getopts ":hdr:x:c:" OPTION; do
  case $OPTION in
    h)
      usage
      exit 0
      ;;
    r)
      DEVICE="$OPTARG"
      ;;
    x)
      COMMANDS_FILE="$OPTARG"
      ;;
    c)
      COMMANDS="$OPTARG"
      ;;
    *)
      error "Invalid option: $OPTARG" true
      ;;
  esac
done
shift $((OPTIND-1))

if [ -n "$1" ]; then
  if [ "$1" == "-" ]; then
    DEVICE="$2"
  else
    DEVICE="$1"
  fi
fi

validate_options
RESULT=$(get_device $DEVICE)
[ $? -ne 0 ] && echo "$RESULT" && exit 1
DEVICE=$(echo $RESULT | cut -d ":" -f 1)
[ $? -ne 0 ] && echo "$DEVICE" && exit 1
VENDOR=$(echo $RESULT | cut -d ":" -f 2)
[ $? -ne 0 ] && echo "$VENDOR" && exit 1
EXEC=$(get_script $VENDOR)
[ $? -ne 0 ] && echo "$EXEC" && exit 1
OPTION=$(get_option)
$EXEC "$OPTION" $DEVICE
{% endhighlight %}

### Usage

{% highlight bash %}
tyler@rancid:~$ ./l -h
Usage: ./l [options]
Options:
  -h: (optional) This help menu
  -r: (required) The device to connect to
  -d: (optional) Enable debug output
  -x: (optional) A file containing a list of commands to execute
  -c: (optional) A command to execute
Notes:
  The -x and -c options are mutually exclusive.
  You can also invoke without the -d option:
    ./l [options] - <device>
tyler@rancid:~$
{% endhighlight %}

### Example

{% highlight bash %}
tyler@rancid:~$ ./l -c 'show ip interface vlan 5' - m2.e
m2.example.com
spawn ssh -c 3des -x -l tyler m2.example.com


User Name:tyler
Password:*************

m2.example.com#
m2.example.com#   show ip interface vlan 5


  Gateway IP Address        Activity status       Type
----------------------- ----------------------- --------


      IP Address          Type
----------------------- ---------
192.168.0.1/24          Static

m2.example.com#exit
Connection to m2.example.com closed.
tyler@rancid:~$
{% endhighlight %}

As you can see, I did not enter the full name of the host.  The script will
match against the first name it comes across in your `router.db` file(s).

This script can use a lot of work.  The error catching isn't always the best,
but I haven't found any bugs yet (that I haven't subsequently fixed, anyway).
The `-d` (debug) option doesn't do anything yet, though, so maybe that's a bug.

This post is part of the [#30in30 challenge][1].

[1]: http://etherealmind.com/challenge-30-blogs-30-days/ "30 Blogs in 30 Days Challenge"
[2]: http://www.shrubbery.net/rancid/ "RANCID"
[3]: http://www.ansible.com/home "Ansible"
[4]: /blog/ansible/2014/11/01/git-gitweb-and-rancid-automated-installation/ "Git Installation with Ansible"
[5]: https://github.com/supertylerc/rancid_scripts "rancid_scripts GitHub Repository"
