---
title: Bourne Again, uplifting the macOS bash shell
date: 2019-01-04
description: Upgrading macOS to a later revision of bash shell
categories:
  - macOS
  - term
  - bash
image: https://source.unsplash.com/Agx5_TLsIf4/2000x1322?a=.png
author_staff_member: oliver
---

Apple laptops and desktops have remained a popular choice for developers. Being based on the BSD kernel, Apple macOS is easily able to offer a familiar UNIX experience when using the command line. The macOS operating system has come with the bash shell for a number of years now, so why is this post even required?

This is a valid question to ask and you may be extremely comfortable with the stock version of bash that is installed by default with macOS. In the current macOS Mojave (10.14.5), the version of bash is:

{% highlight shell %}
$ /bin/bash --version
GNU bash, version 3.2.57(1)-release (x86_64-apple-darwin18)
Copyright (C) 2007 Free Software Foundation, Inc.
{% endhighlight %}

As you can see from the output, the version installed is GNU bash v.3.2.57(1)-release. As the copyright clearly states, this version of GNU bash dates back to 2007. As it's currently 2019, a lot can happen in 12 years, especially in computing and security patching.

The reasoning behind why GNU bash on macOS boils down to good old licensing. GNU bash v3.2 was released under the GNU General Public License v2 (GPLv2). The successor to GNU bash v3.2 is v4.0 and it was this release when GNU bash moved to the [GNU General Public License v3 (GPLv3)](https://www.gnu.org/licenses/gpl.html). Unfortunately for macOS users, Apple does not (currently) support the GPLv3 licensing terms, for whatever their reasons may be. Put another way, the version of GNU bash has remained at GNU bash 3.2 since Mac OS X 10.2 (Jaguar). So, now we know why macOS does not ship with any later releases of GNU bash. What can we do?

Luckily, upgrading is fairly straightforward and you can run multiple versions of GNU bash with no impact to an otherwise stock macOS installation. Later version of GNU bash for Darwin are available from [MacPorts](http://trac.macports.org/browser/trunk/dports/shells/bash), [Homebrew](http://brew.sh/) or [Fink](http://pdb.finkproject.org/pdb/package.php/bash). Alternatively, precompiled macOS packages are available from various websites or can be built directly from source if required.

![GNU bash](https://tiswww.case.edu/php/chet/img/bash-logo-web.png)

# Why bother upgrading?

I've been running later GNU bash releases on my various Mac's that I've owned over the last decade, the motivation behind this post was a recent change of employment and receiving a brand new Apple Macbook Pro 15 forced me to go through the set up. Shortly after, a colleague asked how to upgrade, so I thought I'd throw this post together.

GNU bash v3.2.57 probably works just fine for most people. For me, the main reason to upgrade was to keep my "dot files" operable and in-sync with my Linux and UNIX environments all running current version of GNU bash. At time of writing, the latest release is [GNU bash v5.0.7 release](https://tiswww.case.edu/php/chet/bash/bashtop.html#CurrentStatus). Whilst there's nothing particularly special about my "dot files", I do like to modify the shell behaviour using [shopt built-ins](https://www.gnu.org/software/bash/manual/bash.html#The-Shopt-Builtin) that remain unavailable in GNU bash v3.2. Whilst the scripts handle the version avoiding the unsupported shopt built-ins, I found it lacking these remained unavailable on my macOS. Another common reason is newer releases of GNU bash support programmable tab auto-completion.

Do not worry, upgrading is simple and smooth sailing.

![Drop in the Ocean?](https://source.unsplash.com/DKix6Un55mw/1500x1000?a=.png)

# Installing GNU bash v5.0 on macOS

To install, you will first need to decide whether you are going to install GNU bash via a package or compile from source. In the interest of time and simplicity, I will discuss installing GNU bash v5.0 using [Homebrew](http://brew.sh/), which you're probably already using.

{% highlight shell %}
$ brew install bash
{% endhighlight %}

...and you're done! Well, kind of but not quite. Let's look at what's happened.

Let's first verify what version of GNU bash has been installed:

{% highlight shell %}
$ which -a bash
/usr/local/bin/bash
/bin/bash
{% endhighlight %}

Here we see we have two bash binaries, so let's see what's happening.

{% highlight shell %}
$ /usr/local/bin/bash --version
GNU bash, version 5.0.7(1)-release (x86_64-apple-darwin18.5.0)
Copyright (C) 2019 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>

This is free software; you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
{% endhighlight %}

and

{% highlight shell %}
$ /bin/bash --version
GNU bash, version 3.2.57(1)-release (x86_64-apple-darwin18)
Copyright (C) 2007 Free Software Foundation, Inc.
{% endhighlight %}

So it's clear here that `/bin/bash` is our stock version of GNU bash, supplied by Apple macOS. The version installed by `brew` is `/usr/local/bin/bash`.

By default, /usr/local/bin will feature earlier in your `PATH` environment variable, so whenever you execute a `bash --version`, you will always get the latest version. As such, this new version of GNU bash is now the default.

![MacBook Pro](https://source.unsplash.com/z8lfwpQVXJo/1500x1000?a=.png)

# So it's installed?

Yes, congratulations. You'll probably log out and log back in and realise your shell is still the old version of GNU bash. Why?

UNIX provides a security feature that restricts the shells that can be used by users to a trusted, known list of login shells. As everything in UNIX is a file, the root-owned file in question is the `/etc/shells` file. To add the new version of GNU bash as your login shell, you must update this file.

{% highlight shell %}
$ sudo echo "/usr/local/bin/bash" >> /etc/shells
{% endhighlight %}

Now that this file has been updated, you'll soon realise that `/bin/bash` is still your default shell, not the new version installed. This is because your user profile needs to be updated to use the new shell.

{% highlight shell %}
$ chsh -s /usr/local/bin/bash
{% endhighlight %}

The shell used by your user profile has now been updated to `/usr/local/bin/bash` which is the new GNU bash release installed. Note that this change is only for your current user profile and will takes effect once you end your current session.

If you close your Terminal or iTerm session and reopen, you'll now notice that you're running the later GNU bash release.

Great, well done!

![Success!](https://source.unsplash.com/AndE50aaHn4/1500x1000?a=.png)

# Things to note

As we've seen, you have two versions of GNU bash installed and these will happily coexist without any real issues.

## Shell Scripts

When you write shell scripts, bare in mind you have two versions of bash installed and which version is which. In particular, keep close attention to the shebang of bash scripts:

{% highlight shell %}
#!/bin/bash
echo $BASH_VERSION
{% endhighlight %}

The above script looks basic enough but the shebang explicitly requests the stock version of GNU bash installed on macOS. This can be easily ovrercome by specifying `/usr/local/bin/bash` instead. However, a more elegant way is using the `/usr/bin/env` binary.

{% highlight shell %}
#!/usr/bin/env bash
echo $BASH_VERSION
{% endhighlight %}

This version of the same script leverages `/usr/bin/env` and requests the bash shell, which inspects the `PATH` environment variable and, as mentioned earlier, will find `/usr/local/bin/bash` before `/bin/bash`.

![Link](https://source.unsplash.com/gtVrejEGdmM/1500x1000?a=.png)

## Delete old, Symlink new?

I recall a colleague of mine once suggesting to delete the stock version of bash and symlink the new. This might look something like the below (please do not execute the below).

{% highlight shell %}
$ sudo rm /bin/bash
$ sudo ln -s /usr/local/bin/bash /bin/bash
{% endhighlight %}

Whilst this idea attempts to remove the multiple version of GNU bash installed and shell scripts would in theory leverage a newer GNU bash, it presents another problem.

Apple macOS provides a built-in security feature called the [System Integrity Protection (SIP)](https://developer.apple.com/library/archive/documentation/Security/Conceptual/System_Integrity_Protection_Guide/Introduction/Introduction.html). What this feature does is it stops write access to [specific directories](https://developer.apple.com/library/archive/documentation/Security/Conceptual/System_Integrity_Protection_Guide/FileSystemProtections/FileSystemProtections.html) within macOS even for the root user. This includes the ``/bin`` directory.

Of course SIP can be disabled, changes in `/bin` directory can then be made and SIP can then be enabled again. Apple even describe how to do it [here](https://developer.apple.com/library/archive/documentation/Security/Conceptual/System_Integrity_Protection_Guide/ConfiguringSystemIntegrityProtection/ConfiguringSystemIntegrityProtection.html). It's up to you, but I personally prefer to leave the stock version present to prevent any Apple macOS update issues.

Whichever you decide, enjoy using a recent GNU bash release on your macOS! You'll feel Bourne Again ...
