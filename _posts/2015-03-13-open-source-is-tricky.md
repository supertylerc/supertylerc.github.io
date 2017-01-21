---
layout: post
title: "Open Source is Tricky"
description: ""
category: lessons-learned 
tags: [lessons, social, open-source]
---

This post is part of a series called "Lessons Learned."  In this series, I
talk about the lessons I’ve learned over the past week. These tend to be
"life lessons," but sometimes they’re also technical nuggets of excellence.
This series occurs every Friday.

# Open Source?

I recently came to the realization that simply claiming that a project is open
source does not mean it is actually open source.  When I think about "open
source," I'm not just thinking about the source code.  I'm thinking about the
mentality and community behind a given project.

Take, for example,
[Observium](http://www.observium.org/wiki/Edition_Split#Observium_Community_Edition).
It is "open source" (under the [QPL](http://opensource.org/licenses/QPL-1.0)),
but the "community" isn't known for its friendliness or acceptance of
criticism or patches.  Compare that to the fork
[LibreNMS](http://www.librenms.org).  It is released under the
[GPLv3](http://opensource.org/licenses/gpl-3.0.html).  Why is the difference
in licensing important?  Although the QPL passes the FSF's guidelines, it
fails the
[Debian Free Software Guidelines](https://www.debian.org/social_contract#guidelines).
That's not a particularly painful point, unless a person  believes heavily in
the [Debian Social Contract](https://www.debian.org/social_contract)(I do).
What else is interesting about this comparison of the upstream and fork?
LibreNMS has 19 contributors.  How many does Observium have?  Your guess is as
good as mine.

This isn't the only instance of an "open source" project that has a group of
fairly negative maintainers.  There are plenty of others.  They're either
vaporware or the creators "open sourced" the project just to pay lip service.
I've seen this pretty frequently with large companies.  It's happened at
companies that I've worked for and that I've worked with.  It's happened at
companies that I haven't worked for or with.  I have pull requests open to
projects that just sit around.  Nothing happens.

Of course, this isn't to say that Open Source is bad.  It's actually pretty
awesome.  If I look at [Kirk Byers's](https://twitter.com/kirkbyers)
open source library [Netmiko](https://github.com/ktbyers/netmiko), I can
see that he is very open to the open source model, community, and beliefs.  He
accepts pull requests, and he engages the community.  He's a genuine model of
what Open Source is.  So is [Ansible](https://github.com/ansible/ansible)
(usually).  So is LibreNMS.  But there are many, many projects which exist
solely to pay lip service to open source.

# Contributing is Easy

Along the lines of open source, contributing is actually very easy.  I've had
a few people ask me what to do, and the answer is simple: learn the very
fundamentals of git, find a project that is cool, fork it, submit pull
requests, and watch the magic happen.  If you aren't familiar with git, you
should check out [Try Git](https://try.github.io/levels/1/challenges/1).  It's
pretty cool.  I'll be writing a post in the coming weeks on how to properly
(from my perspective) manage contributing to a GitHub project.

# Static Methods in Python

This one is awesome, and I love it.  Python has a built-in decorator for
writing static methods.  It's `@staticmethod`.  Hard to remember, right?
Here's an example I wrote for Kirk's Netmiko library.

{% highlight python %}
class Utils(object):
    """Utility static methods for working with test data."""
    @staticmethod
    def parse_yaml(yaml_file):
        """Parses a yaml file, returning its contents as a dict."""
        try:
            import yaml
        except ImportError:
            print "Unable to import yaml module."

        try:
            with open(yaml_file) as fname:
                return yaml.load(fname)
        except IOError as (errno, strerr):
            print "I/O Error {0}: {1}".format(errno, strerr)
{% endhighlight %}

> That snippet isn't in the upstream yet.  It's currently (as of this writing)
> in a branch that I'm working on to refactor tests and make them a bit more
> usable for potential contributors.

Why is that awesome?  Because it lets me have an entire utility class that
doesn't have to be instantiated constantly to use its methods!  Winning!

# Aggressive Timelines

This should just be common sense, but I shouldn't commit to aggressive or
optimistic timelines.  Even if it makes someone unhappy (or outright pisses
him/her off), I need to estimate the time that I think it will take and then
add 50% to that estimate.  If I think it will take a week, I commit to 10
days.  A month?  I commit to 6 weeks.  Two months?  I commit to 3 months.  This
may disappoint managers or compliance testers or any number of other people,
but it also ensures I don't slip and miss a timeline as other things come up
or complicate the project.  When I slip and miss a timeline, people get even
angier.

# FBOSS is Cool

It's interesting.  It's cool.  It gives you more options.  But I'm not ready
for it.  I don't have a team of software engineers to maintain it.  I don't
have the time to write controllers for it, since it's really just an agent that
pushes instructions down to an ASIC.  Oh, and I don't have the resources to
the hardware.  Where's the support for it, anyway?  As a network engineer, I
need a NOS that has support behind it.  Especially since I don't have a team
of software engineers ready to debug the source code.

# Ticket All the Things!

If it's not in a ticket, it never happened.  E-mails aren't tickets.  They
don't count.  Everything I ever say needs to be in a ticket.  Tickets don't
go away.  E-mails do.  No matter how hard you try.  And words or chat
conversations?  They're even less reliable than e-mail.  Tickets are like
Certified Mail.  Everything else is like Media Mail...or worse.

# Lay Down the Law

I can't be afraid to say, "No."  It's like I'm ten years old again and
McGruff the Crime Dog is telling me to "Just Say No to Drugs."  Except it's
my professional life instead of drugs.

There are appropriate times to say no, and I can't be afraid of what other
people or management will think if I say no.  Obviously this can be taken
way too far.  If I say no to everything, or if I say no to things that I
really should do, I'm in deep shit.

I have morals.  I have an ethical code to follow.  My commitment is to the
success of the company, but never at the cost of morals and ethics.  Every
company has a Code of Conduct, Business Ethics, or something similar.  I
will say no to any request that requires me to violate those codes--or my own
personal ethical code.

I will do the right thing.  I will make my company successful.  I will follow
the Code of Conduct.  I will follow the Business Ethics.  I will adhere to
the company's core values.  I will say no, no matter the repercussions, if the
request causes me to violate any of those things.
