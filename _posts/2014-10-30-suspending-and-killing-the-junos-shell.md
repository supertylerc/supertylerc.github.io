---
layout: post
title: "When the Junos Shell Stops Responding..."
description: "Suspending and Killing the Junos Shell"
category: junos
tags: [ technical, junos, 30in30 ]
---

I spent a decent portion of one day this week angrily screaming at the screen.
I couldn't do anything.  I was logged into a switch, and I could type commands
that don't actually __do__ anything.  I could enter `configure` mode, I could
even stage changes, but I just couldn't seem to commit them.  The Junos shell
would hang every time.  Obviously, this can create all sorts of problems, such
as exhausting the number of SSH connections allowed.  I couldn't even force
logout a user via the Junos shell!  To continue troubleshooting, this is what
I did:

1. Login as `root`.
2. Type `cli` to start the Junos shell.
3. Attempt to troubleshoot.
4. When the shell inevitably locks during the troubleshooting process:
    1. Press `^z` (control-z) to suspend the shell process.
    2. Once in the shell, enter the following:

{% highlight bash %}
sh
kill -9 $(ps aux | grep [c]li | awk '{print $2}')
{% endhighlight %}

> That's a bit verbose, so I made it into a script that I dropped into root's
> directory on all of my devices.  Now, I just type `sh kill_cli` and POOF!
> Magic!

Here's the script I drop into the root folder:

{% gist supertylerc/0402ca34b7a7804a4043 %}

Just make it executable (`chmod 700 kill_cli`) and then execute it by typing
`sh kill_cli`.

> Be warned that this method will kill _all_ of the Junos shells, including
> your peers'!  You'll need to be a bit more selective if you just want to kill
> your own (i.e., type `ps aux | grep cli | grep -v grep` [which can be done
> from the `csh` BSD shell that root uses by default] and hunt down the PIDs
> [number in the second column] for your user)

### Extra Tip

Remember I mentioned that you can exhaust SSH connections?  Here's how you can
force a user to disconnect from the FreeBSD shell:

{% highlight bash %}
sh
SSH_USER=tyler
kill -9 $(ps aux | egrep [s]shd.*$SSH_USER)
{% endhighlight %}

Replace `tyler` above with the login name of the user you wish to
disconnect.  You can also just omit `SSH_USER` and replace `$(SSH_USER)` above
with the user name (`tyler`, in our example).

> This is another good candidate for a script!

{% gist supertylerc/ad24b99abc2af5805ef8 %}

First, scp the script to the root user's `$HOME` directory.  Make it
executable:

{% highlight bash %}
chmod 700 logout_user
{% endhighlight %}

Finally, execute it with `sh logout_user $SSH_USER`, replacing `$SSH_USER` with
the user you want to kick out (such as `tyler`).

This post is part of the [#30in30 challenge][1].

[1]: http://etherealmind.com/challenge-30-blogs-30-days/ "30 Blogs in 30 Days Challenge"
