---
layout: post
title: "Crash Course - Linux"
date: 2014-01-22 21:19:46 -0700
comments: true
categories:
  - linux
  - crash course
---

**Linux** is a kernel and often a misnomer for an operating system that
makes the life of engineers signifcantly easier.

## The Relevance of Linux <small>The foundation of OSS</small>

Linux plays a significant role in OSS.  That's not to say that it is the
only show in town--in fact, FreeBSD may have been even more pivotal in
OSS.  Linux seems to have merely led to wider-spread adoption.

Many of the utilities in the OSS world were designed on and/or for Linux
or FreeBSD.  These utilities include [LibreNMS][1], [Cacti][2],
[RANCID][3], [TACACS+][4], and many more.

In addition, it hosts tools such as `perl`, `sed`, `grep`, and many
others that can make filtering through information much easier.
Languages such as [Python][5] and [Perl][6] make writing administrative
scripts that automate changes or collect information a breeze.

<!-- more -->

## Crash Course <small>The way of the terminal</small>

The terminal will be your friend.  This article isn't here to teach you
the history of Linux or how to use `ssh`.  I assume that you know how to
SSH to a Linux server or how to open a terminal if you have GUI access
to a Linux system.

> If you're using Ubuntu 12.04 (and presumably later and possibly
> earlier), you can press `ctrl+alt+T` to open a terminal

Now that you have your terminal open, here are a few commands that will
help you out on your journey to becoming an OSS ninja.

> Before we begin, you should know that a single "**dot**" or
> "**period**" (`.`) symbolically represents the current directory you
> are in, while two consecutive "**dots**" or "**periods**" (`..`)
> represents the parent directory, or the directory above your current
> directory.

> A term in angled brackets (`<` and `>`) is optional.  A term in square
> brackets (`[` and `]`) is mandatory.

### ls <small>list files</small>

Format:
``` bash
ls <options> <directory>
```

> Options begin with a hyphen (`-`).

Example:
``` bash
vagrant@precise32:/opt$ ls
oss-conf  tac_plus  vagrant_ruby  VBoxGuestAdditions-4.2.0
vagrant@precise32:/opt$ ls /
bin   dev  home        lib         media  opt   root  sbin     srv  tmp
vagrant  vmlinuz
boot  etc  initrd.img  lost+found  mnt    proc  run   selinux  sys  usr
var
vagrant@precise32:/opt$ ls -R tac_plus/
tac_plus/:
bin  etc  include  lib  share

tac_plus/bin:
tac_plus  tac_pwd

tac_plus/etc:
tac_plus.conf

tac_plus/include:
tacacs.h

tac_plus/lib:
libtacacs.a  libtacacs.la  libtacacs.so  libtacacs.so.1
libtacacs.so.1.0.0

tac_plus/share:
man  tacacs+

tac_plus/share/man:
man5  man8

tac_plus/share/man/man5:
tac_plus.conf.5

tac_plus/share/man/man8:
tac_plus.8  tac_pwd.8

tac_plus/share/tacacs+:
do_auth.py  tac_convert  users_guide
vagrant@precise32:/opt$
```

> If you don't specify a directory, the default is to list the contents
> of the current working directory (commonly represented as a single dot
> `.`).

### cd <small>change directory</small>

Format:
``` bash
cd <options> <destination>
```

Examples:
``` bash
vagrant@precise32:/opt$ cd
vagrant@precise32:~$ cd /opt
vagrant@precise32:/opt$ cd
vagrant@precise32:~$ cd ..
vagrant@precise32:/home$ cd ~
vagrant@precise32:~$
```

> The tilde (`~`) is a shortcut meaning "**home directory**".  The `cd`
> command by itself with no arguments changes to your home directory.
> The two dots (`..`) represents the parent directory.

### cp <small>copy</small>

Format:
``` bash
cp <options> [source] [destination]
```

Examples:
``` bash
vagrant@precise32:~$ ls
postinstall.sh  test
vagrant@precise32:~$ cp /opt/tac_plus/etc/tac_plus.conf .
vagrant@precise32:~$ ls
postinstall.sh  tac_plus.conf  test
vagrant@precise32:~$ ls /opt
oss-conf  tac_plus  test  vagrant_ruby  VBoxGuestAdditions-4.2.0
vagrant@precise32:~$ cp test /opt
vagrant@precise32:~$ ls /opt
oss-conf  tac_plus  test  vagrant_ruby  VBoxGuestAdditions-4.2.0
vagrant@precise32:~$ cp -R /opt/tac_plus/ .
vagrant@precise32:~$ ls
postinstall.sh  tac_plus  tac_plus.conf  test
vagrant@precise32:~$ ls -R tac_plus
tac_plus:
bin  etc  include  lib  share

tac_plus/bin:
tac_plus  tac_pwd

tac_plus/etc:
tac_plus.conf

tac_plus/include:
tacacs.h

tac_plus/lib:
libtacacs.a  libtacacs.la  libtacacs.so  libtacacs.so.1
libtacacs.so.1.0.0

tac_plus/share:
man  tacacs+

tac_plus/share/man:
man5  man8

tac_plus/share/man/man5:
tac_plus.conf.5

tac_plus/share/man/man8:
tac_plus.8  tac_pwd.8

tac_plus/share/tacacs+:
do_auth.py  tac_convert  users_guide
vagrant@precise32:~$
```

> The `ls` command is used to show contents before and after a `cp`
> command.  The `-R` flag on the `cp` command means "**recursive**" and
> is necessary to copy entire folders.

### mv <small>move</small>

Format:
``` bash
mv <options> [source] [destination]
```

Examples:
``` bash
vagrant@precise32:~$ ls /tmp/
vagrant@precise32:~$ mv test /tmp/
vagrant@precise32:~$ mv tac_plus/ /tmp/
vagrant@precise32:~$ ls /tmp
tac_plus  test
vagrant@precise32:~$ ls
postinstall.sh  tac_plus.conf
vagrant@precise32:~$
```

> The `mv` command is very similar to the `cp` command, except that it
> does not require a recursive option to move folders, and when the file
> is moved, it no longer exists at the source location.

### rm <small>remove</small>

Format:
``` bash
rm <options> [file|directory] ...
```

Examples:
``` bash
vagrant@precise32:~$ ls /tmp
tac_plus.conf  test  test2  test3  test4
vagrant@precise32:~$ rm /tmp/tac_plus.conf
vagrant@precise32:~$ ls /tmp
test  test2  test3  test4
vagrant@precise32:~$ rm /tmp/test /tmp/test2
vagrant@precise32:~$ ls /tmp
test3  test4
vagrant@precise32:~$ rm /tmp/*
vagrant@precise32:~$ ls /tmp
vagrant@precise32:~$
vagrant@precise32:~$ ls -R /tmp/
/tmp/:
test

/tmp/test:
happy  hello  sample

/tmp/test/sample:
goodbye  sad
vagrant@precise32:~$ rm -r /tmp/test/
vagrant@precise32:~$ ls -R /tmp/
/tmp/:
vagrant@precise32:~$
```

> The `ls` command is used to illustrate the effects of the `rm`
> command.  The `-r` flag means recursive.  You can specify single
> files, multiple files, directories (but must specify the `-r` flag),
> or wildcards (`*`).

### rmdir <small>remove directory</small>

Format:
``` bash
rmdir <options> [directory] ...
```

Examples:
``` bash
vagrant@precise32:~$ ls -R /tmp
/tmp:
test

/tmp/test:
hello

/tmp/test/hello:
goodbye

/tmp/test/hello/goodbye:
friends

/tmp/test/hello/goodbye/friends:
vagrant@precise32:~$ rmdir /tmp/test/hello/goodbye/friends/
vagrant@precise32:~$ ls -R /tmp/
/tmp/:
test

/tmp/test:
hello

/tmp/test/hello:
goodbye

/tmp/test/hello/goodbye:
vagrant@precise32:~$ rmdir -p /tmp/test/hello/goodbye/
rmdir: failed to remove directory '/tmp': Permission denied
vagrant@precise32:~$ ls -R /tmp
/tmp:
vagrant@precise32:~$
```

> Be careful of the "**parents**" options (`-p`).  You can easily delete
> directories by mistake.

### mkdir <small>make directory</small>

Format:
``` bash
mkdir <options> [directory] ...
```

Examples:
``` bash
vagrant@precise32:~$ ls /tmp
vagrant@precise32:~$ mkdir /tmp/test
vagrant@precise32:~$ ls /tmp
test
vagrant@precise32:~$ mkdir -p /tmp/test/hello/friend/nice/to/meet/you
vagrant@precise32:~$ ls -R /tmp
/tmp:
test

/tmp/test:
hello

/tmp/test/hello:
friend

/tmp/test/hello/friend:
nice

/tmp/test/hello/friend/nice:
to

/tmp/test/hello/friend/nice/to:
meet

/tmp/test/hello/friend/nice/to/meet:
you

/tmp/test/hello/friend/nice/to/meet/you:
vagrant@precise32:~$ mkdir /tmp/sad /tmp/happy
vagrant@precise32:~$ ls -R /tmp
/tmp:
happy  sad  test

/tmp/happy:

/tmp/sad:

/tmp/test:
hello

/tmp/test/hello:
friend

/tmp/test/hello/friend:
nice

/tmp/test/hello/friend/nice:
to

/tmp/test/hello/friend/nice/to:
meet

/tmp/test/hello/friend/nice/to/meet:
you

/tmp/test/hello/friend/nice/to/meet/you:
vagrant@precise32:~$
```

### pwd <small>present working directory</small>

Format:
``` bash
pwd
```

Examples:
``` bash
vagrant@precise32:~$ pwd
/home/vagrant
vagrant@precise32:~$
```

### sudo <small>super-user do</small>

> `sudo` doesn't actually mean "**super-user do**".  It means "**su
> do**", but "**super-user do**" is frequently easier to remember when
> you're starting out.

Format:
``` bash
sudo [command]
```

Example:
``` bash
vagrant@precise32:~$ cd /usr
vagrant@precise32:/usr$ ls
bin  games  include  lib  local  sbin  share  src
vagrant@precise32:/usr$ touch test
touch: cannot touch 'test': Permission denied
vagrant@precise32:/usr$ sudo touch test
vagrant@precise32:/usr$ ls
bin  games  include  lib  local  sbin  share  src  test
vagrant@precise32:/usr$ ls -l test
-rw-r--r-- 1 root root 0 Jan 19 01:49 test
vagrant@precise32:/usr$ whoami
vagrant
vagrant@precise32:/usr$
```

> `sudo` is useful to run commands that require root privileges.  It can
> also be configured such that a user can perform certain actions with
> elevated privileges but not all.

> The above example uses numerous commands you've learned so far to help
> identify the effects of `sudo`.  The `-l` option for `ls` is for
> "**long**", which shows the user and group who owns the file.  In this
> case, it is owned by the "**root**" user and group, even though you
> are "**vagrant**" (or whatever users you happen to be) as identified
> by the `whoami` command.

> This shows one of the "gotchas" of `sudo`: it creates files as the
> `root` user.

### curl <small>client url request library</small>

Format:
``` bash
curl <options> [target] ...
```

Examples:
``` bash
vagrant@precise32:~$ ls
postinstall.sh
vagrant@precise32:~$ curl -O
http://serpentorslair.com/wp-content/uploads/2013/10/darth-vader-16086-1680x1050.jpg
  % Total    % Received % Xferd  Average Speed   Time    Time     Time   Current
                                 Dload  Upload   Total   Spent    Left   Speed
100  397k  100  397k    0     0   305k      0  0:00:01  0:00:01 --:--:-- 493k
vagrant@precise32:~$ ls
darth-vader-16086-1680x1050.jpg  postinstall.sh
vagrant@precise32:~$ curl -o vader.jpg
http://serpentorslair.com/wp-content/uploads/2013/10/darth-vader-16086-1680x1050.jpg
  % Total    % Received % Xferd  Average Speed   Time    Time     Time   Current
                                 Dload  Upload   Total   Spent    Left   Speed
100  397k  100  397k    0     0   276k      0  0:00:01  0:00:01 --:--:-- 484k
vagrant@precise32:~$ ls
darth-vader-16086-1680x1050.jpg  postinstall.sh  vader.jpg
vagrant@precise32:~$
```

### tar <small>tape archive</small>

Format:
``` bash
tar <options> [archive] <files|directories> ...
```

Examples:
``` bash
vagrant@precise32:~$ ls
hello  test1  test3  world
vagrant@precise32:~$ tar -czvf
.bash_history              hello                      test1
.viminfo
.bash_logout               .profile                   test3
world
.bashrc                    .ssh/                      .vbox_version
.cache/                    .sudo_as_admin_successful  .veewee_version
vagrant@precise32:~$ tar -czvf home_dir.tar.gz test1 hello test3 world
test1
hello
test3
world
vagrant@precise32:~$ ls
hello  home_dir.tar.gz  test1  test3  world
vagrant@precise32:~$ rm hello test1 test3 world
vagrant@precise32:~$ ls
home_dir.tar.gz
vagrant@precise32:~$ tar -tzvf home_dir.tar.gz
-rw-rw-r-- vagrant/vagrant   0 2014-01-19 02:26 test1
-rw-rw-r-- vagrant/vagrant   0 2014-01-19 02:26 hello
-rw-rw-r-- vagrant/vagrant   0 2014-01-19 02:26 test3
-rw-rw-r-- vagrant/vagrant   0 2014-01-19 02:26 world
vagrant@precise32:~$ ls
home_dir.tar.gz
vagrant@precise32:~$ tar -xzvf home_dir.tar.gz
test1
hello
test3
world
vagrant@precise32:~$ ls
hello  home_dir.tar.gz  test1  test3  world
vagrant@precise32:~$
```

> `tar` has a number of important options.  The "**verbose**" (`-v`)
> flag prints output to your screen, the "**file**" (`-f`) tells `tar`
> which file, and the "**gzip**" (`-z`) tells `tar` to use "**gzip**"
> compression.

> There are three action flags used as well.  They are:

> "**create**" (`-c`): create the archive

> "**extract**" (`-x`): extract the archive

> "**list**" (`-t`): list the contents of the archive

## vim <small>Editing Files</small>

`vim` is a power text-editing utility that can be difficult to master.
Using it for basic purposes, though, is easy.  Here's a short list of
commands:

* `i`: enter insert mode at the current curson position
* `a`: enter insert mode one character after the current cursor position
* `o`: enter insert mode at the next line
* `dd`: delete the current line
* `yy`: copy the current line
* `:`: enter command mode
* `:wq`: enter command mode and write and close the file
* `:w`: enter command mode and write the file
* `:q!`: enter command mode and close the file, discarding any unsaved
  changes
* `:%s/<old>/<new>/g`: enter command mode and use the subtitution
  command, replacing all instances of "**<old>**" with "**<new>**"
  (note: leave the angled brackets off!)
* `[escape]`: pressing the `[escape]` key aborts the currently entered
  partial command and/or exits command mode

> `vim` is a great tool, but it is difficult to show in a text-based
> tutorial how it's used (irony, huh?).  I strongly recommend [VIM
> Adventures][7] to learn how to use `vim` properly.

## The End

This was only a crash course on the very bare necessities you must
understand to use Linux.  More advanced tools, such as `sed`, `awk`,
`perl`, `cut`, and more require entire posts to explain properly.

This will get you on your way.  Look for future articles in the "**Crash
Course**" series.

[1]: https://github.com/librenms/librenms "LibreNMS"
[2]: http://www.cacti.net/ "Cacti"
[3]: http://www.shrubbery.net/rancid/ "RANCID"
[4]: http://www.shrubbery.net/tac_plus/ "TACACS+"
[5]: http://www.python.org/ "Python"
[6]: http://www.perl.org/ "Perl"
[7]: http://vim-adventures.com/ "VIM Adventures"
