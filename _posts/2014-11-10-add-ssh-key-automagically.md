---
layout: post
title: "Add SSH Key Automagically"
description: "Adding SSH Key and Logging in in One Line"
category: tips
tags: [ Linux, 30in30, tips ]
---

Just a quick little tip.  Add this alias to make adding your SSH keys
everywhere easy on yourself.

{% highlight bash %}
sh_copy_keys () { ssh-copy-id "$1" && ssh "$1" } && alias ssh="ssh_copy_keys"
{% endhighlight %}

> This will break your zsh auto-completion.  If it's that important to you,
> just manually copy your key using the `ssh-copy-id` command to all hosts.

An altnerative to this, if you can loop through every host you have to log
into, is the following:

{% highlight bash %}
HOSTS=(host-a.example.com host-b.example.com host-c.example.com)
for HOST in "${HOSTS[@]}"; do
  ssh-copy-id $HOST
done
{% endhighlight %}
