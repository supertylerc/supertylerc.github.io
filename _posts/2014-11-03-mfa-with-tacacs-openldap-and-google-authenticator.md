---
layout: post
title: "MFA with TACACS+, OpenLDAP, and Google Authenticator"
description: "Deploying a Network Element MFA Solution with Ansible"
category: ansible
tags: [ ansible, automation, security, mfa, tacacs, 30in30 ]
---

# Why

Multi-factor authentication is neat.  It adds a level of security, and we
should all be concerned about security--especially with regards to our critical
infrastructure elements.

## The Stack

[Shrubbery's][2] [`tac_plus`][3] daemon is the approach I decided to take.
I utilize its [PAM][4] capabilities to enable [MFA][5] with [OpenLDAP][7] and
[Google Authenticator][9].

# ansible-role-mfa-tacacs

[Ansible][6] is my preferred framework for automation.  I wrote an Ansible role
that installs `tac_plus` and `google-authenticator` for me because doing
anything more than once is pretty boring.

You'll need to clone [the repository][8] and place the `roles/tacacs`
folder in your `/etc/ansible/roles/` directory.

Next, you'll want to update the variables in
`/etc/ansible/roles/tacacs/vars/main.yml` to match your environment.

> Note: I use Ansible to control my `tac_plus` configuration file, which uses
> the variables in `/etc/ansible/roles/tacacs/vars/main.yml`.  I've only tested
> for Junos.  If you modify this for another vendor, _please_ open a pull
> request to make this role better!

Once you've done that, you can just add the `tacacs` role to a playbook.
If you don't have one or if you're just getting started with Ansible, here's
an example:

{% highlight yaml %}
---

- name: Install TACACS+
  hosts: tacacs.example.com
  sudo: yes
  roles:
    - tacacs
{% endhighlight %}

## Google Authenticator

You'll need to generate your Google Authenticator token.  Type
`google-authenticator` on the server you just installed it on.  I just answered
`yes` to everything, but feel free to configure it to your liking.  The output
should include instructions on how to set your phone up as well (usually a QR
code).  You can look [here][11], though, if it doesn't.

> Note: To increase redundancy, you should copy `$HOME/.google_authenticator`
> to a second server with `tac_plus` and `google-authenticator` installed.  It
> should be the same file with the same path (i.e.,
> `$HOME/.google_authenticator`).

## TODO

Obviously, my role is missing a few things.  The Red Hat-specific packages
aren't there, and some of the paths should be modified for Red Hat
installations.  I welcome the pull requests.

There's also a way to allow roles to be installed via `ansible-galaxy`.  I need
to get around to adapting for that as well.

# Next Steps

This Ansible role will set you up the TACACS+, but you'll still need to do some
of the other setup.  You can find a halfway decent guide to setting up
`tac_plus` on my legacy site [here][10].

> It will be migrated to this domain eventually.  Promise.

# END!

Thanks for reading.  Please open issues on the GitHub page.  If you want to
contribute, please open pull requests.

This post is part of the [#30in30 challenge][1].

[1]: http://etherealmind.com/challenge-30-blogs-30-days/ "30 Blogs in 30 Days Challenge"
[2]: http://shrubbery.net/ "Shrubbery Networks"
[3]: http://shrubbery.net/tac_plus/ "TACACS+ Daemon"
[4]: http://en.wikipedia.org/wiki/Linux_PAM "PAM"
[5]: http://en.wikipedia.org/wiki/Multi-factor_authentication "MFA"
[6]: http://www.ansible.com/home "Ansible"
[7]: http://www.openldap.org/ "OpenLDAP"
[8]: https://github.com/supertylerc/ansible-role-mfa-tacacs "ansible-role-mfa-tacacs on Github"
[9]: http://en.wikipedia.org/wiki/Google_Authenticator "Google Authenticator"
[10]: https://oss-stack.io/blog/categories/tacacs/ "TACACS+ Posts"
[11]: https://code.google.com/p/google-authenticator/wiki/PamModuleInstructions "Google Authenticator PAM Setup Instructions"
