---
layout: post
title: "Junos Annotate Feature"
description: ""
category: junos
tags: [ technical, junos, 30in30, documentation ]
---

A question came up recently on the mailing lists: How do I add a description of
a ticket number to my policy?

Using the Junos [annotate][2] feature, we can do this easily for anything.
Example below.

{% highlight perl %}
tyler@fw1.example.com> configure
warning: Clustering enabled; using private edit
warning: uncommitted changes will be discarded on exit
Entering configuration mode

{primary:node0}[edit]
tyler@fw1.example.com# edit interfaces

{primary:node0}[edit interfaces]
tyler@fw1.example.com# annotate xe-0/0/7 "Turn up port to new QA zone - see ticket OPS-666"

{primary:node0}[edit interfaces]
tyler@fw1.example.com# top

{primary:node0}[edit]
tyler@fw1.example.com# show | compare
[edit interfaces]
+   /* Turn up port to new QA zone - see ticket OPS-666 */
    xe-0/0/7 { ... }

{primary:node0}[edit]
tyler@fw1.example.com#
{% endhighlight %}

This post is part of the [#30in30 challenge][1].

[1]: http://etherealmind.com/challenge-30-blogs-30-days/ "30 Blogs in 30 Days Challenge"
[2]: http://www.juniper.net/techpubs/en_US/junos14.1/topics/task/configuration/junos-software-configuration-comments-adding.html "Adding Comments in a Junos OS Configuration"
