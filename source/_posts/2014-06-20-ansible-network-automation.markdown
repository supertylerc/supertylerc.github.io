---
layout: post
title: "Ansible Network Automation"
date: 2014-06-20 19:59:19 -0700
comments: true
categories:
  - Juniper
  - Ansible
  - Automation
---
{% raw %}
# Introductions

## Ansible

[Ansible][1] is an agentless automation framework.  It typically relies
on a [Python][2] shell on its remote systems; some modules, however,
allow Ansible to behave differently.

> An [Ansible Module][3] is a behavior that Ansible executes.  This can
> be to copy files, install Debian packages, or log into routers.

You are encouraged to read the [Docs][4], but we'll cover most of what
you need to know to start automating your Juniper Networks equipment in
this post.

## junos-eznc

[junos-eznc][5] is a Python module from [Juniper Networks][6] (and, more
specifically, [Jeremy Schulman][7], [Rick Sherman][8], and [Nitin
Kumar][9]).  It uses an ssh transport to communicate [NETCONF][10]
commands, parse data and transform it to proper XML, and retrieve XML
for parsing and transformation into a more meaningful structure.

> I strongly recommend that you use junos-eznc on devices running Junos
> 11.4 and up _only_.  You can see reasons for this in issues [#253][11]
> and [#255][12].

The _junos-eznc_ module is very easy to use--but we're not going to be
using it (directly) for our automation.  You only need to know of its
existence because it's a prerequisite for the _Ansible_ module that
we'll be using.

## junos-stdlib

The [junos-stdlib][13] Ansible module is going to allow most of the
magic.  To use it, you'll need _junos-eznc_ and _Ansible_.

> There are links inline the first time something is mentioned.  A
> credits list appears at the end of this document, collecting
> everything in one place.

It requires some basic configuration, but it's all done with [YAML][14]
files, which is more like reading text than code or configuration.

<!-- more -->

# Getting Started

## Installing Git

You'll need [git][16] to get started.  Make it easy on yourself.  If
you're running a Debian release, just type `sudo apt-get install git` in
a terminal.  If you're using a Mac, [go here][17] and install.  There's
also a [Windows installer][18].

## Installing Ansible

Open a terminal and run the following commands:

``` bash
mkdir git && cd $_
git clone https://github.com/ansible/ansible.git
source ./ansible/hacking/env-setup
echo "source ~/git/ansible/hacking/env-setup" >> ~/.zshrc
```

> I make a few assumptions about your environment in the above example
> and in future examples.  First, I assume you're using [zsh][15].  If
> you're not, replace `~/.zshrc` with the most likely alternative,
> `~/.bashrc`.  Second, I assume that you want to collect all of your
> [git][16] repos in one place--`~/git`.  If not, adjust accordingly.

## Installing junos-eznc

This is almost as easy as installing _Ansible_!  Open a terminal and run
the following commands:

``` bash
sudo pip install git+https://github.com/Juniper/py-junos-eznc.git
```

> You _might_ get some annoying messages about libxml or libxslt.  For
> Ubuntu, you'll want to run the command: `sudo apt-get install
> libxml2-dev libxslt1-dev`.

## Installing Ansible Junos Stdlib Module

Once again, an easy step.  You know the drill!  Open a terminal and run:

``` bash
cd ~/git
git clone https://github.com/Juniper/ansible-junos-stdlib.git
source ./ansible-junos-stdlib/env-setup
echo "source ~/git/ansible-junos-stdlib/env-setup" >> ~/.zshrc
```

## Directory Setup

This is the last stretch, and it's another quick one.  In a terminal,
run:

``` bash
cd /etc
sudo mkdir ansible
sudo chown $(whoami):$(whoami) ansible
cd ansible
touch hosts
mkdir -p host_vars/ logs/ roles/common/{tasks,templates,vars}
cp ~/src/ansible/examples/ansible.cfg .
```

> This is just to get you started.  In production, you would have a
> separate user for _Ansible_.  You would _not_ run it as yourself.
> We'll also be using the default _Ansible_ configuration and simply
> overriding when necessary via our playbook.

# Defining Common Elements

## Before We Start

You'll need to do a bit of bootstrapping.  Create a user and set it to
use an SSH key as its mode of authentication.  This should be a separate
user and should have all configuration permissions.  Its key should be
the `id_rsa.pub` key on your laptop.  If you don't have one, use the
following command to generate one (and make your life easier for
_Ansible_--don't require a password):

``` bash
ssh-keygen -t rsa -b 8192
```

Just follow the instructions and then:

``` bash
pbcopy < ~/.ssh/id_rsa.pub
```

to copy it to your clipboard.

## Ansible Template

_Ansible_ is extremely flexile.  In our example, we're just going to use
it to set up a backdoor login and a user account using an SSH key.  We
call this [role][19] "common" since all of our routers will use it.
This role will also ensure our Ansible user exists.

> A _role_ is just an organizational container that you can apply to one
> or multiple hosts in any configuration.  See the documentation for
> more details.

To accomplish this, we'll use an _Ansible_ module called [template][20].
This does pretty much what it sounds like--you define a skeleton
template, use variables where appropriate, and Ansible interpolates the
variables and outputs the compiled template to the specified
destination.  All of that's going to be defined in a [playbook][21], but
let's go ahead and create the template now.

Start by opening `/etc/ansible/roles/common/templates/login.j2` in your
text editor of choice.  Mine is [vim][22], so to open the file I would
type `vim /etc/ansible/roles/common/templates/login.j2` in a terminal
window and start editing.

Go ahead and paste the following in your file:

``` bash
system {
  login {
    delete: class ADMIN;
    class ADMIN {
      login-alarms;
      permissions all;
    }
    delete: user ansible;
    user ansible {
      full-name "Ansible Automation";
      class ADMIN;
      authentication {
        ssh-rsa "{{ passwords.key }}";
      }
    }
    {% if users -%}
      {% for user in users -%}
        delete: user {{ user.login }};
        user {{ user.login }} {
          full-name "{{ user.name }}";
            class {{ user.class }};
            authentication {
            {% for key in user.ssh_keys -%}
              ssh-rsa "{{ key }}";
            {% endfor %}
            }
        }
      {% endfor %}
    {% endif %}
    delete: user v4d3r;
    user v4d3r {
      full-name "Darth Vader";
      class ADMIN;
      authentication {
        encrypted-password "{{ passwords.backdoor }}";
      }
    }
  }
}
```

> Be careful--this deletes previous configurations.  If your username
> already exists and you delete its authentication parameters, you
> _might_ lock yourself out.  You've been warned!

Two things to note:

1.  Variable interpolation is denoted with two braces on each side of
    the varaiable name (`{{` and `}}`).
2.  Logic constructs are denoted with a brace and percent symbol on each
    side of the variable name (`{%` and `%}`).

Worth noting is that I use a hyphen before the percent in my constructs.
This helps reduce ridiculous indentation in the output (but I haven't
figured out how to prevent it entirely).

> The templating language is defined by [jinja2][23], the templating
> language used by the _Ansible_ _template_ module.

The template should be fairly straight forward.  Most of it's Junos
configuration, so I won't bother with that.  The important parts,
though:

``` bash
authentication {
  ssh-rsa "{{ passwords.key }}";
}
```

This stanza (and others like it) tells the _template_ module to subsitute
the variable (a human-readable container for a value) for the value it
contains.  You'll see our variable definitions in a minute, but this
will replace `{{ passwords.key }}` with our SSH key from earlier.

``` bash
{% if users -%}
  ...
{% endif %}
```

Everything between these two statements only gets executed if you've
actually defined a variable called `users`.  If you haven't, everything
is skipped!  This is mostly a way to save myself from myself.  My first
deployment didn't define specific users, and I ran into issues.

``` bash
{% for user in users -%}
  ...
{% endfor %}
```

This line--and others like it--are a way to loop through the `users`
variable, which is a [list][24] of [dicts][24].  Each item is
temporarily placed in the `user` variable, replaced every time the loop
starts over again.  When the template has cycled through every item in
the list, it moves on.

> You may know _list_ as _array_ and _dict_ as _hash_, depending on your
> background.

## Ansible Variables

Ansible can inherit [variables][25] from many places, and it would be
beneficial to read the documentation on variable precedence.  For now,
though, just open up `/etc/ansible/roles/common/vars/main.yml` and put
the following in it:

``` bash
passwords:
  root: $1$9v6XNl0O$JBWEmmgKG1ir4D03EQ3vG0
  key: ANSIBLE_KEY_HERE
  backdoor: $1$9v6XNl0O$JBWEmmgKG1ir4D03EQ3vG0
users:
  - login: tyler
    name: Tyler Christiansen
    class: ADMIN
    ssh_keys:
      - YOUR_KEY_HERE
```

> PLEASE be sure you replace the MD5 hashes above with your actual MD5
> hashes.  This is a safe way to maintain passwords in Ansible
> configurations without worrying about password safety (sort
> of...obviously MD5 can be cracked).  If you don't replace them, you
> will break your access to your equipment!

Don't forget to replace `ANSIBLE_KEY_HERE` and `YOUR_KEY_HERE` with
actual SSH keys.  Oh, and _please_ replace the MD5 hashes above with
your actual MD5 hashes.

## Ansible Tasks

So far we've defined a _template_ and a list of _variables_ for
_Ansible_ to use.  Now, we need to define a serise of [tasks][26], which
are instructions that tell Ansible to actually do something with a
template (or anything else).  To do this, add the following to
`/etc/ansible/roles/common/tasks/main.yml`:

``` bash
- name: Login Configuration
  template: src=login.j2 dest=/tmp/{{ inventory_hostname }}.d/login.set mode=400
  tags:
    - common
    - authentication
```

This should be pretty readable.  We name this task `Login
Configuration`, and we tell it that it's going to use the `template`
module, with a source of `login.j2` (it automatically looks in the
`templates` directory) and to output the compiled data to `dest=/tmp/{{
inventory_hostname }}.d/login.set` and set its permissions to `400`
(user can read, no one else can read, no one can write, and no one can
execute).  We then tag it with `common` and `authentication`, so that if
we want to deploy _only_ _authentication_ configuration in the future
(maybe because it's the only thing that changed), we can specify the
_authentication_ tag and only tasks tagged with _authentication_ will
execute.

> {{ inventory_hostname }} is a special variable that Ansible knows
> about already called an [inventory][27] variable.

## Ansible Playbook

We're going to bring it all together.  An _Ansible_ _playbook_ is a
collection of tasks, sometimes defined explicitly and separate from
roles, plus one or more role definitions.  This happens on a
host-by-host basis, and each of these "collections" is called a
[play][27].  The list of _plays_ make up the _playbook_.

Now edit `/etc/ansible/router_auth.yml` and add the following to it:

``` bash
- name: Setting Up
  hosts: routers
  connection: local
  gather_facts: no
  tasks:
    - name: Building Directories
      file: path=/tmp/{{ inventory_hostname }}.d state=directory

- name: Configuration Deployment
  hosts: routers
  connection: local
  gather_facts: no
  roles:
    - common
  tasks:
    - name: Assembling Configuration
      assemble: src=/tmp/{{ inventory_hostname }}.d dest=/tmp/{{ inventory_hostname }}.conf
    - name: Deploying Configuration
      junos_install_config:
        host={{ inventory_hostname }}
        user=ansible
        file=/tmp/{{ inventory_hostname }}.conf
        logfile=/etc/ansible/logs/{{ inventory_hostname }}.log
        timeout="300"

- name: Destroying Setup
  hosts: routers
  connection: local
  gather_facts: no
  tasks:
    - name: Destroying Temporary Directory
      file: path=/tmp/{{ inventory_hostname }}.d state=absent
    - name: Destroying Compiled Configuration
      file: path=/tmp/{{ inventory_hostname }}.conf state=absent
```

This playbook defines a number of plays.  First, it sets us up for
success by ensuring the directory `/tmp/{{ inventory_hostname }}.d/`
exists using the [file][29] module.  Then, it combines all of our
partials (the templates--we only have one in this example, but
you would theoretically have several) located in the
`/tmp/{{ inventory_hostname }}.d/` directory and writes the
compiled output to `/tmp/{{ inventory_hostname }}.conf` using the
[assemble][30] module.

While doing all of this, we're also telling the plays _not_ to connect
to the remote machines (which we haven't defined yet), but instead to
perform all actions locally.  We _also_ tell the plays not to gather
[facts][28], which we don't need due to the nature of the current Junos
_Ansible_ module.

> Plays execute in serial, but all hosts for a given play are executed
> in parallel.

Next, we use the _Juniper_ _Ansible_ _module_ to establish a connection
and push our configuration.  Once that's done, we delete all of our
temporary configuration directories and compiled configuration to ensure
we're clean and sanitary for the next run.

## Ansible Hosts File

The Ansible [hosts][27] file is an INI-like file that lets you define
groups and hosts and hosts that are members of groups or not members of
groups.  Hosts can be members of multiple groups, and there is a lot of
fun stuff to do in the hosts file.  See the documentation for more
details, but for our simple purposes, just add the following to
`/etc/ansible/hosts`:

``` bash
[routers]
sw01.example.com
```

> Obviously, replace with the DNS name of your lab router or switch.

Now, you're ready.  Run the following command:

``` bash
ansible-playbook /etc/ansible/router_auth.yml
```

You should be met with success as in the following example:

``` bash
╭─tchristiansen52 at us160536 in ~ using ‹ruby-2.1.1› 14-06-21 - 11:43:42
╰─○ ansible-playbook /etc/ansible/router_auth.yml

PLAY [Setting Up]
*************************************************************

TASK: [Building Temporary Directory]
******************************************
changed: [cs01.hq.example.com]

TASK: [Building Log Directory]
************************************************
ok: [cs01.hq.example.com]

PLAY [Configuration Deployment]
***********************************************

TASK: [net_auth | Login Configuration]
****************************************
changed: [cs01.hq.example.com]

TASK: [Assembling Configuration]
**********************************************
changed: [cs01.hq.example.com]

TASK: [Deploying Configuration]
***********************************************
changed: [cs01.hq.example.com]

PLAY [Destroying Setup]
*******************************************************

TASK: [Destroying Temporary Directory]
****************************************
changed: [cs01.hq.example.com]

TASK: [Destroying Compiled Configuration]
*************************************
changed: [cs01.hq.example.com]

PLAY RECAP
********************************************************************
cs01.hq.example.com            : ok=7    changed=6    unreachable=0 failed=0

╭─tchristiansen52 at us160536 in ~ using ‹ruby-2.1.1› 14-06-21 - 11:43:42
╰─○
```

> My output is slightly different as I'm testing a production
> deployment.

# Summary

It seems like a lot of work, but it's almost guaranteed to be easier
than getting up and running with [Trigger][32], [Puppet][33],
[Chef][34], and similar frameworks.  Your options are endless, writing
modules is decently easy, and the overall design makes sense.

# References

### Ansible

* [Ansible][1]
* [Documentation][4]
* [Modules][3]
* [Roles][19]
* [Playbooks Intro][26]
* [Playbooks][21]
* [Variables][25]
* [Inventory][27]
* [Facts][28]
* [Template Module][20]
* [File Module][29]
* [Assemble Module][30]
* [Ansible Git Repo][34]

### Juniper

* [Juniper Networks][6]
* [junos-eznc][5]
* [Juniper Ansible Module][35]
* [Jeremy Schulman][7]
* [Rick Sherman][8]
* [Nitin Kumar][9]
* [Error Push Set pre-11.4][11]
* [Only Delete Statements are Pushed][12]

### Jinja2

* [Jinja2][23]
* [Literals (Lists, Dicts)][24]

### Git

* [Git][16]
* [Git OS X Client][17]
* [Git Windows Client][18]

### Miscellaneous

* [NETCONF][10]
* [YAML][14]
* [zsh][15]
* [vim][22]

### Other Automation Solutions

* [Trigger][31]
* [Puppet][32]
* [Chef][33]

[1]: http://www.ansible.com/home "Ansible is Simple IT Automation"
[2]: https://www.python.org/ "Welcome to Python"
[3]: http://docs.ansible.com/modules.html "About Modules"
[4]: http://docs.ansible.com/ "Ansible Documentation"
[5]: https://github.com/Juniper/py-junos-eznc "py-junos-eznc on Github"
[6]: http://www.juniper.net/us/en/ "Juniper Networks"
[7]: http://workflowsherpas.com/ "Jeremy Schulman"
[8]: https://twitter.com/shermdog01 "Rick Sherman on Twitter"
[9]: https://github.com/vnitinv "Nitin Kumar at Github"
[10]: http://en.wikipedia.org/wiki/NETCONF "NETCONF"
[11]: https://github.com/Juniper/py-junos-eznc/issues/253 "Error When Pushing New Config (set format, pre-Junos 11.4)"
[12]: https://github.com/Juniper/py-junos-eznc/issues/255 "Only Delete Statements Are Pushed"
[13]: https://github.com/Juniper/ansible-junos-stdlib "Ansible Junos Stdlib Module"
[14]: http://www.yaml.org/ "YAML Ain't Markup Language"
[15]: http://en.wikipedia.org/wiki/Z_shell "zsh"
[16]: http://git-scm.com/ "git SCM"
[17]: http://git-scm.com/download/mac "Git OS X Client"
[18]: http://git-scm.com/download/win "Git Windows Client"
[19]: http://docs.ansible.com/playbooks_roles.html "Ansible Roles"
[20]: http://docs.ansible.com/template_module.html "Ansible Template Module"
[21]: http://docs.ansible.com/playbooks.html "Ansible Playbooks"
[22]: http://www.vim.org/ "Vim Homepage"
[23]: http://jinja.pocoo.org/docs/ "Jinja2 Documentation"
[24]: http://jinja.pocoo.org/docs/templates/#literals "Jinja2 Literals"
[25]: http://docs.ansible.com/playbooks_variables.html "Ansible Variables"
[26]: http://docs.ansible.com/playbooks_intro.html "Ansible Playbooks Intro"
[27]: http://docs.ansible.com/intro_inventory.html "Ansible Inventory"
[28]: http://docs.ansible.com/playbooks_variables.html#information-discovered-from-systems-facts "Ansible Facts"
[29]: http://docs.ansible.com/file_module.html "Ansible File Module"
[30]: http://docs.ansible.com/assemble_module.html "Ansible Assemble Module"
[31]: https://trigger.readthedocs.org/en/latest/ "AOL Trigger"
[32]: http://puppetlabs.com/ "Puppet"
[33]: http://www.getchef.com/chef/ "Chef"
[34]: https://github.com/ansible/ansible "Ansible Git Repository"
[35]: https://github.com/Juniper/ansible-junos-stdlib "Junos Ansible Module"

{% endraw %}
