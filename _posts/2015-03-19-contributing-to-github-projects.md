---
layout: post
title: "Contributing to GitHub Projects"
description: "How to manage pull requests in GitHub"
category: tech-thursday
tags: [git, open-source, tech-thursday]
---

This post is the first in a series called Tech Thursday.  Tech Thursday
posts are whirlwind tours of things that I find interesting.  Not enough information
or detail for the bi-weekly Monday posts, but not quite short enough for Friday's
Lessons Learned.

> You should already be familiar with the basics of git.  If you're not, you should
> [try git](https://try.github.io/).

This post covers the basics of how to properly contribute to GitHub projects without
polluting the upstream.

# Fork It!

First, find the button on a project that says "Fork."

![Fork Project]({{ site.url }}/assets/fork.png)

Click it!  This will create a "fork" under your name.  Forks do not automagically
merge commits to the upstream.  It's a manual process, and that's one of the
more important issues to address: how do you _correctly_ maintain your fork and
submit code?

# Clone It!

Now that you've forked the repository to which you wish to contribute, you need
to clone your fork.  This is pretty easy and can be done via the command line
on Linux and Mac OS X systems:
`git clone git@github.com:supertylerc/netmiko.git`.

> Obviously, that's an example.  You should change the username (`supertylerc`)
> to your GitHub username, and you should change the repository name
> (`netmiko.git`) to your repository name.

# Branch It!

Don't work in master.  Ever.  You shouldn't do it when you're working on
your own source repositories, but you especially shouldn't do it when
contributing to other people's projects.  It just gets messy.

Instead, create a branch.  I like to follow a specific format for naming my
branches.  They start with a prefix.  Here's a short list of some common
ones:

* **feat**: this is a new feature
* **doc**: this is for documentation-only changes
* **bug**: this is for fixing bugs

Follow that with a slash (`/`).  Next, you should reference a ticket number
(when available).  A best practice would be to _always_ have a ticket
number, but I will admit that I don't always follow that practice.  I
should, though, so from now on, I won't be working on anything without a
ticket first!

For GitHub issues, I would just use `gh123`, where `123` is the issue
number in GitHub.  If you're using JIRA, then it would stand to reason
that you should use the entire ticket name: `PROJ-1234`.  You get the
idea.

Follow the ticket number with another slash.  Then, write something
descriptive for your branch name.  Something like `correct-foobar-method`.
A full branch name might look something like this:

`bug/gh123/correct-foobar-method`

And you would create this branch (and check the branch out at the same
time) with the command:
`git checkout -b bug/gh123/correct-foobar-method`.

# Work It!

Now, do all of your work on that branch.  Add it, commit it, and push
it to your fork.  You can push multiple commits--and in fact you are
encouraged to do so on particularly large changes.  One massive commit
is much worse than several smaller commits.  Once that's done, you
should see your new branch in your GitHub repository (the one you
forked).

# Submit It!

You should also see a green button that looks like a counter-clockwise
circle.  A really bad circle.  I'm terrible at describing things, so
here's a picture.

![Submit Pull Request]({{ site.url }}/assets/pr.png)

# Keep Up, Upkeep?

You need to track your pull request in the upstream.  You should
automatically get notifications from GitHub about it, but you should
also be checking in with the upstream maintainers to see if they're
interested in your pull request.  They may not be--and it's okay if
they're not.  Don't let it discourage you.  Remember: the direction
they choose to take is up to them.

Sometimes, though, they may ask you to rewrite certain parts of your
pull request.  You don't need to do anything special--just continue
working on the same branch and submit additional commits to your
fork.

# Success!

You have just contributed to an open source project.  Congratulations!
You've made software better for all mankind.

You can now delete your branch.  Go back to your local machine and
switch to the master branch (`git checkout master`) and delete the
branch: `git branch -d feat/gh3/fix-all-the-things`.  You'll want to
delete it on GitHub as well:
`git push origin :feat/gh3/fix-all-the-things`.

> There's an alternate syntax you can use:
> `git push origin --delete feat/gh3/fix-all-the-things`

# Updating Your Fork

This one is pretty important.  Your fork will get behind the upstream
origin repository.  How do you fix this?  I've seen people submit pull
requests to do it, which is not really the right way to do it.  It
can even pollute the upstream with pull requests unnecessarily!  So
to sync your repository, you need to ensure you have a remote that
tracks the upstream:
`git remote add upstream https://github.com/railsconfig/rails_config.git`.

The next step is to fetch and mergethe upstream repository's
current master branch, then push the changes to your fork:

{% highlight bash %}
git checkout master
git fetch upstream
git merge upstream/master
git push origin master
{% endhighlight %}

If you _really_ wanted to, you could create a local branch, merge
`upstream/master` to it, and push it.  Then you could create a pull
request within your fork on GitHub and merge it.  That's way more
work than it's worth, though, particularly when you're just syncing
your fork to the upstream!

# Testing Pull Requests

Here's an interesting scenario: what if you're the project maintainer,
someone submits a pull request, and you want to test it or see how
it is visually different?

It's actually pretty easy.  You can do it on a one-off basis with the
command `git fetch origin pull/$PR_NUMBER/head:pr$PR_NUMBER`.  To
checkout the pull request, type `git checkout pr$PR_NUMBER`.  Replace
`$PR_NUMBER` with the number of the pull request in GitHub.  So, for
example, if you want to grab `#102` from the `rails_config`
repository, you would type `git fetch origin pull/102/head:pr102`.
To checkout the pull request (and make it part of your working tree),
just type `git checkout pr102`.

There's an easier way to do this, though, if you don't want to type
all of that in every time.  You could create a function in your
shell of choice, or you could just add a remote.  In the repository,
type `git config -e` and add the following lines:

{% highlight bash %}
[remote "pr"]
    url = https://github.com/railsconfig/rails_config.git
    fetch = pull/*/head:refs/remotes/pr/*
{% endhighlight %}

> The above example uses `rails_config` as it's example.  Just
> match the `url` parameter to your `origin` remote's `url`
> parameter.

Now, you can type `git remote update` and get the new pull
requests!  After that, it's as simple as `git fetch pr`.  Remember
to `git checkout pr$PR_NUMBER` to make a specific pull request
your working tree.

There's yet another trick that you can employ, but I prefer the
above method of having an additional remote.  If you don't like
that, though, you can add this line to your `origin` remote
section: `fetch = pull/*/head:refs/remotes/origin/pr/*`.
Don't replace the existing `fetch = ...` line; just add that
one.

# THE END

I hope this whirlwind tour of how to properly manage your GitHub
contribution experience was helpful.  You can reach out to me
via the Twitters if you have any questions or concerns.
