---
layout: post
title: "Git, Gitweb, and RANCID: Automated Installation"
description: ""
category: ansible
tags: [ ansible, automation, rancid, 30in30 ]
---

# rancid-git

[rancid-git][2] is a patched version of [RANCID][3] that allows you to easily
use [`git`][4] as the [RCS][5] for RANCID.

Installing `rancid-git` is a potentially daunting task.  You can either build
a `deb` package and hope it installs correctly, or you can install from source.
If you're using a Red Hat-based distribution, building the `rpm` is completely
untested.  Besides that, you need to do it on at least two servers, and I'm not
a fan of doing anything more than once manually.

# ansible-role-rancid-git

[Ansible][6] is my preferred framework for automation.  I wrote an Ansible role
that installs `rancid-git` and sets up [Gitweb][7] for me, using `nginx` as the
proxy.

You'll need to clone [the repository][8] and place the `roles/rancid-git`
folder in your `/etc/ansible/roles/` directory.

Next, you'll want to update the variables in
`/etc/ansible/roles/rancid-git/vars/main.yml` to match your environment.

Once you've done that, you can just add the `rancid-git` role to a playbook.
If you don't have one or if you're just getting started with Ansible, here's
an example:

{% highlight yaml %}
---

- name: Install RANCID
  hosts: rancid.example.com
  sudo: yes
  roles:
    - rancid-git
{% endhighlight %}

## TODO

Obviously, my role is missing a few things.  The Red Hat-specific packages
aren't there, and some of the paths should be modified for Red Hat
installations.  I welcome the pull requests.

I'd also like to provide a way to specify either `nginx` or `apache` as the web
server, increasing the flexibility.

Certain files aren't currently managed by Ansible (namely the `router.db`
files).  I need to get these to be managed by the Ansible role as well.

There's also a way to allow roles to be installed via `ansible-galaxy`.  I need
to get around to adapting for that as well.

# Next Steps

This Ansible role will set you up the RANCID, but you'll still need to do some
of the other setup (updating the `router.db` files, `.cloginrc`,
`/etc/aliases`, etc).  You can go through a [basic walkthrough here][9].

# END!

Thanks for reading.  Please open issues on the GitHub page.  If you want to
contribute, please open pull requests.

This post is part of the [#30in30 challenge][1].

[1]: http://etherealmind.com/challenge-30-blogs-30-days/ "30 Blogs in 30 Days Challenge"
[2]: https://github.com/dotwaffle/rancid-git "rancid-git"
[3]: http://www.shrubbery.net/rancid/ "RANCID"
[4]: http://git-scm.com/ "Git"
[5]: http://en.wikipedia.org/wiki/Revision_Control_System "Revision Control System"
[6]: http://www.ansible.com/home "Ansible"
[7]: https://git.wiki.kernel.org/index.php/Gitweb "Gitweb"
[8]: https://github.com/supertylerc/ansible-role-rancid-git "ansible-role-rancid-git on Github"
[9]: http://www.linuxhomenetworking.com/wiki/index.php/Quick_HOWTO_:_Ch1_:_Network_Backups_With_Rancid#Initial_Rancid_Configuration "Rancid HOWTO"
