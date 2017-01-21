---
layout: post
title: "Automation Does Not Preclude Networking"
description: "Network Engineers aren't going anywhere.  Here's why."
category: automation
tags: [ansible, automation, networking]
---

I know a few people who think that network automation spells doom for network
engineers.  They're afraid that it somehow obsoletes them.  It doesn't, and
I have two excellent examples of why.

## Code Spaces

Remember these guys?  If not, here's the skinny: they were a cloud-based source
hosting service.  **Were**.  Yeah, they're not anymore.  Why?  You can read the
painful details
[here](http://arstechnica.com/security/2014/06/aws-console-breach-leads-to-demise-of-service-with-proven-backup-plan/),
but the short version is that they sort of failed at security.  Okay, not sort
of failed.  They just failed outright.  But what relevance is this to network
engineers of today?

Let's be honest: these guys probably had plenty of automation.  They probably
didn't spend much time trying to spin up servers or deploy applications.  There
was likely a button that deployed an application, and they probably clicked it.
A lot.

My suspicion, though entirely unfounded, is that they just didn't have security
experts on their payroll.  Why?  Probably because they thought automation was
going to solve the security problem.  Automatically configure `iptables` or
security groups and you're golden.

That wasn't the case.  They needed someone to design, review, implement, and
iterate over security policies and practices.  Even if that person were to do
it once and automate the implementation afterward, you have to keep him on the
payroll to continuously review incidents, best practices, and current
deployments.

Automation did not save them.  AWS did not save them.  Security engineers could
have saved them.  Networking is the same.

## Me

I'm the perfect example, actually.  I have a home lab that I've automated with
Ansible.  It provisions the following for me:

* LDAP
* DNS
* JIRA
* Stash
* Ansible (hashtag Inception!)
* Jenkins

It does quite a bit more for me too.  I could conceivably open a business and
have a fully functioning and automated systems infrastructure from day one.
That's an obscenely horrible idea, though.

LDAP suddenly stops working.  What do I do?  Re-run my Ansible playbook and
hope that it magically works?  I've actually run into this in my lab, and my
solution was to destroy the VM and rebuild it from scratch.  Did it work?
Sure.  Do I have any idea what the problem was?  Nope.  Could I actually fix
the issue?  Even if I knew what it was, there's not a chance.

Here's an interesting data point about my "fix": I lost all of the LDAP
accounts when I destroyed the VM.  If I had another LDAP server with
replication configured, I probably could have saved myself from that.  But I
didn't.  I could blame it on this being a lab, but the truth of the matter is
that I'm not a systems engineer.  I'm not an LDAP expert.

I _could_ be a "DevOps Engineer" at a startup and create their entire
infrastructure from start to finish.  And it would work.  Would it work well?
Would it be secure?  Highly available?  Could I fix it?  Most importantly of
all--would I bet my career and the entire company on my ability to maintain
that infrastructure to a profitable degree?

## Automation and Networking

The above only serves to illustrate a single point: automation does not
preclude networking experts.  Similarly, networking experts do not need to
be automation experts.  They are not mutually exclusive skill sets, but they
are not required to coexist within the same individual, either.  Learn to code
or don't.  Your job won't disappear if you don't, though you will add value if
you do.  When the network automation experts run into issues, or when the
production network is in trouble, you'll be there to save the day.

> I want to take a step back from the polarizing statements above for a moment.
> Automation experts need networking engineers.  Networking engineers need
> automation experts.  They must work together to provide a value-added
> solution for the business, especially at scale.
