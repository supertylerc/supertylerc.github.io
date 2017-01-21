---
layout: post
title: "Automation Workflow"
description: "Proposed Automation Flow with Jenkins, Gerrit, and Ansible"
category: automation
tags: [ automation, cumuluslinux, gerrit, jenkins, ansible ]
---

I gave a [presentation recently](http://www.meetup.com/SF-Network-Automation/events/220445127/)
at [Kirk Byers's](https://twitter.com/kirkbyers)
[San Francisco Network Automation Meetup](http://www.meetup.com/SF-Network-Automation/).
The presentation was supposed to be 10-15 minutes, but it ended up closer to the
45 minute mark (sorry, Kirk+all!).  However, there was a decent bit of feedback
and a lot of questions.

[Cumulus Networks](http://cumulusnetworks.com) hosted the venue, even though the
original talk was going to be from Juniper Networks and even though my
presentation was focused on Juniper Networks.  Many thanks to them and to Kirk
for organizing everything.

Doug talked about using Cumulus, Ansible, and Nutanix together in one big,
happy family.  Read more
[here](http://blog.cnidus.net/2015/03/03/nutanix-cumulus-ansible-demo/).

Kirk presented on Netmiko, which you can read about
[here](https://github.com/ktbyers/netmiko).

I wanted to write a blog post that recaps my presentation but with a different
vendor for the following reasons:

* The recording was destroyed
* [Doug Youd](https://twitter.com/cnudis) thought it would be a good idea
* I wanted to thank Cumulus for hosting
* I wanted to expand my horizons
* I wanted to show exactly how easy this can really be

The last point is important.  To drive it home, I chose to write this blog post
centering around Cumulus Networks.  I took everything I did this past Wednesday
and rewrote it to work with Cumulus Networks--using the same YAML files and
structure.  If you don't know anything about Cumulus Networks, there is one
thing you should know: it's a _drastic_ departure from normal networking
vendors.  The configuration is completely different in nearly every way
imaginable.  With that said, it took me less than eight hours--off and on--to
get everything setup and working the way I wanted.  This included the time it
took to work around lab issues--SOCKS proxies, accessing web interfaces via
SOCKS proxies (not as easy as you think).  It also includes the time it took me
to become familiar enough with Cumulus Linux to configure interfaces, configure
OSPF, install the license, access the switches, and learn how to verify
everything.

> ### Note
> It's definitely worth noting that the nature of Cumulus Linux actually makes
> many things in this process significantly easier.  More on that later.

Let's dig into things a little bit more.

# Culture

I would be remiss if I didn't mention culture.  Culture is Key.  Whenever I talk
about automation, I mention culture.  I try to mention it first, but sometimes
I get ahead of myself and forget until halfway through the conversation.

## Culture is Key

You're doomed to fail if you don't have the culture necessary to pull this off.
The tools actually don't matter--although I am going to present several below,
they are by no means requirements or the only tools for the job.  They may not
even be the best tools.

If you have the culture, the tools will follow.  You'll find them or you'll
create them.  If you have the tools but not the culture, the culture won't
follow.  And when the culture doesn't follow, the tools get misused and abused.
Mistakes happen, and the tools will be blamed when in fact it is the user and
his/her culture.

## Cultivating the Culture

Getting buy-in is paramount.  But how do you get buy-in?  Go buy a copy of
[The Phoenix Project](http://www.amazon.com/Phoenix-Project-DevOps-Helping-Business/dp/0988262509/ref=sr_1_1?ie=UTF8&qid=1425193266&sr=8-1&keywords=the+phoenix+project),
and make it required reading for yourself and everyone on your team.  Then, ask
your manager to read it.  He/she may not, but that's okay--you can paraphrase.

Next, buy a copy of [Kanban in Action](http://www.amazon.com/Kanban-Action-Marcus-Hammarberg/dp/1617291056/ref=sr_1_1?ie=UTF8&qid=1425193275&sr=8-1&keywords=kanban+in+action).
This doesn't need to be required reading for your team, but it should be
required reading for you (and for whomever might implement Kanban, if not you).
I'm not telling you Kanban will solve your problems or make everything better,
but it can definitely help when used right.  You may ultimately decide not to
implement it--that's okay.  The knowledge you will gain from this book will
help you regardless of what you decide to do.

After you've read both books, and after your team has read The Phoenix Project,
talk to them.  Ask them for their thoughts and opinions.  Ask them if any of the
book resonates with them, and ask them what they think can be done within your
organization.  This is a team effort, not just the effort of the team leader.

Now talk to them about the things you learned from Kanban in Action.  Gauge
their reactions.  Ask them for input and help in shaping the future of your
team.  The last step is talking to your manager.  Once you convince him, it's
time to start acting.

To really gauge your progress and culture, you should look at
[the DevOps Checklist](http://devopschecklist.com).  I'm not a huge fan of the
term "DevOps," and I'm even less of a fan of a checklist that tries to define
that term.  That list, however, contains so much useful information for defining
a successful culture that I consider it required reading for everyone on the
team.  Print it out, give it to everyone, and maintain a public copy of it.
Check off items as you implement them.  I don't agree with every item on that
list, and neither should you.  But you should find the value in many--if not
most--of those items.

## CULTURE

I just can't say it enough.  Dabble in automation even if you don't have the
culture, but please don't try to put any of this into production until you have
the culture.  Put the automation stuff in a lab environment to show your
teammates and managers so that you can help build that culture.  But don't do it
live.  Not without the culture in place.  Not without management buy-in.

# Workflow

<iframe width="420" height="315"
  src="https://www.youtube.com/embed/zqXbTKdcGsI"
  frameborder="0" allowfullscreen></iframe>

> ### Thank You!
> A serious thank you to [Doug Youd](https://twitter.com/cnidus) who put the
> above flowchart together for this post.  He put in a lot of time with me
> to make sure the flow made sense.  The flowchart would not have happened
> without him.

The workflow above might look more complicated than it actually is.  To
illustrate how simple it really is, I asked Doug Youd to run through this
workflow--a way of working he had never experienced before.  Here's what he had
to say:

> ### Skype Chat
> [3/1/15, 12:28:03 AM] cnidus101: UI sucks. is my first impression <br />
> [3/1/15, 12:28:11 AM] cnidus101: its not intuitive <br />
> [3/1/15, 12:28:29 AM] cnidus101: as to what its really doing, whats the next step, breadcrumbs etc <br />
> [3/1/15, 12:28:49 AM] cnidus101: like w/o guidance, where do I click? <br />
> [3/1/15, 12:28:52 AM] cnidus101: whats it doing <br />
> [3/1/15, 12:28:54 AM] cnidus101: etc <br />
> [3/1/15, 12:29:27 AM] Tyler Christiansen: So does that comment refer to just Gerrit, or Gerrit and Jenkins? <br />
> [3/1/15, 12:29:39 AM] cnidus101: hover: "this commits the change, then go <here> to see results" <br />
> [3/1/15, 12:30:17 AM] cnidus101: kinda both I guess <br />
> [3/1/15, 12:30:48 AM] cnidus101: its probably just a matter of training, as you say, but Im used to vSphere / etc <br />
> [3/1/15, 12:31:01 AM] cnidus101: where the breadcrumbs are not too bad <br />
> [3/1/15, 12:31:08 AM] cnidus101: still a little obtuse <br />
> [3/1/15, 12:31:57 AM] cnidus101: process is simple enough <br />
> [3/1/15, 12:32:22 AM] cnidus101: with hovers + breadcrumbs and a decent explanation, any (drunken) idiot can  follow it <br />

Don't take my word for it, though.  Let's go through everything step-by-step
here!

## Ansible

[Ansible](http://ansible.com/) is the engine that generates configurations and
deploys them.  You should have a basic familiarity with it--how to install it,
how to use its template module, how to create playbooks, and how to run
playbooks.  You don't really need to understand all of its intricacies--you
don't even need to understand roles to get started!  However, I strongly
encourage you to learn more about it as your automation tool of choice (or
whatever automation tool you ultimately decide to use).  Oh, and look into the
`--check` (dry run).  It will help you with your testing.

Ansible is where the workflow begins.  You need to create your Ansible
automation, and you need to place it in git.

## Git

First, you should have basic familiarity with git.  For this workflow, you
should have a vague understanding of:

* adding files to be tracked by git
* commiting changes
* pushing to remote repositories
* creating branches
* cloning repositories
* fetching the latest changes (and merging them)

Not much at all.  Throughout my workflow, I use about 7 `git` commands: `add`,
`fetch`, `merge`, `checkout`, `add`, `commit`, `review`.  If you're so
inclined, you can actually shorten this to: `commit`, `pull`, `review`,
`checkout`.  You can do this because some of those are shortcuts or offer
shortcut options.  I like using the individual commands, though.

So, what do you do with git?  Well, all of your Ansible stuff (hosts file,
roles, playbooks, etc.) and all of your tests and building tools should be in
git.  You can learn how to use git "good enough"
[here](https://try.github.io/levels/1/challenges/1).

Git is step two.  You place your Ansible automation into git for revision
control.  It's sometimes called source control as well.  When you make a change
to a file, you add it to the local git repository, and when you've finished
making your changes, you commit them to the repository.  Confused?  That's okay.
"Adding" files to your git repository actually doesn't add them to the
repository.  It stages them, telling git to track those files.  Committing the
tracked files is what actually adds them to your local repository.  Once you've
committed them locally, it's time to push them up to Gerrit for integration
testing and peer review.

## Gerrit

Gerrit and Git by themselves are great tools.  They can help bring change
control to your organization even if you don't pursue automation.  Gerrit is
the change control engine of your automation workflow.  It is the gatekeeper.
You can set very flexible permissions on different repositories.  For my
workflow, though, I need a few basic things:

* A "Non-Interactive User" runs integration tests and "verifies" (+1) the change
* A user other than the committer "reviews" (+2) the change
* The committer merges (submits) the change to master (production)

This is pretty simple, but it can be customized to suit your requirements.  You
can have a NOC that must verify changes, or engineers must verify and architects
must review, or any combination of anything.  Ultimately, though, all changes
must have a +1 and a +2.  This isn't math or cumulative in any way.  +1 and +2
are essentially flags.  If you get +1 from three different people, it is _not_
the equivalent of +1 and +2.  There is also a -1 and -2, used when changes fail
verification and/or review.

So when you push your change to Gerrit (using `git review` [a plugin]), it also
notifies Jenkins.  Jenkins does some basic tests on the change--linting YAML,
linting XML, performing any other static analysis.  It can even spin up an
on-demand test/QA environment using something like GNS3 or Juniper vSRX Vagrant
boxes and then deploy to the QA environment and perform tests on it.

Once Jenkins runs its tests, it will either give a +1 or a -1 to the change,
based on the results of the tests.  At this point, someone else should review
the change and then give it +/- 2, as appropriate.  The committer can then
submit the change (if it has +1 and +2), or he can go back to the drawing board,
fix whatever was wrong, and resubmit the change.  The process then repeats.

Once the change is submitted, it gets merged back to master and goes to
production.  How you handle the push to production is up to you, but I would
personally use Jenkins for that as well.

## Jenkins

Jenkins is a Continuous Integration service.  It performs tests that you define
based on events that happen.  One of the things it does in this workflow is
static analysis of the change.  Another it does is deploy the changes every 30
minutes, whether there's a change on the master (production) branch or not.
This makes Jenkins a compliance engine in addition to an automation, testing,
and deployment engine.

Jenkins is highly configurable.  You can see the trends of the jobs over time,
and you can see pretty weather icons on its dashboard.  It has many, many
plugins, but for this workflow I use only `gerrit trigger`, `git`, `ansicolor`,
and `rvm`.

As mentioned above, Jenkins will perform tests on changes and give +/- 1 when
appropriate.  It will push the configuration every 30 minutes, although this
sort of thing is configurable.  It can be once/day, once/week, manual only,
triggered by a merge to master, or just about anything else.  It's very
flexible.

## Emergencies

This question came up several times: "what if there's an emergency?"  In those
cases, this workflow is going to be a hindrance.  How can you adjust it?  Well,
the static analysis is pretty quick, so you shouldn't be slowed down by the
requirement for a +1--and maintaining the +1 requirement also helps ensure
you're not making basic mistakes.  If you have a peer on-hand that can give you
+2 immediately, then you should be clear there as well.  Submit the change and
then it's just a matter of getting out to the right hosts.

To accomplish this, you have a few options:

* Wait for your current policy to push it to production (very slow)
* Manually trigger the deploy in Jenkins (slow)
* Manually trigger the deploy for only the affected host(s) by logging into the Jenkins host and running Ansible manually

The third option is the quickest and most advisable.  However, if the above
process(es) prove to be a hindrance still, here's another suggestion.

Make a manual change to the devices necessary.  Have a policy that requires that
change to be entered into Gerrit and go through the normal deploy policy after
the critical issue is resolved.  This can dangerous, though.  What if you're
using interval-based automatic deploys, an engineer forgets to enter the change
(or it's not merged in time), and the manual change is overwritten?  Then the
outage starts again.  How can you fix this?  Make it a policy to have no
automatic deployments, but instead have a Jenkins job that will log into all of
the devices, get their configurations, and compare them against what your Gerrit
master branch says the configuration can be.  If there's a difference, an e-mail
could be sent to the engineer list (or a manager, senior engineer, or
architect).  Then the appropriate action can be taken by a human.

## Multi-Vendor

This works across any vendor.  You may need to be some extra work in, but it
works.  I have the same definition for OSPF on a Juniper Networks device as I do
on a Cumulus Networks device.  Here's what it looks like:

#### Juniper Definition

{% highlight yaml %}
---

loopback:
  ip: 1.2.3.4

ospf:
 areas:
   - name: 0.0.0.0
     interfaces:
       - name: ge-0/0/0.0
         type: p2p
{% endhighlight %}

#### Cumulus Definition

{% highlight yaml %}
---

loopback:
  ip: 1.2.3.4

ospf:
  areas:
    - name: 0.0.0.0
      interfaces:
        - name: swp2
          type: p2p
{% endhighlight %}

The only difference is the name of the interface.  The format and structure,
however, is identical.  This is _not_ because Cumulus and Juniper are similar.
They are very, very different.  And for the people (like me) who love Juniper,
I'll be the first to say that this isn't a bad thing.

The fact that Cumulus is a new, modern networking company is an awesome thing.
Because they're Linux first, they don't need special modules for interacting
with them.  Because the configuration for OSPF on Cumulus is just another Linux
file (instead of some proprietary shell), it's easy to template, transfer, and
put into effect.  It is so much easier, in fact, that (excluding the `.git` and
`tests` directories) there are 164 _less_ lines than in the equivalent Juniper
demo.  This translates to a major win for automating Cumulus--no external
dependencies, no complicated, convoluted mechanisms for deploying.

# Action!

I just want to briefly show this workflow in action.  It will be a bunch of
screenshots with a few brief words describing what's going on.

### Local Git

{% highlight bash %}
cumulus@wbench:~/enchilada$ git checkout -b bug/fix-ospf
Switched to a new branch 'bug/fix-ospf'
cumulus@wbench:~/enchilada$ vim host_vars/leaf1.yml
cumulus@wbench:~/enchilada$ git diff
diff --git a/host_vars/leaf1.yml b/host_vars/leaf1.yml
index eebaa08..826eb2e 100644
--- a/host_vars/leaf1.yml
+++ b/host_vars/leaf1.yml
@@ -7,5 +7,5 @@ ospf:
   areas:
     - name: 0.0.0.0
       interfaces:
-        - name: swp2
+        - name: swp1
           type: p2p
cumulus@wbench:~/enchilada$ git add host_vars/leaf1.yml
cumulus@wbench:~/enchilada$ git status
# On branch bug/fix-ospf
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#	modified:   host_vars/leaf1.yml
#
cumulus@wbench:~/enchilada$ git commit -m 'fix ospf interface misconfiguration'
[bug/fix-ospf 1fea508] fix ospf interface misconfiguration
 Committer: Customer <cumulus@wbench.lab.local>
Your name and email address were configured automatically based
on your username and hostname. Please check that they are accurate.
You can suppress this message by setting them explicitly:

    git config --global user.name "Your Name"
    git config --global user.email you@example.com

After doing this, you may fix the identity used for this commit with:

    git commit --amend --reset-author

 1 file changed, 1 insertion(+), 1 deletion(-)
cumulus@wbench:~/enchilada$ git review
remote: Resolving deltas: 100% (2/2)
remote: Processing changes: new: 1, refs: 1, done
remote:
remote: New Changes:
remote:   http://100.64.66.2:8088/5 fix ospf interface misconfiguration
remote:
To ssh://supertylerc@localhost:29418/enchilada.git
 * [new branch]      HEAD -> refs/for/master/bug/fix-ospf
cumulus@wbench:~/enchilada$
{% endhighlight %}

Lots of stuff happens here.  First, I start a new branch to fix the OSPF bug
that I introduced previously.  Then, I edit `host_vars/leaf1.yml` and change the
interface.  I show a diff (so you can see the difference between what it was
before and what it is now).  I show the status of my local repository, and then
I tell git to track the file (`git add ...`).  Next, I commit my changes to my
local repository.  Finally, I push the change up to Gerrit for review (`git
review`).

### Gerrit

![Gerrit Change]({{ site.url }}/assets/gerrit.png)

Above is the change pending.  Notice the checkmark--the Jenkins integration test
has already run successfully.

![Gerrit Change Details]({{ site.url }}/assets/gerrit-details.png)

Above, you can see the details and the "+2" button.

![Gerrit Submit]({{ site.url }}/assets/gerrit-submit.png)

Here, you see the submit button.  Clicking it merges the change to master.

### Jenkins

![Jenkins View]({{ site.url }}/assets/jenkins.png)

Here's a basic dashboard for Jenkins.  It shows the current build status.

![Jenkins Test Output]({{ site.url }}/assets/jenkins-test.png)

This shows the output from the Jenkins integration test, which is run prior to
giving +/- 1 to the Gerrit change.

![Jenkins Deploy Output]({{ site.url }}/assets/jenkins-deploy.png)

This shows a sample of the output from an actual deploy initiated by Jenkins.

![Jenkins Trending]({{ site.url }}/assets/jenkins-trending.png)

This shows some trending data available through Jenkins.

# Disclaimers

The lab was sponsored by [Cumulus Networks](http://cumulusnetworks.com).  There
was no mandate for a positive review, but they deserve one anyway.  I did not
receive any compensation for this post.  I did collaborate with
[Doug Youd](https://twitter.com/cnidus) to make this post happen.  He's a
Cumulus employee, and his help was invaluable in understanding the platform
and configuration.  This post would not have happened without Doug and Cumulus.

I also bounced several questions off of
[Matt Stone](https://twitter.com/bigmstone) relating to managing Quagga
effectively.  Although I discovered that I couldn't do some things I wanted to
do, it turned out okay.  I would really prefer that the configuration for
protocols not be unified, because then I can tell people that they don't need to
template the entire Quagga config.  But templating the entire Quagga config is
not too bad.

[Ed Henry](https://twitter.com/NetworkN3rd) reviewed this post prior to it
going live, for which I am eternally grateful.  Thanks, Ed!
