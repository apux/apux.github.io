---
title: How to Install Zsh
date: 2015-09-24 06:00 UTC
tags: [zsh, chakra]
---

## Installation

I use Zsh as my default shell. Here I'm going to show how to install and
configure it.

To use Zsh I recommend the oh-my-zsh framework to make it easier. The first
thing we need to do is install Zsh if we don't have it already installed. We
can do it by typing `sudo pacman -S zsh` or the equivalent one in your distro.

To know what shell we are using, type the following command in your prompt:

    $ ps $0
    bash

We are using bash right now. To use Zsh all you need to do is execute

    $ zsh

It is possible that the first time it is executed, Zsh asks you to configure
it, but we can skip it for now by typing `q` and let `oh-my-zsh` take care of
that later.

Now, we can see that the shell has changed:

    $ echo $0
    zsh

## Make Zsh the default shell

Now we can use Zsh whenever we want, but each time we start a new session or
open a new console, the default shell is still bash.

To know what our default shell is, use the `$SHELL` variable:

   $ echo $SHELL
   bash

To change the default shell, we can use

    $ chsh -s /path/to/shell

Good, but, how can we know this path? The first (wrong) approach I tried was
using `which`:

    $ which zsh
    /usr/bin/zsh

Once we know the path we can use it to change the default shell, I thought. I
tried this:

    $ chsh -s $(which zsh)
    chsh: "/usr/bin/zsh" is not listed in /etc/shells.

As we can see, it didn't work.

Apparently, Zsh executable path and the actual shell path are different. A
better approach was to use `chsh` to know where the shells actually were.

    $ chsh -l
    /bin/sh
    /bin/bash
    /bin/zsh

Now, we can use the information we have.

    $ chsh -s /bin/zsh

If we open a new console, we could expect it to have Zsh by default, but it is
not. It is important to remember that the new configuration will only have
effect when we login for a new session, so you have to logout and login again
in order to have it working.

## Oh my Zsh

The last part is install oh-my-sh.

    $ sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"

We don't have to logout and login again this time, just open a new console and
you will be able to configure and use the themes and plugins oh-my-zsh
includes. Enjoy!
